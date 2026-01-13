

Lazarus Group desplegó una infraestructura compleja y multicapa para la campaña DeTankZone, combinando dominios maliciosos, presencia en redes sociales, y servidores de Command and Control. Esta sección detalla los componentes técnicos de la infraestructura utilizada.

## 4.1. Dominios Maliciosos

La infraestructura de dominios constituye el núcleo de la operación, proporcionando tanto la superficie de ataque inicial como puntos de comunicación con sistemas comprometidos.

### Dominio Principal: detankzone[.]com

**Función:** Dominio principal de distribución del exploit y señuelo del juego falso DeTankZone.

**Características técnicas:**
* Hosting de sitio web con apariencia profesional presentando juego DeFi/NFT.
* Implementación de exploit chain mediante JavaScript ofuscado embebido.
* Servidor web configurado para servir contenido mixto (legítimo + malicioso).
* Capacidad de drive-by download sin interacción adicional del usuario.
* Backend desmantelado en mayo 2024 tras descubrimiento de la campaña.

**Timeline de operación:**
* Registro estimado: Febrero 2024
* Período activo de explotación: Febrero - Mayo 2024
* Desmantelamiento: Mayo 2024 (coincidente con descubrimiento de Kaspersky)

El sitio presentaba un juego completo funcional basado en código robado de DeFiTankLand, proporcionando legitimidad superficial. La arquitectura del sitio separaba claramente el contenido visible (juego, descripciones, gráficos) del código malicioso (script de exploit oculto en el código fuente).

### Dominio Secundario: ccwaterfall[.]com

**Función:** Infraestructura de soporte, probablemente utilizada para C2 o distribución de payloads secundarios.

**Relación con la campaña:** La función exacta de este dominio no está completamente documentada en las fuentes públicas disponibles. Sin embargo, su identificación como IoC relacionado con DeTankZone sugiere roles potenciales:
* Servidor de Command and Control para comunicación con Manuscrypt.
* Repositorio de payloads secundarios descargados post-explotación.
* Infraestructura de exfiltración para datos robados.
* Dominio de respaldo en caso de bloqueo del dominio principal.

El uso de múltiples dominios es consistente con las prácticas operacionales de Lazarus, proporcionando redundancia y dificultando el bloqueo completo de la infraestructura mediante simples blacklists de dominios.

**Características de Registro y Hosting**
Los detalles específicos de registro de dominios (registrar, información WHOIS, geolocalización de servidores) no fueron publicados en las fuentes analizadas. Sin embargo, basándose en patrones históricos de Lazarus Group, es probable que los dominios utilizaran:
* Información WHOIS falsa o anónima mediante servicios de privacidad.
* Registrars en jurisdicciones con políticas laxas de verificación.
* Hosting en proveedores bulletproof o comprometidos.
* Uso de servicios de CDN o proxies inversos para ocultar IPs de origen.
* Domain fronting o técnicas similares para ofuscar tráfico C2.

## 4.2. Sitio Web Falso: DeTankZone

El sitio web DeTankZone representa un ejemplo sofisticado de ingeniería social combinada con capacidades técnicas avanzadas de explotación.

### Apariencia y Diseño

El sitio presentaba diseño profesional emulando plataformas legítimas de juegos blockchain:
* Landing page con gráficos de alta calidad mostrando el juego.
* Descripciones detalladas del gameplay DeFi/NFT MOBA.
* Sección de roadmap del proyecto.
* Información sobre tokenomics y mecánicas NFT.
* Invitación a descargar versión trial del juego (`detankzone.zip`).
* Links a perfiles de redes sociales (X/Twitter, Telegram, Discord).

El contenido visual utilizó generación mediante inteligencia artificial y diseñadores gráficos profesionales, produciendo apariencia indistinguible de proyectos legítimos del sector crypto/gaming.

### Tecnología y Código Fuente

