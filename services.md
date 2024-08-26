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


# Tipos

### ClusterIP

ClusterIP es el tipo de Service por defecto. Asigna una IP interna al Service que solo es accesible dentro del clúster. **No es accesible desde fuera del clúster**.

Ideal para servicios internos que no necesitan ser accedidos desde fuera del clúster, como microservicios que se comunican entre sí.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### NodePort

NodePort expone el Service en un puerto específico en cada nodo del clúster. Esto permite acceder al Service desde fuera del clúster usando la IP del nodo y el puerto asignado. Kubernetes selecciona un puerto en el rango de 30000 a 32767 (puertos predeterminados) para este propósito.

**Útil para pruebas** o cuando necesitas exponer un servicio de manera sencilla para acceso externo sin necesidad de un balanceador de carga. Preferible utilizar el **Ingress**


### LoadBalancer

LoadBalancer crea un balanceador de carga externo (generalmente proporcionado por el proveedor de la nube) y asigna una IP externa al Service. Esta IP puede ser utilizada para acceder al Service desde fuera del clúster.

Ideal para aplicaciones en producción que necesitan ser accesibles desde fuera del clúster y donde un balanceador de carga externo es beneficioso.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-controller
spec:
  type: LoadBalancer
  selector:
    app: ingress-controller
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
    - protocol: TCP
      port: 443
      targetPort: 443
```


### ExternalName

ExternalName permite mapear un Service a un nombre DNS externo. No crea un proxy interno ni balancea carga, simplemente actúa como un alias para un nombre DNS externo.

Utilizado para acceder a servicios externos a Kubernetes sin exponerlos directamente como servicios dentro del clúster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: externalname-service
spec:
  type: ExternalName
  externalName: example.com
```

---

Un tema importante con los services es la manera en que se pueden combinar, ya que estos son los encargados de abstraer la conectividad de los elementos en el cluster. Hay un ejemplo muy típico y muy utilizando que involucra:

1. service LoadBalancer
2. Ingress
3. service ClusterIP

```
Client
  |
  v
LoadBalancer (external IP)
  |
  v
Ingress Controller
  |
  v
Ingress Rules (routing based on hostname/path)
  |
  v
Services (ClusterIP)
  |
  v
Pods
```

El loadBalancer recibe las solicitudes de los clientes, las cuales serán posteriormente balanceadas al ingress, el cuál realizará el enrutamiento.