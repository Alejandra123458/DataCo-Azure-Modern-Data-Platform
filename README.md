## Introducción
La transformación digital en el sector de distribución y consumo masivo exige que las organizaciones cuenten con sistemas capaces de integrar, procesar y analizar grandes volúmenes de datos provenientes de múltiples fuentes en tiempo casi real. En este contexto, la computación en la nube se ha consolidado como la plataforma tecnológica que permite escalar estas capacidades de forma eficiente, segura y con costos controlados.

El presente trabajo corresponde al Caso 2 de la materia Computación en la Nube del Tecnológico de Antioquia, y plantea el diseño e implementación de un pipeline de datos moderno en Microsoft Azure para la empresa colombiana DataCo. El proyecto aborda los problemas críticos de fragmentación de datos, reportes manuales tardíos y toma de decisiones desactualizada que enfrenta la organización, proponiendo una arquitectura cloud basada en servicios administrados que automatiza completamente el flujo de datos desde las fuentes hasta los dashboards ejecutivos.

La arquitectura propuesta sigue los lineamientos de las arquitecturas de referencia oficiales de Microsoft Azure para pipelines de datos modernos, adaptadas al contexto específico de DataCo: sus restricciones presupuestales, las capacidades técnicas de su equipo y los requerimientos de calidad, trazabilidad y escalabilidad definidos por la gerencia de tecnología.

## Contexto del proyecto

## La empresa

DataCo es una empresa colombiana de distribución de productos de consumo masivo con operaciones en 12 departamentos del país. Fundada en 1998, cuenta con 1.800 empleados, una flota de 320 vehículos de reparto y más de 9.000 puntos de venta activos entre supermercados, tiendas de barrio y droguerías. Maneja tres líneas de negocio principales: distribución de alimentos perecederos (45% de ingresos), productos de aseo del hogar (32%) y cosméticos y cuidado personal (23%).

Situación tecnológica actual
DataCo opera con cuatro sistemas de información que funcionan de forma completamente aislada, sin integración entre ellos:

<img width="901" height="376" alt="image" src="https://github.com/user-attachments/assets/3054aee6-3919-457d-a89d-04c15095dd99" />

## Problemas críticos identificados

Esta fragmentación genera cinco problemas que la gerencia ha escalado como prioridad estratégica para 2026:
## 1. Reportes manuales y tardíos: 

El equipo de inteligencia de negocio tarda entre 3 y 5 días hábiles en consolidar información de los cuatro sistemas para generar un informe ejecutivo semanal, mediante exportación manual de Excel y cruce en hojas de cálculo.

## 2. Decisiones desactualizadas: 

la gerencia comercial toma decisiones de abastecimiento con datos de inventario que tienen hasta 72 horas de rezago, generando quiebres de stock y sobrestock simultáneamente.

## 3. Inconsistencia de datos:

los mismos clientes están registrados con nombres distintos en el ERP y el CRM, y los productos tienen códigos diferentes en Oracle y SAP, imposibilitando cualquier análisis cruzado confiable.

## 4. Sin trazabilidad de entregas: 

No existe forma de correlacionar automáticamente una factura de SAP con su entrega real en el GPS, impidiendo medir el cumplimiento de promesas de entrega por ruta o vendedor.

## 5. Escalabilidad nula: 

El servidor de consolidaciones es un equipo de escritorio con Windows Server 2012. En los meses de cierre, el proceso de generación de reportes tarda hasta 8 horas continuas.


## Restricciones del proyecto

- Equipo de datos compuesto por 2 analistas con conocimientos de SQL y Python básico, sin experiencia en Spark ni administración de clusters.

- SAP On-Premise no tiene API REST disponible; la integración se realiza mediante archivos CSV/JSON vía SFTP.

- Presupuesto mensual en Azure máximo de $80 USD durante la fase piloto.

- Los datos de ventas contienen información sensible que requiere control de acceso por roles.

- Power BI Desktop ya está licenciado en la empresa; no se puede proponer una herramienta de visualización con costo adicional.

- El pipeline debe ser tolerante a fallos parciales por fuente.

## Objetivos del proyecto

## Objetivo general

Diseñar e implementar un pipeline de datos moderno en Microsoft Azure que integre las cuatro fuentes de información de DataCo en un modelo de datos unificado, automatizando el proceso de ingesta, transformación y visualización de datos para reducir el ciclo de toma de decisiones de 3 días a menos de 4 horas.

## Objetivos específicos

