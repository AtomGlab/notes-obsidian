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
