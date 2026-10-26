# Aplicacion y GenAI

## Aplicacion web

```text
Analista -> CloudFront + WAF -> React en S3
         -> Cognito -> JWT
React -> CloudFront/ALB -> Spring Boot en ECS Fargate
```

- React-Redux se publica en un bucket S3 privado mediante CloudFront y Origin Access Control.
- WAF aplica reglas administradas y rate limiting; las reglas nuevas se validan primero en modo Count.
- Cognito usa Authorization Code con PKCE para la SPA.
- Spring Boot es stateless, se despliega en subredes privadas y varias zonas de disponibilidad.
- CloudFront enruta `/api/*` a un ALB privado mediante VPC Origin cuando la region lo soporte. Como alternativa, el ALB es publico pero restringe el origen a CloudFront y mantiene las tasks privadas.
- La API valida firma, emisor, audiencia, expiracion, usuario, pais/tenant y parametros.

Cada endpoint elige una ruta explicita:

| Necesidad | Ruta |
|---|---|
| Clasificacion actual | Spring Boot -> DynamoDB |
| Historicos y dashboards | Spring Boot -> Redshift |
| Lenguaje natural | Spring Boot -> AgentCore Runtime |

## GenAI

AgentCore Runtime ejecuta un flujo controlado con:

1. Supervisor que clasifica y planifica la pregunta.
2. Especialista documental para la Managed Knowledge Base.
3. Especialista estructurado para herramientas del Gateway.
4. Sintesis que separa evidencia documental de resultados calculados.

AG-UI permite enviar progreso y texto incremental al chat. Se mide tiempo hasta el primer token y tiempo total, no el SLO de 500 ms.

## Recuperacion documental

Los documentos Gold se sincronizan con una Managed Knowledge Base. AWS administra almacenamiento, parsing, chunking y recuperacion.

- `Retrieve` obtiene fragmentos con metadatos y referencias.
- Los filtros incluyen empresa, pais, fecha, idioma y nivel de acceso.
- La respuesta conserva citas verificables.
- Solo se regeneran embeddings de documentos nuevos o modificados.
- Chunking, modelo de embeddings, borrado y frecuencia de sincronizacion se validan con un conjunto de preguntas real.

## Datos estructurados

```text
Runtime -> herramientas MCP en AgentCore Gateway
        -> API analitica controlada
        -> vistas Redshift read-only
```

El modelo no genera ni ejecuta SQL libre. La herramienta acepta operaciones y parametros tipados, y valida:

- Identidad, pais y tenant.
- Operacion, vista y columnas allowlist.
- Tipos, rangos y fechas.
- Limite de filas, timeout y coste.
- Acceso estrictamente de lectura.

## Seguridad y control de coste

- Preguntas, documentos y parametros del modelo se consideran no confiables.
- Guardrails filtra contenido, pero no sustituye la autorizacion.
- Se limitan tokens de salida, iteraciones, herramientas, timeout y peticiones por usuario.
- Los roles de Runtime, Knowledge Base y Gateway son independientes y de minimo privilegio.
- Logs y trazas sensibles se cifran y tienen retencion limitada.
- La respuesta se abstiene cuando no existe evidencia suficiente.

## Alcance funcional

El MVP soporta por separado preguntas estructuradas y documentales. La combinacion automatica de ambos resultados es una evolucion propuesta y debe evaluarse antes de considerarla obligatoria.

## Calidad propuesta

| Metrica | Objetivo inicial |
|---|---:|
| Precision de citas | `>= 95 %` |
| Groundedness | `>= 90 %` |
| Abstencion correcta | `>= 90 %` |
| Operaciones de escritura desde lenguaje natural | `0` |
| Accesos no autorizados conocidos | `0` |

Estos umbrales requieren un conjunto de evaluacion versionado y revision humana.
