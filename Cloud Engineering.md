Linux, Git

### Certifications to Get:

Aws -> [[Solutions Architect Associate]] 
HashiCorp -> [[Terraform Associate 004]]



### Portfolio Personal Desplegado en AWS con CI/CD

Portfolio personal desplegado automáticamente en AWS con S3 para almacenamiento, Route 53 para gestionar el dominio, CloudFront para distribución global y HTTPS gestionado por ACM.  
La automatización del despliegue se realiza mediante GitHub Actions, que sincroniza archivos con S3 e invalida la caché de CloudFront en cada push a la rama principal. Se utiliza Terraform para gestionar la infraestructura como código, garantizando un proceso de despliegue consistente y repetible.

### Cloud Photos Desplegado en AWS con CI/CD

Una mini-aplicación de almacenamiento de fotos personal construida sobre una arquitectura AWS completamente serverless. El proyecto incluye autenticación federada con Google (Cognito), un backend RESTful usando API Gateway y Lambda, almacenamiento de fotos en S3, gestión de metadatos en DynamoDB y distribución global mediante CloudFront. El despliegue del frontend y la gestión de dominio están automatizados con GitHub Actions.

### Detección de Anomalías en ELB en AWS

Sistema serverless que revisa diariamente los logs del Application Load Balancer en AWS para detectar actividad sospechosa y notificar al equipo de seguridad. Busca patrones de comportamiento como muchas peticiones en poco tiempo, rutas inusuales o intentos de acceso a áreas de administración, y mantiene el estado de alertas almacenado para que pueda revisarse desde Slack. También incluye un botón de verificación para marcar falsos positivos y mantener la lista de IPs de confianza actualizada.



### Resources:

https://portfolio-adrianriera.com/