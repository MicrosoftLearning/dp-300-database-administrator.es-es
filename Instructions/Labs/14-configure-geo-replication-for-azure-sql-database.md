---
lab:
  title: 'Laboratorio 14: configuración de la replicación geográfica para Azure SQL Database'
  module: Plan and implement a high availability and disaster recovery solution
---

# Configuración de la replicación geográfica para Azure SQL Database

**Tiempo estimado: 30 minutos**

Como DBA en AdventureWorks, debes habilitar la replicación geográfica para Azure SQL Database y asegurarte de que funciona correctamente. Además, conmutarás por error manualmente a otra región mediante el portal.

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

## Habilitación de la replicación geográfica

1.  Desde la máquina virtual del laboratorio, o la máquina local si no te se proporcionó ninguna, inicia una sesión del explorador y ve a [https://portal.azure.com](https://portal.azure.com/). Conéctate al portal usando tus credenciales de Azure.

1. En Azure Portal, navega a tu base de datos buscando **sql databases**.

1. Seleccione la base de datos **SQL AdventureWorksLT**.

1. En la hoja de la base de datos, en la sección **Administración de datos**, selecciona **Réplicas**.

1. Selecciona **+ Crear réplica**.

1. En la página **Crear base de datos SQL - Réplica geográfica**, observa que las secciones **Detalles del proyecto** y **Base de datos principal** ya están rellenadas con la suscripción, el grupo de recursos y el nombre de la base de datos.

1. En la sección **Configuración de réplica**, selecciona **Réplica geográfica** para el *Tipo de réplica*.

1. En la sección **Detalles de la base de datos geográfica secundaria**, escribe o selecciona los siguientes valores:

    - **Suscripción**: &lt;el nombre de la suscripción&gt; (usa el mismo que para la base de datos principal).
    - **Grupo de recursos**: &lt;selecciona el mismo grupo de recursos que para la base de datos principal.&gt; 
    - **Nombre de la base de datos**: el nombre de la base de datos aparecerá atenuado y será el mismo que el nombre de la base de datos principal.
    - **Servidor**: seleccione **Crear nuevo**.
    - En la página **Crear servidor de SQL Database**, escribe los valores siguientes:

        - **Nombre del servidor**: escribe un nombre único para el servidor secundario. Los nombres deben ser únicos en todos los servidores de Azure SQL Database.
        - **Ubicación**: selecciona una región diferente de la base de datos principal. Ten en cuenta que es posible que la suscripción no tenga todas las regiones disponibles.
        - Marca la casilla **Permitir que los servicios de Azure accedan al servidor**. Ten en cuenta que, en un entorno de producción, puedes restringir el acceso al servidor.
        - Para la autenticación, selecciona **Autenticación de SQL**. Ten en cuenta que, en un entorno de producción, podrías necesitar usar la autenticación **Usar solo Microsoft Entra**. Escribe **sqladmin* para el nombre de inicio de sesión de administrador y una contraseña segura. La contraseña debe cumplir los requisitos de complejidad de la contraseña de Azure SQL de tener como mínimo 12 caracteres de longitud y contener al menos 1 letra mayúscula, 1 letra minúscula, 1 número y 1 carácter especial.
        - Selecciona **Aceptar** para crear el servidor.

    - **¿Quieres usar un grupo elástico?**: No.
    - **Proceso y almacenamiento**: de uso general, Gen 5, 2 núcleos virtuales, almacenamiento de 32 GB.
    - **Redundancia de almacenamiento de Backup**: almacenamiento con redundancia local (LRS). Ten en cuenta que, en un entorno de producción, es posible que necesites usar el **almacenamiento con redundancia geográfica (GRS)**.

1. Seleccione **Revisar + crear**.

1. Seleccione **Crear**. Llevará unos minutos que se creen el servidor secundario y la base de datos. Una vez completado, el progreso cambiará de **Implementación en curso** a **La implementación se ha completado**.

1. Selecciona **Ir al recurso** para ir a la base de datos del servidor secundario para el paso siguiente.

## Conmutación por error de base de datos SQL a una región secundaria

Ahora que se ha creado la réplica de Azure SQL Database, realizará una conmutación por error.

1. Si aún no está en la base de datos del servidor secundario, busca **bases de datos SQL** en Azure Portal y selecciona la base de datos SQL **AdventureWorksLT** en el servidor secundario.

1. En la hoja principal de la base de datos SQL, en la sección **Administración de datos**, selecciona **Réplicas**.

1. Ten en cuenta que ahora se ha establecido el vínculo de replicación geográfica. El valor *Estado de réplica* de la base de datos principal es **En línea** y el valor de *Estado de réplica* de las réplicas geográficas es **Legible**.

1. Selecciona el menú **...** del servidor secundario y selecciona **Conmutación por error forzada**.

    > &#128221; La conmutación por error forzada cambiará la base de datos secundaria al rol principal. Todas las sesiones se desconectan durante esta operación.

1. Cuando te lo solicite el mensaje de advertencia, haz clic en **Sí**.

1. El estado de la réplica principal cambiará a **Pendiente** y la secundaria a **Conmutación por error**. 

     > &#128221; Ten en cuenta que, dado que la base de datos es pequeña, la conmutación por error será rápida. En un entorno de producción, este proceso puede tardar unos minutos.

1. Cuando hayas finalizado, los roles cambiarán y el secundario será el nuevo principal y el antiguo principal será el secundario. Es posible que tengas que actualizar la página para ver el nuevo estado.

Hemos visto que la base de datos secundaria legible puede estar en la misma región de Azure que la principal o, lo que es más común, en otra región. Este tipo de bases de datos secundarias legibles también se conocen como geográficas secundarias o réplicas geográficas.

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

Ahora ya sabes cómo habilitar la replicación geográfica para Azure SQL Database y conmutarla por error manualmente a otra región mediante el portal.
