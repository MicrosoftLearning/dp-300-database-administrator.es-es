---
lab:
  title: 'Laboratorio 13: implementación de un runbook de automatización para reconstruir índices de manera automática'
  module: Automate database tasks for Azure SQL
---

# Implementación de un runbook de automatización para reconstruir índices de manera automática

**Tiempo estimado: 30 minutos**

Te han contratado como administrador sénior de bases de datos para que ayudes a automatizar las operaciones cotidianas de administración de bases de datos. El objetivo de esta automatización es ayudarle a garantizar que las bases de datos de AdventureWorks sigan funcionando con el máximo rendimiento, así como proporcionar métodos para alertas basadas en determinados criterios. AdventureWorks utiliza SQL Server en las ofertas de infraestructura como servicio (IaaS) y plataforma como servicio (PaaS).

> &#128221; Es posible que en estos ejercicios se te pida que copies y pegues código de T-SQL y que uses los recursos de SQL existentes. Comprueba que el código se ha copiado correctamente antes de ejecutar el código.

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

## Creación de una cuenta de Automation

1. Desde la máquina virtual del laboratorio, o la máquina local si no te se proporcionó ninguna, inicia una sesión del explorador y ve a [https://portal.azure.com](https://portal.azure.com/). Conéctate al portal usando tus credenciales de Azure.

1. En la barra de búsqueda de Azure Portal, escribe *automation*, después, selecciona **Cuentas de Automation** en los resultados de la búsqueda y, después, **+ Crear**.

1. En la página **Crear una cuenta de Automation**, escribe la información siguiente y, después, selecciona **Revisar y crear**.

    - **Grupo de recursos:**&lt;tu grupo de recursos&gt;
    - **Nombre de cuenta de Automation:** autoAccount
    - **Región:** usa la predeterminada.

1. En la página Revisar, seleccione **Crear**.

    > &#128221; La cuenta de Automation puede tardar unos minutos en crearse.

## Conexión a una instancia de Azure SQL Database

1. En Azure Portal, navega a tu base de datos buscando **sql databases**.

1. Seleccione la base de datos **SQL AdventureWorksLT**.

1. En la sección principal de la página de tu base de datos, selecciona **Editor de consultas (versión preliminar)**.

1. Se te pedirán credenciales para iniciar sesión en la base de datos mediante la cuenta de administrador de la base de datos y selecciona **Aceptar**.

    Se abrirá una nueva pestaña en el navegador. Haz clic en **Agregar IP de cliente** y, después, selecciona **Guardar**. Una vez guardado, vuelve a la pestaña anterior y vuelve a seleccionar **Aceptar**.

    > &#128221; Es posible que recibas el mensaje de error *No se puede abrir el servidor "your-sql-server-name" solicitado por el inicio de sesión. No se permite que el cliente con la dirección IP "xxx.xxx.xxx.xxx" acceda al servidor.* Si es así, deberás agregar la dirección IP pública actual a las reglas de firewall de SQL Server.

    Si necesitas configurar las reglas de firewall, sigue estos pasos:

    1. Selecciona **Establecer firewall de servidor** en el menú superior de la página **Información general** de la base de datos.
    1. Selecciona **Agregar dirección IPv4 actual (xxx.xxx.xxx.xxx)** y luego selecciona **Guardar**.
    1. Una vez guardado, vuelve a la página de la base de datos **AdventureWorksLT** y vuelve a seleccionar **Editor de consultas (versión preliminar).**
    1. Se te pedirán credenciales para iniciar sesión en la base de datos mediante la cuenta de administrador de la base de datos y selecciona **Aceptar**.

1. En **Editor de consultas (versión preliminar)**, selecciona **Nueva consulta**.

1. Selecciona el icono examinar *carpeta* y ve a la carpeta **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Module13**. Selecciona el archivo **usp_AdaptiveIndexDefrag.sql** y selecciona **Abrir** y luego selecciona **Aceptar**.

1. Elimina **USE msdb** y **GO** en las líneas 5 y 6 de la consulta y selecciona **Ejecutar**.

1. Expande la carpeta **Procedimientos almacenados** para ver los procedimientos almacenados recién creados.

## Configuración de recursos de cuenta de automatización

Los pasos siguientes consisten en configurar los recursos necesarios para preparar la creación del runbook. Después, seleccione **Cuentas de Automation**.

1. En Azure Portal, en el cuadro de búsqueda superior, escribe **Automation** y selecciona **Cuentas de Automation**.

1. Selecciona la cuenta de Automation **autoAccount** que acabas de crear.

1. Selecciona **Módulos** en la sección **Recursos compartidos** de la hoja de Automation. Después, selecciona **Explorar la galería**.

1. Busca **Sqlserver** dentro de la galería.

1. Selecciona **SqlServer**, que te dirigirá a la pantalla siguiente y, después, selecciona el botón **Seleccionar**.

1. En la página **Agregar un módulo**, selecciona la versión del entorno de ejecución más reciente disponible y después, **Importar**. Esto importará el módulo de PowerShell en la cuenta de Automation.

1. Deberá crear una credencial para iniciar sesión de forma segura en la base de datos. En la hoja de la *cuenta de Automation*, ve a la sección **Recursos compartidos** y selecciona **Credenciales**.

1. Selecciona **+ Agregar una credencial**, escribe la información siguiente y después selecciona **Crear**.

    - Nombre: **SQLUser**
    - Nombre de usuario: **sqladmin**
    - Contraseña: &lt;escribe una contraseña segura, con 12 caracteres de longitud y que contenga al menos 1 letra mayúscula, 1 letra minúscula, 1 número y 1 carácter especial.&gt;
    - Confirmar contraseña: &lt;vuelve a escribir la contraseña que acabas de escribir.&gt;

## Creación de un runbook de PowerShell

1. En Azure Portal, navega a tu base de datos buscando **sql databases**.

1. Seleccione la base de datos **SQL AdventureWorksLT**.

1. En la página **Información general**, copia el **Nombre del servidor** de Azure SQL Database (el nombre del servidor debe empezar por *dp300-lab*). La pegará en los pasos posteriores.

1. En Azure Portal, en el cuadro de búsqueda superior, escribe **Automation** y selecciona **Cuentas de Automation**.

1. Selecciona la cuenta de automatización **autoAccount**.

1. Desplázate hasta la sección **Automatización de procesos** de la hoja de la cuenta de Automation y selecciona **Runbooks**.

1. Seleccione **+ Crear un runbook**.

    > &#128221; Como hemos aprendido, ten en cuenta que hay dos runbooks existentes creados. Estos se crearon automáticamente durante la implementación de la cuenta de Automation.

1. Escriba el nombre del runbook como **IndexMaintenance** y un tipo de runbook de **PowerShell**. Selecciona la versión del entorno de ejecución más reciente disponible y después selecciona **Revisión + crear**.

1. En la página **Creación del runbook**, selecciona **Crear**.

1. Una vez creado el runbook, copia y pega el fragmento de código de PowerShell siguiente en el editor de runbook. 

    > &#128221; Comprueba que el código se ha copiado correctamente antes de guardar el runbook.

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    > &#128221; Ten en cuenta que el código anterior es un script de PowerShell que ejecutará el procedimiento almacenado **usp_AdaptiveIndexDefrag** en la base de datos **AdventureWorksLT**. El script usa el cmdlet **Invoke-Sqlcmd** para conectarse al servidor SQL Server y ejecutar el procedimiento almacenado. El cmdlet **Get-AutomationPSCredential** se usa para recuperar las credenciales almacenadas en la cuenta de Automation.

1. En la primera línea del script, pega el nombre del servidor que copiaste en los pasos anteriores.

1. Seleccione **Guardar** y, a continuación, **Publicar**.

1. Selecciona **Sí** para confirmar la acción de publicación.

1. El runbook *IndexMaintenance* ahora está publicado.

## Creación de una programación para un runbook

A continuación, programará el runbook para que se ejecute de forma periódica.

1. En **Recursos**, en el panel de navegación izquierdo del runbook **IndexMaintenance**, selecciona **Programaciones**. 

1. Selecciona **+ Agregar una programación**.

1. Seleccione **Vincular una programación a su Runbook**.

1. Selecciona **+ Agregar una programación**.

1. Introduce la información y, después, selecciona **Crear**.

    - **Nombre:** DailyIndexDefrag
    - **Descripción:** desfragmentación del índice diario para la base de datos AdventureWorksLT.
    - **Comienza:** 4:00 AM (el día siguiente)
    - **Zona horaria:**&lt;selecciona la zona horaria que coincida con la ubicación.&gt;
    - **Periodicidad:** periódica
    - **Repetir cada:** 1 día
    - **Configurar expiración** No

    > &#128221; Ten en cuenta que la hora de inicio se establece a las 4:00 AM. el día siguiente. Establece la zona horaria en tu zona horaria local. La periodicidad está establecida a cada 1 día. Nunca expira.

1. Seleccione **Crear** y después seleccione **Aceptar**.

1. La programación se ha creado y ahora está vinculada al runbook. Seleccione **Aceptar**.

Azure Automation ofrece un servicio de automatización y configuración basado en la nube que admite una administración coherente en los entornos de Azure y que no son de Azure.

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

### Eliminar la carpeta LabFiles

Si creaste una nueva carpeta LabFiles para este laboratorio y ya no la necesitas, puedes eliminar la carpeta LabFiles para quitar todos los archivos creados en este laboratorio.

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, abre el Explorador de archivos y ve a la unidad **C:\\**.
1. Haz clic con el botón derecho en la carpeta **LabFiles** y selecciona **Eliminar**.
1. Selecciona **Sí** para confirmar la eliminación de la carpeta.

---

Completaste correctamente este laboratorio.

Al completar este ejercicio, has automatizado la desfragmentación de los índices en una base de datos de SQL Server para que se ejecute cada día, a las 4:00 a. m.
