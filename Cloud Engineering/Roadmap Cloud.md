
# Roadmap Cloud Engineering / DevOps — Julio a Noviembre 2026

**Punto de partida:** sin experiencia profesional en cloud/dev.
**Disponibilidad:** ~48h/semana (8h L-V + 8h fin de semana). Esto es full-time real; el plan está calculado a ese ritmo, no a ritmo “side project”.
**Objetivo noviembre 2026:** SAA Associate + Terraform Associate + 3 proyectos de portfolio con mindset de producción, listo para procesos de selección.

-----

## 0. Contexto que cambia las decisiones del plan

Antes del desglose semana a semana, esto es lo que la investigación cambia respecto a tu plan original:

**Vas directo a SAA sin Cloud Practitioner.** El contenido de Cloud Practitioner está contenido dentro del de SAA, así que duplicar esfuerzo no aporta valor de conocimiento. El único riesgo real de saltarlo viniendo de cero absoluto es de ritmo: el SAA exige terminología básica *más* capacidad de razonar entre opciones de arquitectura bajo presión de examen, así que las primeras 1-2 semanas de estudio de SAA harán de facto el papel de “fundamentos” que cubriría Cloud Practitioner, solo que integradas directamente en el temario de SAA. Plan de salvaguarda: si a las dos semanas de estudio notas que vas perdido con conceptos básicos (qué es una región, IAM, Shared Responsibility Model) en vez de con las decisiones de arquitectura en sí, es la señal de bajar el ritmo un par de días para reforzar fundamentos antes de seguir, no de abandonar el examen directo.

**Terraform no puede ir “al final del todo”.** Tu plan original deja Terraform Associate como última pieza, después de los tres proyectos. El problema: dos de tus tres proyectos están diseñados explícitamente para usar Terraform como IaC. Si dejas el *aprendizaje* de Terraform para noviembre, en los meses anteriores estarás escribiendo HCL por imitación de tutoriales sin entender bien módulos, state, workspaces, etc. — exactamente lo que un entrevistador detecta enseguida. Propuesta: separar “aprender Terraform” (agosto, antes de empezar a construir) de “certificarse en Terraform” (se puede hacer en cualquier momento desde que domines la herramienta, incluyendo dejarlo para el final si quieres, pero ya como repaso de algo que usas desde hace meses, no como aprendizaje desde cero).

**El mercado junior en 2026 está duro, y lo será para cuando termines.** Las contrataciones entry-level en tech han caído de forma medible en 2025-2026 por la presión de la IA sobre tareas junior; varios análisis del primer trimestre de 2026 documentan caídas de doble dígito en puestos junior dentro de roles expuestos a IA. Al mismo tiempo, el mercado cloud/DevOps específico en España resiste mejor: el sector de Platform Engineering, Cloud Engineering y DevOps/SRE en España crece con salarios junior entre 22.000€ y 50.000€ y es uno de los perfiles señalados con mayor crecimiento salarial (15-18% anual) en 2025-2026, junto con un déficit de más de 120.000 vacantes TIC sin cubrir en España. La lectura práctica: el hueco existe, pero la barra de entrada para destacar entre candidatos junior es más alta que hace dos años. Esto refuerza tu instinto original: documentación seria, diagramas, costes, mindset de producción no son “un plus”, son el mínimo para no desaparecer entre cientos de candidatos con certificaciones pero sin evidencia real de saber construir.

**Kubernetes es el hueco más grande de tu plan actual.** Aparece de forma recurrente en las búsquedas de mercado español 2026 (DevOps/Platform Engineering con Kubernetes y Terraform es la combinación con mayor crecimiento salarial citada) y en listados de proyectos que recomiendan reclutadores. No te recomiendo meter EKS completo en este roadmap — añadiría semanas y tu objetivo de noviembre es realista solo si no metes alcance nuevo grande. Pero sí incluyo un proyecto extra opcional de Kubernetes básico (local, con kind o minikube, sin coste) que puedes hacer si vas sobrado de tiempo, porque tener algo de Kubernetes que mencionar en entrevista (aunque sea básico) cierra una pregunta muy común: “¿has tocado contenedores/Kubernetes alguna vez?”.

**El “Cloud Resume Challenge” ya existe y tu Proyecto 1 lo está reinventando a medias.** Es el reto más respetado y reconocido en la comunidad cloud para portfolios de entrada (creado por Forrest Brazeal, con más de 12.000 personas en su comunidad). Tiene 16 pasos exactos y un detalle clave que tu proyecto de portfolio actual no tenía: un **backend dinámico** (contador de visitas vía API Gateway + Lambda + DynamoDB, con tests en Python). Esto es importante porque “página estática en S3 con CI/CD” demuestra menos que “página estática + API serverless con tests + CI/CD”, y el coste extra de implementarlo es bajo (ya vas a aprender Lambda/DynamoDB a fondo en el Proyecto 2 de todas formas). Te recomiendo fusionar tu Proyecto 1 con esta estructura.

