Este capítulo documenta la reconstrucción forense de la cadena de ejecución de DeTankZone, desde la manipulación de la memoria en el proceso del navegador hasta la persistencia del código nativo.

## 8.1. Análisis del Exploit V8 (CVE-2024-4947)

La explotación exitosa depende de corromper el motor JavaScript V8 para obtener primitivas de lectura/escritura arbitrarias (`addrOf` y `fakeObj`). El fallo se ubica específicamente en el compilador JIT **Maglev**, introducido recientemente en Chrome para optimizar la generación de código máquina.

### Mecánica de la Confusión de Tipos (Type Confusion)

El compilador Maglev realiza asunciones optimistas sobre los tipos de objetos para omitir comprobaciones de seguridad costosas. El exploit fuerza una desincronización entre el "mapa" del objeto (Hidden Class) percibido por el compilador y el mapa real en el *heap*.

**Reconstrucción de la lógica del exploit (Pseudo-código):**

El siguiente fragmento ilustra cómo se induce la confusión. El atacante define una función que accede a una propiedad de un objeto. Mediante manipulación del prototipo o transiciones de mapa justo antes de la compilación JIT, se engaña a Maglev.

```javascript
// Concepto técnico del Trigger CVE-2024-4947
function trigger_confusion(arr, index, val) {
    // Maglev asume que 'arr' es siempre un array de Dobles (PACKED_DOUBLE_ELEMENTS)
    // debido al profiling previo.
    arr[index] = val; 
}

// 1. Entrenar el JIT con arrays de dobles
for (let i = 0; i < 10000; i++) {
    trigger_confusion(double_array, 0, 1.1);
}

// 2. Optimización Maglev ocurre aquí...

// 3. Modificar el array para cambiar su tipo a PACKED_ELEMENTS (objetos)
// Esto debería invalidar el código optimizado, pero el fallo impide la desoptimización correcta.
let confused_array = [1.1, 2.2];
confused_array.proc = "transition"; // Cambio de Map

// 4. Ejecutar código optimizado con el tipo incorrecto
// El motor escribe un puntero de objeto como si fuera un float, o viceversa.
trigger_confusion(confused_array, 0, object_pointer_as_float);
```

### Primitivas de Memoria Obtenidas

Una vez lograda la confusión, el atacante construye dos primitivas fundamentales:
1.  **addrOf(obj):** Retorna la dirección de memoria de un objeto JavaScript. Se logra confundiendo un objeto con un array de `float` y leyendo su valor.
2.  **fakeObj(addr):** Crea una referencia a un objeto JavaScript falso en una dirección de memoria arbitraria. Se logra escribiendo una dirección (como `float`) en un array de objetos.

## 8.2. Análisis del Bypass del Sandbox V8

Google Chrome implementa un "V8 Sandbox" para impedir que un RCE dentro del motor V8 pueda escribir en la memoria de todo el proceso renderizador. Lazarus utilizó una vulnerabilidad lógica en la implementación de la Máquina Virtual (VM) de V8 para escapar de esta jaula.

**El Fallo del Array de Registros:**
La VM de V8 utiliza un array dedicado para almacenar registros virtuales durante la interpretación de bytecode. Las instrucciones de bytecode contienen índices que referencian a este array.
* **Vulnerabilidad:** Falta de validación de límites (Bounds Check) al decodificar los índices de registros desde el cuerpo de la instrucción.
* **Explotación:** El atacante utiliza la capacidad de escritura obtenida en la fase anterior para inyectar bytecode malicioso con índices de registro "imposibles" (fuera de rango).

**Impacto en Memoria:**
Al acceder a un índice fuera de límites, la VM lee/escribe en estructuras adyacentes al array de registros. Esto permite sobrescribir punteros críticos fuera del sandbox, como la **Global Offset Table (GOT)** o punteros a funciones API importadas, redirigiendo el flujo de ejecución hacia el Shellcode.

## 8.3. Análisis del Shellcode (Validator)

