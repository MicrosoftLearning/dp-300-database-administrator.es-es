---
lab:
  title: 'Laboratorio 3: Autorización del acceso a Azure SQL Database con Microsoft Entra ID'
  module: Implement a Secure Environment for a Database Service
---

# Configuración de la autenticación y la autorización

**Tiempo estimado: 25 minutos**

Los alumnos tomarán la información obtenida de las lecciones para configurar y, posteriormente, implementar la seguridad en Azure Portal y dentro de la base de datos *AdventureWorksLT*.

Le han contratado como administrador sénior de bases de datos para garantizar la seguridad del entorno de la base de datos.

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

1. Una vez completado el script, devolverá el nombre del grupo de recursos, el nombre de SQL Server y el nombre de la base de datos, y el nombre de usuario y la contraseña del administrador. Toma nota de estos valores, ya que los necesitará más adelante en el laboratorio.

---

## Autorización del acceso a Azure SQL Database con Microsoft Entra

Puede crear inicios de sesión desde cuentas de Microsoft Entra como un usuario de base de datos independiente mediante la sintaxis de T-SQL `CREATE USER [anna@contoso.com] FROM EXTERNAL PROVIDER`. Un usuario de base de datos independiente se asigna a una identidad en el directorio de Microsoft Entra asociado a la base de datos y no tiene inicio de sesión en la base de datos `master`.

Con la introducción de los inicios de sesión del servidor de Microsoft Entra en Azure SQL Database, puede crear inicios de sesión a partir de entidades de seguridad de Microsoft Entra en la base de datos virtual `master` de un SQL Database. Puede crear inicios de sesión de Microsoft Entra desde *usuarios, grupos y entidades de servicio* de Microsoft Entra. Para obtener más información, consulte [Entidades de seguridad de servidor de Microsoft Entra](/azure/azure-sql/database/authentication-azure-ad-logins).

Además, solo puede usar Azure Portal para crear administradores y los roles de control de acceso basado en rol de Azure no se propagan a los servidores lógicos de Azure SQL Database. Se deben conceder permisos adicionales de servidor y de base de datos mediante Transact-SQL (T-SQL). Vamos a crear un administrador de Microsoft Entra para SQL Server.

1. Desde la máquina virtual del laboratorio, o la máquina local si no te se proporcionó ninguna, inicia una sesión del explorador y ve a [https://portal.azure.com](https://portal.azure.com/). Conéctate al portal usando tus credenciales de Azure.

1. En Azure Portal, busca y selecciona **servidores SQL** y selecciónalo.

1. Selecciona SQL Server **dp300-lab-xxxxxxxx**, donde *xxxxxxxx* es una cadena numérica aleatoria.

    > &#128221; Ten en cuenta que, si usas tu propio servidor de Azure SQL Server no creado por este laboratorio, debes seleccionar el nombre de ese servidor SQL Server.

1. En la hoja *Información general* , selecciona **No configurado** junto a *Administrador de Microsoft Entra*.

1. En la siguiente pantalla, seleccione **Establecer administrador**.

1. En la barra lateral de **Microsoft Entra ID**, busca el nombre de usuario de Azure con el que iniciaste sesión en Azure Portal y haz clic en **Seleccionar**.

1. Seleccione **Guardar** para completar el proceso. Esto hará que el nombre de usuario sea el administrador de Microsoft Entra para el servidor.

1. En la parte izquierda, seleccione **Información general** y, después, copie el **Nombre del servidor**.

1. Abre SQL Server Management Studio y selecciona **Conectar** > **Motor de base de datos**. En **Nombre del servidor**, pegue el nombre del servidor. Cambia el valor del tipo de autenticación a **MFA de Microsoft Entra**.

1. Seleccione **Conectar**.

## Administración del acceso a objetos de base de datos

En esta tarea, administrará el acceso a la base de datos y sus objetos. Lo primero que va a hacer es crear dos usuarios en la base de datos *AdventureWorksLT*.

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, en SSMS, inicia sesión en la base de datos *AdventureWorksLT* mediante la cuenta de administrador de Azure Server o la cuenta de administrador de Microsoft Entra.

1. Use el **Explorador de objetos** y expanda **Bases de datos**.

1. Haga clic con el botón derecho en **AdventureWorksLT** y seleccione **Nueva consulta**.

1. En la ventana Nueva consulta, copie y pegue T-SQL. Ejecute la consulta para crear los dos usuarios.

    ```sql
    CREATE USER [DP300User1] WITH PASSWORD = 'Azur3Pa$$';
    GO

    CREATE USER [DP300User2] WITH PASSWORD = 'Azur3Pa$$';
    GO
    ```

    **Nota:** Estos usuarios se crean en el ámbito de la base de datos AdventureWorksLT. A continuación, creará un rol personalizado al que agregará los usuarios.

1. Ejecute la siguiente instrucción T-SQL en la misma ventana de consulta.

    ```sql
    CREATE ROLE [SalesReader];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User1];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User2];
    GO
    ```

    A continuación, cree un procedimiento almacenado en el esquema **SalesLT**.

1. Ejecute la instrucción T-SQL siguiente en la ventana de consulta.

    ```sql
    CREATE OR ALTER PROCEDURE SalesLT.DemoProc
    AS
    SELECT P.Name, Sum(SOD.LineTotal) as TotalSales ,SOH.OrderDate
    FROM SalesLT.Product P
    INNER JOIN SalesLT.SalesOrderDetail SOD on SOD.ProductID = P.ProductID
    INNER JOIN SalesLT.SalesOrderHeader SOH on SOH.SalesOrderID = SOD.SalesOrderID
    GROUP BY P.Name, SOH.OrderDate
    ORDER BY TotalSales DESC
    GO
    ```

    Luego, use la sintaxis `EXECUTE AS USER` para probar la seguridad. Esto permite que el motor de base de datos ejecute una consulta en el contexto del usuario.

1. Ejecute la siguiente instrucción T-SQL.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    Se producirá un error con el mensaje:

    <span style="color:red">Msg 229, Level 14, State 5, Procedure SalesLT.DemoProc, Line 1 [Batch Start Line 0] El permiso EXECUTE se denegó en el objeto 'DemoProc', base de datos 'AdventureWorksLT', esquema 'SalesLT'.</span>

1. A continuación, conceda permisos al rol para que este pueda ejecutar el procedimiento de almacenamiento. Ejecute la instrucción T-SQL siguiente.

    ```sql
    REVERT;
    GRANT EXECUTE ON SCHEMA::SalesLT TO [SalesReader];
    GO
    ```

    El primer comando revierte el contexto de ejecución al propietario de la base de datos.

1. Vuelva a ejecutar la instrucción T-SQL anterior.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

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

En este ejercicio, has visto cómo usar Microsoft Entra ID para conceder a las credenciales de Azure acceso a una instancia de SQL Server alojada en Azure. También ha usado la instrucción T-SQL para crear usuarios de base de datos y les ha concedido permisos para ejecutar procedimientos almacenados.
