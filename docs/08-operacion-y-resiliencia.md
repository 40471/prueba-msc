# Operacion y resiliencia

## Observabilidad

CloudWatch centraliza metricas y logs; OpenTelemetry propaga `trace_id` entre frontend, API, agentes y herramientas. Los pipelines utilizan `run_id` para correlacionar sus etapas.

| Area | Senales principales |
|---|---|
| Ingesta | Duracion, watermark, filas, rechazos y errores |
| Scraping | Edad/profundidad SQS, DLQ y errores por dominio |
| Calidad | Reglas fallidas, cuarentena y promociones bloqueadas |
| Iceberg | Archivos pequenos, compactacion y snapshots |
| Redshift | Cola, ejecucion, spill y concurrencia |
| DynamoDB | Latencia, errores y throttling |
| Aplicacion | ALB 4xx/5xx, latencia y salud ECS |
| GenAI | TTFT, tiempo total, tokens, iteraciones, herramientas y errores |

Las alarmas usan tasas y percentiles, no solo promedios. Sus umbrales se ajustan con la linea base real.

## Recuperacion

| Capacidad | Mecanismo |
|---|---|
| Datos recibidos | S3 Versioning y reproceso desde Raw |
| Silver/Gold | Reconstruccion determinista desde Raw y snapshots Iceberg |
| Clasificaciones | DynamoDB PITR y reconstruccion desde Gold |
| Analitica | Snapshots/restore de Redshift y recarga desde Gold |
| Aplicacion | ECS multi-AZ y artefactos versionados |
| GenAI | Desactivar chat sin afectar clasificaciones ni dashboards |

RTO y RPO definitivos requieren validacion de negocio. Como punto de partida se propone: datos recibidos RPO 0 tras confirmacion; serving RTO 1 hora; analitica y documentos RTO 4 horas.

## Degradacion controlada

- El fallo de una fuente no bloquea las demas.
- SQS y DLQ aislan errores de scraping.
- Una carga no valida no reemplaza la publicacion anterior.
- DynamoDB conserva clasificaciones si Redshift o Bedrock fallan.
- La aplicacion puede desactivar temporalmente el chat.
- Los resultados muestran fecha y version para informar sobre frescura.

## CI/CD

El pipeline produce artefactos independientes:

- Frontend React para S3/CloudFront.
- Imagen Spring Boot y scrapers para ECS Fargate.
- Imagen ARM64 para AgentCore Runtime.
- Jobs Glue, reglas DQ y workflows.
- Infraestructura y configuracion versionadas.

Antes de desplegar se ejecutan tests, escaneo de dependencias e imagenes, validacion de esquemas y controles de seguridad. Produccion utiliza aprobacion, despliegue progresivo y rollback basado en health checks y alarmas.

## Objetivos operativos propuestos

| Capacidad | Objetivo inicial |
|---|---|
| Batch semanal | 99 % completo en menos de 4 horas |
| Contenido web capturado | 99 % disponible en menos de 15 minutos desde captura valida |
| GenAI estructurado | TTFT `p95 < 3 s` |
| RAG documental | TTFT `p95 < 4 s` |

Son SLO internos para validacion, no SLA contractuales.
