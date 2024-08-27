Los temas de seguridad son claves y por eso AWS tiene IAM, en donde configuramos usuarios que pueden interactuar en la cuenta AWS y los **roles y permisos**. De esta manera aseguramos quiénes pueden manipular qué recursos, y qué acciones pueden ejecutar sobre ellos.

Kubernetes tiene algo similar aplicado al contexto del cluster. Se define qué usuarios interactúan con nuestros pods

Los usuarios son los desarrolladores, administradores del cluster, auditores, operadores, **software de CI/CD** etc. Es muy importante tener configurado el sistema de permisos para saber **QUIEN ES EL CULPABLE** de realizar acciones, y sobre todo garantizar el **principio de menor privilegio**


RBAC significa Role-Based Access Control (Control de Acceso Basado en Roles)


### Conceptos clave


- **Role:** Define un conjunto de permisos dentro de un namespace específico. Por ejemplo, un Role puede permitir leer o escribir pods solo en un namespace determinado.
- **ClusterRole:** Similar al Role, pero se aplica a todos los namespaces del clúster. Se usa para permisos que deben ser válidos en todo el clúster, como la administración de nodos.
- **RoleBinding:** Asocia un Role a un usuario, grupo o cuenta de servicio en un namespace específico. Esto significa que el RoleBinding le otorga los permisos definidos en el Role a los sujetos en ese namespace.
- **ClusterRoleBinding:** Asocia un ClusterRole a un usuario, grupo o cuenta de servicio en todo el clúster. Esto significa que el ClusterRoleBinding le otorga los permisos definidos en el ClusterRole a los sujetos en todos los namespaces del clúster.
- **Sujetos:** Los sujetos son las entidades a las que se les conceden permisos. Estos pueden ser usuarios, grupos o cuentas de servicio (service accounts) en Kubernetes.
- **Permisos:** Los permisos se definen mediante reglas que especifican los recursos a los que se puede acceder y las acciones que se pueden realizar sobre esos recursos. Por ejemplo, un Role podría permitir la acción de get, list y watch en los pods.


Definimos primero los roles y luego se los asignamos a los usuarios

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-lister
  namespace: mi-namespace
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-lister-binding
  namespace: mi-namespace
subjects:
- kind: User
  name: "usuario-ejemplo"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-lister
  apiGroup: rbac.authorization.k8s.io
```


Pero ¿de dónde salen los usuarios?


En Kubernetes, los usuarios no se gestionan directamente dentro del clúster; en cambio se maneja de manera externa y Kubernetes confía en los sistemas de autenticación para validar la identidad de los usuarios. Entre estos podemos encontrar:

- Certificados TSL
- Tokens Bearer
- Autenticación básica user/password
- Integración con proveedores de identidad como OpenID Connect (OIDC), LDAP, OAuth2 etc.

