> En Kubernetes, los pod controllers son componentes que gestionan el ciclo de vida de los pods y aseguran que el estado deseado del sistema se mantenga. Los controladores supervisan los pods y toman decisiones para alcanzar y mantener el estado deseado.


# ReplicaSet

Aunque es posible utilizar este tipo de controllers, la documentación explícitamente recomienda utilizar **deployments** ya que incluye la misma funcionalidad mejorada

https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/#cuándo-usar-un-replicaset

Sin embargo, para efectos de aprendizaje. El ReplicaSet se encarga de **mantener** un conjunto estable de **réplicas de pods ejecutándose en todo momento**

Si algún pod falla o se elimina, el ReplicaSet **automáticamente crea** nuevos pods para reemplazar los que se han perdido, garantizando así que el número deseado de réplicas esté siempre disponible.

Basta con definir un archivo yaml y aplicarlo a la configuración de kubernetes. [Ver ejemplo](https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/#ejemplo)