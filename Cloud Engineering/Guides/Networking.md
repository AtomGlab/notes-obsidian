
# Guía de Redes: fundamentos generales + AWS Networking para el SAA

Esta guía tiene dos partes intencionalmente separadas. La primera es networking general — lo que sostiene conceptualmente todo lo demás, independiente de AWS. La segunda es networking específico de AWS — lo que realmente se examina en el SAA-C03, y que en la práctica es donde más nota se pierde según coinciden varias fuentes de preparación al examen.

No necesitas convertirte en ingeniero de redes. Necesitas el nivel de un Solutions Architect: entender por dónde fluye el tráfico, por qué, y qué pieza falta cuando algo no conecta.

-----

# PARTE 1: FUNDAMENTOS GENERALES DE REDES

## 1.1 El modelo OSI (7 capas) y por qué te lo vas a encontrar todo el rato

El modelo OSI es un marco conceptual de 7 capas que describe cómo viaja la información en una red. No es lo que realmente “corre” en internet (eso es TCP/IP, más abajo), pero es el vocabulario que usa todo el mundo en networking — vas a escuchar constantemente frases como “esto es un problema de capa 3” o “eso se resuelve en capa 7”, y necesitas saber traducir eso a algo concreto.

|Capa|Nombre         |Qué hace                                 |Ejemplo        |
|----|---------------|-----------------------------------------|---------------|
|7   |Aplicación     |Protocolos que usan las apps directamente|HTTP, DNS, SMTP|
|6   |Presentación   |Formato, cifrado                         |TLS/SSL        |
|5   |Sesión         |Gestión de sesiones                      |—              |
|4   |Transporte     |Entrega fiable o rápida                  |TCP, UDP       |
|3   |Red            |Direccionamiento y enrutamiento          |IP, routers    |
|2   |Enlace de datos|Comunicación dentro de la misma red      |MAC, switches  |
|1   |Física         |Cables, señales, wifi                    |—              |

El truco mental que de verdad importa: cuando algo no funciona, pregúntate en qué capa está el problema. ¿No hay cable/wifi? Capa 1. ¿El switch no encuentra el dispositivo? Capa 2. ¿No hay ruta hacia esa IP? Capa 3. ¿El puerto está cerrado o la conexión se corta? Capa 4. ¿La web da error 500 pero la conexión sí llega? Capa 7. Esto es exactamente el mismo razonamiento que usarás depurando Security Groups, NACLs y Load Balancers en AWS.

## 1.2 TCP/IP — el modelo que realmente usa internet

El modelo OSI es teórico; TCP/IP es el que de verdad corre por debajo de todo. Tiene 4 capas (a veces se describe con 5, separando enlace y física, pero la versión de 4 es la más citada): Aplicación, Transporte, Internet, Acceso a la red. La capa de Aplicación de TCP/IP fusiona las capas 5, 6 y 7 de OSI en una sola — por eso cuando hablas de HTTP o DNS, técnicamente estás en “capa de aplicación” tanto en OSI (capa 7) como en TCP/IP, aunque los modelos no se correspondan exactamente.

## 1.3 TCP vs UDP — la decisión que aparece constantemente en arquitectura

**TCP** (Transmission Control Protocol) garantiza que los datos llegan completos, en orden, y reenvía lo que se pierde. Es más lento por toda esa verificación, pero fiable. Se usa para navegación web, email, transferencia de archivos — cualquier cosa donde perder un trozo de datos es inaceptable.

**UDP** (User Datagram Protocol) no garantiza nada de eso: envía y ya, sin esperar confirmación. Es más rápido precisamente porque no tiene esa sobrecarga. Se usa para streaming de vídeo, llamadas VoIP, gaming online — donde llegar rápido importa más que llegar perfecto (preferible perder un frame de vídeo a tener un segundo de retraso esperando que se retransmita).

Por qué te importa en AWS: vas a tomar decisiones de Load Balancer basadas exactamente en esto — un Application Load Balancer trabaja en capa 7 (HTTP/HTTPS, sobre TCP), mientras que un Network Load Balancer trabaja en capa 4 y puede manejar tanto TCP como UDP cuando necesitas rendimiento extremo o protocolos que no son HTTP.

