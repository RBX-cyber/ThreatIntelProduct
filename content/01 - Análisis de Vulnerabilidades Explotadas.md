

La campaña DeTankZone se fundamentó en la explotación encadenada de dos vulnerabilidades críticas en Google Chrome, específicamente en su motor JavaScript V8. Esta cadena de exploits permitió a Lazarus Group comprometer sistemas objetivo mediante simple visita al sitio malicioso.

## 1.1. [CVE-2024-4947](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2024-4947) - Type Confusion en V8

[CVE-2024-4947](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2024-4947) es una vulnerabilidad de type confusion en el compilador JIT Maglev del motor V8, que constituyó el primer eslabón de la cadena de explotación para obtener acceso inicial al sistema.

### Descripción Técnica Detallada

La vulnerabilidad reside en el manejo incorrecto de asunciones de tipo durante la optimización de código JavaScript por parte del compilador JIT Maglev. Este fallo permite que el motor trate un tipo de dato como si fuera otro, rompiendo las garantías de seguridad del lenguaje.

El compilador Maglev realiza optimizaciones agresivas basándose en tipos inferidos. Cuando estas inferencias son incorrectas, se genera código máquina que accede a memoria de manera insegura. La confusión resultante permite manipular objetos JavaScript de forma que el motor ejecute operaciones con tipos incompatibles.

La explotación exitosa proporciona capacidades de lectura y escritura arbitrarias en el espacio de direcciones del proceso Chrome desde contexto JavaScript. Esto significa que código malicioso puede acceder y modificar memoria fuera de los límites permitidos, incluyendo estructuras de datos críticas del navegador.

### Clasificación de severidad

* **CVSS Score:** 8.8 (High)
* **Tipo:** Type Confusion
* **Componente afectado:** V8 JavaScript Engine - Compilador JIT Maglev
* **Impacto:** Remote Code Execution (RCE), lectura/escritura arbitraria de memoria

### Versiones Afectadas de Chrome

Todas las versiones anteriores a Chrome 125.0.6422.60/61 fueron vulnerables a [CVE-2024-4947](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2024-4947). Google publicó el parche en mayo de 2024, aproximadamente tres meses después del inicio estimado de la campaña (febrero 2024).

**Versiones parcheadas:**
* Chrome 125.0.6422.60 (Linux)
* Chrome 125.0.6422.61 (Windows y macOS)
* Todas las versiones posteriores

La vulnerabilidad también afectó a navegadores basados en Chromium que no integraron el parche, incluyendo versiones anteriores de Microsoft Edge, Brave, Opera y Vivaldi. Durante el período de explotación activa (febrero-mayo 2024), la mayoría de usuarios estaban potencialmente en riesgo.

### Mecanismo de Explotación

El exploit se ejecuta automáticamente al visitar `detankzone[.]com` mediante código JavaScript ofuscado embebido en la página. La secuencia de explotación procede de la siguiente manera:

**Fase 1 - Trigger de Type Confusion:** El código JavaScript malicioso crea objetos específicamente diseñados que explotan la confusión de tipos en el compilador JIT. Estos objetos manipulan al compilador para que genere código optimizado con asunciones de tipo incorrectas.

**Fase 2 - Establecimiento de Primitivas:** Una vez lograda la confusión de tipos, el exploit establece primitivas de lectura/escritura arbitrarias que permiten:

1. **Lectura arbitraria:** Acceso a cualquier dirección de memoria del proceso Chrome para localizar estructuras críticas, bypass de ASLR, y extracción de información sensible (cookies, tokens, credenciales).
2. **Escritura arbitraria:** Modificación de estructuras de control para alterar el flujo de ejecución y lograr RCE.
3. **Bypass de ASLR:** Filtración de direcciones aleatorias mediante primitivas de lectura para calcular ubicaciones base de módulos críticos.

El código está altamente ofuscado y fragmentado entre código legítimo para evadir detección. La fase final prepara el entorno para explotar la segunda vulnerabilidad de bypass del sandbox V8.

## 1.2. Vulnerabilidad de Bypass del Sandbox V8

La segunda vulnerabilidad en la cadena permite eludir completamente el sandbox V8, escalando desde compromiso del navegador hasta ejecución de código arbitrario en el sistema operativo. No tiene CVE público asignado.

### Descripción Técnica

El fallo reside en la implementación de la máquina virtual V8, específicamente en el manejo de registros virtuales durante la ejecución de bytecode. La VM utiliza un array dedicado para almacenar registros, pero no valida adecuadamente los índices de registros decodificados desde las instrucciones de bytecode.

Esta ausencia de bounds checking permite especificar índices fuera de los límites válidos del array de registros. Al acceder a estos índices inválidos, la VM lee/escribe memoria más allá del array, en regiones que pueden contener estructuras críticas del motor o del proceso.

La explotación requiere capacidad de inyectar/manipular bytecode V8, obtenida mediante [CVE-2024-4947](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2024-4947). El acceso a memoria fuera de límites permite:

* Leer estructuras del motor V8 más allá del sandbox, incluyendo punteros a objetos del sistema operativo
* Modificar estructuras de control que determinan las restricciones del sandbox
* Obtener referencias a objetos y capacidades del sistema normalmente inaccesibles

### Impacto de la Vulnerabilidad

El bypass del sandbox V8 anula una de las defensas en profundidad más importantes de Chrome, proporcionando:

* **Acceso directo al sistema operativo:** Invocación de APIs del sistema sin restricciones de sandbox (operaciones de archivos, creación de procesos, manipulación de red)
* **Ejecución de código nativo arbitrario:** Código máquina con privilegios completos del proceso Chrome, suficiente para instalar malware persistente y acceder a archivos del usuario
* **Instalación de malware persistente:** Despliegue del backdoor Manuscrypt que persiste tras cierre del navegador
* **Exfiltración de datos sensibles:** Acceso a documentos, configuraciones, claves de wallets crypto, credenciales y cualquier información del usuario
* **Evasión de detección avanzada:** Manipulación de procesos de seguridad, ocultación de artefactos e interferencia con antivirus

En DeTankZone, el bypass permitió ejecutar un validator script (shellcode) que recopiló información del sistema para determinar el valor de la víctima antes de desplegar Manuscrypt.

### Estado de Parcheo y Naturaleza de la Vulnerabilidad

Google parcheó una vulnerabilidad de acceso fuera de límites en el array de registros de V8 en marzo de 2024, dos meses antes del descubrimiento de DeTankZone (mayo 2024). Esto plantea dos escenarios:

* **Escenario 1 - Zero-Day:** Lazarus descubrió la vulnerabilidad independientemente antes de marzo 2024 y la explotó durante febrero-marzo como zero-day genuino.
* **Escenario 2 - N-Day:** Lazarus realizó ingeniería inversa del parche de marzo 2024 (patch diffing) para desarrollar el exploit, explotándolo contra sistemas no actualizados.

**Factores que sugieren escenario N-day:**
* Período corto entre parche (marzo) y descubrimiento (mayo)
* Lazarus ha demostrado capacidades de patch diffing en campañas previas
* Combinación estratégica de zero-day ([CVE-2024-4947](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2024-4947)) con posible N-day maximiza ROI

Independientemente de su naturaleza, el bypass fue parcheado en marzo 2024. Usuarios con Chrome actualizado estuvieron protegidos contra este componente, aunque permanecieron vulnerables a [CVE-2024-4947](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2024-4947) hasta mayo 2024.