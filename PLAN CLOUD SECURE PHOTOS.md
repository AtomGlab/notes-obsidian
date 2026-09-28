# Plan TFG CloudSecure Photos — 30 semanas

Sep 28, 2026 · @Q

## Resumen y supuestos

El sistema queda desplegado con CI/CD el 20 de diciembre, el motor de detección el 7 de febrero y el alcance se congela el 14 de marzo. El borrador completo llega al tutor el 4 de abril y la entrega es el 25 de abril.

Supuestos del plan (cámbialos si no encajan):

- Unas 15 horas por semana. Con menos, aplica la sección de recortes antes de retrasarte.
- Sabes lo básico de AWS y Python; Terraform lo aprendes en las semanas 1 y 2.
- Entrega final a finales de abril de 2027. Si tu fecha es otra, desplaza las fases enteras.
- El formato de la memoria (TFG español o dissertation de un top-up británico) se confirma en la semana 1.
- Navidad (S13 y S14) y Semana Santa (S26) llevan carga reducida, unas 5 horas.

Cada semana tiene un objetivo, tareas que puedes marcar, un criterio de «Hecho cuando» y lo que toca escribir de la memoria. Si una semana no se cierra, no la arrastres: mira los hitos de control al final.

&#91;embedded content: cronograma · 7 fases y 4 hitos\]

La evaluación experimental (fase 5) empieza con el sistema ya terminado; por eso las fases 2 a 4 no pueden retrasarse.

## Fase 1 · Base y diseño (S1–S4, 28 sep – 25 oct)

Al terminar la fase tienes la cuenta segura, el pipeline haciendo `plan`, la fuente de logs de login decidida y la pregunta de investigación aprobada.

### S1 · 28 sep – 4 oct · Arranque

- [ ] Reunión con el tutor: formato de la memoria, extensión, estilo de citas, fecha exacta de entrega y criterios de evaluación.
- [ ] Preguntar si hace falta aprobación ética para simular ataques contra tu propia infraestructura.
- [ ] Cuenta AWS: MFA en root, acceso diario con IAM Identity Center, nunca root.
- [ ] AWS Budgets con alertas a 5, 15 y 30 USD. Elige una región (p. ej. eu-west-1) y no la cambies.
- [ ] Repo en GitHub con la estructura del README, `.gitignore` y pre-commit (`terraform fmt`, ruff).
- [ ] Tablero en GitHub Projects con las 30 semanas.

**Hecho cuando:** cuenta protegida con presupuesto, repo creado y formato de la memoria confirmado.

**Memoria:** plantilla con los 8 capítulos y Zotero configurado.

### S2 · 5 – 11 oct · Terraform, OIDC y prueba de Cognito

- [ ] Módulo `bootstrap`: bucket de estado con versionado y cifrado, con bloqueo (`use_lockfile` en Terraform ≥ 1.10 o tabla DynamoDB).
- [ ] Etiqueta `Project=cloudsecure` por defecto en el provider, para filtrar costes luego.
- [ ] Rol IAM para GitHub Actions por OIDC y workflow que ejecuta `terraform plan` en cada PR.
- [ ] Prueba desechable: user pool mínimo, 20 logins fallidos y comprobar dónde aparecen (CloudTrail, exportación de eventos de threat protection) y qué campos traen (IP, usuario).
- [ ] Anotar cuándo salta el bloqueo nativo de Cognito tras varios fallos; condiciona la simulación de fuerza bruta.

**Hecho cuando:** un PR ejecuta `plan` sin claves guardadas y tienes notas de la prueba con capturas.

### S3 · 12 – 18 oct · Decisiones que condicionan el detector

- [ ] Elegir la fuente de eventos de login: (a) exportación de Cognito threat protection a CloudWatch (plan Plus, de pago), (b) CloudTrail o (c) endpoint propio `/auth/login` que llama a `InitiateAuth` y registra el resultado. Escribir ADR-006.
- [ ] Diseño en casi tiempo real: subscription filter de CloudWatch Logs → Lambda analizadora, con ventanas en DynamoDB y TTL. Escribir ADR-007.
- [ ] Formato común de log JSON: `timestamp`, `request_id`, `user_id`, `source_ip`, `route`, `status`, `event_type`.
- [ ] Redactar la pregunta de investigación y 3–4 objetivos medibles; enviarlos al tutor.

