Es un recurso que define cómo se deben controlar las comunicaciones entre los Pods y/o servicios dentro de un clúster. Las Network Policies permiten especificar reglas para **permitir o denegar el tráfico de red entre Pods**, lo que es esencial para la seguridad y la segmentación de redes dentro de un clúster.

### Reglas de Tráfico Entrante y Saliente:

- **Ingress:** Define las reglas para el tráfico entrante a un Pod. Puedes especificar qué Pods pueden enviar tráfico a este Pod.

- **Egress:** Define las reglas para el tráfico saliente desde un Pod. Puedes especificar a qué Pods o direcciones IP puede enviar tráfico el Pod.

- **Selector de Pods:** Cada Network Policy tiene un selector de Pods que determina a qué Pods se aplican las reglas. Por ejemplo, podrías definir una Network Policy que se aplique a todos los Pods con una etiqueta específica.

- **Reglas de Permiso:** Las Network Policies utilizan reglas para permitir o denegar el tráfico. Puedes especificar qué IPs, puertos y protocolos se permiten o se deniegan.

- **Prioridad de las Reglas:** Las Network Policies se aplican en conjunto y no en un orden específico. Si una Network Policy permite tráfico de entrada, otra Network Policy no puede denegar ese tráfico si se aplica a los mismos Pods.

### Ejemplo de Network Policy
Aquí tienes un ejemplo básico de una Network Policy que permite el tráfico entrante solo desde Pods con una etiqueta específica y el tráfico saliente a cualquier dirección IP:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-network-policy
spec:
  podSelector:
    matchLabels:
      app: my-app
  ingress:
    - from:
      - podSelector:
          matchLabels:
            role: frontend
      ports:
      - protocol: TCP
        port: 80
  egress:
    - to:
      - ipBlock:
          cidr: 0.0.0.0/0
      ports:
      - protocol: TCP
        port: 80
```

- **podSelector:** Aplica la política a Pods con la etiqueta `app: my-app`.
- **ingress:** Permite el tráfico entrante desde Pods con la etiqueta role: frontend en el puerto TCP 80.
- **egress:** Permite el tráfico saliente a cualquier IP en el puerto TCP 80.

---

No todos los proveedores de red en Kubernetes soportan Network Policies. Asegúrate de usar un plugin de red que sea compatible con Network Policies. Algunos plugins populares que funcionan bien en AWS son Calico, Cilium y Weave

> Si estás usando Amazon EKS (Elastic Kubernetes Service), el proveedor de red predeterminado es AWS VPC CNI. **Este proveedor no soporta Network Policies por sí solo**. Sin embargo, puedes usar un complemento de red adicional, como Calico, para habilitar el soporte de Network Policies en EKS.

Instala un Plugin de Red que Soporte Network Policies:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Este manifiesto instala Calico y configura el clúster para usarlo como el complemento de red.

Una vez que tengas un plugin de red compatible, puedes definir y aplicar Network Policies como lo harías en cualquier otro clúster Kubernetes.
Puedes crear archivos YAML para las Network Policies y aplicarlas con

```sh
kubectl apply -f <archivo>.yaml.
```

Definir una Network Policy:

Crea un archivo YAML para tu Network Policy, por ejemplo network-policy.yaml
```bash
kubectl apply -f network-policy.yaml
```

Verifica la Aplicación de la Network Policy:

Puedes usar comandos como para verificar que tus políticas se están aplicando correctamente
```sh
kubectl get networkpolicies 
```

De esta manera podemos controlar la manera en que funciona la comunicación interna de nuestro cluster. Para agregar mayor seguridad de toda la aplicación siempren es posible complementar los accesos con las ACL, grupos de seguridad y policies de los proveedores cloud, en caso de estar utilizando servicios adicionales.