**Base de código robado:** El juego funcional presentado en el sitio y disponible para descarga estaba basado en código fuente robado del proyecto legítimo DeFiTankLand (DFTL). En marzo 2024, el proyecto DeFiTankLand sufrió un breach donde Lazarus Group robó $20,000 en tokens DFTL2 junto con el código fuente completo del juego.

**Modificaciones realizadas:**
* Rebranding completo de DeFiTankLand a DeTankZone (logos, nombres, assets gráficos).
* Integración del loader YouieLoad en el ejecutable descargable.
* Modificación de mecánicas de juego y parámetros de blockchain.
* Mantenimiento de funcionalidad completa para evitar sospechas.

**Script de exploit oculto:** El sitio web contenía JavaScript malicioso ofuscado que implementaba la cadena de explotación:
* Integrado en archivos JavaScript aparentemente legítimos del sitio.
* Ofuscación multicapa para evadir análisis estático.
* Código fragmentado y distribuido entre múltiples archivos `.js`.
* Trigger automático al cargar la página en Chrome vulnerable.
* Sin indicadores visuales o comportamiento perceptible durante explotación.

**Arquitectura web:**
* Stack tecnológico estándar: HTML5, CSS3, JavaScript.
* Uso de frameworks web modernos para apariencia profesional.
* Separación de contenido legítimo y código de exploit en capas distintas.
* Servidor web configurado para logging mínimo o deshabilitado.

### Funcionalidad del Juego Descargable

El archivo `detankzone.zip` contenía una versión completamente funcional del juego:
* Ejecutable nativo del sistema operativo (Windows `.exe`, posiblemente versiones macOS/Linux).
* Juego funcional basado en código DeFiTankLand modificado.
* Requería registro de usuario en la plataforma.
* Loader YouieLoad integrado ejecutándose durante inicio del juego.
* Gameplay real para mantener ilusión de legitimidad.

La funcionalidad completa del juego servía múltiples propósitos:
1. Reducir sospechas al proporcionar experiencia de usuario real.
2. Vector de infección alternativo vía YouieLoad para usuarios de navegadores no vulnerables.
3. Prolongar tiempo de exposición al mantener víctimas interactuando con la plataforma.
4. Facilitar ingeniería social al tener producto "tangible" que promover.

## 4.3. Presencia en Redes Sociales

Lazarus estableció presencia significativa en múltiples plataformas de redes sociales como parte integral de la infraestructura de ataque, utilizándolas para legitimación, promoción y targeting de víctimas.

### Plataforma X (Twitter)

**Estrategia de operación:**
* Múltiples cuentas creadas con meses de antelación a la campaña activa.
* Construcción orgánica de followers mediante interacción con comunidad crypto.
* Publicación regular de contenido relacionado con DeFi, NFTs y gaming.
* Promoción activa de DeTankZone mediante posts, threads y retweets.
* Uso de hashtags relevantes del sector (DeFi, NFT, PlayToEarn, CryptoGaming).
* Engagement con influencers y figuras prominentes del espacio crypto.

**Contenido generado:**
* Gráficos y renders del juego creados con herramientas de IA generativa.
* Teasers de gameplay y features del juego.
* Anuncios de "milestones" del proyecto (desarrollo, partnerships falsos, próximos lanzamientos).
* Contenido educativo sobre DeFi para posicionarse como expertos.
* Interacción aparentemente genuina con otros usuarios.

La estrategia de X permitió alcance orgánico amplificado mediante algoritmo de la plataforma, exponiendo DeTankZone a audiencias relevantes sin necesidad de publicidad pagada masiva (aunque posiblemente utilizaron promoted tweets selectivamente).

### LinkedIn

**Perfiles profesionales:** Lazarus creó perfiles en LinkedIn presentándose como:
* Desarrolladores de blockchain y smart contracts.
* Fundadores/cofundadores de startups DeFi.
* Representantes de fondos de capital riesgo en crypto.
* Business development managers de empresas gaming.
* Consultores especializados en tokenomics.

