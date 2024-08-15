# Services

Existen por el mismo motivo que existe el autohealing. Imaginemos una situación donde tenemos un deployment y tenemos 3 replicas de nuestro servicio. Como suele pasar, uno de nuestros pod puede fallar y morir, entonces replicaset se encargará de revivirlo.

El problema aparece si suponemos que no existen los services, ya que, suponiendo que nuestros usuarios pueden conectarse directamente al pod, si este muere entonces estaremos en problemas, puesto que nada nos garantiza que cuando replicaset levante uno nuevo este vaya a tener la misma dirección IP. 