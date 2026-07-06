# Caso de Estudio: Arquitectura de Servidores, Migración e Infraestructura Web – Portal de Investigación Universitaria

Este repositorio documenta el proceso técnico de auditoría, despliegue de infraestructura, optimización de rendimiento y resolución de problemas de base de datos llevado a cabo para el portal del Vicerrectorado de Investigación de una universidad nacional.

## ⚙️ Tecnologías Utilizadas
* **Infraestructura & Servidores:** HestiaCP (VPS Linux), Nginx (Proxy Inverso), Apache.
* **Bases de Datos:** MariaDB / MySQL, phpMyAdmin.
* **Automatización & CMS:** WP-CLI, WordPress, Divi Builder.
* **Rendimiento (WPO):** Formato de imagen de última generación (WebP).

---

## 📌 El Desafío (The Challenge)
El proyecto requería migrar de forma integral el portal web institucional desde un entorno de desarrollo local (LocalWP) hacia un servidor de producción dedicado. El escenario presentaba múltiples retos críticos de infraestructura y deudas técnicas acumuladas:

1. **Montaje de Infraestructura desde Cero:** Necesidad de aprovisionar un servidor limpio y configurar un entorno de panel de control robusto, seguro y eficiente.
2. **Deuda Técnica y Basura Histórica (2015-2026):** El sitio acumulaba más de una década de datos residuales, tablas huérfanas de plugins desinstalados, revisiones obsoletas y registros transaccionales basura que inflaban el peso de la base de datos y bloqueaban los límites estándar de importación del servidor.
3. **Seguridad y Enlaces Rotos:** Presencia de "Contenido Mixto" (Mixed Content) persistente tras la instalación del certificado SSL, sumado a fallos strictly estáticos de lectura por parte de Nginx en entornos móviles.
4. **Incompatibilidad Estructural de Plugins Clave:** Desaparición completa de 66 tablas de datos históricos debido a un conflicto de retrocompatibilidad entre la versión clásica del plugin 'League Table' (`wp_dalt_`) y la versión moderna (`wp_daextletal_`), rompiendo los shortcodes y arrojando el error *"There is no table associated with this shortcode"*.

---

## 🛠️ Soluciones Implementadas (The Solution)

### 1. Despliegue de Arquitectura y Configuración del Servidor VPS
* **Aprovisionamiento Core:** Realicé el montaje, securización y configuración completa desde cero del sistema operativo del servidor VPS utilizando **HestiaCP** como panel de control centralizado.
* **Orquestación y Ajustes del Motor (MariaDB):** Configuré la pila de servicios web (Nginx + Apache) y reconfiguré los límites globales del sistema expandiendo el parámetro `max_allowed_packet` para procesar el volcado de la base de datos masiva sin interrupciones. Asimismo, integré phpMyAdmin de manera global en la arquitectura del panel.

### 2. Auditoría, Limpieza y Depuración Profunda de Datos (11 Años de Historial)
* **Data Cleansing Masivo:** Ejecuté un proceso minucioso de auditoría SQL para depurar registros redundantes acumulados desde el año 2015 hasta el 2026 (transitorios expirados, metadatos huérfanos, revisiones y logs), logrando una reducción drástica del peso físico del archivo final y optimizando la velocidad de las consultas.
* **Sanitización con WP-CLI:** Automatizé la corrección de la serialización de datos de WordPress a nivel de terminal empleando herramientas CLI y erradiqué el contenido mixto de URLs inseguras (`http://`) remanentes de la migración.

### 3. Ingeniería Inversa y Resolución de Retrocompatibilidad en Base de Datos
* **Análisis de Dependencias:** Identifiqué mediante ingeniería inversa que el código PHP de la versión del plugin llamaba rígidamente a la estructura de tablas clásica (`wp_dalt_...`).
* **Plan de Contingencia y Sincronización:** Extraje las tablas puras del entorno local conservando intactos los identificadores únicos (`table_id`) y forcé la estructura clásica que el código PHP requería de manera nativa. Esto solucionó los errores de shortcodes huérfanos en el maquetador visual.
* **Gestión de Caché Estática en Divi:** Se detectó que Divi compilaba archivos CSS estáticos avanzados de forma rígida. Se forzó el vaciado técnico y la regeneración de archivos CSS estáticos y cachés del servidor mediante comandos CLI:
  ```bash
  php wp-cli.phar cache flush --allow-root