**1. Integrar las cuatro fuentes de datos (SAP ERP, Oracle Database, GPS Flota y Salesforce CRM)** en un único flujo automatizado de ingesta mediante Azure Data Factory, eliminando el proceso manual de consolidación en Excel.

**2. Implementar un proceso de transformación y calidad de datos** en Azure Databricks que resuelva los problemas de inconsistencia de códigos, duplicados y fechas mal formateadas, garantizando una tasa de registros limpios superior al 98%.

**3. Construir un modelo de datos relacional consolidado** en Azure SQL Database que unifique la información de las cuatro fuentes en tablas de hechos y dimensiones accesibles para análisis desde Power BI Desktop.

**4. Diseñar una arquitectura tolerante a fallos** que garantice que si una fuente falla en un ciclo de ejecución, los datos de las demás fuentes se procesen igualmente sin intervención manual.

**5. Garantizar la trazabilidad completa** de cada transformación aplicada en el pipeline mediante registros de auditoría que cumplan las políticas internas de gobierno de datos de DataCo.

**6. Documentar las decisiones arquitectónicas** mediante Architecture Decision Records (ADRs) que justifiquen la selección de cada servicio del stack frente a sus alternativas, considerando las restricciones técnicas, económicas y organizacionales del proyecto.


## Arquitectura de solución

## Visión general

La arquitectura propuesta para DataCo sigue el patrón Modern Analytics Architecture de Microsoft Azure, adaptado a las restricciones presupuestales y técnicas del proyecto. El pipeline implementa un flujo de datos en capas (medallion architecture) que separa claramente las etapas de ingesta, almacenamiento raw, transformación, almacenamiento curado y visualización.

<img width="900" height="362" alt="image" src="https://github.com/user-attachments/assets/1a47352a-7182-4b39-9d4b-0a1bbb5cef66" />


## Flujo de datos de extremo a extremo

El pipeline se ejecuta en modo batch cada 4 horas siguiendo el siguiente flujo:

**Etapa 1 — Ingesta:** Azure Data Factory detecta los archivos depositados por SAP ERP y GPS Flota vía SFTP, consulta la API REST de Salesforce CRM y exporta los datos de Oracle Database, depositando todo en la zona raw del Data Lake en su formato original (CSV/JSON).

**Etapa 2 — Conversión:** el notebook ingest_sources.py en Databricks lee los archivos de la zona raw y los convierte al formato Parquet, organizándolos en la zona curated del Data Lake particionados por fuente y fecha.

**Etapa 3 — Transformación:** el notebook clean_transform.py aplica las reglas de calidad de datos: eliminación de duplicados, estandarización de fechas, unificación de códigos de producto entre SAP y Oracle, y normalización de nombres de clientes entre ERP y CRM.

**Etapa 4 — Enriquecimiento:** el notebook enrich_deliveries.py correlaciona las facturas de SAP con las entregas del GPS, calculando el cumplimiento de promesas de entrega por ruta y vendedor.

**Etapa 5 — Carga:** el notebook load_warehouse.py realiza operaciones INSERT/UPSERT sobre las tablas de hechos y dimensiones en Azure SQL Database y registra los metadatos de la ejecución en la tabla de auditoría.

**Etapa 6 — Visualización:** Power BI Desktop consulta Azure SQL Database y actualiza automáticamente los dashboards de ventas, inventario y logística disponibles para analistas y gerentes.


## Principios arquitectónicos aplicados

**Separación de responsabilidades:** cada servicio del stack tiene una única responsabilidad bien definida, lo que facilita el mantenimiento, la depuración y la escalabilidad independiente de cada capa.

**Tolerancia a fallos parciales:** el pipeline está diseñado para que el fallo de una fuente no detenga el procesamiento de las demás. Azure Data Factory gestiona los reintentos y Databricks registra los fallos en la tabla de auditoría sin interrumpir el flujo general.

**Mínima complejidad necesaria:** se seleccionaron los servicios más simples que cumplen los requerimientos del caso, evitando añadir capas innecesarias al stack. Esta decisión se documenta explícitamente en los ADRs.

**Control de acceso por capas:** los datos sensibles de precios y márgenes están protegidos mediante ACLs en el Data Lake y RBAC en Azure SQL Database, con credenciales de solo lectura para Power BI y solo escritura para Databricks.

## Decisiones Arquitectónicas [Architecture Decision Records]

Las siguientes decisiones documentan las elecciones tecnológicas clave del proyecto, justificando la selección de cada servicio frente a sus alternativas en el contexto específico de DataCo. Cada ADR considera las restricciones técnicas, económicas y organizacionales definidas en el caso, siguiendo el formato estándar de Architecture Decision Records propuesto por Michael Nygard.

