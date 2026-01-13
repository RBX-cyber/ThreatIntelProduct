

Los IoCs basados en archivos permiten identificar componentes maliciosos mediante características estructurales, nombres y ubicaciones, independientemente de variaciones en hashes específicos.

## Nomenclatura y Ubicaciones

**Archivos del juego DeTankZone:** El archivo descargable `detankzone.zip` se ubicaba típicamente en el directorio de descargas del usuario, con tamaño estimado de 100-500 MB. Tras extracción, el ejecutable principal se instalaba para instalaciones sin privilegios. Los archivos de configuración (`config.json`, `settings.cfg`) residían en el directorio de instalación o en `%APPDATA%`, potencialmente conteniendo configuraciones de C2 ofuscadas.

**Manuscrypt:** El backdoor utiliza nombres variables para evadir detección, frecuentemente imitando procesos del sistema como `svchosts.exe`, `svchost32.exe`, `explorer32.exe`, o `csrss32.exe` en Windows, y nombres como `systemd.bin` en Linux. Las ubicaciones típicas de instalación incluyen `%APPDATA%\`, `%TEMP%\`, y `%LOCALAPPDATA%\` en Windows, mientras que en sistemas Unix se instala en `~/.local/share/`, `/tmp/`, o `/var/tmp/`. El malware crea subdirectorios con nombres aleatorios o que imitan directorios legítimos del sistema, frecuentemente con atributos de archivo oculto.

Los archivos temporales y de staging se identifican por extensiones `.tmp`, `.dat`, `.bin` en directorios temporales, archivos comprimidos conteniendo datos staged, y archivos cifrados con extensiones custom o nombres hexadecimales aleatorios siguiendo el patrón `[a-f0-9]{8,32}`.

## Características de Ejecutables PE

Los ejecutables PE asociados con la campaña presentan compile timestamps potencialmente falsificados, secciones con nombres no estándar y entropía elevada (>7.0) indicando cifrado. Las import tables revelan uso de APIs de red (`WinINet.dll`, `ws2_32.dll`), criptografía (`Crypt32.dll`, `bcrypt.dll`), y manipulación de procesos (`CreateProcess`, `VirtualAlloc`, `WriteProcessMemory`).

Los binarios carecen de firmas digitales válidas o utilizan firmas auto-firmadas. La entropía de secciones ejecutables es elevada (6.5-8.0) sugiriendo empaquetado con UPX, ASPack, o packers custom, con strings ofuscadas ausentes en análisis estático.

**Script de análisis de entropía:**

```python
import pefile, math
from collections import Counter

def calc_entropy(data):
    if not data: return 0
    counter = Counter(data)
    length = len(data)
    return -sum((c/length) * math.log2(c/length) for c in counter.values())

pe = pefile.PE('suspicious_file.exe')
for sec in pe.sections:
    print(f"{sec.Name.decode().rstrip(chr(0))}: {calc_entropy(sec.get_data()):.2f}")
```

## IoCs de Binarios ELF

En sistemas Linux, los binarios ELF de Manuscrypt son típicamente stripped (símbolos removidos), statically linked para portabilidad, y soportan arquitecturas x86-64 y ARM. La persistencia se establece mediante cron jobs en `/var/spool/cron/crontabs/`, systemd services en `/etc/systemd/system/` o `~/.config/systemd/user/`, y modificaciones en `~/.bashrc` o `~/.profile`.

**Comandos de detección:**

```bash
# Binarios ELF sospechosos en /tmp
find /tmp -type f -executable -exec file {} \; | grep ELF

# Enumerar cron jobs
for u in $(cut -f1 -d: /etc/passwd); do crontab -u $u -l 2>/dev/null; done

