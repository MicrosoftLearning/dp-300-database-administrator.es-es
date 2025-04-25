---
lab:
  title: 'Laboratorio 4: configuración de reglas de firewall de Azure SQL Database'
  module: Implement a Secure Environment for a Database Service
---

# Implementación de un entorno seguro

**Tiempo estimado: 30 minutos**

Los alumnos tomarán la información obtenida de las lecciones para configurar y, posteriormente, implementar la seguridad en Azure Portal y dentro de la base de datos *AdventureWorksLT*.

Te han contratado como administrador sénior de bases de datos para garantizar la seguridad del entorno de bases de datos. Estas tareas se centrarán en Azure SQL Database.

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

## Configuración de reglas de firewall de Azure SQL Database

1. Desde la máquina virtual del laboratorio, o la máquina local si no te se proporcionó ninguna, inicia una sesión del explorador y ve a [https://portal.azure.com](https://portal.azure.com/). Conéctate al portal usando tus credenciales de Azure.

1. En Azure Portal, busca *Servidores SQL* en el cuadro de búsqueda de la parte superior y después haz clic en **Servidores SQL** en la lista de opciones.

1. Selecciona SQL Server **dp300-lab-xxxxxxxx**, donde *xxxxxxxx* es una cadena numérica aleatoria.

    > &#128221; Ten en cuenta que, si usas tu propio servidor de Azure SQL Server no creado por este laboratorio, debes seleccionar el nombre de ese servidor SQL Server.

1. En la pantalla *Información general* de SQL Server, a la derecha del nombre del servidor, selecciona el botón **Copiar al portapapeles**.

1. Selecciona **Mostrar configuración de red**.

1. En la página **Redes**, en **Reglas de firewall**, revisa la lista y asegúrate de que aparece la dirección IP del cliente. Si no aparece, selecciona **+ Agregar la dirección IPv4 del cliente (tu dirección IP)** y luego selecciona **Guardar**.

    > &#128221; Observa que la dirección IP del cliente se especificó automáticamente. La incorporación de la dirección IP de tu cliente a la lista te permitirá conectarte a Azure SQL Database mediante SQL Server Management Studio o cualquier otra herramienta de cliente. **Toma nota de la dirección IP del cliente, la usarás más adelante.**

1. Abra SQL Server Management Studio. En el cuadro de diálogo Conectar con el servidor, pega el nombre del servidor de Azure SQL Database e inicia sesión con las siguientes credenciales:

    - **Nombre del servidor:**&lt;_pega el nombre del servidor de Azure SQL Database aquí_&gt;
    - **Autenticación:** autenticación de SQL Server
    - **Inicio de sesión de administrador del servidor:** tu inicio de sesión como administrador del servidor de Azure SQL Database
    - **Contraseña:** tu contraseña como administrador del servidor de Azure SQL Database

1. Seleccione **Conectar**.

1. En Explorador de objetos, expande el nodo del servidor y haz clic con el botón derecho en **Bases de datos**. Selecciona **Importar aplicación de capa de datos**.

1. En la primera pantalla del cuadro de diálogo **Importar aplicación de capa de datos**, selecciona **Siguiente**.

1. En la pantalla **Importar configuración**, haz clic en **Examinar** y desplázate a la carpeta **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\04**, haz clic en el archivo **AdventureWorksLT.bacpac** y después en **Abrir**. De vuelta a la pantalla **Importar aplicación de capa de datos**, haz clic en **Siguiente**.

1. En la pantalla **Configuración de base de datos**, realiza los cambios como se indica a continuación:

    - **Nombre de la base de datos:** AdventureWorksFromBacpac
    - **Edición de Microsoft Azure SQL Database**: básico

1. Seleccione **Siguiente**.

1. En la pantalla **Resumen**, seleccione **Finalizar**. Esto puede tardar unos minutos. Cuando se complete la importación, verá los resultados siguientes. A continuación, seleccione **Cerrar**.

1. De vuelta en SQL Server Management Studio, en el **Explorador de objetos**, expande la carpeta **Bases de datos**. A continuación, haz clic con el botón derecho en la base de datos **AdventureWorksFromBacpac** y selecciona **Nueva consulta**.

1. Ejecuta la siguiente consulta T-SQL; para ello, pega el texto en la ventana de consulta.
    1. **Importante:** reemplaza **000.000.000.000** por la dirección IP del cliente. Seleccione **Execute**(Ejecutar).

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.000', 
            @end_ip_address = '000.000.000.000'
    ```

1. A continuación, crearás un usuario incluido en la base de datos **AdventureWorksFromBacpac**. Seleccione **Nueva consulta** y ejecute el siguiente código T-SQL.

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    > &#128221; Este comando crea un usuario incluido en la base de datos **AdventureWorksFromBacpac**. Probaremos esta credencial en el paso siguiente.

1. Navega hasta el **Explorador de objetos**. Haz clic en **Conectar** y después en **Motor de base de datos**.

1. Intenta conectarte con las credenciales creadas en el paso anterior. Necesitarás tener a mano la siguiente información:

    - **Inicio de sesión:** ContainedDemo
    - **Contraseña**: P@ssw0rd01

     Haga clic en **Conectar**.

     Recibirás el siguiente error.

    <span style="color:red">Error de inicio de sesión del usuario 'ContainedDemo'. (Microsoft SQL Server, Error: 18456)</span>

    > &#128221; Este error se genera porque la conexión intentó iniciar sesión en la base de datos *maestra* y no en **AdventureWorksFromBacpac** donde se creó el usuario. Cambia el contexto de conexión seleccionando **Aceptar** para salir del mensaje de error y, después, selecciona **Opciones >>** en **Conectar con el servidor**.

1. En la pestaña **Propiedades de conexión**, escribe el nombre de la base de datos **AdventureWorksFromBacpac** y después selecciona **Conectar**.

1. Observa que pudiste autenticarte correctamente con el usuario **ContainedDemo**. Esta vez has iniciado sesión directamente en **AdventureWorksFromBacpac**, que es la única base de datos a la que el usuario recién creado tiene acceso.

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

En este ejercicio, has configurado reglas de firewall de base de datos y de servidor para acceder a una base de datos hospedada en Azure SQL Database. También has usado instrucciones T-SQL para crear un usuario independiente y has usado SQL Server Management Studio para comprobar el acceso.
