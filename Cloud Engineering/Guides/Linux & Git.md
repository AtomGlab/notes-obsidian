# Guía completa: Linux y Git para Cloud Engineering

Esta guía cubre exactamente lo que necesitas dominar antes de entrar en AWS, con el nivel de profundidad que realmente se usa en el día a día de cloud/DevOps, ni más ni menos.

-----

# PARTE 1: LINUX

## 1.1 Navegación de filesystem

Lo básico que ya conocerás rápido: `pwd`, `cd`, `ls` (con `-la` para ver ocultos y detalles), `find` para buscar archivos por nombre/tipo/fecha, y `locate` como alternativa más rápida (indexada). La jerarquía que debes tener clara de memoria:

- `/etc` — configuración del sistema y de servicios
- `/var/log` — logs (vivirás aquí constantemente)
- `/home` — directorios de usuario
- `/tmp` — temporal, se vacía en reinicios
- `/usr/local/bin` — binarios instalados manualmente
- `/opt` — software de terceros instalado manualmente

Combo que usarás todo el rato: `find /var/log -name "*.log" -mtime -1` (logs modificados en el último día).

## 1.2 Permisos: chmod y chown

Cada archivo tiene tres categorías (owner, group, others) y tres tipos de permiso (read=4, write=2, execute=1). `chmod 755 archivo` significa: el propietario tiene rwx (7), el grupo r-x (5), otros r-x (5). Los valores que debes memorizar porque aparecen constantemente:

| Octal | Significado | Uso típico                            |
| ----- | ----------- | ------------------------------------- |
| 600   | rw—––       | Archivos privados (claves, secretos)  |
| 644   | rw-r–r–     | Archivos públicos de lectura          |
| 700   | rwx——       | Ejecutables/directorios privados      |
| 755   | rwxr-xr-x   | Directorios públicos y scripts        |
| 750   | rwxr-x—     | Directorios accesibles solo por grupo |
| 400   | r––––       | Secretos solo lectura                 |

Nunca uses 777 en producción — es la forma más rápida de generar un hallazgo de seguridad en cualquier auditoría.

`chmod` también admite modo simbólico: `chmod u+x script.sh` (añade ejecución al owner), `chmod go-w archivo` (quita escritura a group y others). `chown usuario:grupo archivo` cambia propietario y grupo a la vez; con `-R` lo aplicas recursivamente a un directorio completo.

**umask** define los permisos por defecto al crear archivos nuevos. Resta del máximo (666 para archivos, 777 para directorios). `umask 022` (el default en la mayoría de sistemas) produce archivos en 644 y directorios en 755. `umask 077` es el modo estricto (600/700), útil en entornos sensibles.

**Permisos especiales** (los verás menos a menudo pero conviene reconocerlos): SUID (`chmod u+s` o prefijo 4, ej. `4755`) permite que un ejecutable corra con los privilegios de su propietario, no de quien lo ejecuta — es como funciona `passwd`. SGID (`chmod g+s` o prefijo 2) hace que los archivos nuevos creados en un directorio hereden el grupo del directorio, muy útil en carpetas compartidas de equipo. El sticky bit (`chmod +t` o prefijo 1) en un directorio impide que un usuario borre archivos de otro aunque tenga permiso de escritura — así funciona `/tmp`. Si alguna vez ves un binario con SUID que no esperabas, trátalo como posible incidente de seguridad hasta comprobar que es legítimo.

Error clásico que te ahorrarás conociendo esto: permisos de SSH demasiado abiertos. La clave privada debe ir en 600, el directorio `.ssh` en 700, `authorized_keys` en 600 — si no, SSH rechaza la conexión directamente.

## 1.3 Gestión de procesos

Comandos imprescindibles: `ps aux` (todos los procesos con detalle), `top` o `htop` (monitor en vivo de CPU/memoria), `kill <pid>` (termina un proceso, por defecto con SIGTERM), `kill -9 <pid>` (SIGKILL, fuerza terminación cuando el proceso no responde), `pkill nombre` (mata por nombre de proceso en vez de PID).

Diferencia importante entre señales: SIGTERM (15, la default de `kill`) pide al proceso que termine de forma ordenada, dándole oportunidad de cerrar conexiones y guardar estado. SIGKILL (9) lo mata inmediatamente sin darle esa oportunidad — solo úsalo cuando SIGTERM no funciona, porque puede dejar estado a medias (ficheros corruptos, conexiones colgadas).

Procesos en background: `comando &` lo lanza en segundo plano, `jobs` lista los procesos en background de tu sesión, `fg`/`bg` los trae a primer plano o los devuelve a segundo plano, `nohup comando &` lo deja corriendo incluso si cierras la sesión SSH.

## 1.4 Redirecciones y pipes

Esto es el corazón de “pensar en Linux” — encadenar herramientas simples para resolver problemas complejos.

- `>` redirige salida sobreescribiendo el archivo destino
- `>>` redirige salida añadiendo al final sin borrar lo anterior
- `<` redirige entrada desde un archivo
- `2>` redirige solo errores (stderr)
- `&>` redirige tanto salida como errores al mismo destino
- `|` (pipe) conecta la salida de un comando como entrada del siguiente

Ejemplos que usarás de verdad: `comando > /dev/null 2>&1` silencia completamente salida y errores (común en scripts de cron); `cat access.log | grep "ERROR" | wc -l` cuenta cuántas líneas de error hay en un log; `ps aux | grep nginx` busca procesos de nginx en ejecución.

## 1.5 Edición con vim o nano

Para empezar rápido, nano es más amigable (los comandos aparecen en pantalla). Pero en entornos de servidores reales, vim suele ser el único editor disponible, así que conviene tener lo mínimo de vim:

- `i` entra en modo inserción, `Esc` sale de él
- `:wq` guarda y sale, `:q!` sale sin guardar
- `dd` borra una línea, `yy` la copia, `p` la pega
- `/palabra` busca dentro del archivo, `n` salta a la siguiente coincidencia
- `gg` va al inicio del archivo, `G` al final

No necesitas ser experto en vim — necesitas poder abrir un archivo de configuración en un servidor remoto, editar una línea y guardar sin quedarte atascado.

## 1.6 Gestión básica de paquetes (apt)

