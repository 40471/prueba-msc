# Seguridad y red

## Principios

1. Minimo privilegio y roles separados por workload.
2. Cifrado en transito y en reposo.
3. Datos y aplicaciones en redes privadas cuando sea posible.
4. Autorizacion coherente en todas las capas.
5. Secretos fuera del codigo, imagenes y configuracion visible.
6. Auditoria suficiente sin registrar contenido sensible innecesario.

## Controles

| Dominio | Control |
|---|---|
| Identidad humana | Cognito, MFA segun riesgo y federacion corporativa si existe |
| Identidad de servicios | Roles IAM de minimo privilegio y credenciales temporales |
| Secretos | Secrets Manager y rotacion cuando el origen lo permita |
| Cifrado | KMS para datos y logs sensibles; TLS para trafico |
| Aplicacion | WAF, validacion JWT, autorizacion y rate limiting |
| Lakehouse | Lake Formation por rol, pais, dominio y sensibilidad |
| Redshift | Vistas read-only, schemas y roles aislados |
| DynamoDB | Clave `tenant_country#company_id` y autorizacion previa en la API |
| GenAI | Filtros de metadatos, herramientas allowlist y Guardrails |
| Auditoria | CloudTrail, logs cifrados y retencion definida |

Las politicas de confianza incluyen `aws:SourceArn` y `aws:SourceAccount` cuando el servicio lo soporta para reducir el riesgo de confused deputy.

## Topologia propuesta

- ALB, ECS Fargate, Glue y componentes con acceso a datos se ejecutan en subredes privadas y varias AZ.
- VPC endpoints evitan salida publica para servicios AWS compatibles.
- NAT proporciona salida controlada a scrapers; no se acepta trafico entrante desde Internet hacia los workers.
- Security Groups permiten solo los flujos necesarios.
- Teradata y Redshift privados requieren VPN, Direct Connect, peering o Transit Gateway segun su ubicacion real.
- El acceso administrativo utiliza mecanismos corporativos y no expone puertos de gestion publicos.

## Frontera de autorizacion

El pais o tenant debe validarse en Spring Boot y propagarse a:

- Partition keys de DynamoDB.
- Permisos de Lake Formation.
- Vistas o filtros de Redshift.
- Metadatos de la Knowledge Base.
- Herramientas de AgentCore Gateway.

Ninguna capa confia unicamente en un filtro del frontend.

## Datos sensibles y logs

- Redactar tokens, cookies, credenciales y cabeceras `Authorization`.
- No utilizar usuario o empresa como dimension CloudWatch de alta cardinalidad.
- Cifrar logs con KMS y limitar acceso y retencion.
- Habilitar eventos de datos de CloudTrail de forma selectiva donde aporten trazabilidad.
- Confirmar residencia, retencion y obligaciones regulatorias antes del despliegue.

## Pendientes de red

- Cuentas, regiones y VPC existentes.
- CIDR, DNS, rutas y conectividad con origenes.
- Escenarios cross-account.
- Endpoints privados disponibles por servicio y region.
- Egress permitido para scraping y APIs sociales.