**Hecho cuando:** ADR-006, ADR-007 y el esquema de log escritos, y la pregunta aprobada.

**Memoria:** borrador de introducción y objetivos.

### S4 · 19 – 25 oct · Estado del arte y primer despliegue

- [ ] Reunir 15–20 fuentes: documentación de GuardDuty, WAF y Cognito threat protection, pilar de seguridad de Well-Architected, OWASP (Top 10 y Automated Threats), NIST SP 800-61, MITRE ATT&CK (T1110 Brute Force) y artículos sobre detección de anomalías en logs.
- [ ] Diagrama de arquitectura v1 en draw.io.
- [ ] Módulo `storage`: bucket de fotos privado, bloqueo de acceso público, cifrado y versionado, desplegado desde el pipeline.

**Hecho cuando:** el bucket se despliega desde un merge y el estado del arte tiene esqueleto con 15 referencias o más.

**Memoria:** estado del arte al 50 %.

## Fase 2 · Backend (S5–S9, 26 oct – 29 nov)

Al terminar la fase los cinco endpoints funcionan con autenticación, tests y permisos mínimos por función.

### S5 · 26 oct – 1 nov · Autenticación

- [ ] Módulo `auth`: user pool, app client y dominio de hosted UI.
- [ ] Google como proveedor de identidad (credenciales en Google Cloud Console).
- [ ] Política de contraseñas y verificación de email.
- [ ] Si en S3 elegiste la opción (c), crear aquí el endpoint `/auth/login` con su log.

**Hecho cuando:** obtienes un JWT válido con email y con Google.

### S6 · 2 – 8 nov · API y subida

- [ ] Módulo `api`: HTTP API con autorizador JWT de Cognito y throttling por defecto.
- [ ] Lambda `presign` (Python 3.12): valida tipo y tamaño y genera un PUT pre-firmado de 5 minutos con clave `users/{sub}/{photoId}`.
- [ ] Paquete `shared/` con el logging JSON de S3 y las respuestas de error.
- [ ] Tests con pytest y moto desde esta semana.

**Hecho cuando:** subes una foto real con curl y un JWT.

### S7 · 9 – 15 nov · Metadatos

- [ ] Tabla de fotos (PK `USER#`, SK `PHOTO#`) en on-demand con PITR.
- [ ] Lambda `confirm`: comprueba el objeto con `HeadObject` y guarda los metadatos.
- [ ] Lambda `list` paginada.
- [ ] Cobertura de tests del backend al 70 % o más.

**Hecho cuando:** subir, confirmar y listar funciona de extremo a extremo.

### S8 · 16 – 22 nov · Descarga, borrado y propiedad

- [ ] Lambdas `download` (GET pre-firmado) y `delete`, con la propiedad comprobada por el `sub` del JWT, nunca por un parámetro del cliente.
- [ ] Tests negativos: el usuario A intenta leer y borrar fotos de B y recibe 403 o 404.
- [ ] Ruta `$default` que devuelve 404 y registra `route_not_found`; sin ella no hay datos para detectar escaneos.
- [ ] Evento de log `photo_download` en cada descarga.

**Hecho cuando:** los cinco endpoints funcionan y los tests de acceso cruzado pasan.

### S9 · 23 – 29 nov · Mínimos privilegios

- [ ] Un rol por Lambda con ARN concretos (prefijo del bucket, tabla).
- [ ] Pasar IAM Access Analyzer y guardar el antes y el después.
- [ ] Tests de integración con pytest contra la API real de dev.

**Hecho cuando:** Access Analyzer sin hallazgos abiertos y la integración en verde.

**Memoria:** requisitos, modelo de amenazas STRIDE y ADR-001 a ADR-007 pasados al capítulo de diseño.

## Fase 3 · Frontend y CI/CD (S10–S14, 30 nov – 3 ene)

El 20 de diciembre la aplicación está en tu dominio con HTTPS y cada merge la despliega sola. Es el primer hito de control.

### S10 · 30 nov – 6 dic · Frontend

