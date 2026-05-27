ADR-01 — Uso de Azure Data Factory sobre Azure Logic Apps para la orquestación del pipeline Estado: Aceptado

Contexto DataCo requiere orquestar la ingesta de datos desde cuatro sistemas fuente heterogéneos (SAP ERP vía SFTP, Oracle Database vía exportación, GPS Flota vía CSV manual y Salesforce CRM vía REST API) hacia el Data Lake Storage Gen2, ejecutando este proceso cada 4 horas de forma automática y tolerante a fallos parciales. El equipo de datos está compuesto por 2 analistas con conocimientos de SQL y Python básico, sin experiencia en administración de infraestructura. El presupuesto mensual en Azure no debe superar los $80 USD durante la fase piloto.

Alternativas evaluadas

Opción A — Azure Data Factory

Herramienta de orquestación de datos nativa de Azure, diseñada específicamente para pipelines ETL/ELT a escala empresarial. Ofrece conectores nativos para SAP, Oracle, Salesforce y sistemas de archivos SFTP sin necesidad de código adicional. Interfaz visual de arrastrar y soltar que reduce la curva de aprendizaje para el equipo de DataCo. Gestión nativa de dependencias entre actividades, reintentos automáticos y tolerancia a fallos parciales por pipeline. Tier básico gratuito con hasta 5 actividades de bajo costo por mes, suficiente para el volumen del piloto. Monitoreo integrado con logs de ejecución, alertas y métricas de rendimiento. Pensado para volúmenes de datos masivos con soporte para hasta millones de registros por ejecución.

Opción B — Azure Logic Apps

Herramienta de automatización de flujos de trabajo orientada a la integración de aplicaciones y servicios, no a la ingesta masiva de datos. Ofrece conectores para Salesforce y sistemas de archivos, pero sin soporte nativo optimizado para SAP On-Premise ni Oracle Database. El modelo de cobro es por ejecución de acciones individuales, lo que puede generar costos impredecibles con volúmenes altos de datos en ciclos de 4 horas. No está diseñado para manejar tolerancia a fallos parciales entre pipelines de datos interdependientes. Carece de monitoreo especializado para pipelines de datos (sin métricas de registros procesados, sin linaje de datos). Más adecuado para automatización de procesos de negocio ligeros que para orquestación de pipelines ETL a escala.

Decisión

Se selecciona Azure Data Factory como orquestador del pipeline de DataCo. Azure Data Factory es la herramienta técnicamente correcta para este caso porque fue diseñada específicamente para orquestar pipelines de datos a escala, ofrece conectores nativos para todas las fuentes de DataCo y su modelo de costos es predecible dentro del presupuesto del piloto. La tolerancia a fallos parciales por pipeline es un requerimiento explícito del caso y Data Factory lo resuelve de forma nativa sin desarrollo adicional. Azure Logic Apps, aunque es una herramienta válida para automatización de procesos, no fue diseñada para este tipo de carga de trabajo y generaría sobrecostos e ineficiencias operativas en el contexto de DataCo.

Consecuencias

Ventajas obtenidas:

Ingesta automatizada desde las 4 fuentes con conectores nativos, sin desarrollo de integraciones a medida. Tolerancia a fallos parciales garantizada: si una fuente falla, las demás continúan procesándose. Monitoreo centralizado de ejecuciones con alertas y logs auditables. Curva de aprendizaje reducida para el equipo de analistas gracias a la interfaz visual. Costo dentro del presupuesto piloto de $80 USD/mes con el tier básico gratuito.

Trade-offs asumidos:

Azure Data Factory agrega una dependencia adicional al stack de Azure, aumentando levemente la complejidad operativa. El tier gratuito tiene límites en el número de actividades mensuales, lo que requerirá revisión al escalar a producción.
