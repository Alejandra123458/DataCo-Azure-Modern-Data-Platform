ADR-04 — Uso de Azure SQL Database sobre Azure Cosmos DB para el almacén analítico final Estado: Aceptado

Contexto

DataCo necesita un almacén de datos relacional que consolide los resultados del pipeline de transformación en un modelo estructurado, accesible para consultas analíticas desde Power BI Desktop y para consultas SQL directas por parte del equipo de analistas. Los datos de ventas contienen información sensible de precios y márgenes, por lo que el acceso debe estar restringido por roles. El equipo tiene conocimientos de SQL pero ninguna experiencia con bases de datos NoSQL. El presupuesto del piloto no debe superar $80 USD/mes.

Alternativas evaluadas

Opción A — Azure SQL Database

Motor de base de datos relacional completamente administrado basado en SQL Server, optimizado para consultas estructuradas y cargas de trabajo analíticas. Soporte nativo para el modelo dimensional (tablas de hechos y dimensiones) que requiere el caso de DataCo. Conector nativo en Power BI Desktop mediante SQL Server, sin configuración adicional ni licencias extra. Control de acceso por roles (RBAC) mediante usuarios de base de datos con permisos granulares por tabla y vista. Free tier disponible con 32 GB de almacenamiento y 100.000 vCores/mes, suficiente para la fase piloto. Soporte para vistas e índices columnares que optimizan el rendimiento de las consultas analíticas de Power BI. Familiar para el equipo de analistas de DataCo que tiene conocimientos de SQL. Integración directa con Azure Databricks mediante JDBC para carga de datos desde notebooks.

Opción B — Azure Cosmos DB

Base de datos NoSQL distribuida globalmente, optimizada para cargas de trabajo de baja latencia con modelos de datos flexibles (documentos JSON, grafos, clave-valor). No soporta el modelo relacional dimensional requerido por DataCo (tablas de hechos y dimensiones con JOINs complejos). La integración con Power BI Desktop requiere configuración adicional mediante el conector de Cosmos DB o Azure Synapse Link, añadiendo complejidad y posibles costos adicionales. El modelo de consulta basado en particiones y claves de partición no es intuitivo para analistas con experiencia en SQL relacional. Los costos de Cosmos DB (basados en Request Units) son significativamente más altos para cargas de trabajo analíticas con consultas complejas, superando el presupuesto del piloto. Diseñado para aplicaciones transaccionales de alta disponibilidad global, no para almacenes analíticos con consultas ad-hoc complejas.

Decisión

Se selecciona Azure SQL Database como almacén analítico final del pipeline de DataCo. El modelo relacional es la elección natural para este caso por tres razones fundamentales: el equipo tiene experiencia en SQL y puede operar el almacén sin curva de aprendizaje adicional, Power BI Desktop se conecta nativamente sin configuración adicional, y el Free Tier cubre los requerimientos del piloto sin costo. Azure Cosmos DB resolvería problemas que DataCo no tiene (baja latencia global, esquema flexible) mientras generaría nuevos problemas que sí tiene (costos elevados, incompatibilidad con el conocimiento del equipo, complejidad de integración con Power BI).

Consecuencias

Ventajas obtenidas:

Modelo dimensional familiar para el equipo de analistas con conocimientos de SQL. Conexión nativa con Power BI Desktop sin configuración adicional. Control de acceso por roles granular para proteger datos sensibles de precios y márgenes. Costo cero en la fase piloto con el Free Tier de Azure SQL. Soporte para vistas e índices columnares que optimizan consultas analíticas complejas.

Trade-offs asumidos:

Azure SQL Database es un motor relacional optimizado para consultas estructuradas; para cargas de trabajo de análisis masivo a futuro (más de 32 GB), será necesario evaluar Azure Synapse Analytics o migrar a un tier de pago. El Free Tier tiene límites de cómputo (100.000 vCores/mes) que podrían afectar el rendimiento en períodos de alta demanda como cierres de mes.
