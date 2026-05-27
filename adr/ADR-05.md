ADR-05 — Uso de Power BI Desktop sobre Azure Analysis Services para la capa de visualización Estado: Aceptado

Contexto DataCo requiere una herramienta de visualización que permita a los analistas de BI y gerentes comerciales consumir dashboards actualizados automáticamente cada 4 horas con datos de ventas, inventario y logística. La herramienta debe conectarse a Azure SQL Database como fuente de datos. Power BI Desktop ya está licenciado e instalado en los equipos de los analistas de DataCo. El presupuesto del piloto no permite herramientas de visualización con costo adicional. El equipo de datos tiene experiencia básica en herramientas de BI pero no en modelado semántico avanzado.

Alternativas evaluadas

Opción A — Power BI Desktop

Herramienta de visualización y análisis de datos de Microsoft, gratuita en su versión Desktop. Ya está licenciada e instalada en los equipos de los analistas de DataCo, eliminando cualquier costo adicional y tiempo de onboarding. Conector nativo para Azure SQL Database mediante SQL Server, con configuración en menos de 5 minutos. Soporte para actualización programada de datos cada 4 horas, alineado con el ciclo del pipeline. Interfaz de arrastrar y soltar para creación de dashboards, accesible para analistas con conocimientos básicos de BI. Amplia comunidad, documentación en español y recursos de aprendizaje gratuitos disponibles. Permite publicar reportes en Power BI Service (versión cloud) si DataCo decide escalar la distribución de dashboards en el futuro.

Opción B — Azure Analysis Services

Servicio de modelado semántico empresarial en la nube que actúa como capa intermedia entre las fuentes de datos y las herramientas de visualización. Requiere una suscripción de pago (desde $85 USD/mes en el tier Developer), superando el presupuesto total del piloto de DataCo. Añade una capa adicional de complejidad al stack: el modelo semántico debe ser diseñado, mantenido y administrado por el equipo, lo que requiere experiencia en DAX y modelado tabular que el equipo no posee. Su propuesta de valor principal es la capacidad de servir modelos semánticos complejos a miles de usuarios concurrentes, una necesidad que DataCo no tiene en la fase piloto con 2 analistas y 1 gerente. No elimina la necesidad de Power BI Desktop como herramienta de visualización final; simplemente añade una capa intermedia de procesamiento.

Decisión

Se selecciona Power BI Desktop como herramienta de visualización del pipeline de DataCo. La decisión está completamente respaldada por las restricciones explícitas del caso: Power BI Desktop ya está licenciado en DataCo, su costo es cero y se conecta nativamente a Azure SQL sin configuración adicional. Azure Analysis Services resolvería necesidades de escala (miles de usuarios concurrentes, modelos semánticos complejos) que DataCo no tiene en esta fase, mientras generaría un costo mensual que por sí solo superaría el presupuesto total del piloto. La regla de arquitectura aplicada es la de mínima complejidad necesaria: no añadir capas al stack si la herramienta existente cumple los requerimientos.

Consecuencias

Ventajas obtenidas:

Costo cero al aprovechar la licencia existente de Power BI Desktop en DataCo. Conexión nativa a Azure SQL Database sin configuración adicional. Actualización automática de dashboards cada 4 horas alineada con el ciclo del pipeline. Curva de aprendizaje mínima para el equipo de analistas que ya conoce la herramienta. Posibilidad de escalar a Power BI Service en el futuro sin cambiar la herramienta de visualización.

Trade-offs asumidos:

Power BI Desktop es una aplicación local; los dashboards no son accesibles desde el navegador ni desde dispositivos móviles sin migrar a Power BI Service (requiere licencia Pro a futuro). Sin Azure Analysis Services, las consultas analíticas complejas impactan directamente en Azure SQL Database, lo que podría afectar el rendimiento en escenarios de alta concurrencia de usuarios a futuro.
