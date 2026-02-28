# Diseño de la arquitectura

## Enfoque general

El objetivo principal de esta arquitectura era simular un entorno lo más cercano posible a un caso real de empresa:

- Equipos separados en VPCs distintas.
- Backend completamente privado.
- Alta disponibilidad.
- Control del tráfico.

No se trata solo de que funcione, sino de que esté bien planteado desde el punto de vista de red y seguridad.


## Separación por capas (Layered Architecture)

He estructurado ambas VPCs siguiendo una separación clara:

- Subnet pública (solo en la VPC consumidora, para bastión SSH).
- Subnets privadas de aplicación.
- Subnets privadas de datos.

La idea es sencilla: cada capa solo habla con la que necesita.

- El bastión solo sirve para testear.
- Las instancias privadas consumen la API.
- ECS solo habla con el NLB y con la capa de datos.
- RDS y Redis solo aceptan tráfico desde ECS.


## Backend completamente privado

En la VPC productora:

- Las tareas ECS no tienen IP pública.
- El Network Load Balancer es interno.
- RDS PostgreSQL y Redis no son accesibles desde Internet.

El único punto de entrada al backend es el Endpoint Service respaldado por el NLB, que a su vez solo es accesible mediante PrivateLink desde VPCs autorizadas.

Esto significa que la API no puede ser consumida desde Internet bajo ningún concepto.


## Multi-AZ

Todos los componentes críticos están desplegados en múltiples Availability Zones:

- ECS Fargate.
- Subnets privadas.
- Capa de datos.
- VPC Endpoints.

La idea es evitar puntos únicos de fallo y ofrecer una mínima tolerancia a fallos, aunque sea un entorno de laboratorio.


## Desacoplamiento entre VPCs

El uso de PrivateLink permite que la VPC productora no tenga conocimiento directo de la red de la VPC consumidora.

No hay peering.
No hay tablas de rutas compartidas.
No hay coordinación de CIDRs.

Se expone un servicio, no una red completa.

Eso hace que el diseño sea más limpio y más escalable.


## Enfoque en seguridad

En lugar de optar por soluciones rápidas (exponer el NLB públicamente o conectar VPCs con peering completo), se ha priorizado:

- Aislamiento.
- Control.
- Reducción de superficie de ataque.
- Backend 100% privado.

Funciona igual.
Pero está mejor diseñado.