El escalado de recursos es uno de los elementos mas importantes en el mundo empresarial, porque siempre pensamos en el dinero. Si nuestra aplicación tiene bajo tráfico y de repente aumenta el numero de solicitudes, necesitamos que nuestro sistema tolere este aumento y siga funcionando. Es por esto que kubernetes (e incluso otras soluciones como AWS ECS) implmentan el escalado mediante la replicación.

Las métricas de recursos son fundamentales, tales como uso de CPU, RAM etc. Ya que basados en estas métricas nuestra configuración actuará y replicará nuestros servicios.

Hay dos tipos de escalado:

# Escalado Vertical

Este tipo de escalado se basa en el aumento de recursos de un nodo. Suponiendo el caso de un VPS, agregaríamos mas RAM y CPU para que este tenga mas poder de cómputo y nuestro servicio tenga mas espacio para ser ejecutado.

![vertical_scalling](../media/vertical_scalling.png)

Si bien kubernetes no posee un escalado vertical automático, es posible configurar las características de los nodos manualmente

# Escalado Horizontal

Este tipo es el mas común y generalmente el mejor, ya que aún con varios vps de bajo costo podremos suplir nuestras necesidades. El truco es que nuestros pods se distribuirán en las diferentes replicas y mediante el mecanismo de load balancing nuestro servicio estará disponible

![horizontal_scalling](../media/horizontal_scalling.png)

Es importante resaltar que es posible tener pods replicados en el mismo nodo dependiendo de la configuración **affinity y tolerations**

---

Un ejemplo divertido es pensar en una aplicación que lanzamos en un único nodo inicial. Resulta que no calculamos bien los recursos que esto iba a consumir y falla.

Tenemos ambas opciones aquí

- Aumentar los recursos del nodo inicial, de esta manera los servicios podrán iniciarse con tranquilidad, asumiendo los costos y los posibles problemas futuros, ya que al recibir mas tráfico indudablemente se va a colapsar nuevamente el vps
- Habilitar el escalado horizontal, de esta manera el servicio podra iniciar creando una réplica y estaremos bien.

Sin embargo hay un caso mas dramático como este:

Imaginemos el siguiente escenario. Supongamos que tengo dos workernodes limitados en recursos e intento lanzar un pod, que al iniciar supera **siempre** la capacidad de ambos workernodes, es decir, sin importar la replicación va a ser superior el consumo que la disponibilidad de recursos, que ocurría en ese caso?

> Escenario: Pod con requisitos de recursos superiores a la capacidad de los nodos

## Situación
Si intentas lanzar un pod que requiere más recursos de los disponibles en cualquier nodo del clúster, Kubernetes no podrá programar el pod en ninguno de los nodos debido a la falta de capacidad.

## 1. El Pod queda en estado **Pending**
Cuando un pod solicita más recursos de los que cualquier nodo puede ofrecer, Kubernetes intentará crearlo, pero lo dejará en estado **Pending**. Esto indica que el Scheduler no ha encontrado un nodo adecuado para ejecutarlo. En este estado, el pod no se iniciará y Kubernetes continuará intentando programarlo sin éxito hasta que haya un nodo con capacidad suficiente.

## 2. Verificación del estado Pending
Puedes verificar la razón exacta del estado **Pending** ejecutando el siguiente comando:

```bash
kubectl describe pod <nombre-del-pod>
```

En la salida, verás mensajes detallados que indicarán que el pod no pudo programarse debido a la falta de recursos (por ejemplo, falta de CPU o memoria).

## 3. Acciones posibles para solucionar el problema

Para resolver el problema y permitir que el pod se ejecute, tienes varias opciones:

#### 1. Cluster Autoscaler
Si tienes habilitado el Cluster Autoscaler en un entorno en la nube (como AWS, GCP o Azure), el autoscaler intentará agregar nodos con suficiente capacidad para ejecutar el pod. Sin embargo, esto solo funciona si estás en un entorno escalable, donde Kubernetes puede solicitar nodos adicionales a la infraestructura subyacente.