-----

## 1. Orden de certificaciones (revisado)

| Certificación                               | Cuándo                                                                        | Duración estimada a tu ritmo                                                                           | Nota                                                                                                                                                                                                                                                     |
| ------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AWS Solutions Architect Associate (SAA-C03) | Julio (última semana) — agosto                                                | 5-7 semanas a tu ritmo (full-time), incluyendo fundamentos básicos integrados las primeras 1-2 semanas | El examen no ha cambiado de estructura en 2026; sigue siendo SAA-C03, con dominios de arquitectura resiliente, alto rendimiento, seguridad y coste. Sin Cloud Practitioner previo, las primeras semanas serán más densas — ver salvaguarda en sección 0. |
| Terraform Associate (003)                   | Aprender en agosto, certificar cuando quieras (octubre-noviembre como repaso) | Aprendizaje: 2 semanas intensivas. Examen: 1 semana de repaso una vez ya lo usas en los 3 proyectos    | Separamos “aprender” de “examinarse”. El examen en sí es relativamente corto y barato (70$), no necesita un bloque de mes dedicado si ya llevas meses usando la herramienta.                                                                             |

-----

## 2. Linux y Git (julio, primeras 2-3 semanas)

Esto ya estaba en tu plan original, bien situado. Algunas precisiones de qué nivel necesitas realmente para no sobre-invertir tiempo aquí:

**Linux:** navegación de filesystem, permisos (chmod/chown), gestión de procesos, redirecciones y pipes, edición con vim o nano, gestión básica de paquetes (apt), variables de entorno, y muy importante para lo que viene después: cómo leer logs (`tail`, `grep`, `journalctl`) y cómo se comporta un servicio systemd básico. No necesitas convertirte en sysadmin experto; necesitas comodidad total en terminal porque vas a vivir ahí los próximos meses.

**Git:** no solo `add/commit/push`. Necesitas branching real, resolver conflictos de merge, rebase básico, y sobre todo **buenas prácticas de commits** (mensajes descriptivos, commits atómicos) porque tu repo de portfolio será evaluado por reclutadores que sí miran el historial de commits, no solo el resultado final.

-----

## 3. Aprendizaje de Terraform (agosto, antes de tocar AWS en profundidad)

Antes de Solutions Architect, te recomiendo intercalar:

- Sintaxis HCL, providers, resources, data sources
- State y remote state (muy importante: backend en S3 + DynamoDB para locking, que además es exactamente lo que vas a necesitar para tu propio portfolio)
- Variables, outputs, módulos
- Workspaces y entornos (dev/prod)
- `terraform plan/apply/destroy`, y el hábito de SIEMPRE revisar el plan antes de aplicar

Esto no requiere examinarte aún. Solo necesitas poder escribir HCL con soltura cuando llegues a construir los proyectos en septiembre-octubre.

-----

## 4. Calendario mes a mes

### Julio — Fundamentos + arranque de SAA

- Semana 1-2: Linux intensivo + Git intensivo
- Semana 3: arranque de SAA-C03 — primeros dominios, integrando terminología básica (regiones, IAM, Shared Responsibility Model) directamente sobre la marcha, sin examen previo de Cloud Practitioner
- Semana 4: continuar SAA por dominios (EC2, VPC, S3, IAM a fondo) + primeros labs prácticos en Free Tier acompañando cada concepto

### Agosto — SAA + Terraform en paralelo

- Semanas 1-3: estudio SAA-C03 por dominios (resiliencia, alto rendimiento, seguridad, coste) + labs prácticos en Free Tier acompañando cada dominio (no solo vídeos: monta VPC real, EC2+ALB+ASG real, S3 con lifecycle real)
- Semana 3-4 en paralelo (tardes/bloques alternos): aprendizaje de Terraform (sin certificar aún)
- Fin de agosto: examen SAA-C03

### Septiembre — Proyecto 1: Portfolio + Cloud Resume Challenge fusionado

- Construcción completa con Terraform desde el primer commit (no ClickOps y luego migrar — directo a IaC, ya que para entonces dominas Terraform)
- Frontend (HTML/CSS/JS) con contador de visitas
- Backend: API Gateway + Lambda (Python) + DynamoDB para el contador
- Tests Python para la Lambda
- S3 + CloudFront + ACM + Route 53
- CI/CD con GitHub Actions: pipeline que corre tests, y si pasan, despliega infra (Terraform) y sincroniza frontend + invalida caché CloudFront
- README tipo empresa + diagrama de arquitectura + sección de costes + blog post corto explicando decisiones (esto último es literalmente el paso 16 del Cloud Resume Challenge, y sirve directamente como tu primer “blog post corto” mencionado en tu plan original)

### Octubre — Proyecto 2: Cloud Photos (serverless completo)

