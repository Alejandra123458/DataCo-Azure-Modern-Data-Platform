ADR-03 — Uso de Azure Data Lake Storage Gen2 sobre Azure Blob Storage estándar como almacenamiento raw Estado: Aceptado

Contexto

DataCo necesita un sistema de almacenamiento capaz de recibir archivos en formato CSV y JSON desde cuatro fuentes heterogéneas, organizarlos por fuente y fecha de ingesta, y servirlos como entrada al motor de transformación en Databricks. Los datos contienen información sensible de precios y márgenes por cliente, por lo que el acceso debe estar controlado por roles. El pipeline debe escalar para procesar hasta 5 millones de registros y los requerimientos de trazabilidad exigen auditoría completa de cada operación sobre los datos.

Alternativas evaluadas

Opción A — Azure Data Lake Storage Gen2

Servicio de almacenamiento de objetos construido sobre Azure Blob Storage pero con capacidades adicionales para cargas de trabajo analíticas a escala. Implementa un sistema de archivos jerárquico (ADLS) que permite organizar los datos en directorios por fuente, año, mes y día, facilitando la partición y el acceso eficiente desde Spark. Soporte nativo para control de acceso granular mediante ACLs (Access Control Lists) a nivel de archivo y directorio, cumpliendo el requerimiento de restricción de acceso a datos sensibles de DataCo. Integración nativa con Azure Databricks y Azure Data Factory sin configuración adicional. Optimizado para cargas de trabajo analíticas de alto rendimiento con soporte para el formato Parquet. Compatible con el protocolo ABFS (Azure Blob File System), que ofrece mejor rendimiento en operaciones de lectura/escritura masiva comparado con el protocolo REST estándar de Blob Storage. Mismo modelo de costos que Blob Storage estándar (LRS Standard), sin sobrecosto por las capacidades adicionales.

Opción B — Azure Blob Storage estándar

Servicio de almacenamiento de objetos de propósito general de Azure, sin sistema de archivos jerárquico nativo. No soporta ACLs a nivel de directorio o archivo; el control de acceso se limita a nivel de contenedor, lo que impide la granularidad requerida para proteger los datos sensibles de precios y márgenes de DataCo. Menor rendimiento en operaciones de lectura/escritura masiva desde Spark en comparación con Data Lake Gen2 para cargas analíticas. No está optimizado para patrones de acceso analítico (partición por fecha, lectura selectiva de columnas en Parquet). Adecuado para almacenamiento de archivos estáticos, backups o contenido web, no para pipelines de datos analíticos.

Decisión

Se selecciona Azure Data Lake Storage Gen2 como capa de almacenamiento del pipeline de DataCo. El sistema de archivos jerárquico y el control de acceso granular mediante ACLs son los factores determinantes: DataCo maneja datos sensibles de precios y márgenes que requieren restricción de acceso a nivel de directorio, algo que Blob Storage estándar no puede garantizar. Adicionalmente, el rendimiento superior de ADLS Gen2 en operaciones analíticas con Spark es crítico para cumplir el requerimiento de procesar 5 millones de registros por ejecución dentro del ciclo de 4 horas. El hecho de que no represente costo adicional sobre Blob Storage estándar elimina cualquier trade-off económico.

Consecuencias

Ventajas obtenidas:

Control de acceso granular a nivel de archivo y directorio para proteger datos sensibles. Organización jerárquica por fuente y fecha que facilita la partición eficiente en Spark. Rendimiento optimizado para operaciones analíticas masivas. Soporte nativo para Parquet como formato estándar en la zona curated. Sin costo adicional respecto a Blob Storage estándar.

Trade-offs asumidos:

La configuración de ACLs requiere una gestión de identidades más cuidadosa que Blob Storage, añadiendo complejidad operativa inicial. El equipo deberá familiarizarse con el modelo de permisos POSIX de ADLS Gen2, diferente al modelo de roles de Blob Storage.
