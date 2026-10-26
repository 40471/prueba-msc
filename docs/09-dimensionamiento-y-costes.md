# Dimensionamiento y costes

## Datos de referencia

| Magnitud | Valor | Estado |
|---|---:|---|
| Empresas | 500.000-1.000.000 | Aclarado |
| Paises | 10 | Confirmado |
| KPIs por empresa | 100 | Confirmado |
| Historico | 10 anos | Confirmado |
| Datos nuevos | ~1 TB/ano | Confirmado, distribucion desconocida |
| Usuarios estimados | ~500 | Aclaracion ambigua |
| Pico observado | 26 | Aclaracion ambigua |
| Consultas pesadas concurrentes | 40 | Escenario provisional |

## Escenarios

| Driver | Inicial | Objetivo | Estres |
|---|---:|---:|---:|
| Empresas | 500.000 | 1.000.000 | 2.000.000 |
| Observaciones por KPI y ano | 1 | 4 | 12 |
| Datos estructurados nuevos | 1 TB/ano | 1 TB/ano | 2 TB/ano |
| Observaciones historicas de KPI | 500 M | 4.000 M | 24.000 M |
| Documentos nuevos | 1 M/ano | 5 M/ano | 20 M/ano |
| Fragmentos nuevos | 5 M/ano | 25 M/ano | 160 M/ano |
| Consultas normales | 2.500/dia | 10.000/dia | 50.000/dia |
| Consultas naturales | 500/dia | 2.500/dia | 10.000/dia |
| Consultas pesadas concurrentes | 26 | 40 | 80 |

Las observaciones historicas se derivan como `empresas x 100 KPIs x observaciones/KPI/ano x 10 anos`. La frecuencia de KPI, los documentos, fragmentos, consultas y todo el escenario de estres son supuestos para evaluar sensibilidad, no previsiones comerciales.

## Base de precios

| Decision | Valor utilizado |
|---|---|
| Region | `eu-west-1` (Irlanda) |
| Moneda | USD; conversion a EUR pendiente del tipo de cambio de la fecha de entrega |
| Fecha de precios | 25 de octubre de 2026 |
| Modalidad | Precios publicos on-demand, sin Free Tier, creditos ni descuentos privados |
| Mes | 730 horas |
| Entornos | Produccion 24x7; preproduccion y desarrollo con horario reducido |
| Disponibilidad | Multi-AZ en una region; backups y reconstruccion, sin DR activo cross-region |
| Bedrock | `eu.amazon.nova-micro-v1:0` para routing y `eu.anthropic.claude-sonnet-5-5` para sintesis |
| Embeddings | Gestionados por Managed Knowledge Base; Titan V2 como alternativa |
| Redshift | Reutilizacion del existente; Serverless se calcula como alternativa |

Los precios proceden de las paginas oficiales y catalogos publicos de AWS. Se excluyen impuestos, soporte, trabajo humano, licencias de origen y descuentos negociados.

## Supuestos tecnicos de coste

- S3 conserva 35 % de los datos en Standard, 25 % en Standard-IA y 40 % en Deep Archive.
- La huella fisica es 1,5 veces el volumen logico por Raw, Silver, Gold, versiones y snapshots.
- La huella fisica resultante es 16,575, 16,875 y 36 TB; se asume que el 25 % de los datos estructurados llega por SFTP.
- Glue consume 350, 1.300 y 4.500 DPU-horas mensuales.
- La API consume 1.826, 2.630 y 6.570 task-horas; los scrapers procesan 60 documentos por worker-hora y escalan a cero.
- DynamoDB usa capacidad on-demand, lecturas eventuales y un item actual por empresa; las lecturas y escrituras se detallan en los apuntes.
- Managed Knowledge Base almacena 50, 250 y 2.000 GB de documentos originales.
- Las consultas GenAI usan una llamada economica de routing y entre una y tres llamadas de sintesis segun escenario.
- CloudWatch ingiere 100, 500 y 2.500 GB de logs al mes.
- VPC endpoints sustituyen NAT para servicios AWS compatibles.
- Cada NAT incluye su IPv4 publica; el almacenamiento de logs usa una relacion conservadora 1:1 entre GB ingeridos y archivados.

Todos estos valores son supuestos de dimensionamiento y deben sustituirse por metricas reales tras una PoC o prueba de carga.

## Estimacion recurrente

Importes mensuales en USD:

