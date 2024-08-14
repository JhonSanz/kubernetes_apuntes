> En Kubernetes, los pod controllers son componentes que gestionan el ciclo de vida de los pods y aseguran que el estado deseado del sistema se mantenga. Los controladores supervisan los pods y toman decisiones para alcanzar y mantener el estado deseado.


Abhishek Veeramalla hace una comparación interesante. Cuando hablamos de docker, para levantar un contenedor debemos ejecutar un comando en la cli parecido a `docker run image:latest -p 3000:3000 blablabla`. En el mundo de kubernetes sustituimos este comando por un archivo de configuración yaml, el cual kubernetes podrá gestionar, recordando siempre que la diferencia principal es que un pod es una agrupación lógica de contenedores, es decir, un pod puede tener varios contenedores. Como vimos anteriormente, podemos crear un pod manualmente definiéndolo similar a esto

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Funcionará, pero de esta manera estamos desperdiciando las ventajas de kubernetes, ya que así no tenemos ni autohealing, ni autoscalling etc. Pero se ve claramente que podemos levantar un contenedor en kubernetes, incluso esto es muy similar a docker-compose.

Así que para eso llegamos a los pod controllers. Y aquí hay varias cosas interesantes. Para lograr las caracteristicas enterprise para un ambiente productivo kubernetes implementa replicaset


# ReplicaSet

Aunque es posible utilizar este tipo de controllers, la documentación explícitamente recomienda utilizar **deployments** ya que incluye la misma funcionalidad mejorada

https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/#cuándo-usar-un-replicaset

Sin embargo, para efectos de aprendizaje. El ReplicaSet se encarga de **mantener** un conjunto estable de **réplicas de pods ejecutándose en todo momento**

Si algún pod falla o se elimina, el ReplicaSet **automáticamente crea** nuevos pods para reemplazar los que se han perdido, garantizando así que el número deseado de réplicas esté siempre disponible.