`apt update` refresca la lista de paquetes disponibles (siempre antes de instalar algo), `apt install paquete`, `apt remove paquete`, `apt list --installed` para ver qué tienes instalado, `apt search palabra` para buscar paquetes. La diferencia entre `apt` (interfaz moderna, recomendada) y `apt-get` (la herramienta de bajo nivel original) no te bloqueará en nada práctico — usa `apt` y listo.

## 1.7 Variables de entorno

`echo $VARIABLE` muestra el valor, `export VARIABLE=valor` la crea o modifica para la sesión actual, `env` o `printenv` lista todas las variables activas. Las más importantes que verás constantemente: `PATH` (dónde busca el sistema los ejecutables cuando escribes un comando), `HOME`, `USER`, `SHELL`.

Para que una variable persista entre sesiones, se añade a `~/.bashrc` (o `~/.bash_profile` según distro) con `export VARIABLE=valor` y luego se recarga con `source ~/.bashrc`. Esto es exactamente lo que harás constantemente con credenciales y configuración de AWS CLI (`AWS_ACCESS_KEY_ID`, `AWS_DEFAULT_REGION`, etc.) y con Terraform.

## 1.8 Logs: tail, grep y journalctl

### tail y grep

`tail archivo.log` muestra las últimas líneas; `tail -f archivo.log` las sigue en tiempo real, viendo nuevas líneas a medida que se escriben — imprescindible cuando estás depurando un servicio mientras lo pruebas. `tail -n 50 archivo.log` muestra las últimas 50 líneas concretamente.

`grep "patrón" archivo` filtra líneas que coinciden; `grep -i` ignora mayúsculas/minúsculas; `grep -r "patrón" /directorio` busca recursivamente en todos los archivos de un directorio; `grep -v "patrón"` invierte la búsqueda (todo lo que NO coincide), muy útil para filtrar ruido.

Combo de oro para depurar servicios en producción: `tail -f /var/log/app.log | grep ERROR`.

### journalctl (logs de systemd)

En sistemas modernos con systemd (la inmensa mayoría de distribuciones Linux que usarás en EC2), los logs de servicios no viven en archivos planos sueltos sino en el *journal*, centralizado y consultado con `journalctl`. Esto es importante porque la forma de depurar un servicio ya no es solo “buscar el archivo de log correcto” sino usar las opciones de filtrado de journalctl:

```
journalctl                          # todos los logs
journalctl -f                       # seguir en tiempo real (equivalente a tail -f)
journalctl -u nombre-servicio       # logs solo de un servicio concreto
journalctl -u nombre-servicio -f    # seguir en tiempo real solo de ese servicio
journalctl --since "1 hour ago"     # filtrar por tiempo
journalctl --since "2026-06-01" --until "2026-06-02"
journalctl -p err                   # solo errores (prioridades: 0=emerg ... 7=debug)
journalctl -b                       # solo del arranque actual
journalctl -b -1                    # del arranque anterior
journalctl -o json                  # salida en JSON, útil para integrarlo con otras herramientas
```

Cuando un servicio falla, el flujo de depuración estándar es: `systemctl status nombre-servicio` (te dice si está activo, fallido, y muestra las últimas líneas de log directamente) y si necesitas más contexto, `journalctl -u nombre-servicio -n 100` para ver las últimas 100 líneas completas.

Un detalle que te puede sorprender: por defecto algunos sistemas guardan el journal solo en memoria volátil (`/run/log/journal`), lo que significa que se pierde al reiniciar. Para logs persistentes entre reinicios, hace falta crear `/var/log/journal` — algo que en un entorno de producción real querrás verificar.

## 1.9 systemd: cómo se comporta un servicio

systemd es el gestor de servicios estándar en casi todas las distribuciones modernas (Ubuntu, Debian, RHEL, Amazon Linux 2+). Lo mínimo que necesitas dominar:

```
systemctl status nombre        # estado actual + últimas líneas de log
systemctl start nombre         # arrancar
systemctl stop nombre          # detener
systemctl restart nombre       # reiniciar
systemctl enable nombre        # que arranque automáticamente al boot
systemctl disable nombre       # que no arranque automáticamente
systemctl list-units --state=failed   # ver qué servicios están fallando en todo el sistema
```

Un servicio se define mediante una *unit file* (típicamente en `/etc/systemd/system/nombre.service`), con secciones como `[Unit]` (descripción, dependencias), `[Service]` (qué ejecutar, con qué usuario, política de reinicio) y `[Install]` (cuándo arrancar automáticamente). Dos errores típicos que conviene reconocer si te aparecen: el código de estado `203/EXEC` significa que systemd no pudo ejecutar el binario indicado en `ExecStart` (ruta equivocada, falta `chmod +x`, o falta el shebang en un script); `217/USER` significa que el usuario definido en `User=` no existe en el sistema.

No necesitas escribir unit files complejos de memoria. Necesitas poder leer una, entender qué comando ejecuta y con qué usuario, y saber el ciclo de depuración: `systemctl status` → si falla, `journalctl -u` para ver el detalle → corregir → `systemctl daemon-reload` (si modificaste el unit file) → `systemctl restart`.

-----

# PARTE 2: GIT

## 2.1 Branching real

Más allá de `git branch nombre` y `git checkout nombre` (o el atajo `git checkout -b nombre` que crea y cambia a la vez, o el más moderno `git switch -c nombre`):

El flujo estándar de trabajo con ramas en cualquier equipo real: nunca trabajas directamente sobre `main`. Creas una rama por feature o fix (`feature/nombre-descriptivo`, `fix/nombre-del-bug`), trabajas ahí, y la integras de vuelta mediante Pull Request, no con un merge directo sin revisión.

Comandos que usarás constantemente: `git branch` (lista ramas locales), `git branch -a` (incluye remotas), `git branch -d nombre` (borra una rama ya fusionada), `git branch -D nombre` (fuerza el borrado aunque tenga cambios sin fusionar).

## 2.2 Merge vs Rebase — la diferencia que de verdad importa

Ambos hacen lo mismo conceptualmente (traer cambios de una rama a otra), pero de forma fundamentalmente distinta:

**Merge** preserva la historia exactamente como ocurrió, incluyendo el momento y orden real de cada commit, y crea un commit de fusión que registra explícitamente cuándo se integró una rama y de dónde viene. Esto importa en entornos donde la trazabilidad histórica es relevante (auditoría, regulación, o simplemente entender “cuándo entró esto exactamente”).

**Rebase** reescribe la historia: toma tus commits y los “reaplica” uno por uno encima de la última versión de la rama destino, como si los hubieras escrito ahí desde el principio. El resultado es una historia lineal, limpia, sin commits de fusión — más legible, pero ya no representa el orden cronológico real en que ocurrieron las cosas.

La regla de oro que evita problemas reales en equipo: **rebase solo en ramas locales/privadas que nadie más ha tocado**, nunca en ramas públicas que otros ya están usando. Si reescribes una rama que alguien más ya tiene descargada localmente, le rompes su historial y le obligas a rehacer su trabajo. Por eso el patrón habitual en equipos es: usa rebase para limpiar tu propia rama de feature antes de abrir el Pull Request (consolidar commits, mejorar mensajes), y usa merge para integrar esa rama ya limpia dentro de `main`.

Sobre conflictos: con merge, resuelves todos los conflictos de una vez, en un único commit de fusión. Con rebase, puedes encontrarte el mismo conflicto repetido varias veces, una por cada commit que se reaplica — más tedioso si la rama lleva mucho tiempo divergida, pero cada conflicto se resuelve en el contexto específico de ese commit, lo que en according a la práctica ayuda a mantener cada commit coherente y funcional por sí mismo.

Flujo práctico de un rebase con conflicto:

```
git checkout mi-rama
git rebase main
# Git se detiene en el primer commit con conflicto
# Editas el archivo conflictivo manualmente
git add archivo-resuelto
git rebase --continue
# Se repite si hay más commits con conflicto
```

Si en cualquier momento quieres abortar y volver al estado anterior: `git rebase --abort`.

**Rebase interactivo** (`git rebase -i HEAD~5`, por ejemplo) te permite reescribir los últimos 5 commits: combinarlos (`squash`), reordenarlos, editar mensajes, o eliminarlos. Esto es exactamente la herramienta para llegar a un Pull Request con commits limpios y bien explicados aunque tu proceso real de trabajo haya sido más caótico (“wip”, “arreglo typo”, “prueba” — los limpias antes de que nadie más los vea).

## 2.3 Buenas prácticas de commits (esto es lo que de verdad miran)

La investigación es consistente en un punto importante: lo que un reclutador o ingeniero senior valora no es un historial estéticamente perfecto, sino **coherencia entre lo que dice el commit y lo que realmente cambia**, y evidencia de que sabes comunicar el “por qué”, no solo el “qué”. Un historial con muchos commits pequeños de “wip” no es un problema en sí — el problema es no limpiar nada antes de mostrarlo, o tener mensajes como “update”, “fix”, “asdf” sin ningún contexto.

Señales positivas que de verdad se buscan en un repo de portfolio:

- Mensajes de commit descriptivos que explican el porqué de un cambio, no solo repetir el nombre del archivo
- Commits atómicos: cada commit hace una sola cosa coherente, no mezcla un fix de bug con un cambio de estilo y una nueva feature a la vez
- Actividad regular sin huecos enormes (no hace falta commitear cada día, pero un patrón de trabajo sostenido se nota)
- README con instrucciones claras de instalación/uso — muchos evaluadores valoran esto incluso más que el propio historial de commits, porque demuestra que piensas en quien va a usar o mantener tu código después
- Tests presentes y pipeline de CI visible (un badge de GitHub Actions en el README ya transmite “esta persona piensa en calidad”, incluso en proyectos pequeños)

Señales negativas que sí penalizan:

- Repos forkeados sin ningún cambio real, presentados como trabajo propio
- Secretos commiteados en el historial (claves de AWS, contraseñas) — esto es además un riesgo de seguridad real, no solo estético
- Código copiado de tutoriales con los comentarios originales todavía puestos
- Proyectos que se presentan como terminados pero claramente están a medias

### Conventional Commits — el estándar a seguir

No es obligatorio, pero es la convención más extendida en la industria y te da una estructura clara sin tener que inventar tu propio sistema. Formato:

```
<tipo>[ámbito opcional]: <descripción>

[cuerpo opcional explicando el porqué]

[footer opcional]
```

Los tipos principales: `feat` (nueva funcionalidad), `fix` (corrección de bug), `docs` (solo documentación), `refactor` (cambio de estructura sin cambiar comportamiento), `test` (añadir o corregir tests), `chore` (tareas de mantenimiento, dependencias), `ci` (cambios en pipelines de integración continua), `style` (formato, espacios, sin cambio de lógica), `perf` (mejora de rendimiento).

Ejemplos reales aplicados a tus propios proyectos de portfolio:

```
feat(infra): add S3 bucket and CloudFront distribution via Terraform
fix(lambda): handle missing query parameter in visitor counter
docs: add architecture diagram and cost breakdown to README
ci: add GitHub Actions workflow for terraform plan on pull request
test(lambda): add unit tests for DynamoDB visitor count increment
refactor(api): extract DynamoDB client into separate module
chore(deps): bump boto3 to 1.34.0
```

Usar el imperativo presente (“add”, no “added” ni “adding”) es la convención estándar — describe lo que el commit *hace* al aplicarse, no lo que hiciste.

Una práctica adicional que conecta directamente con esto: nombrar las ramas siguiendo el mismo esquema de tipo, por ejemplo `feat/visitor-counter-lambda` o `fix/cloudfront-cache-invalidation`. Mantiene coherencia entre el nombre de la rama, los commits que contiene, y el eventual mensaje del Pull Request.

## 2.4 Resolución de conflictos de merge — el flujo práctico

Cuando Git no puede fusionar automáticamente, marca el archivo con marcadores de conflicto:

```
<<<<<<< HEAD
tu versión del código
=======
la versión de la otra rama
>>>>>>> nombre-rama
```

El proceso: abres el archivo, decides qué contenido se queda (puede ser uno, el otro, o una combinación de ambos), borras los marcadores `<<<<<<<`, `=======`, `>>>>>>>`, guardas, y luego `git add archivo` + `git commit` (en un merge) o `git rebase --continue` (en un rebase). Herramientas como VS Code muestran esto de forma visual con botones de “aceptar versión actual / aceptar versión entrante / aceptar ambas”, lo cual simplifica mucho el proceso cuando estás empezando.

