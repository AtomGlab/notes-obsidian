Udemy Course
https://missioninstituteoftechnology.com/courses/aws-saa-study-arcade/flashcards/

https://cloudnugget.dev/
https://github.com/RonitSachdev/aws-saa-c03-guides

Emuladores de AWS
https://ministack.org/
https://floci.io/

**SAML 2.0-Based Federation by using a Web Identity Federation.**
**Flujo típico:** normalmente se implementa a través de **Amazon Cognito Identity Pools**, que actúan de intermediario — el usuario se loguea con Google → Cognito cambia ese token por credenciales temporales de AWS → la app accede a recursos como S3 o DynamoDB directamente.

# Parte 1.5 — ¿Cómo sé si una IP es privada?

No se puede saber por el número de dispositivos. Se sabe porque hay tres rangos reservados.

Apréndete esta tabla. Es de las pocas cosas que merece la pena memorizar.

|   |   |
|---|---|
|Rango|Uso|
|`10.0.0.0 – 10.255.255.255`|Redes privadas grandes (AWS usa muchísimo este rango).|
|`172.16.0.0 – 172.31.255.255`|Otro rango privado.|
|`192.168.0.0 – 192.168.255.255`|El típico de las casas y routers.|

## Truco visual

IPs privadas

10.x.x.x 172.16–31.x.x 192.168.x.x

IPs públicas

Todo lo que no pertenece a esos rangos.
