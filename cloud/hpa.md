El Horizontal Pod Autoscaler es una característica de Kubernetes que automáticamente ajusta el número de réplicas de un pod en función de la carga. 

HPA utiliza métricas como CPU y RAM para tomar decisiones sobre el escalado. También puedes configurar HPA para que use métricas personalizadas a través de métricas de API.

HPA compara el valor actual de la métrica con el valor objetivo definido en la configuración. Si la métrica supera el umbral establecido, HPA incrementará el número de réplicas del pod y al revés.

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 1
        periodSeconds: 60
      selectPolicy: Min
```

Este ejemplo configura el HPA para que ajuste el número de réplicas entre 2 y 10 basándose en el uso de CPU, con políticas específicas para escalar hacia arriba y hacia abajo, y un periodo de estabilización para evitar fluctuaciones bruscas.

En este ejemplo:

- **scaleTargetRef:** Especifica el recurso (en este caso, un Deployment) que el HPA debe escalar.
- **minReplicas y maxReplicas:** Definen el rango de réplicas entre las que el HPA puede ajustar el número de pods.
- **metrics:** Aquí se define que el HPA debe usar la utilización de CPU como métrica y que debe mantener un promedio del 50% de utilización de CPU. Si la utilización de CPU supera este umbral, el HPA aumentará el número de réplicas para distribuir la carga; si está por debajo, disminuirá el número de réplicas.
- **stabilizationWindowSeconds**: Define el tiempo durante el cual el HPA espera antes de realizar nuevos ajustes después de un cambio en el número de réplicas. Esto ayuda a evitar ajustes bruscos y fluctuaciones.
- **policies** Las políticas determinan el ritmo del escalado.
- **Percent:** Permite escalar el número de réplicas en un porcentaje de los pods actuales. Por ejemplo, un 50% significa que se puede agregar hasta la mitad del número actual de réplicas en cada ajuste.
- **Pods:** Define el número fijo de pods que se pueden agregar o eliminar en cada ajuste. Por ejemplo, el valor 2 significa que el HPA puede añadir hasta 2 pods en cada ciclo de escalado hacia arriba.
- **selectPolicy:** Define la política a utilizar cuando hay múltiples políticas aplicables. Max para scaleUp significa que el HPA tomará la política de escalado más agresiva, mientras que Min para scaleDown significa que tomará la política más conservadora.

https://medium.com/@amirhosseineidy/how-to-make-a-kubernetes-autoscaling-hpa-with-example-f2849c7bbd0b

--- 

### Probar HPA con minikube


Inicia Minikube con el siguiente comando:

```sh
minikube start
```

El Metric Server debe estar habilitado para que el HPA funcione. Minikube puede instalarlo automáticamente, pero asegúrate de que esté corriendo:

```sh
minikube addons enable metrics-server
```

Verifica que el Metric Server esté funcionando:

```sh
kubectl get pods -n kube-system
```

Busca el pod metrics-server en la lista y asegúrate de que esté en estado Running.

#### Desplegar una Aplicación

Crea un archivo YAML para un Deployment que defina los límites de recursos. Aquí hay un ejemplo básico de una aplicación web que puedes usar:

- deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

- Aplica el Deployment
```sh
kubectl apply -f deployment.yaml
```

#### Crear el HPA

Ahora, crea un archivo YAML para definir el HPA. Aquí tienes un ejemplo utilizando autoscaling/v2beta2:

hpa.yaml:

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 1
        periodSeconds: 60
      selectPolicy: Min
```

Aplica el HPA:

```sh
kubectl apply -f hpa.yaml
```

#### Generar Carga en la Aplicación

Para probar el escalado, necesitas generar carga en la aplicación. Puedes usar herramientas como **ab (Apache Benchmark)** o **stress** para esto. A continuación se muestra un ejemplo usando stress:

Primero, abre una terminal en el pod de la aplicación:

```sh
kubectl exec -it $(kubectl get pods -l app=my-app -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```
Luego, instala stress si no está disponible y ejecuta una carga:

```sh
apt-get update && apt-get install -y stress
```
```sh
stress --cpu 4 --timeout 60s
```

Esto debería aumentar el uso de CPU y activar el escalado.

#### Monitorear el HPA

Puedes monitorear el estado del HPA y el escalado con el siguiente comando:

```sh
kubectl get hpa
```
Para ver el uso de CPU y otros detalles de los pods:

```sh
kubectl top pods
```