**Tácticas de targeting:**
* Identificación de figuras influyentes mediante búsquedas avanzadas de LinkedIn.
* Conexión directa con targets de alto valor en el sector crypto.
* Envío de mensajes personalizados solicitando feedback, colaboración o inversión.
* Presentación de DeTankZone como oportunidad de inversión o partnership.
* Uso de cuentas Premium para mayor credibilidad y funcionalidades avanzadas.

La utilización de LinkedIn fue particularmente efectiva para targeting de individuos con capacidad económica significativa y decisores en empresas crypto, aprovechando la cultura profesional de networking activa en la plataforma.

### Plataformas de Mensajería (Telegram, Discord)

Aunque los detalles específicos son limitados en las fuentes públicas, la presencia en plataformas de mensajería es inferida por patrones históricos de Lazarus y menciones en análisis de la campaña:
* Creación de servidor Discord oficial de DeTankZone (probablemente).
* Grupo/canal de Telegram para comunidad del proyecto.
* Mensajería directa a miembros de comunidades crypto existentes.
* Participación en servers Discord de proyectos relacionados.
* Distribución de links al sitio `detankzone[.]com` vía DMs.

### Características Técnicas de la Operación en Redes Sociales

**Gestión de cuentas:**
* Múltiples perfiles operados simultáneamente desde infraestructura distribuida.
* Uso probable de proxies/VPNs para diversificar geolocalización de IPs.
* Scheduling de posts para parecer actividad orgánica en diferentes timezones.
* Interacción coordinada entre cuentas para amplificar alcance.

**Evasión de detección de plataformas:**
* Construcción gradual de presencia (meses antes de campaña activa) para evadir detección de comportamiento spam.
* Contenido mixto legítimo/promocional para mantener ratios normales.
* Evitación de patrones obviamente automatizados en timing y contenido.
* Respuesta a comentarios y DMs para simular operación humana real.

## 4.4. Infraestructura de Command and Control

La infraestructura C2 permitió comunicación persistente con sistemas comprometidos, aunque detalles específicos son limitados debido al desmantelamiento temprano.

### Arquitectura C2

Basándose en capacidades de Manuscrypt y patrones de Lazarus, la arquitectura C2 probablemente incluía:
* **Múltiples servidores C2:** Arquitectura distribuida con servidores en diferentes jurisdicciones para redundancia.
* **Protocolo encriptado:** Comunicaciones cifradas mediante algoritmos personalizados sobre HTTP/HTTPS.
* **Domain fronting:** Uso de CDNs o servicios legítimos como fachada para ocultar destino real.
* **Beaconing flexible:** Intervalos variables con jitter aleatorio para evadir detección por patrones de tráfico.
* **Fallback mechanisms:** Múltiples dominios/IPs configurados en malware para mantener comunicación si servidores primarios caen.

### Funcionalidades C2

El C2 proporcionaba capacidades operacionales para los atacantes:
* **Recepción de datos de validator:** Centralización de información de reconocimiento de víctimas.
* **Envío de comandos:** Instrucciones remotas para ejecución de comandos, descarga de tools adicionales.
* **Gestión de payloads:** Distribución selectiva de Manuscrypt u otros componentes según valor del objetivo.
* **Exfiltración de datos:** Recepción de datos robados (credenciales, wallets, documentos).
* **Actualización de malware:** Capacidad de actualizar componentes instalados para evadir detecciones.

### Estado de la Infraestructura

Para mayo 2024, cuando Kaspersky descubrió la campaña:
* El backend del juego DeTankZone había sido desmantelado por los atacantes.
* Servidores C2 específicos probablemente ya no estaban operacionales.
* Dominios IoC continuaban registrados pero sin servir contenido malicioso activo.
* Infraestructura de redes sociales permanecía parcialmente activa (perfiles existentes pero sin actividad).
* El desmantelamiento sugiere que Lazarus detectó el escrutinio de investigadores de seguridad o completó los objetivos primarios de la campaña, procediendo a eliminar evidencia de infraestructura activa.