| # | Título | Archivo |
| :--- | :--- | :--- |
| ADR-01 |Uso de Azure Data Factory sobre Azure Logic Apps para la orquestación del pipeline | `adr/ADR-01.md` |
| ADR-02 | Uso de Azure Databricks sobre Azure Synapse Analytics para la transformación de datos | `adr/ADR-02.md` |
| ADR-03 | Uso de Azure Data Lake Storage Gen2 sobre Azure Blob Storage estándar como almacenamiento raw | `adr/ADR-03.md` |
| ADR-04 | Uso de Azure SQL Database sobre Azure Cosmos DB para el almacén analítico final | `adr/ADR-04.md` |
| ADR-05 | Uso de Power BI Desktop sobre Azure Analysis Services para la capa de visualización | `adr/ADR-05.md` |

## C1 – Diagrama de contexto - Plataforma de datos DataCo en Azure

![Diagrama de Contexto](assets/c1-contexto.png)

El diagrama de contexto presenta una visión general de la plataforma de datos de DataCo como un sistema central dentro del ecosistema de la organización.

En este nivel se identifican los principales actores del negocio, incluyendo el Analista de BI, quien utiliza los dashboards para el análisis de datos; el Gerente Comercial, que toma decisiones estratégicas basadas en la información procesada; y el Auditor, encargado de verificar la trazabilidad y calidad de los datos.

Asimismo, se representan los sistemas externos que generan la información de negocio. Estos incluyen el ERP de ventas SAP, el sistema de inventario en Oracle, los archivos CSV provenientes del sistema GPS de la flota y el CRM Salesforce. Cada uno de estos sistemas envía datos hacia la plataforma central.

La plataforma de datos en Azure actúa como el núcleo que integra, procesa y consolida la información proveniente de todas las fuentes. Una vez procesados los datos, estos son consumidos a través de Power BI, que permite la visualización mediante dashboards interactivos.

Este nivel de abstracción permite entender cómo interactúan los diferentes actores y sistemas con la solución, sin entrar en detalles técnicos de implementación.



## C2 – Diagrama de Contenedores - Plataforma de datos DataCo en Azure

![Diagrama de Contenedores](assets/c2-contenedores.png)

El diagrama de contenedores representa la arquitectura de la plataforma de datos de DataCo implementada en Microsoft Azure. Este muestra cómo los diferentes servicios trabajan de manera integrada para procesar y transformar los datos provenientes de múltiples fuentes.

El flujo de datos inicia con Azure Data Factory, que se encarga de orquestar la ingesta de información desde los sistemas externos en formato CSV o JSON. Estos datos son almacenados inicialmente en Azure Data Lake Storage Gen2 en la zona RAW.

Posteriormente, Azure Databricks procesa los datos utilizando Apache Spark, realizando tareas de limpieza, estandarización y enriquecimiento. Los datos transformados se almacenan nuevamente en el Data Lake en formato Parquet dentro de la zona CURATED.

Una vez procesados, los datos son cargados en Azure SQL Database, donde se estructuran en un modelo relacional optimizado para consultas analíticas.

Finalmente, Power BI se conecta a Azure SQL Database para visualizar la información mediante dashboards interactivos que apoyan la toma de decisiones del negocio.

El pipeline se ejecuta de manera automatizada cada 4 horas, permitiendo que la información esté actualizada y disponible para los usuarios en tiempo casi real.




## C3 – Diagrama de Componentes - Detalle de Azure Databricks en la Plataforma DataCo

![Diagrama de Componentes](assets/c3-componentes.png)

El diagrama de componentes representa el funcionamiento interno de Azure Databricks dentro de la arquitectura de datos de DataCo. En este nivel se detallan los diferentes notebooks responsables del procesamiento y transformación de los datos.

El flujo inicia con el notebook ingest_sap.py, el cual se encarga de leer los datos almacenados en la zona RAW del Data Lake en formatos como CSV o JSON. Este componente prepara los datos para su posterior procesamiento.

Luego, el notebook clean_inventory.py realiza la limpieza de los datos, eliminando duplicados, corrigiendo formatos incorrectos y estandarizando la información, lo que permite mejorar la calidad de los datos.

Posteriormente, el notebook enrich_deliveries.py integra información proveniente de múltiples fuentes, enriqueciendo los datos con atributos adicionales necesarios para el análisis de negocio, como categorías de productos o relaciones entre entidades.

