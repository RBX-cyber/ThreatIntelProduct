La atribución de la campaña DeTankZone al **Lazarus Group** (y sus subgrupos como BlueNoroff) se sustenta en la coincidencia sistemática de Tácticas, Técnicas y Procedimientos (TTPs). Este capítulo analiza la genealogía del ataque, vinculando la operación con campañas históricas y paralelas de ciberespionaje y robo financiero.

## 7.1. Robo de DeFiTankLand (Marzo 2024)

El hallazgo forense más directo vincula el *software* malicioso con un robo de propiedad intelectual ocurrido apenas un mes antes del despliegue de la campaña.

**Detalles del Incidente:**
* **Objetivo:** Proyecto legítimo de juego *Play-to-Earn* llamado **DeFiTankLand (DFTL)**.
* **Fecha de Compromiso:** Marzo de 2024.
* **Vector:** Penetración en la infraestructura de desarrollo de DFTL, resultando en la exfiltración del código fuente del juego y el robo de ~$20,000 USD en tokens.
* **Reutilización:** Los binarios de DeTankZone son compilaciones directas de este código robado. Esta técnica de "reciclaje" permite a Lazarus desplegar señuelos de alta calidad visual y funcional sin incurrir en costes de desarrollo, dificultando la distinción entre el producto real y el falso por parte de la víctima.

## 7.2. Campañas Relacionadas y Antecedentes

DeTankZone no es un evento aislado, sino una iteración más dentro de una estrategia continuada de ataques contra el sector financiero y tecnológico.

### A. Operación DreamJob (Ingeniería Social vía LinkedIn)
La metodología de captación de víctimas en DeTankZone es una variante directa de la "Operación DreamJob".
* **Modus Operandi:** Los atacantes contactan a empleados de sectores estratégicos (Defensa, Aeroespacial, Cripto) a través de LinkedIn o X, ofreciendo falsas oportunidades laborales o colaboraciones.
* **Vínculo Técnico:** En campañas de 2022, Lazarus utilizó esta misma técnica para desplegar *Zero-Days* de Chrome contra víctimas que visitaban enlaces maliciosos enviados durante la "entrevista". La infraestructura de ingeniería social es compartida.
* **Informe Técnico:** [Google TAG - Countering hackers recruiting unwary job seekers](https://blog.google/threat-analysis-group/countering-hackers-recruiting-unwary-job-seekers/)

### B. Campaña TraderTraitor (Software Cripto Troyanizado)
Esta es la campaña "gemela" de DeTankZone en términos de ejecución técnica.
* **Descripción:** Distribución de aplicaciones falsas de criptomonedas (carteras, bots de trading, juegos NFT) construidas sobre plataformas legítimas o código *open source* modificado.
* **Similitud:** Al igual que en DeTankZone, las aplicaciones de TraderTraitor suelen estar construidas con Electron/Node.js o son binarios nativos que cargan *payloads* maliciosos (como Manuscrypt) tras una ejecución aparentemente normal. El FBI y CISA emitieron alertas específicas sobre esta actividad dirigida a empresas *blockchain*.
* **Informe Técnico:** [CISA/FBI Alert - TraderTraitor: North Korean State-Sponsored APT Targets Blockchain Companies](https://www.cisa.gov/news-events/cybersecurity-advisories/aa22-108a)

### C. Operación AppleJeus (El Origen)
Considerada la precursora de todas las operaciones de *fake crypto software* de Lazarus.
* **Evolución:** Desde 2018, Lazarus ha creado empresas fantasma (Celas Trade, JMT Trading) con sitios web profesionales para distribuir actualizadores de software troyanizados. DeTankZone evoluciona este concepto: en lugar de una herramienta de trading aburrida, utilizan un videojuego para atraer a un perfil demográfico más joven y menos adverso al riesgo (Gamers/NFT Traders).
* **Informe Técnico:** [CISA Alert - AppleJeus: Analysis of North Korean Cryptocurrency Malware](https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-048a)

### D. Campaña SnatchCrypto (BlueNoroff)
Atribuida al subgrupo **BlueNoroff** (especializado en robo financiero dentro de Lazarus).
* **Táctica de "Acecho":** A diferencia de ataques rápidos de *ransomware*, SnatchCrypto se caracteriza por monitorizar a la víctima durante semanas o meses tras la infección inicial, esperando el momento exacto en que se realiza una transacción de alto valor para manipularla o drenar la *wallet*.
* **Relevancia:** El uso de scripts de validación en DeTankZone (para perfilar si la víctima tiene activos valiosos antes de atacar) es una táctica defensiva típica de BlueNoroff para evitar quemar sus herramientas en objetivos sin valor económico.
* **Informe Técnico:** [Kaspersky - The BlueNoroff cryptocurrency hunt](https://securelist.com/the-bluenoroff-cryptocurrency-hunt/105488/)