- [ ] React, TypeScript y Vite con login de Cognito (Amplify Auth u oidc-client-ts).
- [ ] Galería, subida con URL pre-firmada, descarga y borrado.
- [ ] Diseño mínimo y limpio; no inviertas aquí más de una semana.

**Hecho cuando:** el flujo completo funciona en local contra la API de dev.

### S11 · 7 – 13 dic · CDN y dominio

- [ ] Dominio en Route 53 y certificado ACM en us-east-1 (lo exige CloudFront).
- [ ] CloudFront con Origin Access Control para el bucket del frontend.
- [ ] Response headers policy con HSTS y CSP.
- [ ] Dominio propio para la API y CORS del bucket de fotos limitado a tu dominio.

**Hecho cuando:** entras por HTTPS en tu dominio, inicias sesión y subes una foto.

### S12 · 14 – 20 dic · Pipeline completo y escaneos

- [ ] Workflows `deploy-infra`, `deploy-backend` y `deploy-frontend`.
- [ ] Dev se despliega en cada merge a main; prod exige aprobación manual (GitHub environments).
- [ ] En cada PR: ruff, pytest, Checkov o tfsec, Bandit, Gitleaks y npm audit.
- [ ] Guardar el informe de hallazgos inicial, corregir y guardar el informe final.

**Hecho cuando:** un PR pasa lint, tests, escaneos y `plan`, y el merge despliega dev. **Hito de control 1.**

### S13 · 21 – 27 dic · CloudTrail (Navidad, \~5 h)

- [ ] Trail multirregión hacia un bucket dedicado con Object Lock y validación de ficheros de log activada.
- [ ] Object Lock en modo governance en dev; documenta que en producción sería compliance.
- [ ] Evidencia: intentar borrar un log, capturar el error y ejecutar `aws cloudtrail validate-logs`.
- [ ] Retención de 90 días en los log groups de CloudWatch.

**Hecho cuando:** tienes las capturas de la prueba de inmutabilidad.

### S14 · 28 dic – 3 ene · Colchón (Navidad, \~5 h)

- [ ] Terminar lo que quede de S10–S13.
- [ ] Si vas al día: dashboard básico de CloudWatch.

**Memoria:** capítulo de implementación (IaC, CI/CD, backend y frontend) al 60 %.

## Fase 4 · Motor de detección (S15–S19, 4 ene – 7 feb)

El 7 de febrero los tres detectores funcionan en casi tiempo real, el generador de tráfico es reproducible y los umbrales están congelados. Es el segundo hito de control.

### S15 · 4 – 10 ene · Flujo de eventos

- [ ] Subscription filters en los log groups de las Lambdas y en la fuente de login de ADR-006, apuntando a la Lambda `analyzer`.
- [ ] Tabla `SecurityState` con contadores por ventana y TTL; tabla `SecurityAlerts`.
- [ ] Lista de IPs de confianza en DynamoDB, consultada antes de alertar.
- [ ] Medir cuántos segundos tarda un evento en llegar al analyzer.

**Hecho cuando:** un evento generado a mano llega al analyzer y tienes su latencia medida.

### S16 · 11 – 17 ene · Detectores

- [ ] Fuerza bruta, descarga masiva y escaneo de rutas como funciones puras: evento y estado dentro, alerta o nada fuera.
- [ ] Umbrales leídos de configuración, nunca escritos en el código.
- [ ] Tests por detector con secuencias justo por debajo y justo por encima del umbral.

**Hecho cuando:** los tres detectores tienen tests en verde.

### S17 · 18 – 24 ene · Alertas e incidentes

- [ ] SNS por email con severidad baja, media y alta.
- [ ] Ciclo de vida OPEN → ACKNOWLEDGED → RESOLVED o FALSE\_POSITIVE, con un endpoint o un script.
- [ ] Deduplicación: un ataque genera un incidente, no 50 emails.
- [ ] Métricas propias (Embedded Metric Format), alarmas y dashboard de seguridad.

**Hecho cuando:** un ataque manual produce un solo email y un incidente que puedes cerrar.

### S18 · 25 – 31 ene · Generador de tráfico

