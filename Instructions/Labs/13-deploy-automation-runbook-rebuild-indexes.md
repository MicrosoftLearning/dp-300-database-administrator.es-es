---
lab:
  title: 'Laboratorio 13: implementación de un runbook de automatización para reconstruir índices de manera automática'
  module: Automate database tasks for Azure SQL
---

# Implementación de un runbook de automatización para reconstruir índices de manera automática

**Tiempo estimado: 30 minutos**

Te han contratado como administrador sénior de bases de datos para que ayudes a automatizar las operaciones cotidianas de administración de bases de datos. El objetivo de esta automatización es ayudarle a garantizar que las bases de datos de AdventureWorks sigan funcionando con el máximo rendimiento, así como proporcionar métodos para alertas basadas en determinados criterios. AdventureWorks utiliza SQL Server en las ofertas de infraestructura como servicio (IaaS) y plataforma como servicio (PaaS).

**Nota:** En estos ejercicios se le puede pedir tanto que copie y pegue código de T-SQL como que use los recursos de SQL existentes. Comprueba que el código se ha copiado correctamente antes de ejecutar el código.

## Creación de una cuenta de Automation

1. En la máquina virtual del laboratorio, inicia una sesión del explorador y desplázate a [https://portal.azure.com](https://portal.azure.com/). Conéctate al Portal con el **Nombre de usuario** y la **Contraseña** de Azure proporcionados en la pestaña **Recursos** de esta máquina virtual de laboratorio.

    ![Captura de pantalla de la página de inicio de sesión de Azure Portal](../images/dp-300-module-01-lab-01.png)

1. En la barra de búsqueda de Azure Portal, escribe *automation*, después, selecciona **Cuentas de Automation** en los resultados de la búsqueda y, después, **+ Crear**.

    ![Captura de pantalla de la selección de cuentas de Automation.](../images/dp-300-module-13-lab-01.png)

1. En la página **Crear una cuenta de Automation**, escribe la información siguiente y, después, selecciona **Revisar y crear**.

    - **Grupo de recursos:**&lt;tu grupo de recursos&gt;
    - **Nombre:** autoAccount
    - **Ubicación:** usa la ubicación predeterminada.

    ![Captura de pantalla de la pantalla Agregar cuenta de Automation.](../images/dp-300-module-13-lab-02.png)

1. En la página Revisar, seleccione **Crear**.

    ![Captura de pantalla de la pantalla Agregar cuenta de Automation.](../images/dp-300-module-13-lab-29.png)

    > [!NOTE]
    > La cuenta de Automation debe crearse en unos tres minutos.

## Conexión a una instancia de Azure SQL Database

1. En Azure Portal, navega a tu base de datos buscando **sql databases**.

    ![Captura de pantalla de búsqueda de bases de datos SQL existentes.](../images/dp-300-module-13-lab-03.png)

1. Selecciona la base de datos SQL **AdventureWorksLT**.

    ![Captura de pantalla que muestra la selección de la base de datos SQL AdventureWorks.](../images/dp-300-module-13-lab-04.png)

1. En la sección principal de la página de tu base de datos, selecciona **Editor de consultas (versión preliminar)**.

    ![Captura de pantalla de la selección del Editor de consultas (versión preliminar).](../images/dp-300-module-13-lab-05.png)

1. Se le pedirán las credenciales para iniciar sesión en la base de datos. Utiliza esta credencial:

    - **Inicio de sesión:** sqladmin
    - **Contraseña**: P@ssw0rd01

1. Debería recibir el siguiente mensaje de error:

    ![Captura de pantalla del error de inicio de sesión.](../images/dp-300-module-13-lab-06.png)

1. Selecciona el vínculo **Permitir IP ...** proporcionado al final del mensaje de error que se muestra anteriormente. Esto agregará automáticamente la dirección IP del cliente como entrada de regla de firewall para SQL Database.

    ![Captura de pantalla de la creación de regla de firewall.](../images/dp-300-module-13-lab-07.png)

1. Vuelve al editor de consultas y selecciona **Aceptar** para iniciar sesión en la base de datos.

1. Abre una nueva pestaña en el explorador y desplázate a la página de GitHub para acceder al script [**AdaptativeIndexDefragmentation**](https://github.com/microsoft/tigertoolbox/blob/master/AdaptiveIndexDefrag/usp_AdaptiveIndexDefrag.sql). Después, selecciona **Sin procesar**.

    ![Captura de pantalla de selección de Raw en GitHub.](../images/dp-300-module-13-lab-08.png)

    Esto proporcionará el código en un formato en el que puede copiarlo. Seleccione todo el texto (<kbd>CTRL</kbd> + <kbd>A</kbd>) y cópielo en el portapapeles (<kbd>CTRL</kbd> + <kbd>C</kbd>).

    >[!NOTE]
    > La finalidad de este script es realizar una desfragmentación inteligente en uno o varios índices, así como la actualización de estadísticas necesaria, para una o varias bases de datos.

1. Cierre la pestaña del explorador de GitHub y vuelva a Azure Portal.

1. Pega el texto que copiaste en el panel **Consulta 1**.

    ![Captura de pantalla del pegado del código en una nueva ventana de consulta.](../images/dp-300-module-13-lab-09.png)

1. Elimine `USE msdb` y `GO` en las líneas 5 y 6 de la consulta (que se resaltan en la captura de pantalla) y seleccione **Ejecutar**.

1. Expanda la carpeta **Procedimientos almacenados** para ver lo que se ha creado.

    ![Captura de pantalla de los nuevos procedimientos almacenados.](../images/dp-300-module-13-lab-10.png)

## Configuración de recursos de cuenta de automatización

Los pasos siguientes consisten en configurar los recursos necesarios para preparar la creación del runbook. Después, seleccione **Cuentas de Automation**.

1. En Azure Portal, en el cuadro de búsqueda superior, escriba **Automation**.

    ![Captura de pantalla de la selección de cuentas de Automation.](../images/dp-300-module-13-lab-11.png)

1. Seleccione la cuenta de Automation que acaba de crear.

    ![Captura de pantalla de la selección de la cuenta de Automation.](../images/dp-300-module-13-lab-12.png)

1. Selecciona **Módulos** en la sección **Recursos compartidos** de la hoja de Automation. Después, selecciona **Explorar la galería**.

    ![Captura de pantalla de la selección del menú Módulos.](../images/dp-300-module-13-lab-13.png)

1. Busque **sqlserver** dentro de la galería.

    ![Captura de pantalla de la selección del módulo SqlServer.](../images/dp-300-module-13-lab-14.png)

1. Selecciona **SqlServer**, se te dirigirá a la pantalla siguiente y, después, selecciona **Seleccionar**.

    ![Captura de pantalla de la selección de Seleccionar.](../images/dp-300-module-13-lab-15.png)

1. En la página **Agregar un módulo**, selecciona la versión del entorno de ejecución más reciente disponible y después, **Importar**. Esto importará el módulo de PowerShell en la cuenta de Automation.

    ![Captura de pantalla de la selección de Importar.](../images/dp-300-module-13-lab-16.png)

1. Deberá crear una credencial para iniciar sesión de forma segura en la base de datos. En la hoja de la cuenta de Automation, vaya a la sección **Recursos compartidos** y seleccione **Credenciales**.

    ![Captura de pantalla de la selección de la opción Credenciales.](../images/dp-300-module-13-lab-17.png)

1. Selecciona **+ Agregar una credencial**, escribe la información siguiente y después selecciona **Crear**.

    - Nombre: **SQLUser**
    - Nombre de usuario: **sqladmin**
    - Contraseña: **P@ssw0rd01**
    - Confirmar contraseña: **P@ssw0rd01**

    ![Captura de pantalla de la adición de credenciales de cuenta.](../images/dp-300-module-13-lab-18.png)

## Creación de un runbook de PowerShell

1. En Azure Portal, navega a tu base de datos buscando **sql databases**.

    ![Captura de pantalla de búsqueda de bases de datos SQL existentes.](../images/dp-300-module-13-lab-03.png)

1. Selecciona la base de datos SQL **AdventureWorksLT**.

    ![Captura de pantalla que muestra la selección de la base de datos SQL AdventureWorks.](../images/dp-300-module-13-lab-04.png)

1. En la página **Información general**, copia el **Nombre del servidor** de Azure SQL Database tal y como se muestra a continuación (el nombre del servidor debe empezar por *dp300-lab*). La pegará en los pasos posteriores.

    ![Captura de pantalla de la copia del nombre del servidor.](../images/dp-300-module-13-lab-19.png)

1. En Azure Portal, en el cuadro de búsqueda superior, escriba **Automation**.

    ![Captura de pantalla de la selección de cuentas de Automation.](../images/dp-300-module-13-lab-11.png)

1. Seleccione la cuenta de Automation que acaba de crear.

    ![Captura de pantalla de la selección de la cuenta de Automation.](../images/dp-300-module-13-lab-12.png)

1. Desplázate hasta la sección **Automatización de procesos** de la hoja de la cuenta de Automation, selecciona **Runbooks** y, después, **+ Crear un runbook**.

    ![Captura de pantalla de la página Runbooks, con la selección de Crear un runbook.](../images/dp-300-module-13-lab-20.png)

    >[!NOTE]
    > Como hemos aprendido, ten en cuenta que hay dos runbooks existentes creados. Estos se crearon automáticamente durante la implementación de la cuenta de Automation.

1. Escriba el nombre del runbook como **IndexMaintenance** y un tipo de runbook de **PowerShell**. Selecciona la versión del entorno de ejecución más reciente disponible y después selecciona **Crear**.

    ![Captura de pantalla de la creación de un runbook.](../images/dp-300-module-13-lab-21.png)

1. Una vez creado el runbook, copia y pega el fragmento de código de PowerShell siguiente en el editor de runbook. En la primera línea del script, pega el nombre del servidor que copiaste en los pasos anteriores. Seleccione **Guardar** y, a continuación, **Publicar**.

    **Nota:** comprueba que el código se ha copiado correctamente antes de guardar el runbook.

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    ![Captura de pantalla del pegado de código recortado.](../images/dp-300-module-13-lab-22.png)

1. Si todo va bien, deberías recibir un mensaje correcto.

    ![Captura de pantalla de un mensaje correcto para la creación del runbook.](../images/dp-300-module-13-lab-23.png)

## Creación de una programación para un runbook

A continuación, programará el runbook para que se ejecute de forma periódica.

1. En **Recursos**, en el panel de navegación izquierdo del runbook **IndexMaintenance**, selecciona **Programaciones**. A continuación, seleccione **+ Agregar una programación**.

    ![Captura de pantalla de la página Programaciones, con la selección de Agregar una programación.](../images/dp-300-module-13-lab-24.png)

1. Seleccione **Vincular una programación a su Runbook**.

    ![Captura de pantalla de la selección de Vincular una programación a su Runbook.](../images/dp-300-module-13-lab-25.png)

1. Selecciona **+ Agregar una programación**.

    ![Captura de pantalla del vínculo de crear una programación.](../images/dp-300-module-13-lab-26.png)

1. Proporcione un nombre descriptivo para la programación y una descripción, si lo quiere.

1. Especifica la hora de inicio a las **4:00 a. m.** del día siguiente en la zona horaria **Hora del Pacífico**. Configure la periodicidad para cada **1** día. No establezca una expiración.

    ![Captura de pantalla del elemento emergente Nueva programación, completado con información de ejemplo.](../images/dp-300-module-13-lab-27.png)

1. Seleccione **Crear** y después seleccione **Aceptar**.

1. La programación se ha creado y ahora está vinculada al runbook. Seleccione **Aceptar**.

    ![Captura de pantalla de la programación creada.](../images/dp-300-module-13-lab-28.png)

Azure Automation ofrece un servicio de automatización y configuración basado en la nube que admite una administración coherente en los entornos de Azure y que no son de Azure.

Al completar este ejercicio, has automatizado la desfragmentación de los índices en una base de datos de SQL Server para que se ejecute cada día, a las 4:00 a. m.