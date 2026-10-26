# Analitica y clasificacion

## Objetivo

Separar las consultas historicas complejas del serving rapido de clasificaciones para que ambas cargas escalen y fallen de forma independiente.

## Analitica en Redshift

Gold publica incrementos en marts y vistas de Redshift para:

- Series historicas de KPIs.
- Comparaciones entre empresas.
- Agregaciones por pais, sector y periodo.
- Dashboards y herramientas estructuradas de GenAI.

La opcion inicial es reutilizar el Redshift existente con schemas, roles y workload management aislados. Si una prueba muestra colas, spill o interferencia, los marts pasan a un workgroup Redshift Serverless independiente.

## Calculo de clasificaciones

Glue/PySpark calcula el score fuera de la peticion del usuario:

1. Lee features versionadas de Gold.
2. Ejecuta reglas o modelo segun la definicion vigente.
3. Registra `score_version`, `model_or_rule_version`, `computed_at`, `run_id` y explicaciones.
4. Conserva el historico en Gold y Redshift.
5. Publica la ultima version en DynamoDB mediante escritura condicional.

El batch semanal cubre cambios financieros. Correcciones o eventos relevantes pueden activar mini-batches solo para empresas afectadas; esta frecuencia debe validarse con negocio.

Si la clasificacion se basa en reglas, sus formulas, umbrales y vigencia se versionan junto con los resultados. Si evoluciona a ML, se anaden dataset de entrenamiento, evaluacion, registro y aprobacion del modelo, monitorizacion de drift y rollback. En ambos casos, las features y el codigo de calculo deben permitir reproducir cada score.

## Serving en DynamoDB

Modelo propuesto:

```text
PK = tenant_country#company_id
SK = CURRENT
```

- `GetItem` para una empresa.
- `BatchGetItem` para un conjunto conocido.
- La clave compuesta distribuye empresas y conserva la frontera de autorizacion.
- Los listados amplios se resuelven en Redshift; un GSI shardeado solo se incorpora si existe un listado operacional concreto.
- Nunca `Scan`, joins o calculo de score en la ruta critica.
- Escritura condicionada por version o fecha para impedir sobrescrituras antiguas.

El tamano del item, la distribucion de trafico, `BatchGetItem` y posibles particiones calientes se verifican mediante carga antes de elegir capacidad provisionada.

El historico no se duplica en DynamoDB; permanece en el lakehouse y Redshift.

## Objetivos de servicio propuestos

mscope no ha confirmado un SLA contractual. Los siguientes valores son SLO de diseno sujetos a validacion:

| Capacidad | SLO propuesto |
|---|---|
| Perfil, KPIs actuales y clasificacion | `p95 < 500 ms`, backend autenticado |
| Historico estandar | `p95 < 5 s` dentro de limites soportados |
| Analitica pesada | `p95 < 30 s`, escenario de 40 consultas concurrentes |

Los 500 ms no aplican a Redshift ni a una respuesta LLM completa. La arquitectura esta disenada para aspirar al objetivo; solo una prueba puede demostrarlo.

## Degradacion

- Si Redshift falla, DynamoDB mantiene la ultima clasificacion publicada.
- Si un scoring falla, no se publica una version parcial.
- La API muestra `computed_at` y version para hacer visible la frescura.
- DynamoDB puede reconstruirse desde el historico Gold.

## Metricas

- Latencia p50/p95/p99 y errores de `GetItem`/`BatchGetItem`.
- Throttling, consumo y distribucion de claves DynamoDB.
- Tiempo en cola, ejecucion, spill y concurrencia Redshift.
- Duracion del scoring, empresas procesadas y escrituras rechazadas por version.
