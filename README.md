# Oracle Cloud Developer Workshop - Microservice Helidon CD con Oracle Pipeline y Oracle Engine para Kubernetes (OKE)

## Antes de que empieces

Este taller le muestra cómo usar Oracle Pipeline (Wercker) para construir y enviar imágenes de Docker al Registro OCI y desplegar la imagen como un contenedor en un clúster de OCI Container Engine for Kubernetes (OKE).

*Fondo*

Como parte del proceso de desarrollo, los desarrolladores escriben un nuevo código y vuelven a fusionar su código con el maestro (código fuente). En este tutorial, el repositorio de este código fuente es GitHub.

Wercker se integra en GitHub de modo que, por ejemplo, cuando hay un compromiso (el nuevo código o los cambios se realizan en una rama o en el maestro), Wercker puede generar automáticamente una imagen de contenedor.

En el caso de un compromiso con el maestro, Wercker ejecuta una canalización y construye la imagen, empuja la imagen a OCIR y luego implementa el contenedor en una instancia de OKE, reemplazando los contenedores / pods en ejecución y, por lo tanto, actualizando la aplicación.

*¿Que necesitas?*

• Complete el tutorial Creación de un clúster con Oracle Cloud Infrastructure Container Engine para Kubernetes y despliegue de una aplicación de muestra (Si no fue previamente hecho por el instructor)

• Cuenta GitHub

