# Implementaci贸n de Cloud Object Storage sobre un cluster de Openshift

Se recomienda hacer esta implementaci贸n sobre un SO Linux para agilizar la instalaci贸n de herramientas y ejecuci贸n de comandos. Estas implementaciones se realizan suponiendo que se tiene un SO Linux.

## Tabla de contenido 馃搼
1. Instalaci贸n del plugin de Cloud Object Storage sobre el cl煤ster utilizando Helm
2. Almacenamiento de la informaci贸n del Cloud Object Storage en el Cluster
3. Conexi贸n del Bucket con el cluster

## Pre-Requisitos :pencil:
* La cuenta tiene una instancia en plan Standard de Cloud Object Storage <a href="https://cloud.ibm.com/objectstorage/create"> IBM Cloud Object Storage </a>
* Haber hecho login sobre IBM Cloud desde la CLI con el siguiente comando: "ibmcloud login"
* Tener acceso al cl煤ster de Openshift mediante los comandos kubectl, de no ser as铆, ir al cl煤ster de Openshift sobre IBM Cloud, hacer click en el men煤 鈥淎ctions鈥?, elegir la opci贸n 鈥淐onnect via CLI鈥?, desplegar el token que se muestra en la nueva ventana y ejecutar, a trav茅s de una terminal, el comando que se muestra al haber desplegado el token

## Consideraciones 馃搼
* Estos comandos se pueden ejecutar desde la terminal de su computadora personal o desde la terminal de IBM Cloud
* Para copiar los comandos de esta gu铆a debe de omitir las "" que se encuentran al inicio y al final de este
* Todos los par谩metros de los comandos que se van a usar en esta gu铆a y que esten dentro de <> deben de ser modificados acorde a lo que se especifica

## 1. Instalaci贸n del plugin de Cloud Object Storage sobre el cl煤ster utilizando Helm
   
1. Ingresar a su terminal o a la terminal de IBM Cloud

2. Ingrese el siguiente comando para iniciar la instalaci贸n: "brew install helm"

3. Ingrese el siguiente comando para agregar el repositorio ibmc: "helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm"

4. Ingrese el siguiente comando para actualizar los repositorios: "helm repo update"

5. Ingrese el siguiente comando para desempaquetar el repositorio descargado: "helm fetch --untar ibm-helm/ibm-object-storage-plugin"

6. Ingrese el siguiente comando para la instalaci贸n del plugin de forma local: "helm plugin install ./ibm-object-storage-plugin/helm-ibmc"
   <br />
   <br />
   **Posibles errores**
   * ```Para sistemas operativos MAC```: Ingrese el siguiente comando para dar permisos de escritura y lectura: "chmod 755 ~/Library/helm/plugins/helm-ibmc/ibmc.sh"
   * ```Para sistemas operativos Windows```: Ingrese el siguiente comando para dar permisos de escritura y lectura: "chmod 755 ~/.helm/plugins/helm-ibmc/ibmc.sh"

7. Ingrese el siguiente comando para verificar la instalaci贸n del plugin: "helm ibmc 鈥揾elp"

8. Ingrese el siguiente comando para la instalaci贸n del plugin en el cluster: "helm ibmc install ibm-object-storage-plugin ibm-helm/ibm-object-storage-plugin --set license=true --set bucketAccessPolicy=true"
   <br />
   <br />
   **Tener en cuenta**
   * ```En caso que el cluster no se encuentre dentro de una VPC```: Modificar la porci贸n del comando "bucketAccessPolicy=true" por "bucketAccessPolicy=false" para permitir la conexi贸n del bucktet con el cluster
   * ```En caso que el cluster se encuentre dentro de una VPC```: Mantener la porci贸n del comando "bucketAccessPolicy=true"

9. Ingrese el siguiente comando para verificar los Pods creados para el plugin: "kubectl get pod --all-namespaces -o wide | grep object"
   <br />
   <br />
   **Tener en cuenta**
   * Debemos tener un Pod ejecutando y una cantidad igual a la cantidad de worker nodes de nuestro cl煤ster de ibmcloud-object-storage-driver ejecutandose

10. Ingrese el siguiente comando para verificar los Storage Class creados para el plugin: "kubectl get storageclass | grep s3" 
<br />


## 2. Almacenamiento de la informaci贸n del Cloud Object Storage en el Cluster