- [ ] Tráfico legítimo: navegación, subidas, descargas normales, picos y un usuario que descarga su álbum entero (el caso límite de exfiltración).
- [ ] Ataques: fuerza bruta (teniendo en cuenta el bloqueo nativo de Cognito), descarga masiva y escaneo de rutas.
- [ ] Cada petición registrada en un CSV local de ground truth (`run_id`, `label`, `attack_type`), nunca en algo que vea el detector.
- [ ] Semilla aleatoria fija para que cada escenario sea reproducible.

**Hecho cuando:** un comando lanza un escenario y deja su CSV de ground truth.

### S19 · 1 – 7 feb · Diseño experimental

- [ ] Plan de experimentos escrito antes de ejecutar nada: escenarios, métricas (precisión, recall, F1, tasa de falsos positivos, tiempo de detección p50 y p95, coste) y repeticiones.
- [ ] Ajustar los umbrales solo con un conjunto de ajuste y congelarlos.
- [ ] Script de evaluación que cruza las alertas de DynamoDB con el ground truth.
- [ ] Aprobación ética concedida, si la necesitas.

**Hecho cuando:** el tutor aprueba el plan de experimentos y los umbrales están congelados. **Hito de control 2.**

**Memoria:** diseño del detector y metodología experimental.

## Fase 5 · Evaluación experimental (S20–S24, 8 feb – 14 mar)

El 14 de marzo tienes todas las tablas y gráficas de resultados y congelas el alcance: desde ahí solo se corrigen errores.

### S20 · 8 – 14 feb · Activar los comparadores

- [ ] Activar GuardDuty con S3 Protection. La prueba gratuita dura 30 días, así que cubre hasta aproximadamente el 10 de marzo.
- [ ] WAF: web ACL asociado al user pool de Cognito (los logins no pasan por CloudFront) y otro en CloudFront, con reglas rate-based y managed rules.
- [ ] Cognito threat protection en modo auditoría, si no lo usas ya como fuente de logs.
- [ ] Escenario base solo con tráfico legítimo para medir falsos positivos.

**Hecho cuando:** los cuatro sistemas (tu motor, GuardDuty, WAF y Cognito) están activos y registrando.

### S21 · 15 – 21 feb · Experimentos sobre el conjunto de test

- [ ] Generar el conjunto de test con otra semilla, distinta de la de ajuste.
- [ ] Ejecutar cada escenario de ataque con N repeticiones.
- [ ] Recoger alertas propias, hallazgos de GuardDuty, logs de WAF y eventos de Cognito.

**Hecho cuando:** tienes la matriz de «qué detectó cada sistema» con datos brutos guardados.

### S22 · 22 – 28 feb · Sensibilidad, tiempo real y evasión

- [ ] Barrido de umbrales (p. ej. 5, 10 y 20 intentos) y gráfica de precisión frente a recall.
- [ ] Comparar el tiempo de detección en casi tiempo real frente a un cron horario, procesando los mismos logs por lotes.
- [ ] Ataques evasivos: fuerza bruta lenta, por debajo del umbral, y distribuida desde 2 o 3 orígenes (tu máquina y Lambdas en otras regiones).

**Hecho cuando:** tienes las gráficas de sensibilidad y la tabla de tiempos de detección.

### S23 · 1 – 7 mar · Rendimiento y coste

- [ ] k6: latencia p95 de la API y de la subida, cold starts y concurrencia sostenida.
- [ ] Cost Explorer filtrado por la etiqueta `Project`, por servicio.
- [ ] Coste por sistema de detección: tu motor, GuardDuty, WAF y Cognito Plus.
- [ ] Desactivar GuardDuty antes de que acabe la prueba si ya tienes sus datos.

**Hecho cuando:** todas las casillas «fill in» del README tienen un valor medido.

### S24 · 8 – 14 mar · Análisis y congelación

- [ ] Tablas y gráficas finales; repetir los experimentos que fallaron.
- [ ] README actualizado y tabla Well-Architected con ⚠️ justificados donde toque.
- [ ] Apagar WAF, Cognito Plus y GuardDuty.
- [ ] Congelar el alcance: no se añade código nuevo.

**Hecho cuando:** resultados cerrados y servicios de pago apagados. **Hito de control 3.**

**Memoria:** apartado de resultados con tablas y gráficas.

## Fase 6 · Redacción (S25–S27, 15 mar – 4 abr)

