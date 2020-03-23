## Contiene un ejemplo simple de servicio Springboot con las siguientes características:

1) Puedes comprobar la salud del servicio en http://<dominio/ip>:8080/actuator/health

2) Incluye Dockerfile para empaquetar imagen del servicio

3) Imagen está publicada en repo Docker Hub spmoreno/labsbch:ms-arquetipo-bch-v1.0

4) Carpeta k8s contiene dos archivos:
	- deployment.yml: Contiene un objeto deployment que despliega la imagen del MS con replica=1 en puerto 8080. Hasta acá el MS es solo visible dentro del cluster.
	- service.yml: Contiene objeto service que expone el MS desplegado en el punto 4.1 hacia internet mediante el uso de un LoadBalancer (el LoadBalancer puede ser modificado para exponga localmente el MS en caso de probar con minikube o microk8s por ejemplo).
	- El LoadBalancer expondrá el puerto 80 e internamente traducirá las llamadas al pod en el puerto 8080

5) Para desplegar en un cluster K8s seguir los siguientes pasos:
	- Se requiere acceso a un cluster K8s en OCI
	- Se requiere el cli kubectl configurado para acceder al cluster
	- Desde la raíz de este repo correr el comando kubectl create -f k8s/.
	- Esperar a que el LoadBalancer sea provisionado. Se puede revisar el estado con el siguiente comando `kubectl get svc ms-v1-service -o yaml`

El resultado del comando será algo como esto:
<pre>apiVersion: v1
kind: Service
metadata:
  annotations:
    field.cattle.io/publicEndpoints: '[{"addresses":["150.136.184.65"],"port":80,"protocol":"TCP","serviceName":"default:ms-v1-service","allNodes":false}]'
  creationTimestamp: "2020-03-23T13:47:59Z"
  name: ms-v1-service
  namespace: default
  resourceVersion: "359738"
  selfLink: /api/v1/namespaces/default/services/ms-v1-service
  uid: 346512d9-ed6f-4b81-a5f3-ada3f6e20a51
spec:
  clusterIP: 10.96.140.119
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30127
    port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: ms-v1
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 150.136.184.65
</pre>

En la última línea se indica la ip pública ``150.136.184.65``.

Para probar el MS se debe ingresar a ``http://150.136.184.65/actuator/health``
La url debería responder con:
``{"status":"UP"}``

6) Para eliminar los servicios en el cluster:
	- Simplemente correr el siguiente comando desde la raíz de este repositorio: ``kubectl delete -f k8s/.``
	- Se eliminarán ambos objetos (deployment y service) y OCI se encargará de eliminar el LoadBalancer como infraestructura