## 1.4 Direcciones IP, máscaras de subred y CIDR

Una dirección IPv4 son 32 bits, escritos como cuatro números separados por puntos (ej. 192.168.1.10), donde cada número va de 0 a 255. La máscara de subred determina qué parte de esa dirección identifica la “red” y qué parte identifica el “host” dentro de esa red.

**CIDR** (Classless Inter-Domain Routing) es la notación moderna: en vez de escribir la máscara completa (255.255.255.0), escribes solo cuántos bits son de red, con una barra: `/24`. Esto significa “los primeros 24 bits son de red, los últimos 8 son de host”.

### Tabla de referencia que conviene memorizar

|CIDR|Máscara        |Direcciones totales|Hosts usables|
|----|---------------|-------------------|-------------|
|/24 |255.255.255.0  |256                |254          |
|/25 |255.255.255.128|128                |126          |
|/26 |255.255.255.192|64                 |62           |
|/27 |255.255.255.224|32                 |30           |
|/28 |255.255.255.240|16                 |14           |
|/29 |255.255.255.248|8                  |6            |
|/30 |255.255.255.252|4                  |2            |
|/16 |255.255.0.0    |65.536             |65.534       |

La fórmula detrás: con *n* bits de host, tienes 2^n direcciones totales, y 2^n - 2 usables (se restan siempre 2: la dirección de red, que identifica la subred en sí, y la dirección de broadcast, que identifica “todos los hosts de esa subred”). Excepción: un /31 (RFC 3021) no reserva ninguna de las dos, usado solo en enlaces punto a punto entre routers.

### El truco para calcular subnetting sin binario

No necesitas hacer binario a mano cada vez. El truco práctico: fíjate en el “octeto especial” — el primer grupo de números que no es 255 en la máscara. Resta ese valor de 256, y obtienes el “salto” entre subredes consecutivas.

Ejemplo: tienes `192.168.1.0/26`. La máscara /26 es `255.255.255.192`. El octeto especial es 192. 256 - 192 = 64. Eso significa que las subredes empiezan en múltiplos de 64: `192.168.1.0`, `192.168.1.64`, `192.168.1.128`, `192.168.1.192`. Cada una tiene 64 direcciones totales (62 usables).

Otro ejemplo real de AWS: tu VPC es `10.0.0.0/16` (65.536 direcciones). Quieres crear subredes /24 dentro de ella. El salto entre subredes /24 es 256 (256 - 0, ya que el tercer octeto pasa a ser parte de la red). Así que tus subredes serían `10.0.0.0/24`, `10.0.1.0/24`, `10.0.2.0/24`, etc. — cada una con 256 direcciones (254 usables en redes normales, pero recuerda que en AWS son menos, ver más abajo).

### Rangos privados (RFC 1918) — los que usarás siempre en VPC

- `10.0.0.0/8` (10.0.0.0 – 10.255.255.255)
- `172.16.0.0/12` (172.16.0.0 – 172.31.255.255)
- `192.168.0.0/16` (192.168.0.0 – 192.168.255.255)

Estas direcciones nunca son enrutables en internet público — por eso son las que usas dentro de tu VPC, y necesitas NAT para que cualquier instancia que las tenga pueda salir a internet.

## 1.5 DNS — el “directorio telefónico” de internet

DNS traduce nombres de dominio (`midominio.com`) en direcciones IP, porque a los humanos nos cuesta recordar números y a las máquinas les da igual el nombre. El proceso (simplificado): tu dispositivo pregunta a un resolver DNS, que si no tiene la respuesta en caché, pregunta a los servidores raíz, luego a los servidores del TLD (.com, .es, etc.), y finalmente al servidor autoritativo de ese dominio concreto, que es quien tiene la respuesta real.

Comandos útiles para depurar DNS desde terminal: `dig dominio.com` (la herramienta moderna, da información completa) y `nslookup dominio.com` (más antigua, pero todavía muy usada). Útil saber también que en Linux, `/etc/hosts` permite mapear nombres a IPs manualmente sin consultar DNS — el sistema mira primero ahí antes de preguntar a un servidor DNS externo.

