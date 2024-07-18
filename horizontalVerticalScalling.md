El escalado de recursos es uno de los elementos mas importantes en el mundo empresarial, porque siempre pensamos en el dinero. Si nuestra aplicación tiene bajo tráfico y de repente aumenta el numero de solicitudes, necesitamos que nuestro sistema tolere este aumento y siga funcionando. Es por esto que kubernetes (e incluso otras soluciones como AWS ECS) implmentan el escalado mediante la replicación.

Las métricas de recursos son fundamentales, tales como uso de CPU, RAM etc. Ya que basados en estas métricas nuestra configuración actuará y replicará nuestros servicios.

Hay dos tipos de escalado:

# Escalado Vertical

Este tipo de escalado se basa en el aumento de recursos de un nodo. Suponiendo el caso de un VPS, agregaríamos mas RAM y CPU para que este tenga mas poder de cómputo y nuestro servicio tenga mas espacio para ser ejecutado.

![vertical_scalling](media/vertical_scalling.png)

Si bien kubernetes no posee un escalado vertical automático, es posible configurar las características de los nodos manualmente

# Escalado Horizontal

Este tipo es el mas común y generalmente el mejor, ya que aún con varios vps de bajo costo podremos suplir nuestras necesidades. El truco es que nuestros pods se distribuirán en las diferentes replicas y mediante el mecanismo de load balancing nuestro servicio estará disponible

![horizontal_scalling](media/horizontal_scalling.png)

Es importante resaltar que los pods se replican pero no van a estar viviendo en el mismo nodo, es decir, no encontraremos dos pods replicados identicos en el mismo nodo, eso no tendría mucho sentido.

---

Un ejemplo divertido es pensar en una aplicación que lanzamos en un único nodo inicial. Resulta que no calculamos bien los recursos que esto iba a consumir y falla.

Tenemos ambas opciones aquí

- Aumentar los recursos del nodo inicial, de esta manera los servicios podrán iniciarse con tranquilidad, asumiendo los costos y los posibles problemas futuros, ya que al recibir mas tráfico indudablemente se va a colapsar nuevamente el vps
- Habilitar el escalado horizontal, de esta manera el servicio podra iniciar creando una réplica y estaremos bien.