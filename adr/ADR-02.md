ADR-02 — Uso de Azure Databricks sobre Azure Synapse Analytics para la transformación de datos Estado: Aceptado

Contexto DataCo necesita un motor de transformación capaz de limpiar, estandarizar, enriquecer y consolidar datos provenientes de cuatro sistemas fuente con problemas graves de calidad: códigos de producto inconsistentes entre SAP y Oracle, nombres de clientes duplicados entre ERP y CRM, fechas mal formateadas y registros duplicados. El motor debe procesar hasta 5 millones de registros por ejecución para soportar cierres de mes. El equipo tiene conocimientos básicos de Python y SQL pero ninguna experiencia en Spark ni en administración de clusters. El presupuesto es de máximo $80 USD/mes y se requiere una solución gratuita para la fase piloto.

Alternativas evaluadas

Opción A — Azure Databricks Community Edition

Plataforma de procesamiento distribuido basada en Apache Spark, con soporte nativo para Python (PySpark) y SQL. La Community Edition es completamente gratuita, sin necesidad de tarjeta de crédito, con clusters de hasta 15 GB de RAM. Entorno de notebooks interactivos que facilita el desarrollo iterativo y la depuración de transformaciones, ideal para un equipo con experiencia básica en Python. Soporte nativo para lectura y escritura de archivos Parquet desde Azure Data Lake Gen2. Capacidad de procesar millones de registros mediante procesamiento distribuido en Spark, cumpliendo el requerimiento de escalabilidad del caso. Amplia documentación, comunidad activa y curva de aprendizaje progresiva para equipos que inician con Spark. Integración nativa con Azure Data Factory para orquestación de notebooks.

Opción B — Azure Synapse Analytics

Plataforma unificada de análisis que combina procesamiento de datos, almacén de datos y herramientas de BI en un solo servicio. No dispone de un tier completamente gratuito equivalente a Databricks Community Edition; los costos de los pools de Spark dedicados superan el presupuesto piloto de $80 USD/mes. Mayor complejidad de configuración y administración, especialmente para equipos sin experiencia en plataformas de datos distribuidos. Adecuado para organizaciones que buscan consolidar análisis, ingesta y visualización en una única plataforma, lo cual no es el objetivo de DataCo en esta fase. La propuesta de valor principal de Synapse (unificación de servicios) no aplica en este caso, ya que DataCo ya tiene definido un stack con servicios especializados.

Decisión

Se selecciona Azure Databricks Community Edition como motor de transformación del pipeline de DataCo. La decisión está fundamentada en tres factores determinantes del caso: costo cero durante el piloto, capacidad de procesamiento distribuido para 5 millones de registros y entorno de notebooks compatible con las habilidades actuales del equipo (Python básico). Azure Synapse Analytics, aunque es técnicamente más completo, excede el presupuesto del piloto y su complejidad de configuración representa un riesgo operativo para un equipo de 2 analistas sin experiencia en plataformas distribuidas.

Consecuencias

Ventajas obtenidas:

Costo cero en la fase piloto con Databricks Community Edition. Capacidad de procesar hasta 5 millones de registros por ejecución mediante Spark distribuido. Entorno de notebooks que facilita el desarrollo iterativo y la colaboración del equipo. Escalabilidad garantizada para temporadas altas y cierres de mes.

Trade-offs asumidos:

La Community Edition no tiene SLA de disponibilidad, lo que la hace inadecuada para entornos de producción críticos a futuro. Los clusters se terminan automáticamente tras períodos de inactividad, lo que puede generar tiempos de inicio en cada ejecución del pipeline. Al escalar a producción, será necesario migrar a un tier de pago de Databricks o evaluar nuevamente Synapse Analytics.