Esto te va a importar directamente en tu Proyecto 1 del portfolio: Route 53 es exactamente un servicio de DNS gestionado, y vas a configurar registros (A, CNAME, ALIAS) para que tu dominio apunte a CloudFront.

## 1.6 HTTP/HTTPS

HTTP es el protocolo sobre el que funciona la web, construido sobre TCP. Cada petición HTTP tiene un método (GET para leer, POST para crear, PUT/PATCH para actualizar, DELETE para borrar — esto es literalmente lo que vas a usar al diseñar tu API REST en Cloud Photos) y recibe una respuesta con un código de estado: 200 (OK), 301/302 (redirección), 403 (prohibido), 404 (no encontrado), 500 (error del servidor). Conocer bien estos códigos te ahorra mucho tiempo depurando APIs.

HTTPS es HTTP cifrado mediante TLS/SSL — la “S” significa que el contenido va cifrado en tránsito, y por eso necesitas un certificado (en AWS, gestionado por ACM, que usarás en el Proyecto 1).

## 1.7 NAT (Network Address Translation)

NAT traduce direcciones privadas en una dirección pública compartida (y viceversa), lo cual permite que muchos dispositivos con IP privada salgan a internet usando una sola IP pública. Es exactamente el mecanismo detrás del NAT Gateway de AWS: tus instancias en una subred privada, sin IP pública propia, pueden salir a internet a través de un NAT Gateway que sí tiene IP pública, pero nadie desde fuera puede iniciar una conexión hacia esas instancias privadas directamente.

-----

# PARTE 2: NETWORKING ESPECÍFICO DE AWS (lo que examina el SAA)

Varias fuentes de preparación al SAA-C03 coinciden en que networking es uno de los dominios donde más se pierde nota, no porque el contenido sea exótico, sino porque hay muchas piezas pequeñas que hay que coordinar mentalmente a la vez (VPC, subred, tabla de rutas, gateway, Security Group, NACL). Vamos pieza por pieza.

## 2.1 VPC (Virtual Private Cloud)

Una VPC es tu propia red privada y aislada dentro de AWS. Tú decides su rango de direcciones (CIDR), divides ese rango en subredes, y configuras gateways y reglas de seguridad. Cada cuenta AWS tiene una VPC por defecto en cada región (normalmente con CIDR `172.31.0.0/16`), pero en cualquier proyecto serio crearás tu propia VPC personalizada.

Datos importantes: una VPC admite hasta 5 bloques CIDR asociados, y el tamaño de cada bloque debe estar entre /16 (máximo, 65.536 direcciones) y /28 (mínimo, 16 direcciones). El rango CIDR de tu VPC no se puede modificar después de creada — si te equivocas, tienes que crear una VPC nueva y migrar.

## 2.2 Subredes públicas vs privadas

Una **subred pública** tiene en su tabla de rutas una entrada que dirige el tráfico hacia un Internet Gateway. Una **subred privada** no tiene esa ruta — sus recursos no son directamente accesibles desde internet, ni pueden iniciar conexiones salientes a internet sin ayuda de algo más (NAT Gateway).

Patrón estándar de arquitectura en 3 niveles que verás constantemente, incluyendo en tus propios proyectos: subred pública para el Load Balancer y NAT Gateway, subred privada para los servidores de aplicación, subred privada (a veces llamada “de datos”) para la base de datos, todo esto replicado en al menos dos Availability Zones para alta disponibilidad.

### Detalle importante de AWS que no aparece en el subnetting “normal”

En una subred normal, una `/24` reserva 2 direcciones (red y broadcast) de sus 256, dejando 254 usables. En AWS, cada subred reserva **5** direcciones, no 2: la dirección de red, la del router de la VPC, la usada para DNS, una reservada para uso futuro, y la de broadcast (aunque AWS no soporta broadcast real, la reserva igual). Esto significa que una subred `/24` en AWS solo tiene 251 direcciones usables, no 254. Es un detalle pequeño pero que aparece en preguntas de examen sobre sizing.

## 2.3 Internet Gateway (IGW)

