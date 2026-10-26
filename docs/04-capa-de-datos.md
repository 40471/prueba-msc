# Capa de datos

## Objetivo

Transformar entradas originales en datos fiables, reproducibles y gobernados para analitica, scoring y GenAI.

## Capas

| Capa | Contenido | Regla principal |
|---|---|---|
| Raw/Bronze | Copia original y metadatos de ingesta | No modificar; las correcciones crean otra version |
| Silver | Datos normalizados, deduplicados y relacionados | Sin reglas finales de presentacion |
| Gold | KPIs, features, documentos curados y datasets de consumo | Publicar solo tras superar calidad |

S3 Versioning, permisos de escritura restringidos y politicas de retencion respaldan la reproducibilidad de Raw. Object Lock se evaluara si existe un requisito regulatorio de inmutabilidad.

## Procesamiento

Glue/PySpark procesa solo cargas, empresas o periodos afectados:

1. Valida el contrato de entrada.
2. Normaliza fechas, monedas, paises y conceptos.
3. Deduplica mediante claves de negocio y hashes.
4. Resuelve una identidad empresarial global.
5. Calcula KPIs y features versionadas.
6. Escribe Silver y Gold mediante `MERGE` de Iceberg.
7. Registra el resultado y publica solo si las reglas criticas pasan.

## Calidad

Glue Data Quality valida obligatoriedad, unicidad, rangos, formatos, integridad referencial y coherencia temporal.

- Fallo critico: bloquea la promocion.
- Fallo de registro: envia a cuarentena con causa y `run_id`.
- Advertencia: publica con metrica y seguimiento.
- Reconciliacion: compara origen, Raw, Silver y Gold.

Los umbrales concretos deben acordarse con los propietarios de datos.

## Modelo minimo

| Entidad | Contenido |
|---|---|
| `company` | Identidad global, pais e identificadores de origen |
| `financial_fact` | Empresa, periodo, concepto, valor, moneda, fuente y version |
| `kpi_definition` | Formula, unidad, vigencia y version del KPI |
| `company_kpi` | Empresa, periodo, KPI, valor y version de formula |
| `company_score_history` | Score, version, fecha, features y explicaciones |
| `document` | Contenido, empresa, fuente, URL, fecha, idioma, licencia y hash |

El modelo de KPI es largo, por empresa, periodo y definicion. Un cambio de formula recalcula solo el alcance afectado y conserva la version anterior.

## Apache Iceberg

Iceberg aporta `MERGE`, snapshots, evolucion de esquema y time travel. La operacion debe incluir compactacion de archivos pequenos, expiracion controlada de snapshots y limpieza de archivos huerfanos.

## Catalogo y gobierno

- Glue Data Catalog registra tablas, columnas, formatos y ubicaciones.
- Lake Formation aplica permisos por rol, dominio, sensibilidad y pais/tenant.
- Catalogo y Lake Formation no mueven datos: describen y gobiernan los objetos en S3.
- Cada dataset debe tener owner, clasificacion, contrato, calidad y politica de retencion.

## Identidad empresarial

La resolucion combina identificadores oficiales, fuente, pais y reglas de matching. Los casos ambiguos requieren revision o una confianza explicita. Las reglas se versionan para reproducir resultados historicos.