1. Anotar el nombre del Cloud Object Storage
   <br />
   <br />
   **Pasos para obtener el nombre del servicio de Cloud Object Storage**
   * En el portal de IBM Cloud expandir la barra lateral izquierda
   * Dentro de esta barra ingresar a la secci贸n "Resource list"
   * En el listado de los recursos ingresar al apartado "Storage"
   * Copiar y almacenar el nombre del CLoud Object Storage

2. Anotar el APIKEY de tu cuenta de IBM Cloud
   <br />
   <br />
   **Pasos para ubicar y generar una APIKEY**
   * Verificar que la cuenta de IBM Cloud posea permisos de Manager para la creaci贸n de las credenciales de acceso
   * En el portal de IBM Cloud dirigirse al bot贸n "Manage" dentro de este bot贸n ingresar a la secci贸n "Access (IAM)"
   * Dentro de la barra lateral izquierda ingresar a la secci贸n "API keys"
   * Seleccionar el bot贸n "Create +"
   * Rellenar los datos que te piden para la creaci贸n del APIKEY
   * Copiar y almacenar el APIKEY generado

3. Anotar el GUID del Cloud Object Storage
   <br />
   <br />
   **Pasos para obtener el GUID del servicio de Cloud Object Storage**
   * En el terminal donde se viene trabajando ingresar el siguiente comando para visualizar el GUID: "ibmcloud resource service-instance <service_name> | grep GUID"
   * Copiar y almacenar el GUID mostrado

4. Ingrese el siguiente comando para crear un Secret y almacenarlo en el cluster: "kubectl create secret generic <nombre_secret> --type=ibm/ibmc-s3fs --from-literal=api-key=<api_key> --from-literal=service-instance-id=<service_instance_guid>"

5. Copiar y almacenar el <nombre_secret> que se ha creado
<br />
 
## 3. Conexi贸n del Bucket con el Cluster

1. Crear el bucket desde la consola de IBM Cloud
   <br />
   <br />
   **Pasos para ubicar y crear un Bucket**
   * En el portal de IBM Cloud expandir la barra lateral izquierda
   * Dentro de esta barra ingresar a la secci贸n "Resource list"
   * En el listado de los recursos ingresar al apartado "Storage"
   * Ingresar a la instancia de CLoud Object Storage que posee
   * Dentro de la barra lateral izquierda ingresar a la secci贸n "Bucktes"
   * Seleccionar el bot贸n "Create bucket +"
   * Dentro de esta vista seleccionar el icono "->" dentro del cuadrado que posee el t铆tulo de "Quickly get started"
   * Seguir la gu铆a de configuraci贸n del Bucket
   * Copiar y almacenar el nombre del Bucket creado

2. Crear el archivo de configuraci贸n "pvc.yaml" para configurar los par谩metros del Bucket dentro del cluster
   <br />
   <br />
   **Pasos crear el archivo de configuraci贸n**
   * Descargar el archivo de configuraci贸n "pvc.yaml" que se encuentra en el repositorio
   * Modificar el archivo de configuraci贸n en base a las variables que posea, puede guiarse del archivo de configuraci贸n "pvcTemplate.yaml" que se encuentra en el repositorio
   * En caso necesite m谩s informaci贸n acerca del archivo de configuraci贸n ingrese a la siguiente <a href="https://cloud.ibm.com/docs/openshift?topic=openshift-storage_cos_apps&mhsrc=ibmsearch_a&mhq=Persistent+Volume+Claim"> documentaci贸n </a>
   * Guardar los cambios realizados en el archivo de configuraci贸n "pvc.yaml"
   * Ingresar al terminal donde se viene trabajando e ingrese el siguiente comando para ejecutar el yaml en el cluster: "kubectl apply -f pvc.yaml"

3. Corroborar que el proceso se realiz贸 de manera correcta ejecutando el siguiente comando: "kubectl get pvc"
   <br />
   <br />
   **Tener en cuenta**
   * El estado que debe de tener el persistent volume debe de ser 鈥淏OUND鈥? 
<br />

## Referencias :mag:
* <a href="https://cloud.ibm.com/objectstorage/create"> IBM Cloud Object Storage in IBM Portal</a>. 
* <a href="https://cloud.ibm.com/docs/openshift?topic=openshift-storage_cos_apps&mhsrc=ibmsearch_a&mhq=Persistent+Volume+Claim"> Adding object storage to apps </a>. 

## Autores :black_nib:
Italo Silva IBM Cloud Tech Sales Per煤.