Basta con definir un archivo yaml y aplicarlo a la configuración de kubernetes. [Ver ejemplo](https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/#ejemplo)


Entonces, como hemos visto, kubernetes explícitamente ya nos recomienda dos cosas que **no debemos hacer**, la primera definir los pods manualmente y la segunda utilizar replicaset

# Deployment 

Entonces aquí aparece deployment, y en términos generales es un wrapper de replicaset con funciones adicionales. Así que vamos a ver las diferencias entre ambos

### ReplicaSet
- **Propósito**: Asegurar que un número específico de réplicas de un pod estén corriendo en todo momento.
- **Función principal**: Monitoriza el estado de los pods y, si alguno falla o se elimina, el ReplicaSet creará nuevos pods para reemplazarlos y mantener el número deseado de réplicas.
- **Configuración**: Define el número de réplicas deseadas y el selector para los pods que debe controlar.
- **Uso directo**: Se usa más comúnmente como un componente interno del Deployment. Aunque puedes usar ReplicaSets directamente, no es la práctica recomendada para la mayoría de los casos.
### Deployment
- **Propósito**: Facilitar el despliegue, actualización y rollback de aplicaciones.
- **Función principal**: Administra el ciclo de vida de los pods a través de ReplicaSets. Permite hacer despliegues y actualizaciones de manera controlada y ordenada.
- **Actualizaciones**: Permite realizar actualizaciones continuas de la aplicación sin tiempo de inactividad, gracias a estrategias como la actualización progresiva (rolling updates).
- **Rollbacks**: Permite revertir a una versión anterior de la aplicación en caso de que algo salga mal durante una actualización.
- **Historial**: Mantiene un historial de versiones y puede gestionar múltiples ReplicaSets asociados a diferentes versiones de la aplicación.
- **Uso**: Es la forma preferida de gestionar la aplicación en producción, ya que proporciona más control y flexibilidad para realizar cambios y gestionar el estado de la aplicación.
### Resumen
- **ReplicaSet**: Garantiza que un número específico de pods esté en ejecución, pero no ofrece funcionalidades para actualizar o revertir cambios.
- **Deployment**: Utiliza ReplicaSets para gestionar los pods, además de ofrecer herramientas para despliegues, actualizaciones y rollbacks.

Así se ve un deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Aqui es donde aparece lo interesante. Debido al hecho de que deployment solo se encarga de **mantener la cantidad de replicas** que configuremos mediante replicaset, es necesario utilizar **HPA** para crecer o decrecer la infraestructura basado en métricas de CPU y RAM, lo cual es muy importante para optimizar costos, utilizando instancias spot y ese tipo de cosas. Adicionalmente para saltar a la nube es necesario configurar el **Cluster Autoscaler**.


# ROLLBACKS

Es el proceso de revertir la aplicación a una versión anterior, lo cual es útil especialmente para regresar a una versión segura en caso de que algo falle.

### ¿Cómo Funciona el Rollback?
Cuando aplicas cambios a un Deployment, Kubernetes **crea un nuevo ReplicaSet** para manejar la nueva versión de los pods. El Deployment **mantiene un historial** de estos ReplicaSets, lo que te permite revertir a una versión anterior si es necesario.

1. **Historial de Versiones**: Cada vez que actualizas un Deployment (por ejemplo, cambiando la imagen del contenedor), Kubernetes crea un nuevo ReplicaSet. El Deployment guarda información sobre estos ReplicaSets y sus configuraciones anteriores.
2. **Estrategias de Actualización**: Kubernetes utiliza estrategias de actualización, como la actualización progresiva (rolling update), que despliega los cambios de manera gradual. Si la nueva versión falla, puedes hacer un rollback a una versión anterior para restaurar el estado anterior.
3. **Realización del Rollback**: Puedes realizar un rollback manualmente usando el comando kubectl rollout undo. Esto revertirá el Deployment a la configuración de ReplicaSet anterior. Kubernetes seleccionará el ReplicaSet anterior y lo hará la versión actual.
   1. `kubectl rollout history deployment/my-app`
   2. `kubectl rollout undo deployment/nombre-del-deployment`
   3. `kubectl rollout undo deployment/nombre-del-deployment --to-revision=2`
4. **Verificación del Estado:** Después de hacer un rollback, es importante verificar que el Deployment esté funcionando correctamente. Puedes usar comandos como `kubectl get deployments` y `kubectl describe deployment nombre-del-deployment` para verificar el estado actual y asegurarte de que los pods están en la versión deseada.

Para tutorial completo sobre rollback vistiar https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment


# Otros parámetros para gestionar deployments

1. **Estrategia de actualización**: Puedes definir cómo se realizarán las actualizaciones a través del campo strategy en el archivo YAML del Deployment.
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```
- **Rolling Update**: La estrategia predeterminada que despliega los cambios de manera gradual. Puedes controlar el ritmo de la actualización con los parámetros maxSurge y maxUnavailable.
- **Recreate**: La estrategia que elimina todos los pods existentes antes de crear los nuevos pods. Esto puede ser configurado si no puedes permitir una transición gradual.
- **maxSurge**: Define el número máximo de pods que se pueden crear por encima del número deseado durante una actualización.
- **maxUnavailable**: Define el número máximo de pods que pueden estar fuera de servicio durante una actualización.

2. **readinessProbe**: Las Readiness Probes son una característica crucial para asegurar que los pods están listos para recibir tráfico antes de que sean considerados como disponibles.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app:v1
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

3. **Liveness Probes**: Es una verificación que Kubernetes realiza para determinar si un contenedor está en un estado saludable y operativo. A diferencia de las Readiness Probes, que aseguran que un contenedor esté listo para recibir tráfico, una Liveness Probe verifica si un contenedor está vivo y funcionando correctamente. Si la Liveness Probe falla, Kubernetes considera que el contenedor está en un estado de error y puede reiniciarlo automáticamente para intentar recuperarlo.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app:v1
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 5
          failureThreshold: 3
```
4. Variables de entorno
5. secrets
6. configmaps