Los ConfigMaps son recursos en Kubernetes que permiten almacenar datos de configuración en forma de pares clave-valor. Estos datos pueden ser utilizados por los Pods para configurar aplicaciones sin tener que modificar la imagen del contenedor.

Esto funciona muy similar a los archivos .env

### Creación de un ConfigMap
Puedes crear un ConfigMap de varias maneras, como desde un archivo de configuración o directamente desde la línea de comandos.

1. Desde un archivo de configuración (configmap.yaml):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  APP_ENV: production
  APP_LOG_LEVEL: debug
```

2. Desde la línea de comandos:
```sh
kubectl create configmap my-config --from-literal=config1=value1 --from-literal=config2=value2
```

### Usar el configmap en un deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: APP_ENV
        - name: APP_LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: APP_LOG_LEVEL
```