- Cognito con federación Google
- API Gateway + Lambda (RESTful)
- S3 para fotos + DynamoDB para metadatos
- CloudFront
- Terraform de principio a fin
- CI/CD con GitHub Actions
- Aquí es donde añades: logging centralizado con CloudWatch bien estructurado (logs estructurados en JSON, no solo print), manejo de errores y reintentos en las Lambdas, IAM de mínimo privilegio por función (no un rol genérico para todo)

### Noviembre — Proyecto 3: Detección de anomalías en ELB + cierre de portfolio

- Primeras 2 semanas: sistema serverless de revisión de logs ALB (EventBridge programado, Lambda de análisis, almacenamiento de estado de alertas, notificación a Slack, botón de verificación de falsos positivos)
- Esta es la pieza más sofisticada lógicamente — por eso va última, cuando ya tienes Terraform, Lambda, DynamoDB y CloudWatch dominados de los proyectos anteriores
- Resto de noviembre: documentación final de los 3 proyectos al mismo nivel (README, diagramas, costes, decisiones justificadas), preparar respuestas a las preguntas típicas de entrevista por proyecto, y si llegas con margen, examen Terraform Associate como cierre

-----

## 5. Production mindset — checklist transversal para los 3 proyectos

Esto no es una fase, es un estándar a aplicar en cada proyecto desde el primer commit:

- **Logging centralizado:** CloudWatch Logs con formato estructurado, métricas custom donde aporte, sin logs ruidosos sin sentido
- **Manejo de errores:** las Lambdas no deben fallar en silencio; captura de excepciones, respuestas de error claras en la API, dead-letter queues donde tenga sentido (especialmente útil de mencionar en el proyecto de anomalías ELB)
- **Reintentos / fallbacks:** entender y configurar retries en Lambda/SQS, y saber explicar qué pasa si un servicio downstream falla
- **Seguridad / IAM mínimo privilegio:** cada Lambda con su propio rol, con permisos exactos a los recursos que toca, nunca `*` salvo que lo justifiques
- **Versionado de infraestructura:** todo en Terraform, en repos públicos, con tags de versión cuando tenga sentido
- **Testing:** no solo en el Cloud Resume Challenge — añade tests básicos (pytest) también en Cloud Photos y en el sistema de detección de anomalías
- **Costes:** para cada proyecto, una tabla real con estimación de coste a tráfico bajo (tu caso real, probablemente 0€ en Free Tier) y a escala (10K, 100K usuarios), con cifras, no solo “es barato”

-----

## 6. Preparación de entrevistas (a partir de octubre, en paralelo)

Para cada uno de los 3 proyectos, prepara por escrito (no solo mentalmente):

- Explicación del sistema de 2 minutos, sin mirar notas
- 3-5 preguntas específicas de “qué se rompe aquí” basadas en tus decisiones reales (no genéricas): ejemplo para Cloud Photos — “¿qué pasa si dos usuarios suben una foto con el mismo nombre simultáneamente?”, “¿cómo manejas que Cognito esté caído?”
- Una variante de arquitectura que dibujarías distinto si tuvieras que escalar a 1M usuarios
- El coste real estimado a esa escala

-----

## 7. Extra opcional (solo si vas sobrado de tiempo)

**Kubernetes básico (local, sin coste):** un mini-proyecto con kind o minikube, desplegando un contenedor sencillo, con un Service y un Deployment, quizás un ConfigMap. No es EKS en AWS — es simplemente para poder decir en entrevista “he tocado Kubernetes” con algo concreto detrás, dado que aparece repetidamente como skill de alto crecimiento salarial en el mercado español 2026. El propio Cloud Resume Challenge tiene una extensión de Kubernetes (“Kubernetes Resume Challenge”) con pasos muy claros si quieres replicarla.

**Blog posts cortos adicionales:** ya tienes uno integrado en el Proyecto 1 (paso 16 del Cloud Resume Challenge). Si tienes margen, uno por cada proyecto adicional (“cómo detecté anomalías de seguridad con Lambda y EventBridge”) suma visibilidad en LinkedIn/Dev.to y es evidencia adicional de comunicación técnica, algo que el contexto de mercado 2026 señala como diferenciador junior.

-----

## 8. Riesgos del plan a vigilar

- **Burnout:** 48h/semana sostenidas durante 5 meses es exigente. Construye descansos reales, no solo “menos densos”.
- **Perfeccionismo en Proyecto 1:** es el primero y el más tentador de pulir sin fin. Pon fecha límite dura de fin de septiembre y pasa al Proyecto 2 aunque no esté perfecto — siempre puedes volver a refinarlo en noviembre.
- **Dejar Terraform Associate fuera si se acumula retraso:** si en noviembre el tiempo aprieta, prioriza tener los 3 proyectos bien documentados sobre la certificación Terraform. Un portfolio sólido con Terraform usado en producción real pesa más en una entrevista que la insignia de certificación sin proyectos que la respalden.