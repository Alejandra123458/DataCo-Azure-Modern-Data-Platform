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

## Sin trazabilidad de entregas: 

No existe forma de correlacionar automáticamente una factura de SAP con su entrega real en el GPS, impidiendo medir el cumplimiento de promesas de entrega por ruta o vendedor.

## Escalabilidad nula: 

El servidor de consolidaciones es un equipo de escritorio con Windows Server 2012. En los meses de cierre, el proceso de generación de reportes tarda hasta 8 horas continuas.


## Restricciones del proyecto

- Equipo de datos compuesto por 2 analistas con conocimientos de SQL y Python básico, sin experiencia en Spark ni administración de clusters.

- SAP On-Premise no tiene API REST disponible; la integración se realiza mediante archivos CSV/JSON vía SFTP.

- Presupuesto mensual en Azure máximo de $80 USD durante la fase piloto.

- Los datos de ventas contienen información sensible que requiere control de acceso por roles.

- Power BI Desktop ya está licenciado en la empresa; no se puede proponer una herramienta de visualización con costo adicional.

- El pipeline debe ser tolerante a fallos parciales por fuente.

