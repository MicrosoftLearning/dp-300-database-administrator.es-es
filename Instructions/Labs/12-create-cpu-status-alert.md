---
lab:
  title: "Laboratorio 12: creación de una alerta de estado de CPU para SQL\_Server"
  module: Automate database tasks for Azure SQL
---

# Creación de una alerta de estado de CPU para SQL Server en Azure

**Tiempo estimado**: 20 minutos

Le han contratado como ingeniero sénior de datos para que ayude a automatizar las operaciones cotidianas de administración de bases de datos. El objetivo de esta automatización es ayudarle a garantizar que las bases de datos de AdventureWorks sigan funcionando con el máximo rendimiento, así como proporcionar métodos para alertas basadas en determinados criterios.

> &#128221; En estos ejercicios se te pide tanto que copies y pegues código de T-SQL como que uses los recursos de SQL existentes. Comprueba que el código se ha copiado correctamente antes de ejecutar el código.

## Configura el entorno

Si la máquina virtual de laboratorio se proporcionó y está preconfigurada, deberías encontrar los archivos de laboratorio listos en la carpeta **C:\LabFiles**. *Dedica un momento a comprobar si los archivos ya están allí, y omite esta sección*. Sin embargo, si usas tu propia máquina o faltan los archivos para laboratorio, deberás clonarlos desde *GitHub* para continuar.

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, inicia una sesión de Visual Studio Code.

1. Abre la paleta de comandos (CTRL+Mayús+P) y escribe **Git: Clone**. Selecciona la opción **Git: Clone**.

1. Pega la siguiente dirección URL en el campo **Repository URL** y selecciona **Enter**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Guarda el repositorio en la carpeta **C:\LabFiles** en la máquina virtual del laboratorio o en tu máquina local si no se te proporcionó ninguna (crea la carpeta si no existe).

## Configuración de SQL Server en Azure

Inicia sesión en Azure y comprueba si tienes una instancia de Azure SQL Server existente que se ejecuta en Azure. *Omite esta sección si ya tienes una instancia de SQL Server que se ejecuta en Azure*.

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, inicia una sesión de Visual Studio Code y ve al repositorio clonado de la sección anterior.

1. Haz clic con el botón derecho del ratón en la carpeta **/Allfiles/Labs** y selecciona **Open in Integrated Terminal**.

1. Conéctate a Azure mediante la CLI de Azure. Escribe el comando siguiente y selecciona **Enterr**.

    ```bash
    az login
    ```

    > &#128221; Ten en cuenta que se abrirá una ventana del explorador. Use sus credenciales de Azure para iniciar sesión.

1. Una vez que hayas iniciado sesión en Azure, es el momento de crear un grupo de recursos, si aún no existe, y crear un servidor SQL Server y una base de datos en ese grupo de recursos. Escribe el comando siguiente y selecciona **Enterr**. *Este script tarda unos minutos en completarse*.

    ```bash
    cd ./Setup
    ./deploy-sql-database.ps1
    ```

    > &#128221; Ten en cuenta que, de forma predeterminada, este script creará o un grupo de recursos denominado **contoso-rg**, o usará un recurso cuyo nombre empiece por *contoso-rg*, si existe. De forma predeterminada, también creará todos los recursos de la región **Oeste de EE. UU. 2** (westus2). Por último, se generará una contraseña aleatoria de 12 caracteres para la **contraseña de administrador de SQL**. Puedes cambiar estos valores mediante uno o varios de los parámetros **-rgName**, **-location** y **-sqlAdminPw** con tus propios valores. La contraseña tendrá que cumplir los requisitos de complejidad de la contraseña de Azure SQL, al menos 12 caracteres de longitud y contener al menos 1 letra mayúscula, 1 letra minúscula, 1 número y 1 carácter especial.

    > &#128221; Ten en cuenta que el script agregará la dirección IP pública actual a las reglas de firewall de SQL Server.

1. Una vez completado el script, devolverá el nombre del grupo de recursos, el nombre de SQL Server y el nombre de la base de datos, y el nombre de usuario y la contraseña del administrador. *Toma nota de estos valores, ya que los necesitarás más adelante en el laboratorio*.

---

## Creación de una alerta cuando una CPU excede un promedio de 80 %

