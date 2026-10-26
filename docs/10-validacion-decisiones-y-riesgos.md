# Validacion, decisiones y riesgos

## Plan de validacion

Estas pruebas estan propuestas; no se han ejecutado.

| Area | Prueba | Criterio principal |
|---|---|---|
| Ingesta | Carga historica y replay semanal | Sin duplicados ni watermarks incorrectos |
| Fallos | Error parcial, retry y DLQ | Una fuente no bloquea las demas |
| Calidad | Registros invalidos | Cuarentena y bloqueo de publicacion |
| Iceberg | `MERGE`, snapshot y rollback | Resultado reproducible |
| Clasificacion | Mezcla `GetItem`/`BatchGetItem` | Medir p50/p95/p99 con autenticacion |
| Redshift | Consultas representativas concurrentes | Cola, ejecucion, spill y aislamiento |
| Autorizacion | Acceso a otro pais/tenant | Cero accesos no autorizados |
| Herramientas IA | Parametros y operaciones maliciosas | Cero bypass de allowlist o escrituras |
| RAG | Preguntas con evidencia conocida | Citas, groundedness y abstencion |
| Recuperacion | Restaurar o reconstruir componentes | Cumplir RTO/RPO aprobados |

## Decisiones y alternativas

| Decision | Motivo | Cuando reconsiderar |
|---|---|---|
| Glue/PySpark | Stack alineado y sin cluster fijo | Jobs largos o tuning especial: EMR Serverless |
| Step Functions Standard | Auditoria, retries y espera de Glue | Ecosistema Airflow existente: MWAA |
| SQS + Fargate para scraping | Backpressure y runtime flexible | Tareas cortas: Lambda; stream real: Kinesis/MSK |
| S3/Iceberg | Historico economico, `MERGE` y snapshots | Dataset pequeno y solo warehouse |
| Reutilizar Redshift | Menor coste y migracion | Contencion demostrada: Redshift Serverless |
| DynamoDB para ultimo score | Lectura directa por clave | Transacciones relacionales o consultas ad-hoc |
| AgentCore Runtime | Grafo y streaming controlados | MVP simple: AgentCore Harness |
| Managed Knowledge Base | Menor operacion RAG | Control obligatorio de motor o ranking |
| Sin Kafka/MSK | No hay requisito que lo justifique | Alta tasa, orden, replay y varios consumidores |

## Riesgos principales

| Riesgo | Mitigacion |
|---|---|
| Alcance ambiguo de 500 ms | SLO separado y validacion con negocio |
| Concurrencia no caracterizada | Mezcla de consultas y benchmark |
| Identidad empresarial multipais | Fuentes de autoridad, matching versionado y revision |
| Conectividad de origen desconocida | Discovery de red, permisos y capacidades incrementales |
| Mala calidad o duplicados | Contratos, DQ, cuarentena e idempotencia |
| Fuga entre paises | Autorizacion en API, datos, RAG y herramientas |
| SQL o parametros peligrosos | Operaciones allowlist, tipos, limites y rol read-only |
| Alucinaciones | Evidencia, citas, abstencion y evaluacion |
| Coste GenAI impredecible | Limites, cuotas, metricas unitarias y budgets |
| Contencion Redshift | Workload isolation y alternativa Serverless |
| Cambios o bloqueo de webs | APIs oficiales, rate limit, backoff y DLQ |
| Publicacion de fuentes privadas | Repositorio privado y permiso expreso |

## Validaciones previas al despliegue

1. Confirmar region, cuotas y disponibilidad de servicios AgentCore y modelos.
2. Acordar SLO, RTO/RPO, retencion y residencia.
3. Validar topologia y permisos de SFTP, Teradata y Redshift.
4. Medir usuarios activos, curva horaria y consultas representativas.
5. Definir formulas de KPIs, scoring y reglas de identidad.
6. Construir el conjunto de evaluacion de RAG.
7. Calcular costes con precios vigentes y escenarios aprobados.