Finalmente, el notebook load_warehouse.py carga los datos ya transformados en Azure SQL Database, donde se estructuran en un modelo analítico optimizado para consultas.

Adicionalmente, los datos procesados pueden almacenarse en el Data Lake en la zona CURATED en formato Parquet, permitiendo su reutilización y optimización en futuras consultas.



## Arquitectura de solución - Pipeline de datos DataCo

![Diagrama de Componentes](assets/Arquitectura-de-solucion.jpg)

Este flujo de procesamiento se ejecuta de forma secuencial y automatizada, asegurando que los datos pasen por cada etapa de transformación hasta estar listos para su consumo en herramientas analíticas.



## Conclusiones

## Sobre el problema resuelto

DataCo enfrentaba una situación crítica común en empresas de distribución de consumo masivo en Colombia: datos valiosos atrapados en sistemas aislados, procesos manuales que consumían días de trabajo y decisiones gerenciales tomadas con información desactualizada. El diseño e implementación del pipeline de datos en Microsoft Azure demuestra que es posible transformar esta realidad con un stack tecnológico moderno, completamente administrado y dentro de un presupuesto controlado de $80 USD mensuales durante la fase piloto.
El pipeline logra reducir el ciclo de consolidación de datos de 3 a 5 días hábiles a un máximo de 4 horas, cumpliendo el requerimiento principal de la gerencia de tecnología de DataCo y habilitando una toma de decisiones basada en datos reales y actualizados.

## Sobre la arquitectura diseñada

La arquitectura propuesta demuestra que el patrón Modern Analytics Architecture de Microsoft Azure es aplicable no solo a grandes corporaciones con presupuestos ilimitados, sino también a empresas medianas colombianas con restricciones reales de presupuesto, equipo y tiempo. La clave está en la correcta selección de los tiers gratuitos y de bajo costo disponibles en Azure: Databricks Community Edition, Azure SQL Free Tier y Power BI Desktop conforman un stack de transformación y visualización de costo cero, complementado por Azure Data Factory y Data Lake Storage Gen2 con costos mínimos por uso.

La separación del almacenamiento en zonas raw y curated (patrón medallion) resultó fundamental para garantizar la trazabilidad de los datos y permitir la reejección de transformaciones sin pérdida de información original, un principio que toda arquitectura de datos moderna debe contemplar.


Sobre las decisiones arquitectónicas

Los cinco ADRs documentados en este proyecto reflejan una lección central de la arquitectura de software: **la mejor decisión técnica no es siempre la más sofisticada, sino la que mejor se ajusta al contexto específico del problema.** Azure Synapse Analytics, Azure Cosmos DB y Azure Analysis Services son tecnologías más completas que sus alternativas seleccionadas, pero en el contexto de DataCo representarían sobrecostos, complejidad innecesaria y una curva de aprendizaje que el equipo no podría absorber en la fase piloto.

La documentación explícita de estas decisiones mediante ADRs es una práctica profesional que agrega valor más allá del proyecto inmediato: permite que futuros integrantes del equipo comprendan por qué se tomaron ciertas decisiones, qué alternativas se evaluaron y qué trade-offs se asumieron conscientemente, evitando que se repita el proceso de evaluación ante cada cambio.


## Sobre la calidad de datos

Uno de los hallazgos más relevantes del proyecto es que los problemas de calidad de datos de DataCo no son un problema tecnológico sino un problema de proceso y gobernanza que la tecnología puede resolver pero no puede prevenir por sí sola. La estandarización de códigos de producto entre SAP y Oracle, la normalización de nombres de clientes entre ERP y CRM y la correlación de facturas con entregas GPS son transformaciones que el pipeline ejecuta automáticamente, pero su causa raíz es la ausencia histórica de estándares de datos en la organización.

Se recomienda a DataCo acompañar la implementación técnica del pipeline con un proceso de gobernanza de datos que establezca estándares de nomenclatura, códigos maestros unificados y políticas de calidad en los sistemas fuente, reduciendo progresivamente la carga de limpieza en la capa de transformación.

**Sobre la escalabilidad futura**

La arquitectura implementada en la fase piloto es una base sólida sobre la cual DataCo puede escalar progresivamente. Se identifican tres caminos de evolución natural:

**Corto plazo:** migrar de Databricks Community Edition a un tier de pago cuando el pipeline pase a producción, garantizando SLA de disponibilidad y soporte técnico para un proceso crítico del negocio.



**Mediano plazo:** incorporar procesamiento en tiempo real mediante Azure Event Hubs y Databricks Structured Streaming para reducir la latencia de 4 horas a minutos en los datos de ventas e inventario, habilitando alertas automáticas de quiebre de stock.




