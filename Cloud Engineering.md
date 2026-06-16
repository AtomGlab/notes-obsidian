Julio 2026: Linux, Git 

### Certifications to Get:

 Agosto/noviembre 2026 -> [[Solutions Architect Associate]]  
HashiCorp -> [[Terraform Associate 004]]

Objetivo portfolio noviembre 2026
### Portfolio Personal Desplegado en AWS con CI/CD

Portfolio personal desplegado automáticamente en AWS con S3 para almacenamiento, Route 53 para gestionar el dominio, CloudFront para distribución global y HTTPS gestionado por ACM.  
La automatización del despliegue se realiza mediante GitHub Actions, que sincroniza archivos con S3 e invalida la caché de CloudFront en cada push a la rama principal. Se utiliza Terraform para gestionar la infraestructura como código, garantizando un proceso de despliegue consistente y repetible.

### Cloud Photos Desplegado en AWS con CI/CD

Una mini-aplicación de almacenamiento de fotos personal construida sobre una arquitectura AWS completamente serverless. El proyecto incluye autenticación federada con Google (Cognito), un backend RESTful usando API Gateway y Lambda, almacenamiento de fotos en S3, gestión de metadatos en DynamoDB y distribución global mediante CloudFront. El despliegue del frontend y la gestión de dominio están automatizados con GitHub Actions.

### Detección de Anomalías en ELB en AWS

Sistema serverless que revisa diariamente los logs del Application Load Balancer en AWS para detectar actividad sospechosa y notificar al equipo de seguridad. Busca patrones de comportamiento como muchas peticiones en poco tiempo, rutas inusuales o intentos de acceso a áreas de administración, y mantiene el estado de alertas almacenado para que pueda revisarse desde Slack. También incluye un botón de verificación para marcar falsos positivos y mantener la lista de IPs de confianza actualizada.

## Simular entorno de empresa real

- documentación tipo empresa (README serio)
- diagramas de arquitectura (muy importante)
- costes aproximados en AWS
- decisiones justificadas (“por qué Lambda y no EC2”)

- “hace proyectos”  
    vs
- “piensa como cloud engineer” 

## Un entorno “production mindset”

- logging centralizado (CloudWatch bien usado)
- gestión de errores
- retries / fallbacks
- security (IAM bien hecho, mínimo privilegio)
- versionado de infra

Importante para entrevistas.

## Preparación de entrevistas

Capacidad de:
- explicar todo tu sistema sin mirar notas
- diseñar variantes en pizarra
- responder preguntas tipo:
    - “qué pasa si Lambda falla”
    - “cómo escalas esto a 1M usuarios”
    - “qué coste tiene esto”
    
- README muy bien explicado
- screenshots o diagramas
- quizá blog posts cortos (“cómo hice X en AWS”)

### Resources:

https://portfolio-adrianriera.com/