- Si se agrega un nodo con recursos suficientes para el pod, el Scheduler lo detectará y programará el pod en ese nuevo nodo.
- Si el Cluster Autoscaler no puede agregar un nodo (por ejemplo, debido a restricciones de cuota de recursos o limitaciones de configuración), el pod permanecerá en estado Pending.
#### 2. Optimizar las solicitudes de recursos
Si es posible, podrías reducir las solicitudes de recursos (requests y limits) del pod para que se ajusten a los recursos disponibles en los nodos actuales. Esto solo funcionará si el pod puede ejecutarse con menos recursos y aún así funcionar correctamente.

#### 3. Agregar manualmente nodos más grandes
Si estás en un entorno controlado (por ejemplo, un clúster on-premise o sin autoscaling), podrías añadir manualmente un nodo con más capacidad para permitir que el pod se programe y se ejecute. Esto implica modificar el clúster para incluir un nodo con más recursos (CPU o memoria) que el que requiere el pod.

#### 4. Redistribuir la carga
Si algunos de tus pods están consumiendo recursos de manera innecesaria, puedes intentar optimizar su uso de recursos o mover ciertos pods a otros nodos para liberar capacidad. Esto puede implicar eliminar pods menos prioritarios o reducir la cantidad de réplicas.

### Ejemplo práctico
Imagina que tienes dos nodos con 2 CPU y 4 GB de RAM cada uno, y un pod requiere 4 CPU y 8 GB de RAM para ejecutarse. En este caso:

- Kubernetes intentará lanzar el pod pero no podrá programarlo, ya que ningún nodo tiene los recursos suficientes.
- El pod quedará en estado Pending.
- Si tienes habilitado el Cluster Autoscaler en la nube, este puede añadir un nodo con suficientes recursos (por ejemplo, uno con 4 CPU y 8 GB de RAM), y el pod se programará allí.
- Si el Cluster Autoscaler no está disponible, el pod permanecerá en Pending hasta que realices alguna de las acciones mencionadas.

En resumen, Kubernetes no intentará iniciar el pod en un nodo si los recursos son insuficientes, ya que esto podría causar problemas de rendimiento y estabilidad. El pod solo se iniciará cuando haya capacidad suficiente.


# Métricas

Como se mencionó anteriormente la clave del funcionamiento del autoscalling son las métricas de los nodos. En AWS se utiliza el buen CloudWatch para obtener datos de EC2 sobre la RAM, CPU etc. 

Para hacer el simil entre ECS y Kubernetes, podemos comparar CloudWatch con HPA. Obtenemos los datos de los nodos y basados en las métricas replicamos los nodos según las necesidades.

Entonces en resumidas cuentas:

> Su objetivo principal es asegurar que la aplicación siempre tenga suficientes réplicas (instancias) en ejecución para manejar la carga de trabajo actual, garantizando al mismo tiempo que no haya recursos subutilizados que generen costos innecesarios.

### Funcionamiento de HPA:
1. Métricas de recursos: HPA utiliza métricas específicas para decidir cuándo escalar los pods. Las métricas más comunes son el uso de CPU y la utilización de memoria. Kubernetes monitoriza continuamente estas métricas en tiempo real.

2. Configuración de HPA: Para configurar el HPA, debes definir:
   - Métricas de autoscaling: Decide qué métricas (CPU, memoria u otras métricas personalizadas) se utilizarán para escalar.
   - Objetivos de uso de recursos: Establece los valores de utilización de recursos que deseas mantener (por ejemplo, mantener el uso de CPU en promedio al 50%).

3. Decisiones de escalado: Basado en las métricas configuradas, el HPA evalúa continuamente si hay necesidad de ajustar el número de réplicas de los pods.
   - Escalar hacia arriba: Si la carga aumenta y las métricas superan los límites definidos, el HPA incrementará el número de réplicas de los pods para manejar la carga adicional.
   - Escalar hacia abajo: Si la carga disminuye y las métricas caen por debajo de los límites establecidos, el HPA reducirá el número de réplicas, lo que ahorra recursos y costos.
   - Interacción con los controladores de réplicas: El HPA interactúa con los controladores de réplicas de Kubernetes para gestionar automáticamente el escalado de los pods. No es necesario intervenir manualmente una vez que se ha configurado correctamente.