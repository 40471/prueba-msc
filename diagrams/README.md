# Diagramas

## Arquitectura oficial

La arquitectura definitiva se entrega en varios formatos:

- [`arquitectura-general.drawio`](arquitectura-general.drawio): fuente editable y normativa.
- [`arquitectura-general.drawio.png`](arquitectura-general.drawio.png): imagen utilizada por la documentacion.
- [`arquitectura-general.drawio.svg`](arquitectura-general.drawio.svg): exportacion vectorial.
- [`arquitectura-general.drawio.html`](arquitectura-general.drawio.html): recorrido interactivo.
- [`arquitectura.ir.json`](arquitectura.ir.json): representacion semantica reproducible.

![Arquitectura oficial](arquitectura-general.drawio.png)

## Contenido

1. Fuentes: SFTP, Teradata, Redshift origen y web/social.
2. Ingesta: Transfer Family, Glue/JDBC, `UNLOAD`, Step Functions, SQS y Fargate.
3. Lakehouse: S3 Raw, Glue/PySpark + Data Quality, Silver/Gold Iceberg y gobierno.
4. Analitica: Redshift, scoring y DynamoDB.
5. Aplicacion: CloudFront, WAF, Cognito, Spring Boot y las tres rutas de consulta.
6. GenAI: AgentCore Runtime, Managed Knowledge Base, Gateway y Redshift read-only.
7. Transversal: seguridad, observabilidad y CI/CD.

## Reglas visuales

- Separar `Teradata -> Glue/JDBC -> S3 Raw` de `Redshift origen -> UNLOAD -> S3 Raw`.
- Mostrar el scraping como `worker -> fuente web` para la llamada HTTPS y `worker -> S3 Raw` para el resultado.
- Mostrar `scoring -> Gold` para historico y `scoring -> DynamoDB` para el ultimo score.
- Diferenciar Redshift como origen de Redshift como destino analitico, aunque inicialmente puedan ser el mismo cluster con schemas aislados.
- Etiquetar los 500 ms como `SLO propuesto`, no como SLA confirmado.

## Boceto inicial

El directorio [`boceto/`](boceto/) conserva el mapa conceptual inicial realizado en Excalidraw. Sirve para mostrar la evolucion del razonamiento, pero no constituye una fuente de implementacion ni sustituye al diagrama oficial.

![Boceto conceptual inicial](boceto/arquitectura-general-boceto.png)