El 4 de abril el tutor recibe el borrador completo. Si escribiste cada fase a su tiempo, aquí solo quedan evaluación, discusión y cierre.

### S25 · 15 – 21 mar · Evaluación y discusión

- [ ] Capítulo de evaluación completo: resultados y discusión.
- [ ] Responder la pregunta de investigación: qué detecta cada sistema, qué se le escapa y por qué.
- [ ] Cada afirmación con su tabla, gráfica o captura.

**Hecho cuando:** el capítulo de evaluación se puede leer de principio a fin.

### S26 · 22 – 28 mar · Cierre (Semana Santa, \~5 h)

- [ ] Limitaciones: tráfico sintético, escala pequeña, una sola región.
- [ ] Trabajo futuro: geo-anomalía, bloqueo automático con WAF, ML, multirregión.
- [ ] Conclusiones contrastadas una a una con los objetivos de S3.

### S27 · 29 mar – 4 abr · Borrador completo

- [ ] Resumen o abstract e introducción final.
- [ ] Bibliografía revisada en el estilo exigido; figuras y tablas numeradas.
- [ ] Enviar el borrador completo al tutor.

**Hecho cuando:** el borrador está enviado. **Hito de control 4.**

## Fase 7 · Correcciones y defensa (S28–S30, 5 – 25 abr)

El 25 de abril entregas la versión final y tienes la defensa ensayada dos veces.

### S28 · 5 – 11 abr · Correcciones

- [ ] Aplicar las correcciones del tutor.
- [ ] Pasada de estilo y comprobación del límite de palabras o páginas.
- [ ] Repo final: README completo, diagramas definitivos y tag `v1.0`.

### S29 · 12 – 18 abr · Demo y presentación

- [ ] Guion de demo de 3 minutos: login, subida, lanzar un ataque y ver llegar la alerta por email en directo.
- [ ] Grabar un vídeo de respaldo de la demo.
- [ ] Diapositivas de la defensa.
- [ ] Respuestas escritas a preguntas típicas: «¿por qué no usar solo GuardDuty?», «¿cómo sabes que tus umbrales son correctos?», «¿qué pasa con 100.000 usuarios?».

### S30 · 19 – 25 abr · Ensayo y entrega

- [ ] Dos ensayos completos de la defensa, uno con alguien que te interrumpa con preguntas.
- [ ] Comprobar el entorno de demo el día antes.
- [ ] Entrega final.
- [ ] Después de la defensa, `terraform destroy` de todo lo que no necesites para el portfolio.

## Hitos de control, recortes y costes

Si un hito no se cumple, recorta en esa misma semana; nunca recortes la evaluación experimental.

| Semana | Debe estar hecho | Si no lo está |
| --- | --- | --- |
| S3 · 18 oct | Fuente de logs de login decidida | Elige la opción (c), el endpoint propio: es la que más controlas |
| S9 · 29 nov | Cinco endpoints con tests y permisos mínimos | Quita Google OAuth y deja solo email |
| S12 · 20 dic | App desplegada con CI/CD | Frontend mínimo y un solo entorno; prod queda como diseño documentado |
| S19 · 7 feb | Detectores, generador y plan de experimentos | Reduce a dos detectores: fuerza bruta y descarga masiva |
| S24 · 14 mar | Experimentos completos | Quita Cognito threat protection de la comparativa; mantén GuardDuty y WAF |
| S27 · 4 abr | Borrador enviado al tutor | Comprime S28–S30 y prepara la defensa en paralelo |

Orden de recorte si vas tarde: Slack, geo-anomalía, multirregión y ML. Todo eso va a trabajo futuro.

Control de costes (cifras aproximadas, compruébalas en la calculadora de AWS):

- Lambda, API Gateway, DynamoDB y S3 cuestan céntimos con tu tráfico, así que puedes dejarlos encendidos.
- La zona de Route 53 cuesta unos 0,50 USD al mes y el dominio se paga aparte, al año.
- WAF (unos 5 USD por web ACL al mes, más cada regla), Cognito Plus y GuardDuty tras la prueba solo se encienden en S20–S24.
- Revisa Cost Explorer cada domingo; son 2 minutos y evitan sustos.