Es el componente que permite que una VPC tenga comunicación bidireccional con internet. Solo se puede tener un IGW adjunto a una VPC a la vez. Adjuntar el IGW no es suficiente por sí solo — además necesitas que la tabla de rutas de la subred tenga una ruta hacia él (normalmente `0.0.0.0/0 → igw-xxxx`), y que las instancias tengan IP pública o Elastic IP. Las tres cosas son necesarias a la vez; falta una y no hay conectividad, que es exactamente el tipo de pregunta de examen que aparece sobre este tema.

## 2.4 NAT Gateway vs NAT Instance

Ambos resuelven el mismo problema (dar salida a internet a recursos en subred privada sin exponerlos a conexiones entrantes), pero de forma distinta. El **NAT Gateway** es un servicio totalmente gestionado por AWS: escala automáticamente, tiene alta disponibilidad dentro de su AZ, y no requiere que tú lo administres. La **NAT Instance** es una EC2 normal configurada para hacer de NAT: más barata, pero tienes que gestionarla tú (parchearla, escalarla, asegurar su disponibilidad), y además requiere desactivar la comprobación de origen/destino (`source/dest check`) en la instancia, algo que no es intuitivo si no lo sabes de antemano. La recomendación estándar, y la respuesta esperada en el examen casi siempre, es NAT Gateway por el menor esfuerzo operativo, salvo que el coste sea una restricción muy fuerte.

Un detalle de examen frecuente: el NAT Gateway se coloca en la subred pública (no en la privada), y las tablas de rutas de las subredes privadas son las que apuntan hacia él.

Para tráfico IPv6 específicamente, en vez de NAT Gateway se usa un **Egress-Only Internet Gateway**, que da salida a internet por IPv6 sin permitir conexiones entrantes iniciadas desde fuera.

## 2.5 Tablas de rutas

Cada subred está asociada a una tabla de rutas, que decide hacia dónde se dirige el tráfico según su destino. Toda VPC tiene una tabla de rutas “principal” (main route table) que se asocia automáticamente a cualquier subred nueva si no le asignas explícitamente otra. Cada VPC tiene también una ruta implícita “local” que permite que todas las subredes de esa VPC se comuniquen entre sí — esa ruta siempre está ahí, no la puedes borrar.

## 2.6 Security Groups vs Network ACLs — la diferencia más preguntada en el examen

Esta es probablemente la comparación más citada en cualquier guía de preparación al SAA, y vale la pena memorizarla bien:

|          |Security Group                                                |Network ACL (NACL)                                              |
|----------|--------------------------------------------------------------|----------------------------------------------------------------|
|Nivel     |Instancia (ENI)                                               |Subred                                                          |
|Estado    |**Stateful** (el tráfico de vuelta se permite automáticamente)|**Stateless** (hay que permitir explícitamente entrada Y salida)|
|Reglas    |Solo “permitir” (no hay “denegar” explícito)                  |Permitir y denegar explícitamente                               |
|Evaluación|Se evalúan TODAS las reglas antes de decidir                  |Se evalúan en orden numérico, la primera que coincide gana      |
|Alcance   |Por instancia                                                 |Afecta a todo lo que esté en la subred                          |

La consecuencia práctica de “stateful vs stateless”: si un Security Group permite tráfico HTTP entrante, automáticamente permite la respuesta saliente sin que tengas que configurar nada para la salida. Pero si usas una NACL para lo mismo, tienes que crear una regla de entrada para el HTTP entrante Y una regla de salida separada para el tráfico de vuelta (que normalmente sale por un puerto efímero alto, no por el 80), o el tráfico se cortará en la vuelta aunque la entrada se permitiera.

Caso de uso típico que aparece en examen: tienes que bloquear un rango de IPs maliciosas específico. Como los Security Groups solo permiten reglas de “permitir”, no puedes usarlos para denegar explícitamente — necesitas una NACL con una regla de “deny” con un número de prioridad bajo (se evalúan en orden, y la primera coincidencia gana, así que el deny debe ir antes que cualquier allow que pudiera aplicar a esas IPs).

## 2.7 VPC Peering vs Transit Gateway

