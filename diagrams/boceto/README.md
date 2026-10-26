# Boceto inicial

Este directorio conserva la exploracion inicial de la arquitectura realizada en Excalidraw.

![Boceto conceptual de la arquitectura](arquitectura-general-boceto.png)

## Proposito

El boceto muestra el mapa mental utilizado para identificar inicialmente:

1. Fuentes de informacion estructuradas y documentales.
2. Procesos de ingesta y transformacion.
3. Almacenamiento historico y capas de consumo.
4. Analitica, clasificacion y aplicacion web.
5. Consultas en lenguaje natural.

## Evolucion del diseno

La arquitectura definitiva desarrolla y concreta el boceto mediante:

1. Separacion entre ingesta batch y scraping near real-time.
2. Capas Raw, Silver y Gold sobre S3 e Iceberg.
3. Calidad con Glue Data Quality y gobierno con Lake Formation.
4. Separacion entre analitica historica en Redshift y serving en DynamoDB.
5. Scoring precalculado y versionado fuera de la peticion del usuario.
6. RAG documental con Managed Knowledge Base.
7. Herramientas estructuradas y controladas mediante AgentCore Gateway.
8. Seguridad, observabilidad y CI/CD como capacidades transversales.

## Estado

Este PNG es un artefacto exploratorio y no normativo. La fuente oficial para revision e implementacion es [`../arquitectura-general.drawio`](../arquitectura-general.drawio), acompañada por su [documentacion semantica](../arquitectura.ir.json).
