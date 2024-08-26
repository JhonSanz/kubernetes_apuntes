Este es un componente muy importante, especialmente para las aplicaciones web. Este es el componente encargado de ser el punto de entrada de nuestra aplicación, recibiendo el tráfico y enrutandolo basados en hostname o path al lugar deseado.  

Facilita la exposición de múltiples servicios a través de un único punto de entrada.

Es muy similar al servicio AWS APIgateway

### Componentes Clave de Ingress

- **Ingress Resource**: Es la definición que especifica las reglas de enrutamiento para el tráfico entrante. Define cómo se deben dirigir las solicitudes a los diferentes servicios basados en el hostname y/o el path de la solicitud.

- **Ingress Controller**: Es un componente que implementa la lógica de enrutamiento definida en el recurso Ingress. El controlador de Ingress se encarga de interpretar las reglas del Ingress Resource y configurar el enrutamiento del tráfico a los servicios internos. Ejemplos de controladores incluyen NGINX Ingress Controller, Traefik, y HAProxy Ingress.


```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  tls:
    - hosts:
        - example.com
      secretName: tls-secret
  rules:
    - host: example.com
      http:
        paths:
          - path: /service1
            pathType: Prefix
            backend:
              service:
                name: service1
                port:
                  number: 80
          - path: /service2
            pathType: Prefix
            backend:
              service:
                name: service2
                port:
                  number: 80

```