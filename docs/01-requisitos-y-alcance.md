# Requisitos y alcance

## Objetivo

Definir una plataforma de datos completa en AWS para integrar informacion empresarial, conservar historicos, calcular KPIs y clasificaciones y ponerlos a disposicion de analistas mediante una aplicacion web y lenguaje natural.

## Requisitos confirmados

| ID | Requisito | Fuente |
|---|---|---|
| FR-01 | Ingerir datos de SFTP, Teradata, Redshift, paginas corporativas, noticias y redes sociales. | Reto, pp. 2-3 |
| FR-02 | Procesar semanalmente las fuentes batch. | Reto, p. 2 |
| FR-03 | Incorporar contenido web en near real-time. | Reto, p. 2 |
| FR-04 | Proporcionar una herramienta web para analistas financieros. | Reto, p. 3 |
| FR-05 | Servir clasificaciones empresariales. | Reto, pp. 2-3 |
| FR-06 | Permitir consultas en lenguaje natural. | Reto, pp. 2-3 |
| FR-07 | Gestionar unos 100 KPIs por empresa y 10 anos de historico. | Reto, p. 2 |
| NFR-01 | Ejecutar la plataforma en AWS. | Reto, p. 2 |
| NFR-02 | Cubrir 10 paises y aproximadamente 1 TB nuevo al ano. | Reto, p. 2 |
| NFR-03 | Optimizar escalabilidad, rendimiento y costes operativos. | Reto, p. 3 |
| NFR-04 | Alcanzar una latencia inferior a 500 ms; su alcance exacto permanece ambiguo. | Reto, p. 3; aclaraciones, lineas 15-16 |

## Decisiones de diseño

| ID | Decision provisional |
|---|---|
| ASM-01 | Aplicar un SLO `p95 < 500 ms` al perfil, KPIs actuales y clasificacion precalculada. |
| ASM-02 | Excluir de ese SLO la analitica pesada y la respuesta generativa completa. |
| ASM-03 | Precalcular clasificaciones tras cargas validas y recalcular selectivamente las empresas afectadas. |
| ASM-04 | Disenar por separado las consultas estructuradas y documentales; dejar las hibridas como evolucion. |
| ASM-05 | Usar 40 consultas pesadas concurrentes como escenario objetivo provisional. |
| ASM-06 | Comparar costes mediante escenarios inicial, objetivo y estres sin inventar un presupuesto. |


## Cuestiones abiertas

- Alcance contractual y medicion exacta de los 500 ms.
- Region, residencia, retencion y clasificacion regulatoria de los datos.
- RTO/RPO y ventanas de mantenimiento acordadas con negocio.
- Conectividad y capacidad incremental de SFTP, Teradata y Redshift.
- Formulas, granularidad y ownership de los KPIs.
- Tecnologia y frecuencia real del scoring.
- Usuarios activos, mezcla de consultas y curva horaria.
- Alcance final de las consultas en lenguaje natural.
- Presupuesto y plan de soporte.

## Trazabilidad resumida

| Necesidad | Solucion propuesta | Validacion |
|---|---|---|
| Ingesta multipais | Transfer Family, Glue/JDBC, `UNLOAD`, SQS/Fargate | Replay e idempotencia |
| Historico y KPIs | S3, Iceberg, Glue/PySpark | Calidad, reconciliacion y time travel |
| Clasificacion rapida | Scoring previo y DynamoDB | Carga p50/p95/p99 |
| Analitica pesada | Marts y vistas Redshift | Concurrencia y aislamiento |
| Lenguaje natural | AgentCore, Managed Knowledge Base y Gateway | Exactitud, citas y autorizacion |
| Control operacional | CloudWatch, OpenTelemetry y CloudTrail | Alarmas y simulacion de fallos |