| Componente | Inicial | Objetivo | Estres |
|---|---:|---:|---:|
| S3 lakehouse | $200,89 | $222,45 | $516,55 |
| Transfer Family | $219,85 | $219,85 | $220,71 |
| Glue | $154,00 | $572,00 | $1.980,00 |
| Step Functions y SQS | $0,61 | $3,08 | $15,00 |
| ECS Fargate API | $36,06 | $103,88 | $519,03 |
| ECS Fargate scrapers | $54,87 | $274,29 | $2.194,46 |
| ALB, CloudFront y WAF | $49,69 | $86,05 | $250,27 |
| Cognito | $2,75 | $2,75 | $5,50 |
| DynamoDB y PITR | $0,81 | $3,83 | $16,49 |
| Redshift: almacenamiento incremental | $49,15 | $122,88 | $491,52 |
| CloudWatch | $60,00 | $300,00 | $1.500,00 |
| NAT, IPv4 y VPC endpoints | $108,35 | $205,73 | $338,74 |
| KMS, Secrets, ECR y CloudTrail | $19,60 | $45,50 | $121,00 |
| Modelos Bedrock | $231,42 | $2.973,15 | $23.776,80 |
| Managed Knowledge Base | $265,00 | $1.362,50 | $10.600,00 |
| AgentCore Runtime y Gateway | $27,12 | $271,26 | $1.627,80 |
| **Total incremental mensual** | **$1.480,18** | **$6.769,20** | **$44.173,87** |
| **Total incremental anual** | **$17.762,12** | **$81.230,45** | **$530.086,48** |

El total incremental reutiliza el Redshift actual y no incluye una asignacion de su computo existente. Debe sumarse `factura mensual actual de Redshift x porcentaje atribuible` para obtener TCO completo.

## Alternativa Redshift Serverless

| Escenario | RPU-horas/mes | Coste Serverless incluido almacenamiento | Total plataforma/mes |
|---|---:|---:|---:|
| Inicial | 960 | $420,67 | $1.851,70 |
| Objetivo | 5.760 | $2.352,00 | $8.998,32 |
| Estres | 46.080 | $18.324,48 | $62.006,83 |

Serverless no se recomienda por defecto. Se adopta si la prueba de concurrencia demuestra contencion, colas o necesidad de aislamiento que el cluster actual no puede resolver.

El almacenamiento incremental usa como proxy la tarifa RMS de `$0,024/GB-mes`. Si el Redshift existente no usa almacenamiento administrado, este importe se sustituye por el coste real de resize, nodos o capacidad atribuida.

## Carga historica inicial

Estimacion de Glue, staging temporal y primera escritura de scores:

| Escenario | Coste unico estimado |
|---|---:|
| Inicial | $1.351,74 |
| Objetivo | $4.228,68 |
| Estres | $19.489,80 |

Es un limite inferior: no incluye horas de ingenieria, licencias, transferencia desde origen, remediacion de calidad ni ampliaciones temporales de los sistemas fuente.

## Metricas unitarias

| Metrica | Inicial | Objetivo | Estres |
|---|---:|---:|---:|
| Coste incremental por empresa y mes | $0,0030 | $0,0068 | $0,0221 |
| Coste GenAI por sesion | $0,0349 | $0,0614 | $0,1200 |

El coste GenAI incluye modelos, Knowledge Base, Runtime y Gateway. No incluye la parte compartida de API, red y observabilidad.

## Sensibilidades

1. Tokens, llamadas de sintesis y contexto recuperado dominan el crecimiento de GenAI.
2. Managed Knowledge Base crece con los GB originales indexados y las recuperaciones.
3. La duracion y selectividad de Glue determinan el coste de reproceso.
4. El throughput de scraping modifica Fargate y NAT.
5. La verbosidad de logs puede convertir CloudWatch en un coste material.
6. Redshift existente solo tiene coste marginal bajo mientras no requiera resize o concurrency scaling.
7. Retencion, versiones y compactacion modifican la huella fisica de S3.

## Controles

- AWS Budgets al 80 %, 100 % y 120 % de la prevision mensual.
- Etiquetas por entorno, dominio, pais y componente.
- Limites de tokens, iteraciones, timeout y consultas por usuario.
- Alarmas sobre DPU-horas, RPU-horas, logs, NAT y sesiones GenAI.
- Revision mensual del coste total y unitario.
- Sin Savings Plans ni compromisos hasta medir demanda estable.

## Limites

Esta estimacion es orientativa y no una cotizacion. Debe actualizarse con AWS Pricing Calculator y metricas reales antes de aprobar presupuesto. Tambien debe validarse la disponibilidad regional de Managed Knowledge Base y los modelos seleccionados.

Los totales se calculan con precision completa y se redondean despues; por ello, la suma visual de componentes redondeados puede diferir en un centimo. Los GET modelados acceden solo a S3 Standard; no se incluyen restauraciones de archivo, borrado anticipado ni recuperaciones Standard-IA. CloudWatch incluye logs y un mes de almacenamiento, pero no custom metrics, alarmas, dashboards, X-Ray o consultas Logs Insights. Estos consumos deben medirse y agregarse antes de aprobar presupuesto.