• Cuenta Wercker
    * [Aplicación Wercker](https://app.wercker.com) *

> * [Parametros del taller](workshop-data.md) *


## Aplicación Clone Helidon Microservice

En esta sección, creará una aplicación Wercker de una aplicación GitHub.

1. Inicie sesión en su cuenta de GitHub. Abra la aplicación * [Helidon Microservice] (https://github.com/pasimoes/helidon-quickstart-se) * en Github y haga clic en ** Fork **.

![Fork del proyecto Helidon QuickStart SE](resources/images/helidon-quickstart-se-fork.png)

2. Ajuste el nombre de la aplicación en los siguientes archivos:

    Donde `` `quickstart-se``` incluye tu número de participante, por ejemplo` `` quickstart-se01```
    - kubernetes_deployment.yml.template
    
    ![Ajustar en la plantilla de implementación de K8s] (resources/images/kubernetes-deployment-yml-template-adjust.png)
    
    - kubernetes_service.yml.template

    ![Ajustar en la plantilla de servicio de K8s] (resources/images/kubernetes-service-yml-template-adjust.png)

    - wercker.yml
    
    ![Ajustar en Wercker yml] (resources/images/wercker-yml-adjust.png)
    
3. Seleccione el archivo * wercker.yml * para abrirlo.

4. Cualquier imagen de Docker creada por la aplicación Wercker se etiquetará con la confirmación Git correspondiente que activó su ejecución. Esta es una de las mejores prácticas de Wercker que garantiza que una revisión determinada de su fuente se incluya en una imagen de artefacto único conocida. Esto ayuda a la observabilidad, además de facilitar a Kubernetes los nuevos cambios en la aplicación. Las variables de entorno que deben pasarse a Wercker serán:
    - OCIR_USERNAME
    - OCIR_PASSWORD
    - OCIR_REPO
    - OCIR_ADDR

## Crear la aplicación Wercker

4. Abra e inicie sesión en su cuenta de Wercker. Haga clic en la opción `` `+` `` y elija ** Agregar aplicación **.

![Wercker Add Application] (resources/images/wercker-add-app.png)

5. Asegúrese de que su usuario esté seleccionado para el # 1 y GitHub esté seleccionado para el # 2 y haga clic en ** Siguiente **.

![Wercker Create New Application Step 1] (resources/images/wercker-createapp-step1.png)

6. Seleccione la aplicación `` `helidon-quickstart-se``` que bifurque previamente y haga clic en ** Siguiente **.

![Wercker Create New Application Step 2] (resources/images/wercker-createapp-step2.png)

7. Acepte el valor predeterminado para verificar el código y haga clic en ** Siguiente **.

![Wercker Create New Application Step 3] (resources/images/wercker-createapp-step3.png)

8. Haga clic en ** Crear **.

![Wercker Create New Application Step 4] (resources/images/wercker-createapp-step4.png)

9. Su aplicación fue creada exitosamente. En la siguiente sección, se definen las variables de entorno. Haga clic en la pestaña ** Entorno **.

![Wercker define las variables de entorno] (resources/images/wercker-env-01.png)

## Establecer variables de entorno de aplicación

1. Cree cada una de las siguientes variables de entorno y haga clic en Agregar después de cada una.

    - El nombre de usuario del Registro OCI debe incluir el `<nombre de tenencia> / <nombre de usuario>`
    - La contraseña de registro de OCI es el `auth_token` para su clúster. Haga clic en la casilla de verificación ** Protegido **. NOTA: No debe contener un carácter `$`.
    - OCI Registry Repo debe incluir `<region-code> .ocir.io / <tenancy name> / <registry name>`
    - Dirección de Registro OCI
    Cuando haya terminado, haga clic en la pestaña ** Ejecutar **.

![Wercker define las variables de entorno] (resources/images/wercker-env-02.png)

2. Pruebe que la aplicación se puede compilar y enviar a OCIR. Navegue a la pestaña ** Ejecuciones ** y haga clic en ** para activar un enlace de compilación ahora ** en la parte inferior de la página.

![Wercker Run 01] (resources/images/wercker-runs-01.png)

![Wercker Run 02] (resources/images/wercker-runs-02.png)

3. La construcción se completa con éxito.

![Wercker Run 03] (resources/images/wercker-runs-03.png)

## Configure el cluster para extraer imágenes del registro OCI

Para poder extraer las imágenes durante la implementación, debe configurar el clúster creando un secreto de imagen y configurando algunos parámetros adicionales en su aplicación Wercker.

1. El archivo de configuración de Kubernetes hace referencia al * secreto de imagen * utilizando la variable de entorno `` OKE_IMAGESECRET``, que debe crear como una variable de entorno en su aplicación Wercker.

    - Clave: `` OKE_IMAGESECRET`` (proporcionada por el instructor)

2. Cambie a Wercker y haga clic en la pestaña ** Entorno **.

3. Ingrese la clave `` OKE_IMAGESECRET`` y el valor `<nombre secreto>` y haga clic en ** Agregar **.

4. Repita la inclusión de parámetros:
    - Clave: `` OKE_MASTER`` (proporcionada por el instructor)
    - Clave: `` OKE_TOKEN`` (proporcionada por el instructor)

![Wercker define las variables de entorno] (resources/images/wercker-env-03.png)

5. Para revisar el script cuando se realiza un despliegue en kubernetes, cambie a GitHub y abra el archivo `` `wercker.yml```.

6. Desplácese hasta el área `` `deploy-to-kubernetes```. El primer paso es que se eliminan todas las extensiones .template. Luego moverá todos los archivos de configuración de Kubernetes a un directorio limpio para que los comandos de kubectl los consuman.

7. Estos pasos en el archivo de configuración hacen lo siguiente:
    - Establezca un tiempo de espera en la implementación de 60 segundos, lo que le da al tiempo de implementación para iniciar correctamente el contenedor de la aplicación antes de que se agote el tiempo de espera.
    - Observe el estado del despliegue hasta que todos los pods hayan subido. Si se alcanza el tiempo de espera, esto devolverá inmediatamente un código de salida distinto de cero y hará que la ejecución de la tubería falle. Esto significa que su canalización tendrá éxito solo si su aplicación se ha implementado con éxito, de lo contrario falla

![Wercker Build Automatic] (resources/images/wercker-autobuild-01.png)

## Agregar flujo de trabajo a canalización en la aplicación Wercker

Para implementar el motor de contenedor OCI para Kubernetes (OKE), debe crear un flujo de trabajo `` Deploy-to-Kubernetes``` en su aplicación Wercker.

1. Cambie a su aplicación Wercker y en la pestaña ** Flujos de trabajo **. Haga clic en Agregar * Nueva tubería *.

![Nuevo canal de flujo de trabajo de Wercker] (resources/images/wercker-new-pipeline.png)

2. Ingrese ** deploy-to-kubernetes ** para Nombre y Nombre de canalización YML y haga clic en ** Crear **.

![Canalización de configuración de flujo de trabajo de Wercker] (resources/images/wercker-config-pipeline.png)

3. Haga clic en la pestaña ** Flujos de trabajo **.

4. En el Editor de flujo de trabajo, haga clic en `'+'`, para crear una nueva cadena de tubería después de la construcción. Seleccione ** deploy-to-kubernetes ** para Ejecutar canalización y haga clic en ** Agregar **.

![Canalización Wecker Workflow Exec] (resources/images/wercker-exec-pipeline-config.png)

![Wercker Workflow Exec Pipeline Config] (resources/images/wercker-exec-pipeline-config-02.png)

5. El nuevo cambio en el flujo de trabajo fue creado con éxito. En la siguiente sección, implementa la imagen OCI en kubernetes.

## Implementar el contenedor OCI en el motor del contenedor OCI para Kubernetes

1. Cambie a la pestaña ** Runs **. Haga clic en la última tubería de ** build ** para mostrar su ejecución. On ** Actions ** button Haga clic en `` `deploy-to-kubernetes```
El canal de implementación se inicia y realiza la implementación del contenedor en OKE.

![Wercker Deploy Pipeline] (resources/images/wercker-run-deploy.png)

![Ejecución de Wercker Deploy Pipeline] (resources/images/wercker-deploy-k8s-01.png)

2. Su despliegue se completó con éxito.

![Ejecución exitosa de Wercker Deploy Pipeline 02] (resources/images/wercker-deploy-k8s-02.png)

![Ejecución exitosa de Wercker Deploy Pipeline 03] (resources/images/wercker-deploy-k8s-03.png)

![Ejecución exitosa de Wercker Deploy Pipeline 04] (resources/images/wercker-deploy-k8s-04.png)

## Servicio de verificación en el motor de contenedores OCI para gobernadores

Puede verificar el servicio ejecutando la aplicación en OCI Container Engine for Kubernetes.

> Acceso desde la terminal nativa de contenedores en OCI.
>
> Para los participantes que no tienen instalado OCI Cli y Kubectl, preparamos algunos * Terminales nativos de contenedores *.
>
> $ ssh opc @ `` <node-address> `` -i `` <ssh-key> ``
> [opc @ oracledev ~] $
>


1. Desde su ventana de terminal, ejecute lo siguiente:

    `` `
    exportar KUBECONFIG = ~ / kubeconfig
    Kubectl get services
    `` `


2. Pegue el valor de EXTERNAL-IP en su navegador para ejecutar la aplicación.

3. Arme la url para acceder a la aplicación helloworld, en la forma `` http: // <node-address>: <port-number> ``. Obtenga los valores de `` <node-address> `` y `` <port-number> `` en el Panel de Kubernetes de la siguiente manera:

    - Para averiguar el `` <node-address> ``, consulte la información sobre el pod que ejecuta la aplicación ** quickstart-se ** y obtenga la dirección del nodo que lo ejecuta en la columna NODE. Por ejemplo, 132.145.140.4.
    `` `
        $ kubectl get pods --output = wide
        NOMBRE LISTO LISTO RESTARTS EDAD NOD IP
        hello-k8s-deployment-6dcbb9998b-jps7d 1/1 Running 0 8d 10.244.1.2 132.145.140.4
        quickstart-se-7bcfd74777-8rt4b 1/1 Running 0 1h 10.244.1.11 132.145.140.4
    `` `
![kubectl show pods] (resources/images/kubectl-pods-01.png)

    - Para averiguar el `` <port-number> ``, consulte la información acerca de ** quickstart-se **, y obtenga el puerto en el que se está ejecutando el servicio desde la columna PORT (S). Por ejemplo, el puerto 30151.
    `` `
        $ kubectl get services quickstart-se
        NOMBRE TIPO CLÚSTER-IP EXTERNO-IP PUERTO (S) EDAD
        quickstart-se NodePort 10.96.137.252 <none> 8090: 30151 / TCP 1h
    `` `
![kubectl muestra los detalles del servicio] (resources/images/kubectl-service-details.png)

4. Abra una nueva ventana del navegador e ingrese la URL para acceder a la aplicación ** quickstart-se ** en el campo URL del navegador. Por ejemplo, la url completa podría verse como http://132.145.140.4:31030/greet

    Cuando el navegador carga la página, la página muestra un mensaje como:
    `{" mensaje ":" ¡Hola mundo! "}`

![microservice running 01] (resources/images/helidon-microservice-running-02.png)

## Ahora practicemos la entrega continua

La canalización se inicia automáticamente cuando realiza un cambio en uno de sus archivos de aplicación en GitHub.

1. Cambie a su aplicación GitHub y seleccione el archivo ** GreetService.java **.

![actualización de helidon 01] (resources/images/helidon-GreetService-java-01.png)

2. Edita el archivo.

3. Desplázate hasta el método `` `getDefaultMessage``` y cambia la palabra" Mundo "a ** tu mensaje **. Ingrese una descripción para confirmar y haga clic en ** Confirmar cambios **.

![actualización 02 de helidon] (resources/images/helidon-GreetService-java-02.png)

![actualización 03 de helidon] (resources/images/helidon-GreetService-java-03.png)

4. Su cambio fue cometido. Cambie a Wercker y haga clic en la pestaña ** Runs **.

    - Tenga en cuenta que la tubería se ejecutó automáticamente.
    
    ![actualización 04 de helidon] (resources/images/wercker-building-GreetService-01.png)
    
    - Una vez finalizada la compilación, se ejecuta el flujo de trabajo de despliegue

5. Su despliegue completado con éxito. Haga clic en ** deploy-to-kubernetes ** para ver los detalles.

    - Desplácese hasta la parte inferior para verificar que todos los pasos se completaron correctamente.

6. Abra una nueva ventana del navegador e ingrese la url para acceder a la aplicación ** quickstart-se ** en el campo URL del navegador. Por ejemplo, la url completa podría verse como http://132.145.140.4:31030/greet

    Cuando el navegador carga la página, la página muestra un mensaje como:
    `{" mensaje ":" ¡Hola Brasil! "}`

![microservice running 03] (resources/images/helidon-microservice-running-03.png)

¡Felicidades! Ha implementado con éxito el microservicio ** Helidon Quick Start ** en un nodo en el nuevo clúster y ha verificado que la aplicación funciona como se esperaba.
