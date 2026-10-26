# Ingesta y orquestacion

## Objetivo

Integrar fuentes con protocolos y frecuencias diferentes sin acoplarlas al procesamiento posterior. Toda entrada aceptada se conserva primero en S3 Raw.

## Flujos batch

| Fuente | Flujo | Control incremental |
|---|---|---|
| SFTP | Proveedor -> Transfer Family -> S3 Raw | Checksum, nombre logico, periodo y manifiesto |
| Teradata | Step Functions -> Glue/JDBC -> S3 Raw | Watermark, particionado y pushdown |
| Redshift origen | Step Functions -> `UNLOAD` -> S3 Raw | Ventana temporal o clave incremental |

El primer procesamiento realiza la carga historica; las siguientes ejecuciones leen solo cambios. El watermark avanza despues de una escritura y validacion correctas.

## Scraping near real-time

```text
EventBridge/Step Functions -> SQS -> Fargate workers -> HTTPS fuentes -> S3 Raw
                                   \-> DLQ
```

- SQS absorbe picos y evita saturar workers o fuentes externas.
- Fargate permite navegadores, librerias especificas y procesos mas largos que una funcion corta.
- El autoescalado utiliza profundidad y edad de cola.
- Se aplican limites por dominio, backoff, `robots.txt`, APIs oficiales cuando existan y condiciones de uso.
- El mensaje se elimina solo despues de confirmar la escritura en S3.
- Los fallos permanentes pasan a DLQ sin detener otras fuentes.

## Orquestacion

EventBridge Scheduler inicia Step Functions Standard. El workflow coordina dependencias, jobs Glue, controles de calidad, reintentos y publicacion. Standard aporta historial auditable y espera sincronica de jobs.

EventBridge y Step Functions transportan control; los datos permanecen en S3 o en los sistemas de origen.

## Idempotencia y procedencia

Cada entrega o captura registra:

- `run_id`, fuente, pais y fecha de ingesta.
- Bucket, key, `versionId`, checksum y manifiesto para objetos S3.
- URL canonica o identificador de origen.
- Checksum o hash de contenido.
- Esquema y version del parser.
- Estado HTTP, idioma, licencia y fecha de captura cuando aplique.
- Watermark inicial y final.

Repetir una ejecucion con el mismo alcance no debe duplicar datos ni avanzar el watermark dos veces.

## Seguridad y conectividad

- Credenciales JDBC y APIs en Secrets Manager.
- TLS y cifrado KMS.
- Glue y Fargate en subredes privadas.
- VPN o Direct Connect para origenes privados, sujeto a la topologia real.
- NAT y salida controlada solo para los scrapers que acceden a Internet.
- Rol de `UNLOAD` y bucket policy limitados al prefijo de Redshift.

## Motivo para no usar streaming

No existe un requisito de eventos continuos de alta tasa, orden por clave o multiples consumidores con replay prolongado. SQS y procesos incrementales reducen coste y operacion. Kinesis o MSK se reconsiderarian si esos requisitos aparecen.

## Metricas

- Duracion, filas leidas/escritas/rechazadas y watermark por `run_id`.
- Edad y profundidad de SQS, mensajes visibles y DLQ.
- Errores y codigos HTTP por dominio.
- Retraso desde recepcion o captura hasta disponibilidad.

Para near real-time se separan dos indicadores: publicacion externa hasta deteccion, condicionado por la cadencia de rastreo, y captura valida hasta disponibilidad interna. La cadencia y el presupuesto por tipo de fuente deben acordarse antes de fijar el primer objetivo.
