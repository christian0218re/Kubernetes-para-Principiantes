# Introducción a Kubernetes

<p align=center>
<img src="img/Kubernetes-Logo.jpg" width="500" >
</p>

Kubernetes es una plataforma portátil, extensible y de código abierto para gestionar cargas de trabajo y servicios en contenedores, que facilita tanto la configuración declarativa como la automatización. Tiene un ecosistema grande y de rápido crecimiento. Los servicios, el soporte y las herramientas de Kubernetes están ampliamente disponibles.



# Alcance de este curso:

- Retrocidiendo en el tiempo
- Arquitectura
- Instalación
- Contextos
- Imperativo vs Declarativo
- Namespaces
- Pod
- Selectores
- Pods Multi-Contenedores
- ReplicaSet
- Deployments
- Stateful
- Job
- CronJob
- Rolling Updates
- Despliegue azul y verde
- ClusterIP
- NodePort
- LoadBalancer
- Persistence Volumen
- ConfigMap
- Secrets
- LivenessProbe

# Retrocediendo en el timepo

<p align=center>
<img src="img/2.png" width="600" >
</p>

 - **Era de implementación tradicional**: Al principio, las organizaciones ejecutaban aplicaciones en servidores físicos. No había forma de definir límites de recursos para las aplicaciones en un servidor físico y esto causaba problemas de asignación de recursos. Por ejemplo, si se ejecutan varias aplicaciones en un servidor físico, puede haber casos en los que una aplicación consumiría la mayor parte de los recursos y, como resultado, las otras aplicaciones tendrían un rendimiento inferior. Una solución para esto sería ejecutar cada aplicación en un servidor físico diferente. Pero esto no creció porque los recursos estaban infrautilizados y a las organizaciones les resultaba costoso mantener muchos servidores físicos.

 - **Era de la implementación virtualizada**: Como solución se introdujo la virtualización. Le permite ejecutar múltiples máquinas virtuales (VM) en la CPU de un único servidor físico. La virtualización permite aislar las aplicaciones entre máquinas virtuales y proporciona un nivel de seguridad ya que otra aplicación no puede acceder libremente a la información de una aplicación. 
 
    La virtualización permite una mejor utilización de los recursos en un servidor físico y permite una mejor escalabilidad porque una aplicación se puede agregar o actualizar fácilmente, reduce los costos de hardware y mucho más. Con la virtualización puedes presentar un conjunto de recursos físicos como un grupo de máquinas virtuales desechables.

    Cada VM es una máquina completa que ejecuta todos los componentes, incluido su propio sistema operativo, sobre el hardware virtualizado.

 -  **Era de implementación de contenedores**: Los contenedores son   similares a las máquinas virtuales, pero tienen propiedades de  aislamiento relajadas para compartir el sistema operativo (SO) entre las aplicaciones. Por tanto, los contenedores se consideran ligeros. Al igual que una máquina virtual, un contenedor tiene su propio sistema de archivos, CPU compartida, memoria, espacio de proceso y más. Como están desacoplados de la infraestructura subyacente, son portátiles a través de nubes y distribuciones de sistema operativo. 

# Arquitectura
Cuando se despliega Kubernetes, se obtiene un clúster.

Un clúster Kubernetes consiste en un conjunto de máquinas de trabajo, llamadas nodos, que ejecutan aplicaciones en contenedores. Cada clúster tiene al menos un nodo trabajador.

El nodo(s) trabajador(es) aloja(n) los Pods que son los componentes de la carga de trabajo de la aplicación. El plano de control gestiona los nodos trabajadores y los Pods en el cluster. En entornos de producción, el plano de control suele ejecutarse en varios ordenadores y un clúster suele ejecutar varios nodos, lo que proporciona tolerancia a fallos y alta disponibilidad.

<p align=center>
<img src="images/k8s/3.svg" width="1000" >
</p>

### Componentes del plano de control

Los componentes del plano de control toman decisiones globales sobre el clúster (por ejemplo, programación), así como detectan y responden a eventos del clúster (por ejemplo, iniciar un nuevo pod cuando el campo de réplicas de un despliegue no está satisfecho).

Los componentes del plano de control pueden ejecutarse en cualquier máquina del cluster. Sin embargo, por simplicidad, los scripts de configuración normalmente inician todos los componentes del plano de control en la misma máquina, y no ejecutan contenedores de usuario en esta máquina. Vea Creando clusters altamente disponibles con kubeadm para un ejemplo de configuración del plano de control que corre a través de múltiples máquinas.