1. Desde la máquina virtual del laboratorio, o la máquina local si no te se proporcionó ninguna, inicia una sesión del explorador y ve a [https://portal.azure.com](https://portal.azure.com/). Conéctate al portal usando tus credenciales de Azure.

1. Desde Azure Portal, en la barra de búsqueda de la parte superior, escribe **Bases de datos de SQL** y selecciona **Bases de datos SQL**. Selecciona el nombre de la base de datos **AdventureWorksLT** que se muestra.

1. En la hoja principal de la base de datos **AdventureWorksLT**, desplázate hacia abajo hasta la sección Supervisión. Seleccione **Alertas**.

1. Seleccione **Crear regla de alertas**.

1. En la página **Crear una regla de alertas**, selecciona **Porcentaje de CPU**.

1. En la sección **Lógica de alertas**, selecciona **Estático** para el **Tipo de umbral**. Comprueba que el tipo de **Agregación** sea **Promedio** y que la propiedad **Valor es** sea **Mayor que**. A continuación, en **Umbral**, escribe el valor **80**. Revisa los valores *check every* y *lookback period*.

1. Selecciona **Siguiente: Acciones**.

1. En la pestaña **Acciones**, selecciona **Crear grupo de acciones**.

1. En la pantalla **Grupo de acciones**, escribe **emailgroup** en los campos **Nombre del grupo de acciones** y **Nombre para mostrar** y, después, selecciona **Siguiente: notificaciones**.

1. En la pestaña **Notificaciones**, ingresa la siguiente información:

    - **Tipo de notificación:** correo electrónico, mensaje SMS, notificación push o mensaje de voz

        > &#128221; Al seleccionar esta opción, aparecerá un control flotante de correo electrónico, mensaje SMS, notificación push o mensaje de voz. Comprueba la propiedad Email y escribe el nombre de usuario de Azure con el que inició sesión. Seleccione **Aceptar**.

    - **Nombre:** DemoLab

1. Seleccione **Revisar y crear** y, luego, **Crear**.

1. De nuevo en la página **Crear una regla de alertas**, selecciona **Siguiente: Detalles** y asigna un nombre único a la regla de alertas.

1. Seleccione **Revisar y crear** y, luego, **Crear**.

1. Con la alerta configurada, si el uso medio de la CPU supera el 80 %, se envía un correo electrónico.

---

## Recursos de limpieza

Si no usas Azure SQL Server para ningún otro propósito, puedes limpiar los recursos que creaste en este laboratorio.

### Eliminar el grupo de recursos

Si creaste un nuevo grupo de recursos para este laboratorio, puedes eliminar el grupo de recursos para quitar todos los recursos creados en este laboratorio.

1. En Azure Portal, selecciona **Grupos de recursos** en el panel de navegación izquierdo o busca **Grupos de recursos** en la barra de búsqueda y selecciónalo desde los resultados.

1. Ve al grupo de recursos que creaste para este laboratorio. El grupo de recursos contendrá Azure SQL Server y otros recursos creados en este laboratorio.

1. Seleccione **Eliminar grupo de recursos** del menú superior.

1. En el panel **Eliminar un grupo de recursos**, escribe el nombre del grupo de recursos para confirmarlo y selecciona **Eliminar**.

1. Espera a que se elimine el grupo de recursos.

1. Cierra Azure Portal.

### Eliminar solo los recursos del laboratorio

Si no creaste ningún grupo de recursos nuevo para este laboratorio y deseas dejar intacto el grupo de recursos y sus recursos anteriores, aún puedes eliminar los recursos creados en este laboratorio.

1. En Azure Portal, selecciona **Grupos de recursos** en el panel de navegación izquierdo o busca **Grupos de recursos** en la barra de búsqueda y selecciónalo desde los resultados.

1. Ve al grupo de recursos que creaste para este laboratorio. El grupo de recursos contendrá Azure SQL Server y otros recursos creados en este laboratorio.

1. Selecciona todos los recursos con el prefijo del nombre de SQL Server que especificaste anteriormente en el laboratorio.

1. Seleccione **Eliminar** en el menú superior.

1. En el cuadro de diálogo **Eliminar recursos**, escribe **delete** y selecciona **Eliminar**.

1. Para confirmar la eliminación, escribe el nombre del recurso y selecciona **Eliminar**.

1. Espera a que se eliminen los recursos.

1. Cierra Azure Portal.

---

Completaste correctamente este laboratorio.

Las alertas pueden enviarte un correo electrónico o llamar a un webhook cuando alguna métrica (por ejemplo, el tamaño de la base de datos o el uso de la CPU) alcanza el umbral definido. Acabas de ver cómo puedes configurar fácilmente alertas para bases de datos de Azure SQL.
