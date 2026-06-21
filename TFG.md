# ☁️ CloudSecure Photos — Serverless Photo Storage with Anomaly Detection

> Plataforma serverless de almacenamiento de fotos con sistema integrado de detección de anomalías de seguridad, desplegada completamente en AWS mediante Infrastructure as Code.

![AWS|76](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws) ![Terraform](https://img.shields.io/badge/IaC-Terraform-purple?logo=terraform) ![Python](https://img.shields.io/badge/Backend-Python-blue?logo=python) ![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-black?logo=github-actions) ![License](https://img.shields.io/badge/License-MIT-green)

---

## 📋 Tabla de Contenidos

- [Descripción General](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#descripci%C3%B3n-general)
- [Arquitectura](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#arquitectura)
- [Servicios AWS Utilizados](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#servicios-aws-utilizados)
- [Módulos del Proyecto](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#m%C3%B3dulos-del-proyecto)
- [Estructura del Repositorio](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#estructura-del-repositorio)
- [Requisitos Previos](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#requisitos-previos)
- [Despliegue](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#despliegue)
- [CI/CD Pipeline](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#cicd-pipeline)
- [Sistema de Detección de Anomalías](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#sistema-de-detecci%C3%B3n-de-anomal%C3%ADas)
- [Seguridad](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#seguridad)
- [Estimación de Costes](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#estimaci%C3%B3n-de-costes)
- [Decisiones de Arquitectura](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#decisiones-de-arquitectura)
- [Evaluación Well-Architected](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#evaluaci%C3%B3n-well-architected)
- [Roadmap](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#roadmap)
- [Autor](https://claude.ai/chat/5941d80f-2dab-484a-979f-5e2082b94bd8#autor)

---

## Descripción General

**CloudSecure Photos** es una aplicación de almacenamiento de fotos personal construida sobre arquitectura serverless en AWS. El proyecto nace con un doble objetivo: proporcionar una plataforma funcional de gestión de imágenes y demostrar cómo implementar seguridad proactiva mediante análisis de logs y detección automática de comportamientos anómalos.

El sistema está compuesto por dos capas principales:

- **Capa de Aplicación (Cloud Photos):** autenticación federada, gestión de fotos y distribución global.
- **Capa de Seguridad (Anomaly Detection):** análisis periódico de logs, detección de patrones sospechosos y notificación automática al equipo.

Toda la infraestructura está definida como código con Terraform y desplegada automáticamente mediante GitHub Actions en cada push a la rama principal.

---

## Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                        USUARIO FINAL                           │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTPS
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CAPA DE DISTRIBUCIÓN                        │
│                                                                 │
│   Route 53 (DNS)  →  CloudFront (CDN)  →  ACM (certificado)   │
└──────────────┬──────────────────────────────────────────────────┘
               │
       ┌───────┴────────┐
       │                │
       ▼                ▼
  [Frontend]      [API Gateway]
  S3 (static)          │
                        │ invoca
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CAPA DE APLICACIÓN                          │
│                                                                 │
│   Cognito (Auth)  ←→  Lambda (Python)  ←→  S3 (fotos)         │
│                              │                                  │
│                              └──→  DynamoDB (metadatos)        │
└──────────────────────────────┬──────────────────────────────────┘
                               │ genera logs
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CAPA DE SEGURIDAD                           │
│                                                                 │
│   CloudWatch Logs  →  EventBridge (cron)  →  Lambda Analyzer   │
│                                                    │            │
│                              ┌─────────────────────┘            │
│                              │                                  │
│                    DynamoDB (alertas)  →  SNS  →  Email/Slack   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Servicios AWS Utilizados

|Categoría|Servicio|Uso|
|---|---|---|
|**Auth**|Amazon Cognito|Autenticación de usuarios + federación con Google|
|**API**|API Gateway (HTTP API)|Endpoints REST para la aplicación|
|**Compute**|AWS Lambda (Python 3.12)|Lógica de negocio y análisis de anomalías|
|**Storage**|Amazon S3|Almacenamiento de fotos y frontend estático|
|**Database**|Amazon DynamoDB|Metadatos de fotos y estado de alertas|
|**CDN**|Amazon CloudFront|Distribución global y caché|
|**DNS**|Amazon Route 53|Gestión del dominio personalizado|
|**TLS**|AWS Certificate Manager|Certificado HTTPS gratuito|
|**Seguridad**|AWS IAM|Roles y políticas de mínimo privilegio|
|**Secretos**|AWS Secrets Manager|Credenciales y configuración sensible|
|**Observabilidad**|Amazon CloudWatch|Logs centralizados y métricas|
|**Eventos**|Amazon EventBridge|Scheduler para análisis periódico de logs|
|**Notificaciones**|Amazon SNS|Alertas de seguridad por email|
|**IaC**|Terraform|Gestión de toda la infraestructura|
|**CI/CD**|GitHub Actions|Pipeline de despliegue automatizado|

---

## Módulos del Proyecto

### Módulo 1 — Cloud Photos

Aplicación serverless de gestión de fotos con las siguientes funcionalidades:

- **Registro e inicio de sesión** con email/contraseña o cuenta de Google (Cognito + OAuth2)
- **Upload de fotos** mediante presigned URLs de S3 (el archivo va directo a S3, no pasa por Lambda)
- **Galería personal** con listado de fotos y metadatos almacenados en DynamoDB
- **Descarga y eliminación** de fotos con validación de propiedad
- **Distribución global** del frontend y las imágenes mediante CloudFront

### Módulo 2 — Anomaly Detection

Sistema de análisis de logs que se ejecuta cada hora mediante EventBridge y detecta los siguientes patrones:

|Patrón|Descripción|Umbral|
|---|---|---|
|**Fuerza bruta**|Múltiples intentos de login fallidos desde la misma IP|>10 intentos en 5 minutos|
|**Exfiltración masiva**|Un usuario descarga un volumen inusual de fotos|>50 fotos en 10 minutos|
|**Escaneo de rutas**|Peticiones a endpoints inexistentes (bots, reconocimiento)|>20 rutas distintas en 1 minuto|

Cuando se detecta una anomalía:

1. Se registra en DynamoDB con estado `OPEN`
2. Se envía notificación por email vía SNS
3. Se puede marcar como falso positivo desde Slack (webhook)
4. Las IPs de confianza se mantienen en una lista blanca actualizable

---

## Estructura del Repositorio

```
cloudsecure-photos/
│
├── infrastructure/              # Terraform — toda la infraestructura
│   ├── modules/
│   │   ├── networking/          # VPC, subnets, security groups
│   │   ├── auth/                # Cognito user pool y identity pool
│   │   ├── api/                 # API Gateway + Lambda functions
│   │   ├── storage/             # S3 buckets + DynamoDB tables
│   │   ├── cdn/                 # CloudFront + Route53 + ACM
│   │   └── security/            # IAM roles, Secrets Manager
│   ├── environments/
│   │   ├── dev/                 # Variables para entorno de desarrollo
│   │   └── prod/                # Variables para entorno de producción
│   └── main.tf
│
├── backend/                     # Lambdas en Python
│   ├── photos/
│   │   ├── upload.py            # Genera presigned URL para S3
│   │   ├── list.py              # Lista fotos del usuario
│   │   ├── delete.py            # Elimina foto y metadatos
│   │   └── download.py          # Genera URL de descarga
│   ├── anomaly_detector/
│   │   ├── analyzer.py          # Motor de análisis de logs
│   │   ├── patterns/
│   │   │   ├── brute_force.py
│   │   │   ├── mass_download.py
│   │   │   └── path_scanning.py
│   │   └── notifier.py          # Envío de alertas por SNS
│   └── shared/
│       ├── auth.py              # Validación de tokens Cognito
│       └── db.py                # Cliente DynamoDB compartido
│
├── frontend/                    # React (estático, desplegado en S3)
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── services/            # Llamadas a la API
│   └── public/
│
├── .github/
│   └── workflows/
│       ├── deploy-infra.yml     # Terraform plan + apply
│       ├── deploy-backend.yml   # Tests + deploy Lambdas
│       └── deploy-frontend.yml  # Build + sync S3 + invalidar CF
│
├── docs/
│   ├── architecture/
│   │   ├── architecture-diagram.png
│   │   └── data-flow-diagram.png
│   ├── adr/                     # Architecture Decision Records
│   │   ├── ADR-001-serverless-vs-containers.md
│   │   ├── ADR-002-dynamodb-vs-rds.md
│   │   └── ADR-003-cognito-auth.md
│   └── cost-analysis.md
│
├── tests/
│   ├── unit/
│   └── integration/
│
└── README.md
```

---

## Requisitos Previos

- [AWS CLI](https://aws.amazon.com/cli/) configurado con credenciales válidas
- [Terraform](https://www.terraform.io/) >= 1.6
- [Python](https://www.python.org/) >= 3.12
- [Node.js](https://nodejs.org/) >= 20 (para el frontend)
- Dominio registrado en Route 53 (o transferido desde otro registrador)

---

## Despliegue

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/cloudsecure-photos.git
cd cloudsecure-photos
```

### 2. Configurar variables de entorno

```bash
cp infrastructure/environments/dev/terraform.tfvars.example \
   infrastructure/environments/dev/terraform.tfvars

# Editar con tus valores:
# - domain_name
# - aws_region
# - google_client_id (para federación con Google)
```

### 3. Inicializar y aplicar Terraform

```bash
cd infrastructure/environments/dev
terraform init
terraform plan
terraform apply
```

### 4. Desplegar el backend

```bash
cd backend
pip install -r requirements.txt
# El despliegue de Lambdas se gestiona desde Terraform
```

### 5. Desplegar el frontend

```bash
cd frontend
npm install
npm run build
aws s3 sync dist/ s3://$(terraform output -raw frontend_bucket_name)
aws cloudfront create-invalidation \
  --distribution-id $(terraform output -raw cloudfront_distribution_id) \
  --paths "/*"
```

> En producción, todos estos pasos se ejecutan automáticamente mediante GitHub Actions.

---

## CI/CD Pipeline

Cada push a `main` desencadena el siguiente pipeline:

```
Push a main
    │
    ├── [Test] Unit tests + linting (Python + JS)
    │
    ├── [Infra] terraform plan → terraform apply
    │
    ├── [Backend] Package Lambdas → Deploy vía Terraform
    │
    └── [Frontend] npm build → S3 sync → CloudFront invalidation
```

Los entornos de desarrollo y producción tienen pipelines independientes. Los PRs solo ejecutan `terraform plan` sin aplicar cambios.

---

## Sistema de Detección de Anomalías

### Flujo de análisis

```
EventBridge (cada hora)
        │
        ▼
Lambda Analyzer
        │
        ├── Lee logs de CloudWatch (última hora)
        │
        ├── Aplica patrones de detección
        │         ├── BruteForcePattern
        │         ├── MassDownloadPattern
        │         └── PathScanningPattern
        │
        ├── Si anomalía detectada:
        │         ├── Guarda alerta en DynamoDB
        │         └── Publica en SNS → Email
        │
        └── Actualiza métricas en CloudWatch
```

### Gestión de alertas

Las alertas tienen los siguientes estados:

|Estado|Descripción|
|---|---|
|`OPEN`|Anomalía detectada, pendiente de revisión|
|`ACKNOWLEDGED`|Revisada por el equipo de seguridad|
|`FALSE_POSITIVE`|Marcada como falso positivo|
|`RESOLVED`|Incidente resuelto|

### Lista blanca de IPs

Las IPs de confianza (oficinas, VPNs corporativas) se almacenan en DynamoDB y el analizador las excluye automáticamente del análisis. Se pueden actualizar sin necesidad de redesplegar.

---

## Seguridad

Este proyecto aplica el principio de **mínimo privilegio** en todos los componentes:

- Cada Lambda tiene su propio rol IAM con permisos exclusivamente necesarios para su función
- Los buckets S3 tienen acceso público bloqueado; las fotos se sirven únicamente mediante presigned URLs con expiración
- Los secretos (credenciales de Google, tokens) se almacenan en Secrets Manager, nunca en variables de entorno ni en código
- Cognito gestiona la autenticación; las Lambdas nunca almacenan contraseñas
- CloudFront fuerza HTTPS en todas las conexiones
- Los logs de API Gateway se envían a CloudWatch con retención de 90 días

---

## Estimación de Costes

Estimación para uso personal con ~1.000 fotos y tráfico bajo (capa gratuita de AWS incluida):

|Servicio|Uso estimado|Coste mensual|
|---|---|---|
|Lambda|~10.000 invocaciones/mes|~$0.00 (free tier)|
|API Gateway|~10.000 requests/mes|~$0.035|
|S3|~5 GB almacenamiento|~$0.115|
|DynamoDB|~1 GB, on-demand|~$0.25|
|CloudFront|~10 GB transferencia|~$0.85|
|Route 53|1 hosted zone|~$0.50|
|CloudWatch|Logs básicos|~$0.50|
|SNS|~100 notificaciones|~$0.00 (free tier)|
|**TOTAL**||**~$2.25/mes**|

> Los costes reales dependen del tráfico. Para un uso real de producción con miles de usuarios, consultar el [AWS Pricing Calculator](https://calculator.aws/).

---

## Decisiones de Arquitectura

Las decisiones técnicas relevantes están documentadas como Architecture Decision Records (ADR) en `/docs/adr/`. A continuación un resumen de las principales:

**¿Por qué Lambda y no EC2 o ECS?** La carga de trabajo es intermitente y orientada a eventos. Lambda elimina la gestión de servidores, escala automáticamente a cero y su modelo de pago por invocación es óptimo para este volumen. EC2 tendría un coste fijo 24/7 innecesario.

**¿Por qué DynamoDB y no RDS?** Los datos de metadatos de fotos son simples y no requieren joins complejos. DynamoDB ofrece latencia de un dígito en milisegundos, escala sin gestión y el modelo on-demand evita costes de instancias RDS siempre activas.

**¿Por qué Cognito y no Auth0 o sistema propio?** Cognito es nativo de AWS, se integra directamente con API Gateway y IAM, soporta federación con Google sin coste adicional y elimina la responsabilidad de gestionar contraseñas.

**¿Por qué HTTP API y no REST API en API Gateway?** HTTP API es más barato (~70% menos), tiene menor latencia y soporta JWT authorization nativo con Cognito. REST API solo se justifica cuando se necesitan características avanzadas como caching por endpoint o API keys.

---

## Evaluación Well-Architected

Este proyecto ha sido evaluado contra los 6 pilares del [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/):

|Pilar|Estado|Notas|
|---|---|---|
|**Excelencia Operativa**|✅|IaC completo, CI/CD automatizado, logs centralizados|
|**Seguridad**|✅|IAM mínimo privilegio, Secrets Manager, HTTPS forzado|
|**Fiabilidad**|⚠️|Single-region; multi-region fuera del alcance actual|
|**Eficiencia de Rendimiento**|✅|Serverless, CloudFront con caché, presigned URLs|
|**Optimización de Costes**|✅|Pay-per-use, sin instancias ociosas, free tier aprovechado|
|**Sostenibilidad**|✅|Serverless reduce huella de carbono vs servidores dedicados|

---

## Roadmap

- [x] Infraestructura base con Terraform
- [x] Autenticación con Cognito + Google
- [x] Upload/download/delete de fotos
- [x] Sistema de detección de anomalías
- [x] Notificaciones por email via SNS
- [ ] Dashboard web para gestión de alertas
- [ ] Detección de acceso desde geolocalización inusual
- [ ] Soporte multi-región para alta disponibilidad
- [ ] Integración con AWS WAF para bloqueo automático de IPs

---

## Autor

**[Tu Nombre]** Trabajo de Fin de Grado — Ingeniería del Software Universidad [Nombre] · Curso 2026/2027

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://linkedin.com/in/tu-perfil) [![GitHub](https://img.shields.io/badge/GitHub-Follow-black?logo=github)](https://github.com/tu-usuario)

---

> _Este proyecto forma parte del TFG "Diseño e implementación de una plataforma serverless en AWS con sistema integrado de detección de anomalías de seguridad". La infraestructura está desplegada en una cuenta AWS personal con fines académicos y de portfolio profesional._