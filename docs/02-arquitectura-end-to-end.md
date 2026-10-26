# Arquitectura end-to-end

## Principios

1. Separar serving operacional, analitica y GenAI.
2. Mantener S3 como historico reproducible.
3. Procesar incrementos en lugar de releer diez anos.
4. Precalcular las clasificaciones fuera de la peticion.
5. Tratar las salidas de los modelos como datos no confiables.
6. Introducir servicios complejos solo cuando exista una necesidad demostrada.

## Flujo general

```text
SFTP --------> Transfer Family -----------------------> S3 Raw
Teradata ----> Glue/JDBC incremental -----------------> S3 Raw
Redshift ----> UNLOAD incremental --------------------> S3 Raw
Web/social --> SQS/DLQ --> Fargate scrapers ----------> S3 Raw
                                                         |
                                                         v
                                              Glue/PySpark + DQ
                                                         |
                                                         v
                                             S3/Iceberg Silver y Gold
                                                |        |        |
                                                v        v        v
                                           Redshift   Scoring   Managed KB
                                                        |
                                                        v
                                                     DynamoDB

Analista -> CloudFront/WAF/Cognito -> Spring Boot -> DynamoDB | Redshift | AgentCore
```

![Arquitectura end-to-end](../diagrams/arquitectura-general.drawio.png)

El diagrama se entrega en formato [draw.io editable](../diagrams/arquitectura-general.drawio), [SVG](../diagrams/arquitectura-general.drawio.svg) y [HTML interactivo](../diagrams/arquitectura-general.drawio.html). Seguridad, observabilidad y CI/CD se representan como capacidades transversales, no como nuevas rutas de datos.

## Componentes

| Capa | Componentes | Responsabilidad |
|---|---|---|
| Fuentes | SFTP, Teradata, Redshift, web/social | Proporcionar datos estructurados y documentales |
| Ingesta | Transfer Family, Glue, EventBridge, Step Functions, SQS, Fargate | Recibir, extraer y desacoplar cargas |
| Lakehouse | S3, Glue/PySpark, Glue Data Quality, Iceberg | Conservar, transformar y validar |
| Gobierno | Glue Data Catalog, Lake Formation | Catalogar y autorizar |
| Analitica | Redshift | Historicos, comparaciones y agregaciones |
| Clasificacion | Glue/PySpark, DynamoDB | Calcular y servir el ultimo score |
| Aplicacion | CloudFront, WAF, Cognito, ALB, ECS Fargate | Interfaz y API para analistas |
| GenAI | AgentCore Runtime, Managed Knowledge Base, Gateway | Consultas documentales y estructuradas controladas |
| Transversal | IAM, KMS, Secrets Manager, CloudTrail, CloudWatch, OpenTelemetry | Seguridad, auditoria y operacion |

## Separacion de cargas

| Ruta | Motor | Motivo |
|---|---|---|
| Perfil y score actual | DynamoDB | Lectura directa por clave y latencia predecible |
| Historico y analitica | Redshift | Joins, filtros y agregaciones columnares |
| Documentos | Managed Knowledge Base | Recuperacion semantica con metadatos y citas |
| Transformacion | Glue/PySpark | Procesamiento distribuido e incremental |

## Evolucion

- Reutilizar inicialmente el Redshift existente si permite aislar schemas, roles y workloads.
- Migrar los marts a Redshift Serverless solo si una prueba demuestra contencion.
- Mantener SQS y mini-batches mientras no exista una necesidad real de Kinesis o MSK.
- Usar AgentCore Runtime cuando el grafo multiagente sea necesario; un MVP simple podria usar AgentCore Harness.