-----

# Resumen de prioridad práctica

Si tuvieras que ordenar por “qué se usa más a diario” una vez estés trabajando en los proyectos del portfolio: `cd`/`ls`/`find` y `tail -f`/`grep`/`journalctl -u -f` para depurar son lo que tocarás constantemente; `chmod`/`chown` aparecerán sobre todo al configurar SSH y permisos de scripts; `systemctl status` será tu primer comando reflejo cuando algo no arranque; y en Git, el ciclo `branch → commits atómicos con Conventional Commits → rebase interactivo para limpiar → Pull Request → merge` será literalmente tu flujo de trabajo en cada uno de los tres proyectos del portfolio.



# Ejercicios prácticos: Linux y Git

Cómo usar este documento: haz cada bloque en tu terminal real, sin mirar la guía anterior salvo que te quedes completamente atascado. Las soluciones están todas al final, separadas por sección — resuélvelos primero, revisa después. El objetivo no es acertar a la primera, es notar en qué momento dudas, porque eso es justo lo que necesitas reforzar.

Algunos ejercicios requieren que prepares un escenario antes (crear archivos, un repo, etc.) — esa preparación está incluida en el propio ejercicio.

---

## BLOQUE 1 — Navegación de filesystem

**1.1** Crea esta estructura de directorios en tu home con un solo comando (pista: `mkdir -p`):

```
proyecto/
  src/
  logs/
  config/
```

**1.2** Dentro de `proyecto/logs/`, crea tres archivos vacíos llamados `app.log`, `error.log` y `access.log` con un único comando.

**1.3** Usa `find` para localizar, dentro de `proyecto/`, todos los archivos que terminen en `.log`.

**1.4** Usa `find` para localizar todos los archivos modificados en el último minuto dentro de `proyecto/` (útil para detectar qué se ha tocado recientemente).

**1.5** Desde tu home, navega a `proyecto/src` usando una ruta relativa, y luego vuelve a tu home usando `cd` sin escribir la ruta completa (pista: hay un atajo de un solo carácter).

---

## BLOQUE 2 — Permisos (chmod, chown, umask)

**2.1** Crea un archivo llamado `secreto.txt`. Compruébalo con `ls -l` y dime qué permisos tiene por defecto.

**2.2** Cambia los permisos de `secreto.txt` a que solo tú puedas leerlo y escribirlo, nadie más pueda hacer nada (un solo valor octal).

**2.3** Crea un script `script.sh` con el contenido `echo "hola"`. Intenta ejecutarlo con `./script.sh`. ¿Qué error obtienes? Arréglalo con chmod en modo simbólico (no octal).

**2.4** ¿Qué comando usarías para comprobar tu umask actual? Cámbialo temporalmente a `077` y crea un archivo nuevo. Comprueba con `ls -l` qué permisos tiene.

**2.5** Sin ejecutarlo, dime qué hace exactamente este comando y por qué es peligroso si se usa en el sitio equivocado:

```
chmod -R 777 /
```

---

## BLOQUE 3 — Gestión de procesos

**3.1** Lanza el comando `sleep 300` en segundo plano (background).

**3.2** Usa `ps aux` junto con `grep` para encontrar el PID de ese proceso `sleep` que acabas de lanzar (sin usar `jobs`).

**3.3** Termina ese proceso usando `kill` de forma "educada" (sin forzar).

**3.4** Lanza otro `sleep 300 &`, y esta vez mátalo de forma forzada porque imagina que no responde al intento anterior. ¿Qué flag/número de señal usas?

**3.5** Explica con tus palabras la diferencia entre lo que ha pasado en 3.3 y en 3.4 a nivel del proceso, no solo del comando.

---

## BLOQUE 4 — Redirecciones y pipes

**4.1** Genera un archivo `numeros.txt` con los números del 1 al 20, cada uno en una línea (pista: `seq`).

**4.2** Usando pipes, cuenta cuántas líneas de `numeros.txt` contienen un dígito par sin escribir un script, solo encadenando comandos (pista: `grep -E` con un patrón de números pares, y `wc -l`).

**4.3** Ejecuta un comando que falle a propósito (por ejemplo `ls /carpeta/que/no/existe`) y redirige SOLO el error a un archivo llamado `errores.txt`, dejando que si hubiera salida normal, se siga viendo en pantalla.

**4.4** Ahora redirige tanto la salida normal como los errores de ese mismo comando fallido a un único archivo `todo.txt`, sin que se muestre nada en pantalla.

**4.5** Crea un archivo `lista.txt` con varias líneas, algunas repetidas. Encadena comandos para mostrar solo las líneas únicas, ordenadas (pista: `sort` y `uniq`).

---

## BLOQUE 5 — Edición con vim

Haz estos pasos en orden, dentro del propio editor, sin salir entre paso y paso salvo que se indique:

**5.1** Abre con vim un archivo nuevo llamado `notas.txt`.

**5.2** Entra en modo inserción y escribe tres líneas: "primera", "segunda", "tercera".

**5.3** Sin salir de vim, guarda el archivo pero NO cierres el editor.

**5.4** Borra la línea "segunda" usando el comando de borrado de línea de vim.

**5.5** Pega de vuelta esa línea borrada justo donde estaba.

**5.6** Guarda y cierra en un solo comando.

**5.7** Abre de nuevo el archivo, ve directamente a la última línea con un solo comando de vim, y luego al principio del archivo con otro.

---

## BLOQUE 6 — Gestión de paquetes (apt)

**6.1** ¿Qué comando ejecutarías siempre antes de instalar cualquier paquete nuevo, y por qué es importante no saltárselo?

**6.2** Comprueba si tienes instalado el paquete `tree`. Si no lo tienes, instálalo.

**6.3** Usa `tree` para ver la estructura de tu carpeta `proyecto/` del bloque 1.

**6.4** Lista todos los paquetes que tienes instalados y cuenta cuántos son con un pipe (sin contarlos a mano).

---

## BLOQUE 7 — Variables de entorno

**7.1** Muestra el valor actual de tu variable `PATH`.

