# Uso de AWS PrivateLink

## Problema

En muchas organizaciones, distintos equipos trabajan en VPCs separadas por motivos de aislamiento, seguridad y organización.

Sin embargo, a veces un equipo necesita consumir una API o servicio interno desplegado en otra VPC.

La solución tradicional podría ser VPC Peering, pero eso implica:

- Conectividad de red completa entre ambas VPCs.
- Coordinación de rangos CIDR.
- Mayor superficie de ataque.
- Menor desacoplamiento entre equipos.

No siempre queremos conectar redes enteras. Solo queremos exponer un servicio concreto para que el resto pueda consumirlo.


## Solución: AWS PrivateLink

AWS PrivateLink permite exponer un servicio específico (siempre con un Network Load Balancer) a otras VPCs mediante un Endpoint Service.

La VPC consumidora crea un Interface VPC Endpoint que:

- Genera una ENI con IP privada dentro de su propia subnet.
- Proporciona un DNS privado.
- Redirige el tráfico internamente hacia la VPC productora.

Desde el punto de vista del cliente, parece que está llamando a un recurso interno.


## Flujo real de tráfico

1. Un  cliente dentro de la VPC del consumer resuelve el DNS privado del Interface Endpoint.
2. El tráfico va a la ENI del Interface Endpoint (IP privada).
3. PrivateLink dirige el tráfico al Endpoint Service de la VPC Productora.
4. El tráfico llega al NLB, que distribuye al Target Group con las tareas ECS.
5. ECS llama a Redis y RDS según sea necesario.
6. Una vez el código en ECS responde, la respuesta sigue el camino inverso.

En ningún momento el tráfico atraviesa Internet público.


## Ventajas

- No se requiere VPC Peering.
- No es necesario coordinar rangos CIDR.
- Se expone únicamente el servicio necesario.
- Escala fácilmente a múltiples VPCs consumidoras.
- Reduce la superficie de ataque.
- Mantiene el backend completamente privado.

PrivateLink desacopla redes y permite consumir servicios internos como si fuesen productos dentro de una organización.