### - kube-apiserver
El servidor API es un componente del plano de control de Kubernetes que expone la API de Kubernetes. El servidor API es la interfaz del plano de control de Kubernetes.

La principal implementación de un servidor API de Kubernetes es kube-apiserver. kube-apiserver está diseñado para escalar horizontalmente, es decir, escala desplegando más instancias. Puede ejecutar varias instancias de kube-apiserver y equilibrar el tráfico entre esas instancias.

### - etcd
Almacén de valores clave consistente y de alta disponibilidad utilizado como almacén de respaldo de Kubernetes para todos los datos del clúster.

### - kube-scheduler
Componente del plano de control que busca Pods recién creados sin nodo asignado, y selecciona un nodo para que se ejecuten.

### - kube-controller-manager
Componente del plano de control que ejecuta procesos de controlador. Lógicamente, cada controlador es un proceso independiente, pero para reducir la complejidad, todos se compilan en un único binario y se ejecutan en un único proceso.

Hay muchos tipos diferentes de controladores. Algunos ejemplos son:

- Controlador de nodos: Se encarga de avisar y responder cuando los nodos se caen.
- Controlador de trabajos: Busca objetos Job que representan tareas puntuales y crea Pods para ejecutar esas tareas hasta su finalización.
- Controlador EndpointSlice: Rellena los objetos EndpointSlice (para proporcionar un vínculo entre los servicios y los pods).
- Controlador ServiceAccount: Crea ServiceAccounts por defecto para nuevos espacios de nombres.

### Componentes de nodo
Los componentes de nodo se ejecutan en cada nodo, manteniendo los pods en ejecución y proporcionando el entorno de ejecución de Kubernetes.

### - kubelet
Un agente que se ejecuta en cada nodo del clúster. Se asegura de que los contenedores se están ejecutando en un Pod.

El kubelet toma un conjunto de PodSpecs que se proporcionan a través de diversos mecanismos y se asegura de que los contenedores descritos en esos PodSpecs se están ejecutando y en buen estado. El kubelet no gestiona contenedores que no hayan sido creados por Kubernetes.

### - kube-proxy
kube-proxy es un proxy de red que se ejecuta en cada nodo de su clúster, implementando parte del concepto de Servicio Kubernetes.

kube-proxy mantiene reglas de red en los nodos. Estas reglas de red permiten la comunicación de red a sus Pods desde sesiones de red dentro o fuera de su cluster.

kube-proxy utiliza la capa de filtrado de paquetes del sistema operativo si existe y está disponible. De lo contrario, kube-proxy reenvía el tráfico por sí mismo.

### - Runtime del contenedor
Componente fundamental que permite a Kubernetes ejecutar contenedores de forma eficaz. Se encarga de gestionar la ejecución y el ciclo de vida de los contenedores dentro del entorno Kubernetes.

# Instalación de Kubernetes

 Para crear un cluster de `Kubernetes` localmente utilizaremos `minikube`.  Es una herramienta de código abierto que permite a los usuarios crear y administrar clústeres de Kubernetes en entornos locales o virtuales. Facilita el desarrollo, las pruebas y el aprendizaje de Kubernetes al proporcionar un entorno local que simula las características de producción, sin requerir acceso a un clúster completo en la nube o en un entorno de producción.
 
 ```consola
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

$ minikube start

$ minikube dashboard
 ```

Probemos que la instalación de minikube fue exitosa:

    kubectl cluster-info

# Contextos

Los contextos son atajos que le permiten cambiar rápidamente entre diferentes configuraciones de clúster sin escribir comandos largos y tediosos. Los detalles del contexto se almacenan en el archivo kubeconfig , un archivo de configuración YAML utilizado por Kubernetes para autenticarse y conectarse al servidor API de Kubernetes. El archivo normalmente se encuentra en el directorio de inicio del usuario.

1. Obtengamos el contexto actual:

       kubectl config current-context

2. Listamos todos los contextos:

       kubectl config get-contexts

3. Usar un contexto:

       kubectl config use-context [contextName]

4. Renombrar contexto:

       kubectl config rename-context [old-name] [new-name]

5. Eliminar contexto:

       kubectl config delete-context [contextName]