El *payload* inicial (Validator Script) es un bloque de *Position Independent Code* (PIC) inyectado directamente en la memoria del proceso `chrome.exe` o `renderer`.

### Resolución de APIs (API Hashing)
Para evitar detección por escaneos de strings estáticos, el shellcode no contiene nombres de funciones (e.g., `CreateProcessA`). Utiliza *hashing* dinámico para resolver direcciones.

**Algoritmo de Hashing (Reconstrucción Python):**
El shellcode recorre la `Export Address Table` de `kernel32.dll` y `ntdll.dll`, calcula el hash de cada función y lo compara con un valor precalculado (e.g., `0xDEADC0DE`).

```python
def ror(val, rot, width=32):
    return ((val >> rot) | (val << (width - rot))) & ((1 << width) - 1)

def calc_hash(func_name):
    hash_val = 0
    for char in func_name:
        # Rotación común en shellcodes de Lazarus
        hash_val = ror(hash_val, 13) 
        hash_val += ord(char)
    return hash_val

# Ejemplo: Resolver "VirtualAlloc"
print(hex(calc_hash("VirtualAlloc")))
```

### Lógica de Recolección de Información
El análisis del desensamblado muestra que el shellcode busca patrones específicos en el sistema de archivos:
1.  **Enumeración de Discos:** Itera desde `A:\` hasta `Z:\`.
2.  **Búsqueda Recursiva:** Busca archivos con extensiones `.wallet`, `.kdbx` (KeePass), y cadenas específicas en `%APPDATA%`.
3.  **Serialización:** Concatena los resultados en un buffer en memoria, lo cifra (posiblemente XOR o RC4 con clave estática) y lo envía vía `WinHttpSendRequest`.

## 8.4. Artefactos Forenses en Disco

A pesar de que gran parte del ataque ocurre en memoria ("fileless"), la interacción con el navegador y la persistencia dejan rastros recuperables.

### 1. Cache del Navegador (Chrome)
El script del exploit (`.js`) y los recursos del juego falso quedan almacenados temporalmente.
* **Ubicación:** `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Cache\Cache_Data\`
* **Recuperación:** Herramientas como *ChromeCacheView* o *Hindsight* pueden reconstruir el archivo JS original si no ha sido sobrescrito. Buscar entradas con el dominio `detankzone[.]com`.

### 2. Crash Dumps
La explotación de V8 es inestable. Intentos fallidos provocan el cierre inesperado de la pestaña o del navegador.
* **Evidencia:** Archivos `.dmp` en `%LOCALAPPDATA%\Google\CrashReports\`.
* **Análisis:** Cargar el volcado en *WinDbg*. Un fallo de *Access Violation* (0xC0000005) en `v8.dll` o `chrome.dll` con registros apuntando a direcciones inusuales es un indicador fuerte de intento de explotación.

### 3. Event Logs de Windows
Si el malware escala privilegios o establece persistencia, genera eventos auditables.
* **ID 4688 (Process Creation):** Buscar procesos hijos de `chrome.exe` lanzando `cmd.exe` o `powershell.exe` (comportamiento anómalo).
* **ID 11 (Sysmon - File Create):** Creación de ejecutables o DLLs en carpetas temporales (`AppData\Local\Temp`).

## 8.5. Descifrado de Comunicaciones C2 (Manuscrypt)

El backdoor Manuscrypt utiliza un protocolo HTTP personalizado. El cuerpo del mensaje POST suele estar cifrado y codificado en Base64.

**Esquema de Cifrado Típico:**
1.  **Header Fake:** Cabeceras HTTP estándar para parecer tráfico de navegación o telemetría.
2.  **Payload:** `Datos = Base64(RC4(Key, Data))`.
3.  **Key:** La clave RC4 suele estar *hardcodeada* en el binario o derivada de un *handshake* inicial.

Para el análisis de tráfico capturado (PCAP), se requiere extraer la clave del volcado de memoria del proceso infectado para descifrar el flujo exfiltrado.