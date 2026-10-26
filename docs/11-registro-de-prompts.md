# Registro de prompts y uso de IA

## Uso realizado

Se utilizo OpenCode con el modelo `openai/gpt-5.6-sol` para estructurar requisitos, contrastar alternativas AWS, redactar documentacion y revisar inconsistencias. Las decisiones finales requieren validacion humana.

Los prompts siguientes son una **version consolidada** de las iteraciones principales: eliminan instrucciones operativas repetidas, pero conservan su objetivo y restricciones. No se incluyen prompts de sistema, mensajes rutinarios ni credenciales.

| Campo | Valor |
|---|---|
| Periodo de trabajo | 23-25 de octubre de 2026 |
| Herramienta | OpenCode |
| Modelo | `openai/gpt-5.6-sol` |
| Validacion | Revision y aprobacion final del candidato |
| Criterio | Conservar objetivo, contexto, restricciones, resultado utilizado y decisiones humanas |

## 1. Requisitos

> Analiza el enunciado, las respuestas del CTO y la descripcion del puesto. Extrae requisitos funcionales, no funcionales, restricciones, cifras, contradicciones y cuestiones abiertas. Separa informacion confirmada, aclaraciones y supuestos. No disenes todavia la arquitectura y referencia el origen de cada requisito.

**Resultado utilizado:** linea base de requisitos y deteccion de ambiguedades sobre escala, latencia y concurrencia.

## 2. Casos de uso y acceso

> Convierte los requisitos en actores, casos de uso y patrones de acceso. Distingue lecturas puntuales, historicas, analiticas y generativas; batch y near real-time; datos estructurados y documentos. No selecciones servicios todavia.

**Resultado utilizado:** separacion de workloads y rutas de consumo.

## 3. Niveles de servicio

> Propone SLO medibles para serving, historicos, analitica, batch, scraping y lenguaje natural. Distingue SLO internos de SLA contractual, define la medicion y propone RTO/RPO. No presentes valores propuestos como requisitos confirmados.

**Resultado utilizado:** SLO separados y correccion del tratamiento de los 500 ms.

## 4. Dimensionamiento

> Estima volumen, crecimiento y carga para escenarios inicial, objetivo y estres. Usa 500.000-1.000.000 de empresas, 10 paises, 100 KPIs, 10 anos y 1 TB anual. Distingue datos confirmados, derivados y supuestos. No conviertas consultas diarias en QPS sin una curva horaria.

**Resultado utilizado:** escenarios de capacidad y sensibilidades.

## 5. Arquitectura end-to-end

> Disena una arquitectura AWS end-to-end para SFTP, Teradata, Redshift y scraping. Cubre lakehouse, calidad, gobierno, analitica, clasificacion, aplicacion web y lenguaje natural. Justifica cada servicio, alternativas y trade-offs. No introduzcas streaming si no existe una necesidad concreta.

**Resultado utilizado:** arquitectura general y separacion entre S3/Iceberg, Redshift, DynamoDB y AgentCore.

## 6. Ingesta

> Disena la ingesta batch y near real-time. Compara Transfer Family, Glue/JDBC, `UNLOAD`, Step Functions, SQS, Fargate, Lambda y MSK. Incluye watermarks, idempotencia, procedencia, retries, DLQ, backpressure y controles legales de scraping.

**Resultado utilizado:** flujos independientes por fuente y descarte razonado de Kafka/MSK.

## 7. Capa de datos

> Disena Raw, Silver y Gold sobre S3 e Iceberg. Incluye normalizacion, deduplicacion, identidad empresarial, KPIs versionados, `MERGE`, snapshots, compactacion, calidad, cuarentena, Glue Catalog y Lake Formation.

**Resultado utilizado:** modelo de procesamiento y gobierno.

## 8. Analitica y clasificacion

> Separa analitica historica en Redshift y serving de clasificaciones precalculadas en DynamoDB. Define scoring incremental, historico, versionado, claves de acceso, aislamiento de workloads y pruebas de latencia. Trata los 500 ms como objetivo propuesto.

**Resultado utilizado:** rutas OLAP y operacional independientes.

## 9. Aplicacion y GenAI

> Disena la aplicacion con React, CloudFront, WAF, Cognito y Spring Boot en Fargate. Para lenguaje natural usa AgentCore Runtime, Managed Knowledge Base y Gateway. Incluye citas, filtros, autorizacion, streaming, proteccion frente a prompt injection, SQL controlado y limites de coste.

**Resultado utilizado:** aplicacion web, RAG y herramientas estructuradas seguras.

## 10. Revision critica

> Revisa la propuesta como Senior Data Engineer y arquitecto AWS. Busca servicios innecesarios, afirmaciones no respaldadas, puntos unicos de fallo, costes ocultos, problemas de seguridad y pruebas ausentes. Produce una lista priorizada de correcciones y trade-offs.

**Resultado utilizado:** correccion de la atribucion del SLO, concurrencia y alcance hibrido; consolidacion de riesgos y validaciones.

## Trazabilidad

| Prompts | Documentos principales | Decision humana |
|---|---|---|
| 1-4 | Requisitos, SLO y dimensionamiento | Mantener ambiguedades como supuestos |
| 5-8 | Arquitectura, ingesta, datos y clasificacion | Separar OLTP, OLAP e historico |
| 9 | Aplicacion y GenAI | No permitir SQL libre ni confiar autorizacion al modelo |
| 10 | Todos | Corregir afirmaciones no respaldadas y registrar riesgos |

## Validacion humana

Antes de implantar se deben verificar servicios y cuotas regionales, precios vigentes, conectividad, permisos, restricciones legales de scraping, formulas de KPIs, scoring, SLO y calidad de las respuestas GenAI. La IA no sustituye estas decisiones ni las pruebas de carga y seguridad.
