# CI/CD pipeline Jenkins, despliegue en AWS EKS

Esta guia esta diseñada para ver la interaccion entre aplicaciones que permitan gestionar el despliegue de forma automatica en la actualizacion de un Container, para ser desplegado en EKS de Aws de tal forma que permita un flujo de trabajo mas eficiente.

## Requerimientos:

    Cluster de Kubernetes EKS en AWS.
    Servidor Jenkins Server, con las siguiente herramientas:
        Jenkins Plugins.
            - Pipeline: AWS Steps.
            - Docker
            - Docker Pipeline.
        AWS CLI
            - aws configure
            - aws credencials (AWS Access Key ID and Secret Access Key).
        Kubectl

## Pasos del pipeline:

    Descarga el repositorio de github.
    Creamos la imagen con el archivo Dockerfile
    Subimos la imagen al repositorio DockerHub, Tag del build.
    Hacemos el despliegue en AWS EKS de un Deploy con la nueva imagen en su version latest.
    Nos manda una confirmacion de la direccion donde podemos observar nuestra aplicacion.
    Termina el pipeline y al cabo de unos minutos que se habilita el LoadBalancer vemos nuestra App.

## Algunos puntos a tener en cuenta:

    Configurar los permisos para que jenkins pueda leer los archivos:
        Si se colocan los archivos en el Workspace de Jenkins de forma manual.
    Agregar el usuario jenkins para que pueda ejecutar Docker.
        sudo usermod -aG docker jenkins
    Configurar AWS-CLI en la terminal y dar de alta la configuracion
        aws configure