**7.2** Crea una variable de entorno llamada `MI_PROYECTO` con el valor `cloud-portfolio`, solo para la sesión actual.

**7.3** Comprueba que existe usando `echo`.

**7.4** Abre una nueva pestaña/sesión de terminal y comprueba si `MI_PROYECTO` sigue existiendo ahí. ¿Por qué pasa esto?

**7.5** Haz que `MI_PROYECTO` persista en todas tus sesiones futuras (pista: tienes que tocar tu archivo de configuración del shell, `.bashrc` o `.zshrc` según cuál uses, y luego recargarlo sin cerrar la terminal).

---

## BLOQUE 8 — Logs: tail, grep

**8.1** Genera un archivo de log simulado con 100 líneas, donde unas 10 contengan la palabra "ERROR" en posiciones aleatorias del archivo (puedes hacerlo manualmente o con un bucle simple, tú decides el método).

**8.2** Muestra solo las últimas 15 líneas de ese archivo.

**8.3** Cuenta cuántas líneas contienen "ERROR".

**8.4** Muestra las líneas que contienen "ERROR" pero NO la palabra "critical" (asumiendo que añades alguna línea con ambas palabras para hacer la prueba real).

**8.5** Abre dos terminales. En la primera, deja corriendo `tail -f` sobre el archivo. En la segunda, añade una línea nueva al archivo con `echo "nueva linea ERROR" >> archivo.log`. ¿Qué ves en la primera terminal?

---

## BLOQUE 9 — systemd y journalctl

Nota: para hacer esto de verdad necesitas un servicio systemd real corriendo. Si tu Mac no usa systemd nativamente (macOS no lo usa, es exclusivo de Linux), haz este bloque conceptualmente respondiendo con precisión, y repítelo de forma práctica en cuanto tengas tu primera EC2 — es importante que no te lo saltes del todo.

**9.1** ¿Qué comando usarías para ver el estado actual de un servicio llamado `nginx`?

**9.2** Ese comando te dice que el servicio está en estado "failed". ¿Cuál es tu siguiente comando para investigar por qué, y qué información concreta esperas encontrar ahí?

**9.3** Quieres ver únicamente los logs de ese servicio de la última hora. Escribe el comando exacto.

**9.4** Quieres seguir los logs de ese servicio en tiempo real mientras pruebas algo. Escribe el comando.

**9.5** Has corregido el problema y necesitas reiniciar el servicio. ¿Qué comando usas, y si además modificaste el propio archivo de unit (`.service`), qué comando adicional necesitas ejecutar antes de reiniciar?

**9.6** Un compañero te dice "el servicio me da código de salida 203/EXEC". Sin buscarlo, explica con tus palabras qué tres causas típicas podrían estar detrás.

---

## BLOQUE 10 — Escenario integrador de Linux (mezcla todo lo anterior)

Imagina esta situación real: tienes una aplicación corriendo como servicio systemd llamado `miapp`. Un usuario te dice que la aplicación "no responde". Tienes acceso SSH al servidor.

Describe, comando por comando, exactamente qué harías desde que te conectas hasta que identificas la causa probable. Da por hecho que en algún punto vas a necesitar mirar permisos de algún archivo de configuración, y que en algún punto vas a necesitar matar un proceso colgado que no responde a una señal normal.

---

# GIT

## BLOQUE 11 — Branching básico

**11.1** Crea un repositorio nuevo llamado `practica-git`, entra en él, y haz un primer commit con un archivo `README.md` que contenga solo el título del proyecto.

**11.2** Crea una rama llamada `feature/login` y cámbiate a ella en un solo comando.

**11.3** Dentro de esa rama, crea un archivo `login.txt` con el contenido "pantalla de login" y commitea con un mensaje en formato Conventional Commits.

**11.4** Vuelve a `main` y comprueba que `login.txt` no existe ahí (porque sigue solo en la rama feature).

**11.5** Lista todas las ramas que tienes, marcando cuál es la actual.

---

## BLOQUE 12 — Conflictos de merge (provócalo tú mismo)

**12.1** En `main`, edita `README.md` añadiendo la línea "Versión 1.0" y commitea.

**12.2** Cámbiate a `feature/login` y edita TÚ TAMBIÉN `README.md`, pero añadiendo en el mismo sitio la línea "Versión beta" en vez de "Versión 1.0". Commitea.

**12.3** Vuelve a `main` e intenta hacer `git merge feature/login`. Deberías obtener un conflicto. Pégame (o anota para ti) exactamente qué aspecto tienen los marcadores de conflicto en el archivo.

**12.4** Resuelve el conflicto decidiendo quedarte con ambas líneas, una debajo de la otra. Termina el merge correctamente.

**12.5** Comprueba con `git log --oneline --graph` cómo quedó el historial. ¿Ves el commit de merge?

---

## BLOQUE 13 — Rebase y su diferencia práctica con merge

**13.1** Crea una nueva rama `feature/registro` desde `main`. Haz dos commits separados en ella: uno que añade `registro.txt` con "formulario de registro", y otro que añade una segunda línea a ese mismo archivo "validación de email".

**13.2** Mientras tanto, en `main`, haz un commit que añada una línea nueva a `README.md` ("Versión 2.0").

**13.3** Desde `feature/registro`, haz `git rebase main`. Observa qué pasa con tus commits (pista: usa `git log --oneline` antes y después para comparar).

**13.4** Ahora deshaz mentalmente lo anterior (o créate una rama nueva idéntica desde cero) y en vez de rebase, haz `git merge main` desde `feature/registro`. Compara el resultado de `git log --oneline --graph` entre ambos métodos.

**13.5** Explica con tus propias palabras: si ambas ramas (`feature/registro` original y esta copia) llegaran al mismo resultado final de archivos, ¿qué diferencia exacta hay en el historial entre haber usado rebase o merge?

**13.6** Provócate un conflicto durante un rebase (edita la misma línea de un archivo en ambas ramas antes de rebasar) y practica el flujo completo: resolver, `git add`, `git rebase --continue`.

---

## BLOQUE 14 — Rebase interactivo (limpieza de commits)

**14.1** En una rama nueva, haz 4 commits "sucios" a propósito: "wip", "arreglo", "wip de nuevo", "ya funciona", cada uno tocando el mismo archivo con cambios mínimos.

