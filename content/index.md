# Informe de Inteligencia: Campaña DeTankZone

Este repositorio documenta el análisis táctico y técnico de la campaña [DeTankZone](https://nquiringminds.com/cybernews/lazarus-group-exploits-chrome-zeroday-vulnerability-in-detankzone-campaign/), una operación avanzada de ciberespionaje y robo financiero dirigida contra el sector de las criptomonedas. La campaña ha sido atribuida con alta confianza al [Lazarus Group](https://www.darkreading.com/cyberattacks-data-breaches/lazarus-group-exploits-chrome-zero-day-campaign) (APT de Corea del Norte).

## Resumen Ejecutivo

La campaña DeTankZone (activa principalmente entre febrero y mayo de 2024) destaca por el uso de una cadena de exploits Zero-Day en Google Chrome para comprometer a las víctimas sin necesidad de interacción (técnica drive-by download).

**Puntos Clave de la Investigación:**

* **Vector de Infección (Zero-Day):** Los atacantes explotaron la vulnerabilidad [CVE-2024-4947](https://www.incibe.es/en/incibe-cert/early-warning/vulnerabilities/cve-2024-4947/) (Type Confusion en el motor V8 de Chrome) encadenada con un Bypass del Sandbox V8. Esto permitía la Ejecución Remota de Código (RCE) simplemente visitando el sitio web malicioso detankzone[.]com.
* **Ingeniería Social:** Se clonó el código fuente de un juego DeFi legítimo (DeFiTankLand) para crear un señuelo completamente funcional promocionado en redes sociales.
* **Malware Desplegado:**
    * **Validator Script:** Shellcode en memoria que perfila el sistema de la víctima.
    * **Manuscrypt:** Backdoor principal utilizado para persistencia y control.
    * **YouieLoad:** Un loader personalizado oculto dentro del ejecutable del juego.
* **Objetivos:** Inversores de criptomonedas y empresas Web3.

## Índice del Informe

A continuación se listan las secciones del análisis táctico. Selecciona un enlace para navegar al detalle:

### [[01 - Análisis de Vulnerabilidades Explotadas]]
* Detalle técnico del [CVE-2024-4947](https://www.incibe.es/en/incibe-cert/early-warning/vulnerabilities/cve-2024-4947/) (Type Confusion en V8).
* Mecanismo de evasión del Sandbox V8 y obtención de RCE.

### [[02  - Cadena de Ataque (Kill Chain)]]
* Desglose de la operación fase por fase: Reconocimiento, Weaponization, Entrega, Explotación, Instalación, C2 y Acciones sobre Objetivos.

### [[03 - Análisis Técnico del Malware]]
* Ingeniería inversa del backdoor Manuscrypt.
* Funcionamiento del loader YouieLoad y el Validator Script.

### [[04 - Infraestructura de Ataque]]
* Análisis de dominios (detankzone[.]com), sitio web falso y presencia en RRSS.

### [[05 - IoCs de Archivos]]
* Reglas de detección y resumen de Indicadores de Compromiso críticos.

### [[06 - Análisis de ingeniería social]]
* Perfil de las víctimas, técnicas de engaño y canales de distribución.

### [[07 - Conexión con incidentes previos]]
* Vínculos con el robo de DeFiTankLand (Marzo 2024) y campañas anteriores.

### [[08 - Análisis forense técnico]]
* Reconstrucción del flujo de ejecución del exploit.