# Archivos modificados recientemente
find ~/ -type f -mtime -7 -ls 2>/dev/null
```

## IoCs de JavaScript Malicioso

El exploit JavaScript presenta ofuscación mediante variables de nombres sin sentido, uso extensivo de `eval()`, `Function()`, strings codificados en hexadecimal/Base64/Unicode, y control flow ofuscado. Los patrones característicos incluyen referencias a objetos internos de V8 (`%OptimizeFunctionOnNextCall`), manipulación de typed arrays para type confusion, spray de memoria con arrays grandes, y timing deliberado con `setTimeout`/`setInterval` para evasión de sandboxes.

El JavaScript malicioso se encuentra en el cache de Chrome ubicado en `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Cache\` (Windows), `~/.cache/google-chrome/Default/Cache/` (Linux), o `~/Library/Caches/Google/Chrome/Default/Cache/` (macOS).

## 5.1. Reglas de Detección

### YARA - Detección de YouieLoad

```yara
rule DeTankZone_YouieLoad {
    meta:
        description = "Detects YouieLoad loader in DeTankZone"
        date = "2024-10-24"
    strings:
        $s1 = "YouieLoad" ascii wide
        $s2 = "DeTankZone" ascii wide nocase
        $vm1 = "VBOX" ascii wide nocase
        $vm2 = "VMware" ascii wide nocase
        $api1 = "URLDownloadToFile" ascii
        $api2 = "CryptDecrypt" ascii
        $shell = { 55 8B EC 83 EC ?? 53 56 57 }
    condition:
        uint16(0) == 0x5A4D and filesize < 10MB and
        ((1 of ($s*)) or (2 of ($vm*) and 2 of ($api*)) or
        ($shell and 3 of ($api*)))
}
```

### YARA - Manuscrypt Genérico

```yara
rule Lazarus_Manuscrypt {
    meta:
        description = "Generic Manuscrypt detection"
        family = "Manuscrypt"
    strings:
        $str1 = "Mozilla/4.0 (compatible; MSIE 8.0;" ascii
        $cfg = { C7 45 ?? ?? ?? ?? ?? C7 45 ?? ?? ?? ?? ?? }
        $xor = { 8A 0? [2-4] 32 0? [2-4] 88 0? [2-4] 4? 3? ?? 7? }
        $api1 = "HttpSendRequestA" ascii
        $api2 = "InternetConnectA" ascii
    condition:
        uint16(0) == 0x5A4D and filesize < 5MB and
        (($str1 and 2 of ($api*)) or ($xor and $cfg and 2 of ($api*)))
}
```

### Sigma - Acceso a Wallets Crypto

```yaml
title: Suspicious Cryptocurrency Wallet Access
id: a1b2c3d4-e5f6-7890-abcd-ef1234567890
date: 2024/10/24
tags: [attack.collection, attack.t1005]
detection:
    wallet_paths:
        TargetFilename|contains:
            ['\Bitcoin\wallet.dat', '\Ethereum\keystore\', '\Exodus\exodus.wallet\']
    suspicious_proc:
        Image|endswith: ['\cmd.exe', '\powershell.exe']
    filter:
        Image|contains: ['\Bitcoin\', '\Exodus\']
    condition: wallet_paths and suspicious_proc and not filter
level: high
```

### Snort - Tráfico C2

```snort
alert tcp $HOME_NET any -> $EXTERNAL_NET $HTTP_PORTS (
    msg:"Lazarus DeTankZone C2 Detected";
    flow:to_server,established;
    content:"POST"; http_method;
    content:"/api/"; http_uri;
    pcre:"/\/api\/[a-f0-9]{32}\//U";
    classtype:trojan-activity; sid:1000001;
)

alert tcp $HOME_NET any -> $EXTERNAL_NET $HTTP_PORTS (
    msg:"DeTankZone Domain Communication";
    flow:to_server,established;
    content:"Host|3a 20|detankzone.com"; http_header;
    sid:1000002;
)
```

### Queries de Threat Hunting

**Splunk - Correlación de IoCs:**

```splunk
index=* (detankzone.com OR ccwaterfall.com OR "B2DC7AEC2C6D2FFA28219AC288E4750C")
| stats count by src_ip, dest_ip, _time | sort -count
```

**PowerShell - Event Log Hunting:**

```powershell
# Descargas de archivos sospechosos
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=11} | Where {$_.Message -match 'detankzone'} | Select TimeCreated, Message

# Conexiones a dominios IoC
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} | Where {$_.Message -match 'detankzone\.com'} | Select TimeCreated, Message
```

## 5.2. Resumen de IoCs Críticos

| Tipo | Valor | Descripción | Prioridad |
| :--- | :--- | :--- | :--- |
| Dominio | `detankzone[.]com` | Sitio distribución | Crítica |
| Dominio | `ccwaterfall[.]com` | Infra secundaria | Alta |
| MD5 | `B2DC7AEC2C6D2FFA28219AC288E4750C` | Exploit CVE-2024-4947 | Crítica |
| SHA-256 | `7353AB9670133468...C417833A` | Exploit completo | Crítica |
| MD5 | `8312E556C4EEC999204368D69BA91BF4` | Juego ZIP | Alta |
| SHA-256 | `59A37D7D2BF4CFFE...AB65A4CC` | Juego completo | Alta |
| CVE | CVE-2024-4947 | Vulnerabilidad | Crítica |
| Malware | Manuscrypt/NukeSped | Backdoor | Crítica |

**Acciones recomendadas:**
Prioridad crítica requiere bloqueo inmediato de dominios IoC en firewalls/DNS, actualización de Chrome a versión >= 125.0.6422.60/61, implementación de detección de hashes en gateways, y despliegue de reglas Manuscrypt en EDR/AV. Alta prioridad (24-48h) incluye escaneo retroactivo de endpoints, análisis de logs históricos, implementación de reglas YARA/Sigma, y revisión de accesos a wallets crypto. Media prioridad (1 semana) comprende threat hunting proactivo, educación de usuarios, hardening de navegadores, e implementación de monitorización mejorada de comportamiento.