**14.2** Usa `git rebase -i HEAD~4` y combina (squash) los 4 commits en uno solo, con un mensaje final limpio en formato Conventional Commits.

**14.3** Comprueba con `git log --oneline` que ahora solo hay un commit nuevo en vez de 4.

---

## BLOQUE 15 — Conventional Commits aplicados

Sin mirar la guía anterior, escribe el mensaje de commit correcto en formato Conventional Commits para cada una de estas situaciones:

**15.1** Has añadido un endpoint nuevo a una API que gestiona usuarios.

**15.2** Has corregido un bug donde la API devolvía error 500 si el campo email venía vacío.

**15.3** Has actualizado solo el README explicando cómo desplegar el proyecto.

**15.4** Has subido la versión de una dependencia (por ejemplo boto3) sin cambiar nada de tu código.

**15.5** Has reorganizado el código de un módulo de autenticación moviéndolo a otro archivo, sin cambiar su comportamiento.

**15.6** Has añadido un pipeline de GitHub Actions que corre los tests automáticamente.

**15.7** Has hecho un cambio que rompe la compatibilidad con la versión anterior de tu API (cambias el formato de respuesta de XML a JSON).

---

## BLOQUE 16 — Escenario integrador de Git (mezcla todo lo anterior)

Estás trabajando en tu proyecto de portfolio. Llevas tres días en una rama `feature/contador-visitas` con 12 commits, varios de ellos con mensajes como "prueba", "arreglo de nuevo", "funciona ahora sí". Mientras tú trabajabas, en `main` se han añadido cambios al README por otra razón (imagina que los añadiste tú mismo simulando ser "otra persona", para tener algo real con lo que practicar).

Describe paso a paso qué harías para: limpiar tu historial de los 12 commits dejándolo en 3-4 commits con mensajes en Conventional Commits que tengan sentido, actualizar tu rama con los cambios que hay en `main` sin perder tu trabajo, resolver cualquier conflicto que surja, y finalmente dejarlo listo para abrir un Pull Request con una historia legible.

---

---

# SOLUCIONES

## Bloque 1 — Navegación de filesystem

**1.1**

```
mkdir -p proyecto/{src,logs,config}
```

El flag `-p` crea también los directorios padre si no existen, y la sintaxis `{src,logs,config}` expande a tres rutas a la vez.

**1.2**

```
touch proyecto/logs/{app.log,error.log,access.log}
```

**1.3**

```
find proyecto/ -name "*.log"
```

**1.4**

```
find proyecto/ -mmin -1
```

`-mmin -1` significa "modificado hace menos de 1 minuto". `-mtime -1` sería "menos de 1 día".

**1.5**

```
cd proyecto/src
cd ~
```

El `~` (tilde) es el atajo universal a tu home, sin importar dónde estés.

---

## Bloque 2 — Permisos

**2.1** Por defecto, en la mayoría de sistemas con umask 022, un archivo nuevo creado con `touch` sale en `644` (rw-r--r--).

**2.2**

```
chmod 600 secreto.txt
```

**2.3** El error es `Permission denied`, porque al crear el script no tiene permiso de ejecución (sale en 644, sin la `x`). Se arregla con:

```
chmod u+x script.sh
```

(modo simbólico: añade ejecución solo al usuario propietario)

**2.4**

```
umask
umask 077
touch archivo_estricto.txt
ls -l archivo_estricto.txt
```

Con umask 077 el archivo nuevo sale en `600` en vez del `644` habitual.

**2.5** Este comando cambiaría recursivamente TODOS los archivos y directorios del sistema (empezando en la raíz `/`) a permisos 777, es decir, lectura/escritura/ejecución para absolutamente cualquier usuario. Es catastrófico porque rompe la seguridad de todo el sistema (cualquiera puede modificar binarios del sistema, archivos de configuración, claves) y además probablemente deja el sistema inestable o inutilizable, ya que muchos programas (especialmente los relacionados con SSH) rechazan funcionar si detectan permisos demasiado abiertos en archivos sensibles.

---

## Bloque 3 — Procesos

**3.1**

```
sleep 300 &
```

**3.2**

```
ps aux | grep sleep
```

(el PID aparece en la segunda columna de la salida)

**3.3**

```
kill <PID>
```

Por defecto manda SIGTERM (señal 15), pidiendo al proceso que termine de forma ordenada.

**3.4**

```
kill -9 <PID>
```

o equivalentemente `kill -SIGKILL <PID>`. La señal 9 es SIGKILL, fuerza la terminación inmediata sin dar oportunidad al proceso de limpiar nada.

**3.5** En 3.3, el proceso recibe la petición de cerrarse y, si está bien programado, puede ejecutar lógica de cierre (liberar recursos, cerrar conexiones, guardar estado) antes de terminar realmente. En 3.4, el kernel termina el proceso de forma inmediata sin pasar ningún aviso al propio proceso — no hay oportunidad de limpieza, lo cual puede dejar archivos a medio escribir o conexiones de red colgadas. `sleep` no tiene estado que cuidar, así que en este ejemplo concreto no notarás diferencia visible, pero en una aplicación real (por ejemplo una Lambda o un servidor web) la diferencia es crítica.

---

## Bloque 4 — Redirecciones y pipes

**4.1**

```
seq 1 20 > numeros.txt
```

**4.2**

```
grep -E "[0-9]*[02468]$" numeros.txt | wc -l
```

(el patrón busca números que terminan en cifra par)

**4.3**

```
ls /carpeta/que/no/existe 2> errores.txt
```

**4.4**

```
ls /carpeta/que/no/existe &> todo.txt
```

o de forma equivalente y más explícita: `ls /carpeta/que/no/existe > todo.txt 2>&1`

**4.5**

```
sort lista.txt | uniq
```

---

## Bloque 5 — vim

**5.1** `vim notas.txt`

**5.2** Pulsas `i` para entrar en modo inserción, escribes las tres líneas con Enter entre cada una, y pulsas `Esc` para volver a modo normal.

**5.3** `:w` (guarda sin salir)

**5.4** Con el cursor en la línea "segunda", pulsas `dd`

