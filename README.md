# Oracle Cloud Developer Workshop - Microservice Helidon CD con Oracle Pipeline y Oracle Engine para Kubernetes (OKE)

## Antes de que empieces

Este taller le muestra cómo usar Oracle Pipeline (Wercker) para construir y enviar imágenes de Docker al Registro OCI y desplegar la imagen como un contenedor en un clúster de OCI Container Engine for Kubernetes (OKE).

*Fondo*

Como parte del proceso de desarrollo, los desarrolladores escriben un nuevo código y vuelven a fusionar su código con el maestro (código fuente). En este tutorial, el repositorio de este código fuente es GitHub.

Wercker se integra en GitHub de modo que, por ejemplo, cuando hay un compromiso (el nuevo código o los cambios se realizan en una rama o en el maestro), Wercker puede generar automáticamente una imagen de contenedor.

En el caso de un compromiso con el maestro, Wercker ejecuta una canalización y construye la imagen, empuja la imagen a OCIR y luego implementa el contenedor en una instancia de OKE, reemplazando los contenedores / pods en ejecución y, por lo tanto, actualizando la aplicación.

*¿Que necesitas?*

• Complete el tutorial Creación de un clúster con Oracle Cloud Infrastructure Container Engine para Kubernetes y despliegue de una aplicación de muestra

• Cuenta GitHub

• Cuenta Wercker
    * [Aplicación Wercker] (https://app.wercker.com) *

> * [Parametros del taller] (workshop-data.md) *


## Aplicación Clone Helidon Microservice

En esta sección, creará una aplicación Wercker de una aplicación GitHub.

1. Inicie sesión en su cuenta de GitHub. Abra la aplicación * [Helidon Microservice] (https://github.com/pasimoes/helidon-quickstart-se) * en Github y haga clic en ** Fork **.

! [Horquilla del proyecto Helidon QuickStart SE] (recursos / imágenes / helidon-quickstart-se-fork.png)

2. Ajuste el nombre de la aplicación en los siguientes archivos:

    Donde `` `quickstart-se``` incluye tu número de participante, por ejemplo` `` quickstart-se01```
    - kubernetes_deployment.yml.template
    
    ! [Ajustar en la plantilla de implementación de K8s] (resources / images / kubernetes-deployment-yml-template-adjust.png)
    
    - kubernetes_service.yml.template

    ! [Ajustar en la plantilla de servicio de K8s] (resources / images / kubernetes-service-yml-template-adjust.png)

    - wercker.yml
    
    ! [Ajustar en Wercker yml] (resources / images / wercker-yml-adjust.png)
    
3. Seleccione el archivo * wercker.yml * para abrirlo.

4. Cualquier imagen de Docker creada por la aplicación Wercker se etiquetará con la confirmación Git correspondiente que activó su ejecución. Esta es una de las mejores prácticas de Wercker que garantiza que una revisión determinada de su fuente se incluya en una imagen de artefacto único conocida. Esto ayuda a la observabilidad, además de facilitar a Kubernetes los nuevos cambios en la aplicación. Las variables de entorno que deben pasarse a Wercker serán:
    - OCIR_USERNAME
    - OCIR_PASSWORD
    - OCIR_REPO
    - OCIR_ADDR

## Crear la aplicación Wercker

4. Abra e inicie sesión en su cuenta de Wercker. Haga clic en la opción `` `+` `` y elija ** Agregar aplicación **.

! [Wercker Add Application] (recursos / imágenes / wercker-add-app.png)

5. Asegúrese de que su usuario esté seleccionado para el # 1 y GitHub esté seleccionado para el # 2 y haga clic en ** Siguiente **.

! [Wercker Create New Application Step 1] (recursos / imágenes / wercker-createapp-step1.png)

6. Seleccione la aplicación `` `helidon-quickstart-se``` que bifurque previamente y haga clic en ** Siguiente **.

! [Wercker Create New Application Step 2] (recursos / imágenes / wercker-createapp-step2.png)

7. Acepte el valor predeterminado para verificar el código y haga clic en ** Siguiente **.

! [Wercker Create New Application Step 3] (recursos / imágenes / wercker-createapp-step3.png)

8. Haga clic en ** Crear **.

! [Wercker Create New Application Step 4] (recursos / imágenes / wercker-createapp-step4.png)

9. Su aplicación fue creada exitosamente. En la siguiente sección, se definen las variables de entorno. Haga clic en la pestaña ** Entorno **.

! [Wercker define las variables de entorno] (resources / images / wercker-env-01.png)

## Establecer variables de entorno de aplicación

1. Cree cada una de las siguientes variables de entorno y haga clic en Agregar después de cada una.

    - El nombre de usuario del Registro OCI debe incluir el `<nombre de tenencia> / <nombre de usuario>`
    - La contraseña de registro de OCI es el `auth_token` para su clúster. Haga clic en la casilla de verificación ** Protegido **. NOTA: No debe contener un carácter `$`.
    - OCI Registry Repo debe incluir `<region-code> .ocir.io / <tenancy name> / <registry name>`
    - Dirección de Registro OCI
    Cuando haya terminado, haga clic en la pestaña ** Ejecutar **.

! [Wercker define las variables de entorno] (resources / images / wercker-env-02.png)

2. Pruebe que la aplicación se puede compilar y enviar a OCIR. Navegue a la pestaña ** Ejecuciones ** y haga clic en ** para activar un enlace de compilación ahora ** en la parte inferior de la página.

! [Wercker Run 01] (recursos / imágenes / wercker-runs-01.png)

! [Wercker Run 02] (recursos / imágenes / wercker-runs-02.png)

3. La construcción se completa con éxito.

! [Wercker Run 03] (recursos / imágenes / wercker-runs-03.png)

## Configure el cluster para extraer imágenes del registro OCI

Para poder extraer las imágenes durante la implementación, debe configurar el clúster creando un secreto de imagen y configurando algunos parámetros adicionales en su aplicación Wercker.
