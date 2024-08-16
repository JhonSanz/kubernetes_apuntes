# Services

Existen por el mismo motivo que existe el autohealing. Imaginemos una situación donde tenemos un deployment y tenemos 3 replicas de nuestro servicio. Como suele pasar, uno de nuestros pod puede fallar y morir, entonces replicaset se encargará de revivirlo.

El problema aparece cuando nos damos cuenta de que cuando replicaset revive el pod, este **ya no tiene la misma dirección IP**, entonces no podrá se encontrado ni por los demás pods, ni por accesos externos al cluster, es decir, ese pod revive pero está desconectado de todo.

Entonces encontramos ahí la utilidad de los services, y también los diversos casos en donde debe aplicarse.

### **Comunicación interna entre pods**
Permite que los Pods dentro del mismo clúster se comuniquen entre sí de manera confiable usando nombres en lugar de IPs. Esto se logra a través de las etiquetas que asignamos a los pod mediante el manifiesto del deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend # creamos el label backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: my-backend-image
          ports:
            - containerPort: 8080
```

Y luego lo utilizamos en el service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend # lo utilizamos aqui
  ports:
    - protocol: TCP
      port: 80       
      targetPort: 8080  
  type: ClusterIP

```

Es **muy importante** tener en cuenta que nuestro service ahora es la url para comunicar nuestros pods `http://backend-service` (DNS creado por kubernetes) suponiendo que el frontend accede a algun recurso del backend. De esta manera superamos la dificultad de encontrar los pods con la IP, y en su lugar el servicio hace el *sevice discovery* mediante los label.

Una vez los pods fueron encontrados las solicitudes deben ser distribuidas entre ellos, por lo cual en este paso se realiza un balanceo de carga. Aquí es donde encontramos **la relación entre kube-proxy y los services**, ya que kube-proxy es el encargado de manejar los temas de red en los nodos, mediante IPTables, IPVS y otros mecanismos.

En resumen, el Service en Kubernetes y kube-proxy trabajan en conjunto para manejar el balanceo de carga interno y el enrutamiento del tráfico. El Service proporciona una IP virtual y un nombre DNS, mientras que kube-proxy se encarga de redirigir el tráfico hacia los Pods seleccionados utilizando reglas de red que se actualizan dinámicamente. Este enfoque garantiza que el tráfico se distribuya de manera equitativa y eficiente entre los Pods disponibles.