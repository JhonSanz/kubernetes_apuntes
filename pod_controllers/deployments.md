> En Kubernetes, los pod controllers son componentes que gestionan el ciclo de vida de los pods y aseguran que el estado deseado del sistema se mantenga. Los controladores supervisan los pods y toman decisiones para alcanzar y mantener el estado deseado.

- Objetivo: Mantener un número deseado de réplicas de pods en todo momento.
- Función Principal: Garantizar que siempre haya el número especificado de pods en ejecución, creando o eliminando pods según sea necesario.
- Configuración: Puedes especificar el número de réplicas deseadas en la configuración del Deployment. Esto se puede ajustar manualmente mediante comandos o actualizando el archivo de configuración YAML.
- Sin HPA: El escalado es estático y requiere intervención manual para ajustar el número de réplicas. El Deployment no ajusta automáticamente el número de réplicas en función del uso de recursos.

Y aqui es donde aparece lo interesante. Debido al hecho de que deployment solo se encarga de **mantener la cantidad de replicas** que configuremos, es necesario utilizar **HPA** para crecer o decrecer la infraestructura basado en métricas de CPU y RAM, lo cual es muy importante para optimizar costos, utilizando instancias spot y ese tipo de cosas. Adicionalmente para saltar a la nube es necesario configurar el **Cluster Autoscaler**.