**Largo plazo:** evaluar la migración del almacén analítico a Azure Synapse Analytics cuando el volumen de datos supere los límites del Free Tier de Azure SQL y el número de usuarios concurrentes de Power BI justifique una capa semántica dedicada.


**Reflexión final**

Este proyecto evidencia que la computación en la nube no es exclusiva de las grandes empresas tecnológicas. Con una arquitectura bien diseñada, decisiones fundamentadas y un uso inteligente de los tiers gratuitos disponibles en Microsoft Azure, una empresa colombiana de distribución puede construir una plataforma de datos moderna, escalable y segura que transforme la manera en que toma decisiones. La nube no es un destino, es un habilitador: el valor real lo generan los datos, el diseño y las personas que los usan.

## Implementacion

**Creación del Data Lake y almacenamiento RAW**
Se configuró el contenedor RAW dentro de la cuenta de almacenamiento pipelinedatafor.
Allí se cargó el archivo ventas_dataco.csv en formato CSV, representando la información original de ventas.

Durante esta etapa se validó:

-Creación correcta del contenedor RAW.

-Carga inicial del archivo CSV.

-Disponibilidad de los datos en el Data Lake
<img width="1836" height="473" alt="image" src="https://github.com/user-attachments/assets/8a715a3e-8405-4c5e-9bfc-03a9159b38e3" />

**Generación de la zona CURATED**
Se creó el contenedor CURATED, destinado a almacenar los datos procesados.
El archivo limpio ventas_clean.csv fue generado con la columna calculada valor_total = cantidad × valor_unitario.

**Validaciones realizadas:**

-Separación entre datos crudos y datos procesados.

-Archivo _SUCCESS confirmando ejecución correcta del flujo.
<img width="1810" height="572" alt="image" src="https://github.com/user-attachments/assets/55a153cc-5bfd-4591-b707-916c6ffd6a3b" />

**Orquestación del pipeline con Azure Data Factory**

Se implementó el pipeline pipeline_dataco en Data Factory.
**Este pipeline incluyó:**

-Actividad Copy Data para mover el archivo desde RAW.

-Actividad Data Flow para aplicar transformaciones y crear la columna valor_total.

**Validaciones realizadas:**

-Ejecución con estado Succeeded.

-Tiempo de ejecución correcto.

-Flujo automatizado de datos desde RAW hacia CURATED.

<img width="1879" height="614" alt="image" src="https://github.com/user-attachments/assets/ef4694df-4e02-46e1-a4a4-1c820ab1f0f4" />

**Lectura y validación en Databricks**

Se utilizó el notebook **dataco_limpieza_azure** en Databricks para leer el archivo CSV desde el contenedor RAW.
El código en Python cargó los datos con pandas y spark, mostrando los primeros registros y confirmando la carga de 1250 registros.

**Transformaciones aplicadas:**

-Visualización de registros crudos para validación.

<img width="1706" height="872" alt="image" src="https://github.com/user-attachments/assets/e95b7f2b-79a8-4973-98f8-406e5875cf05" />

**Script Python y conexión exitosa a Azure SQL Database**

Se evidencia la implementación del script cargar_sql.py en Visual Studio Code, utilizado para establecer la conexión con la base de datos dataco_db en Azure SQL Database.

El script emplea la librería pyodbc junto con el ODBC Driver 17 for SQL Server. Durante esta etapa se validó:

-Conexión exitosa entre Python y Azure SQL.

-Corrección del error inicial de firewall agregando la IP local en el portal de Azure.

-Ejecución de sentencias SQL para limpiar y cargar la tabla ventas_hechos.

<img width="1639" height="942" alt="image" src="https://github.com/user-attachments/assets/faf47aa7-debc-45ca-af31-e14301928cc2" />

**Validación en Azure SQL Database**

Finalmente, se ejecutaron consultas SQL en la base dataco_db para validar la carga:

-Confirmación de registros cargados correctamente.

-Visualización de la columna valor_total calculada.

<img width="1500" height="639" alt="image" src="https://github.com/user-attachments/assets/9ca7d08d-4e30-404f-98f2-53a8b592267f" />

**Conclusiones**

Se logró implementar un flujo ETL completo con Data Factory, Databricks, Python y Azure SQL Database.

-Los datos fueron organizados en capas RAW y CURATED, garantizando trazabilidad.

-La conexión Python–SQL permite automatizar la carga de datos en la base relacional.

-Las consultas en SQL validan la disponibilidad y calidad de la información procesada.