**VPC Peering** conecta dos VPCs directamente mediante IP privada, como si estuvieran en la misma red. Importante: no es transitivo — si A está peered con B, y B está peered con C, A no puede hablar con C automáticamente; necesitarías un peering directo entre A y C también. Esto se vuelve inviable rápidamente cuando tienes muchas VPCs (necesitarías una conexión por cada par).

**Transit Gateway** resuelve ese problema actuando como un hub central de enrutamiento regional: cada VPC, VPN o conexión Direct Connect se “adjunta” a él, y tú decides qué puede hablar con qué mediante tablas de rutas del propio Transit Gateway. Es la solución estándar cuando tienes más de un puñado de VPCs que necesitan comunicarse entre sí.

## 2.8 VPC Endpoints (PrivateLink)

Permiten que tus recursos dentro de una VPC accedan a servicios de AWS (S3, DynamoDB, y muchos más) sin que el tráfico salga a internet público, ni siquiera pasando por un NAT Gateway. Hay dos tipos: **Gateway Endpoint** (solo para S3 y DynamoDB, gratuito, se añade como entrada en la tabla de rutas) y **Interface Endpoint** (para el resto de servicios soportados, tiene coste por hora y por tráfico procesado, crea una interfaz de red privada dentro de tu subred). La recomendación estándar: usa Gateway Endpoint siempre que el servicio sea S3 o DynamoDB porque es gratis y no añade latencia; usa Interface Endpoint para todo lo demás cuando necesites acceso privado.

## 2.9 Direct Connect y VPN

**Site-to-Site VPN** conecta tu red on-premise con tu VPC a través de internet público, pero cifrado — rápido de montar (minutos a horas) pero con el rendimiento variable de internet público. **Direct Connect** es una conexión dedicada y privada entre tu datacenter y AWS, sin pasar por internet público en absoluto — mucho más rendimiento y consistencia, pero puede tardar semanas o meses en aprovisionarse físicamente. En arquitecturas serias, la VPN se usa frecuentemente como respaldo (failover) de una conexión Direct Connect principal.

## 2.10 Route 53

Es el servicio DNS gestionado de AWS — y mucho más que solo DNS. Permite registrar y gestionar dominios, hacer health checks de tus endpoints y desviar tráfico automáticamente si algo falla (DNS failover), aplicar políticas de enrutamiento avanzadas (latencia, geolocalización, peso), y crear zonas DNS privadas dentro de una VPC para resolver nombres internos sin exponerlos al internet público.

Para tu Proyecto 1: usarás un registro tipo ALIAS (una mejora de AWS sobre el CNAME estándar, sin coste adicional de consulta) apuntando tu dominio hacia la distribución de CloudFront.


## 2.11 Load Balancers y en qué capa trabaja cada uno

**Application Load Balancer (ALB)**: trabaja en capa 7, entiende HTTP/HTTPS, permite enrutamiento basado en contenido (path, host header), ideal para aplicaciones web modernas y microservicios.

**Network Load Balancer (NLB)**: trabaja en capa 4, maneja TCP/UDP a rendimiento extremo y baja latencia, preserva la IP del cliente de forma nativa, se usa cuando necesitas throughput muy alto o protocolos que no son HTTP.

**Gateway Load Balancer**: pensado para insertar appliances de terceros (firewalls, sistemas de detección de intrusiones) de forma transparente en el flujo de tráfico.

-----

## 2.12 Resumen práctico: el checklist mental para cualquier pregunta de conectividad

Cuando te encuentres con un escenario de “¿por qué no conecta X con Y?”, repasa en orden: ¿la instancia tiene IP pública o Elastic IP (si necesita ser accesible desde internet)? ¿La subred es pública (tiene ruta a un IGW)? ¿La tabla de rutas de esa subred apunta correctamente? ¿El Security Group de la instancia permite ese tráfico de entrada/salida? ¿La NACL de la subred permite ese tráfico en ambas direcciones (recuerda que es stateless)? Si todas estas piezas están bien y sigue sin funcionar, el problema probablemente está a nivel de aplicación (capa 7), no de red.

Este es exactamente el orden de razonamiento que se espera de un Solutions Architect, y el que te va a servir tanto para el examen como para depurar tus propios proyectos cuando algo no conecte como esperabas.