**5.5** Mueves el cursor a donde estaba esa línea y pulsas `p` (pega después de la línea actual) o `P` (pega antes)

**5.6** `:wq`

**5.7** Para ir al final: `G` (mayúscula). Para ir al principio: `gg`

---

## Bloque 6 — apt

**6.1** Siempre `sudo apt update` antes de instalar nada. Es importante porque refresca el índice local de paquetes disponibles y sus versiones; si no lo haces, `apt` puede intentar instalar una versión que ya no existe en los repositorios o perderte actualizaciones de seguridad recientes.

**6.2**

```
apt list --installed | grep tree
sudo apt install tree
```

**6.3**

```
tree proyecto/
```

**6.4**

```
apt list --installed | wc -l
```

---

## Bloque 7 — Variables de entorno

**7.1** `echo $PATH`

**7.2** `export MI_PROYECTO=cloud-portfolio`

**7.3** `echo $MI_PROYECTO`

**7.4** No existirá en la nueva sesión. `export` solo afecta a la sesión de shell actual y a los procesos hijos que lance desde ahí — no se propaga a otras sesiones ni persiste tras cerrar la terminal, porque vive solo en memoria del proceso del shell.

**7.5**

```
echo 'export MI_PROYECTO=cloud-portfolio' >> ~/.bashrc
source ~/.bashrc
```

(o `~/.zshrc` si usas zsh, que es el shell por defecto en Mac desde hace varias versiones)

---

## Bloque 8 — tail, grep

**8.1** Una forma simple:

```
for i in $(seq 1 100); do
  if [ $((RANDOM % 10)) -eq 0 ]; then
    echo "linea $i ERROR algo fallo"
  else
    echo "linea $i todo normal"
  fi
done > simulado.log
```

**8.2**

```
tail -n 15 simulado.log
```

**8.3**

```
grep -c "ERROR" simulado.log
```

(o `grep "ERROR" simulado.log | wc -l`, equivalente)

**8.4**

```
grep "ERROR" simulado.log | grep -v "critical"
```

**8.5** En la primera terminal verás aparecer en tiempo real la nueva línea justo cuando la añades desde la segunda terminal, sin necesidad de volver a ejecutar el comando. Esto es exactamente el comportamiento que usarás constantemente para depurar una aplicación mientras la pruebas en otra ventana.

---

## Bloque 9 — systemd / journalctl

**9.1** `systemctl status nginx`

**9.2** `journalctl -u nginx -n 50` (o sin `-n`, para ver todo). Esperas encontrar el motivo concreto del fallo: un error de configuración, un puerto ya en uso, un permiso denegado, o un código de salida específico del proceso.

**9.3** `journalctl -u nginx --since "1 hour ago"`

**9.4** `journalctl -u nginx -f`

**9.5** `systemctl restart nginx`. Si modificaste el archivo `.service`, primero necesitas `sudo systemctl daemon-reload` para que systemd relea la definición del servicio, y solo entonces `systemctl restart nginx`.

**9.6** Las tres causas típicas: la ruta del binario indicada en `ExecStart` está mal escrita o no existe; el archivo no tiene permiso de ejecución (falta `chmod +x`); o si es un script, le falta la línea shebang al principio (`#!/bin/bash`, por ejemplo) y el sistema no sabe con qué intérprete ejecutarlo.

---

## Bloque 10 — Escenario integrador Linux

Un flujo razonable y completo sería:

1. `ssh usuario@servidor` para conectar.
2. `systemctl status miapp` — primer comando reflejo, te dice de inmediato si el servicio está activo, parado, o en estado failed, y muestra ya las últimas líneas de log directamente en la salida.
3. Si dice "failed" o "inactive", `journalctl -u miapp -n 100` para ver el contexto completo de qué pasó antes de caer.
4. Si en los logs aparece algo como "permission denied" al intentar leer un archivo de configuración, revisas con `ls -l /ruta/del/archivo` los permisos, y los corriges con `chmod`/`chown` según corresponda.
5. Si en cambio los logs sugieren que el proceso sigue "vivo" pero colgado sin responder (por ejemplo, lleva horas en el mismo punto sin avanzar), localizas el proceso con `ps aux | grep miapp`, intentas primero `kill <PID>` (SIGTERM, educado), esperas unos segundos y compruebas con `ps aux` si ha terminado.
6. Si sigue ahí después de SIGTERM, usas `kill -9 <PID>` para forzar la terminación.
7. Una vez limpio, `systemctl restart miapp` (con `daemon-reload` antes si tocaste el unit file).
8. Verificas con `systemctl status miapp` que ahora está activo, y con `journalctl -u miapp -f` sigues los logs en tiempo real un rato para confirmar que se comporta bien antes de darlo por resuelto.

---

## Bloque 11 — Branching básico

**11.1**

```
mkdir practica-git && cd practica-git
git init
echo "# Práctica Git" > README.md
git add README.md
git commit -m "docs: initial commit with project title"
```

**11.2**

```
git checkout -b feature/login
```

(o `git switch -c feature/login`, sintaxis más moderna)

**11.3**

```
echo "pantalla de login" > login.txt
git add login.txt
git commit -m "feat(login): add login screen placeholder"
```

**11.4**

```
git checkout main
ls
```

`login.txt` no aparecerá, porque solo existe en el historial de `feature/login`.

**11.5**

```
git branch
```

La rama actual aparece marcada con un asterisco `*` delante.

---

## Bloque 12 — Conflictos de merge

**12.1**

```
git checkout main
echo "Versión 1.0" >> README.md
git add README.md
git commit -m "docs: add version 1.0 note"
```

**12.2**

```
git checkout feature/login
echo "Versión beta" >> README.md
git add README.md
git commit -m "docs: add beta version note"
```

**12.3**

```
git checkout main
git merge feature/login
```

El archivo `README.md` mostrará algo como:

```
<<<<<<< HEAD
Versión 1.0
=======
Versión beta
>>>>>>> feature/login
```

**12.4** Editas el archivo manualmente dejando ambas líneas (borrando los marcadores `<<<<<<<`, `=======`, `>>>>>>>`):

```
Versión 1.0
Versión beta
```

Luego:

```
git add README.md
git commit -m "merge: resolve version conflict keeping both notes"
```

**12.5**

