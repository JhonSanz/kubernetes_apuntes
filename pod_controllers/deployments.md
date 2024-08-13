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

Aqui es donde aparece lo interesante. Debido al hecho de que deployment solo se encarga de **mantener la cantidad de replicas** que configuremos mediante replicaset, es necesario utilizar **HPA** para crecer o decrecer la infraestructura basado en métricas de CPU y RAM, lo cual es muy importante para optimizar costos, utilizando instancias spot y ese tipo de cosas. Adicionalmente para saltar a la nube es necesario configurar el **Cluster Autoscaler**.
