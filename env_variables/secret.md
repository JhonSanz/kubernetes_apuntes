# ¿Qué son?
Secrets se utilizan para almacenar datos sensibles como contraseñas, tokens de acceso o claves SSH. Los Secrets permiten gestionar estos datos de manera segura, evitando que aparezcan en el archivo de configuración en texto plano.

¿Cómo se Configuran?
Definición de un Secret: Puedes crear un Secret utilizando un archivo YAML o a través de la línea de comandos.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  password: dG9wU2VjcmV0UGFzcw==  # "topSecretPass" en base64
```

Y una vez definido se puede utilizar en un deployment

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
      - name: my-app-container
        image: my-app:v1
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
```