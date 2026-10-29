# Despliegue de MISP v2.5 y Alimentación de Inteligencia de Amenazas (OSINT Feeds)

## 📌 Descripción del Proyecto
Este repositorio documenta el despliegue manual, endurecimiento y configuración avanzada de la plataforma de compartición de información de malware **MISP v2.5** sobre un entorno limpio de **Ubuntu 24.04 LTS (Noble Numbat)**. 

El objetivo de este laboratorio práctico ( Bootcamp Analista SOC Nivel 2) es simular el flujo de trabajo real de un analista de ciberseguridad: aprovisionar infraestructura core, mitigar excepciones complejas del servidor web, estructurar incidentes bajo taxonomías internacionales y automatizar la ingesta de indicadores de compromiso (IoCs) mediante fuentes abiertas.

---

## 🛠️ Arquitectura e Infraestructura Desplegada
*   **Hipervisor:** Oracle VirtualBox (Configuración de red en modo *Adaptador Puente / Bridged*).
*   **Servidor Web:** Apache2 (Ajuste de parámetros con redirecciones nativas HTTPS y directivas `ProxyPreserveHost On`).
*   **Base de Datos Relacional:** MariaDB Server (Inyección limpia del esquema SQL base).
*   **Almacenamiento en Memoria / Caché:** Redis-Server (Gestión activa de tokens y sesiones).
*   **Entorno de Ejecución:** PHP 8.3 & Python 3 Virtualenv (Instalación de dependencias core vía Composer).

---

## 🚀 Fases del Laboratorio y Contribuciones Técnicas

### 1. Resolución de Bloqueos de Permisos y Políticas de Red
Durante el aprovisionamiento manual, se identificaron y resolvieron diversos conflictos de dependencias y políticas de seguridad:
*   **Mitigación de la excepción CSP (Content Security Policy):** Se corrigieron las variables `'baseurl'` y `'external_baseurl'` en `config.php` para evitar bloqueos del navegador (*Blackhole Exception*) provocados por el enrutamiento virtual defectuoso del framework.
*   **Gestión de Permisos Cruzados:** Sincronización criptográfica del parámetro `Security.salt` y reasignación de privilegios de lectura/escritura (`www-data:www-data`) sobre las carpetas temporales de CakePHP.

### 2. Modelado de Incidentes: Campaña de Phishing "Banco Ejemplo"
Se simuló la ingesta y clasificación estructurada de un reporte de amenazas corporativo:
*   **Ingesta de IoCs:** Utilización de la herramienta *Freetext Import Tool* para registrar de forma masiva direcciones IP de comando y control (C2), dominios de phishing y el hash criptográfico SHA256 del adjunto malicioso.
*   **Clasificación Internacional:** Enriquecimiento del evento mediante etiquetas **TLP:AMBER** y **PAP:AMBER** para el control estricto de la distribución.
*   **Contexto Táctico:** Vinculación directa con el framework global **MITRE ATT&CK** asociando los patrones de ataque:
    *   *T1566.001 — Spearphishing Attachment*
    *   *T1566.002 — Spearphishing Link*
    *   *T1071.001 — Web Protocols*

### 3. Automatización y Orquestación de Feeds OSINT
Activación masiva del ecosistema de recolección inteligente de MISP:
*   Carga de metadatos maestros y habilitación de caché en memoria para optimizar la velocidad de búsqueda.
*   Sincronización e importación completa (*Fetch*) de eventos analizados por agencias de ciberseguridad de alto nivel como **CIRCL OSINT Feed**, **Botvrij.eu** y **URLhaus (abuse.ch)**.

---

## 💡 Aprendizaje Clave de Ingeniería de Detección
El verdadero reto de implementar soluciones de Threat Intelligence de código abierto no reside únicamente en la interpretación de los artefactos maliciosos, sino en la capacidad de realizar un *troubleshooting* profundo de sistemas. Entender cómo interactúan las capas del servidor web (Apache), los motores de bases de datos (MariaDB) y el motor de políticas de seguridad del navegador es fundamental para un Analista SOC que busca desplegar herramientas robustas de defensa proactiva en un entorno empresarial.
