# NAT vs VPC Endpoints

## Enfoque inicial: NAT Gateway

Cuando empecé a estudiar AWS y a tocar conceptos de redes, siempre ponía una NAT Gateway y, desde mi código, llamaba a los servicios que había añadido. Por ejemplo: desde mi código en ECS llamaba a una base de datos RDS o Dynamo, añadía eventos a una cola SQS, llamaba a Lambda...

Es una solución totalmente válida y funciona sin problemas. Las tareas ECS, al estar en subnets privadas, no tienen IP pública, por lo que necesitan un NAT para poder salir a Internet y acceder a estos servicios.

El flujo sería:

ECS → NAT Gateway → Internet Gateway → Endpoint público del servicio AWS

Aunque el tráfico nunca salga físicamente de la infraestructura de AWS, a nivel de red se considera tráfico público. Además, el NAT tiene un coste fijo por hora y por tráfico procesado, lo que puede encarecer la arquitectura si se despliega en múltiples AZs.


## Alternativa: VPC Endpoints

Sin embargo, más adelanté descubrí los VPC Endpoints y ahí abrí los ojos. En el pasado estaba en un recurso privado, llamando a un recurso interno de AWS, saliendo por Internet público, lo cual viéndolo con perspectiva es un sinsentido. Por eso, ahora utilizo VPC endpoints.

Los VPC Endpoints permiten acceder a servicios regionales de AWS sin necesidad de atravesar Internet. El tráfico permanece completamente dentro del backbone interno de AWS.

Hay dos tipos:

- **Gateway Endpoints** (para S3 y DynamoDB)
- **Interface Endpoints** (para el resto: ECR, Secrets Manager, CloudWatch, etc.)

En este caso, creando Interface Endpoints para ECR y Secrets Manager, las tareas ECS pueden comunicarse directamente con estos servicios de forma privada:

ECS → Interface Endpoint (ENI privada) → Servicio AWS

Sin NAT.
Sin Internet Gateway.
Sin exposición adicional.


## Comparación real

El NAT es más simple de montar y puede ser suficiente en muchos escenarios, especialmente en entornos pequeños o de laboratorio.

Sin embargo, los VPC Endpoints ofrecen:

- Mejor seguridad.
- Mayor control sobre el tráfico.
- Arquitectura más alineada con entornos empresariales.
- Eliminación de dependencia de Internet para servicios internos.

La contrapartida es que cada endpoint tiene coste por AZ y requiere algo más de configuración.

Desde un punto de vista de diseño, si el objetivo es construir un backend completamente privado y reducir al máximo la superficie de ataque, los VPC Endpoints son la mejor opción.

Ambas opciones funcionan y son completamente válidas. Pero para una arquitectura privada y que sea lo más profesional posible, yo personalmente optaría por VPC Endpoints.