```
git log --oneline --graph
```

Sí, verás un commit de merge explícito que junta ambas ramas — esto es justo la diferencia clave con rebase, que no genera este commit.

---

## Bloque 13 — Rebase vs merge

**13.1**

```
git checkout main
git checkout -b feature/registro
echo "formulario de registro" > registro.txt
git add registro.txt
git commit -m "feat(registro): add registration form placeholder"
echo "validación de email" >> registro.txt
git add registro.txt
git commit -m "feat(registro): add email validation note"
```

**13.2**

```
git checkout main
echo "Versión 2.0" >> README.md
git add README.md
git commit -m "docs: add version 2.0 note"
```

**13.3**

```
git checkout feature/registro
git log --oneline
git rebase main
git log --oneline
```

Verás que tus dos commits de `feature/registro` ahora aparecen "después" del commit de versión 2.0 de `main` en la historia, como si los hubieras escrito a partir de ahí desde el principio. Los hashes de tus commits habrán cambiado, porque rebase los reescribe.

**13.4**

```
git checkout main
git checkout -b feature/registro-merge registro.txt  # o recrea la rama igual que en 13.1 desde antes de rebasar
git merge main
git log --oneline --graph
```

Comparado con el resultado de rebase, aquí verás un commit de merge explícito y las ramas siguen siendo visualmente distinguibles en el `--graph` hasta el punto de unión.

**13.5** Con rebase, la historia final es lineal: parece que escribiste tus commits directamente encima de los últimos cambios de `main`, sin rastro de que hubo divergencia real. Con merge, la historia preserva exactamente cómo ocurrió: que hubo dos líneas de trabajo en paralelo y en qué momento exacto se unieron, mediante un commit de merge explícito. El contenido final de los archivos puede ser idéntico en ambos casos — lo que cambia es si quieres que el historial cuente la verdad cronológica completa, o una versión simplificada y lineal más fácil de leer.

**13.6** Provocar el conflicto: edita la misma línea de `README.md` de forma distinta en `main` y en tu rama antes de rebasar, luego:

```
git rebase main
# Git se detiene y avisa del conflicto
# Edita el archivo a mano resolviendo el conflicto
git add README.md
git rebase --continue
```

Si el rebase tiene más de un commit con el mismo conflicto repetido, este ciclo se repite tantas veces como conflictos surjan.

---

## Bloque 14 — Rebase interactivo

**14.1**

```
echo "intento 1" > archivo.txt && git add archivo.txt && git commit -m "wip"
echo "intento 2" >> archivo.txt && git add archivo.txt && git commit -m "arreglo"
echo "intento 3" >> archivo.txt && git add archivo.txt && git commit -m "wip de nuevo"
echo "intento 4" >> archivo.txt && git add archivo.txt && git commit -m "ya funciona"
```

**14.2**

```
git rebase -i HEAD~4
```

Se abre un editor con los 4 commits listados, cada uno precedido de `pick`. Cambias los tres últimos de `pick` a `squash` (o su abreviatura `s`), dejando el primero como `pick`:

```
pick a1b2c3 wip
squash d4e5f6 arreglo
squash g7h8i9 wip de nuevo
squash j1k2l3 ya funciona
```

Al guardar y cerrar, Git te pide escribir el mensaje final combinado — borras todo y escribes algo como:

```
feat(archivo): add complete content in single coherent commit
```

**14.3**

```
git log --oneline
```

Deberías ver un único commit nuevo con el mensaje limpio, en vez de los 4 originales.

---

## Bloque 15 — Conventional Commits

**15.1** `feat(users): add endpoint to retrieve user list`

**15.2** `fix(api): handle empty email field to prevent 500 error`

**15.3** `docs: add deployment instructions to README`

**15.4** `chore(deps): bump boto3 to latest version`

**15.5** `refactor(auth): move authentication logic to separate module`

**15.6** `ci: add GitHub Actions workflow to run tests automatically`

**15.7** `feat(api)!: change response format from XML to JSON` (el `!` señala explícitamente un breaking change; también es válido añadir en el cuerpo del commit una línea `BREAKING CHANGE: response format changed from XML to JSON`)

---

## Bloque 16 — Escenario integrador Git

Un flujo razonable:

1. Primero, asegurarte de que tu rama está actualizada localmente con los últimos cambios remotos de ambas ramas: `git fetch origin`.
2. Limpiar tu propio historial antes de mezclar nada de fuera: `git rebase -i HEAD~12` desde tu rama `feature/contador-visitas`, marcando los commits relevantes como `pick` y combinando (`squash`) los de prueba/arreglo en los commits a los que pertenecen lógicamente, hasta dejar 3-4 commits con sentido, reescribiendo cada mensaje en formato Conventional Commits (por ejemplo: `feat(contador): add DynamoDB table for visit count`, `feat(contador): add Lambda to increment and return count`, `test(contador): add unit tests for counter Lambda`, `docs: document counter architecture in README`).
3. Una vez tu propia rama está limpia, traer los cambios de `main`: `git checkout main && git pull origin main` para asegurarte de tener la versión más reciente.
4. Volver a tu rama y traer esos cambios sin perder tu trabajo: `git checkout feature/contador-visitas` y `git rebase main` (preferible a merge aquí, ya que tu rama sigue siendo privada/local y nadie más depende de ella, así que mantener historia lineal es seguro).
5. Si surge conflicto (por ejemplo, ambos tocasteis el README), Git se detiene en el commit conflictivo: abres el archivo, resuelves manualmente decidiendo qué contenido se queda, `git add README.md`, `git rebase --continue`. Repetir si hay más de un commit con conflicto.
6. Una vez completado el rebase sin errores, comprobar con `git log --oneline --graph` que la historia queda lineal y legible: tus 3-4 commits limpios, encima de los últimos cambios de `main`.
7. Subir la rama (con `--force-with-lease` en vez de `--force` a secas, porque has reescrito commits que ya habías subido antes con los mensajes sucios): `git push --force-with-lease origin feature/contador-visitas`.
8. Abrir el Pull Request: ahora la lista de commits que verá cualquier revisor es coherente, legible, y cuenta una historia clara de cómo se construyó la funcionalidad, en vez de mostrar el proceso real lleno de intentos fallidos.