---
lab:
  title: 'Laboratorio 5: habilitación de Microsoft Defender para SQL y clasificación de datos'
  module: Implement a Secure Environment for a Database Service
---

# Habilitación de Microsoft Defender para SQL y clasificación de datos

**Tiempo estimado: 30 minutos**

El alumnado tomará la información obtenida en las lecciones para configurar y, posteriormente, implementar la seguridad en Azure Portal y dentro de la base de datos AdventureWorks.

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

## Habilitar Microsoft Defender para SQL

1. Desde la máquina virtual del laboratorio, o la máquina local si no te se proporcionó ninguna, inicia una sesión del explorador y ve a [https://portal.azure.com](https://portal.azure.com/). Conéctate al portal usando tus credenciales de Azure.

1. En Azure Portal, busca *Servidores SQL* en el cuadro de búsqueda de la parte superior y después haz clic en **Servidores SQL** en la lista de opciones.

1. Selecciona SQL Server **dp300-lab-xxxxxxxx**, donde *xxxxxxxx* es una cadena numérica aleatoria.

    > &#128221; Ten en cuenta que, si usas tu propio servidor de Azure SQL Server no creado por este laboratorio, debes seleccionar el nombre de ese servidor SQL Server.

1. En la hoja *Información general*, selecciona **No configurado** junto a *Microsoft Defender para SQL*.

1. Selecciona la **X** en la esquina superior derecha para cerrar el panel Información general de*Microsoft Defender for Cloud*.

1. Selecciona **Habilitar** en *Microsoft Defender para SQL*.

1. En un entorno de producción, deben aparecer varias recomendaciones. Es recomendable que selecciones **Ver todas las recomendaciones en Defender for Cloud** y revises todas las recomendaciones de *Microsoft Defender* que se enumeran para Azure SQL Server e implementarlas según corresponda.

## Evaluación de vulnerabilidad

1. En la hoja principal de Azure SQL Server, desplázate a la sección **Configuración**, selecciona **Bases de datos SQL** y después selecciona el nombre de la base de datos **AdventureWorksLT**.

1. En **Seguridad**, selecciona el ajuste de **Microsoft Defender for Cloud**.

1. Selecciona la **X** en la esquina superior derecha para cerrar el panel Información general de *Microsoft Defender for Cloud* y ver el panel de **Microsoft Defender for Cloud** de la base de datos `AdventureWorksLT`.

1. Para empezar a revisar las funcionalidades de la Evaluación de vulnerabilidades, en **Vulnerability assessment findings** (Resultados de la Evaluación de vulnerabilidades), seleccione **View additional findings in Vulnerability Assessment** (Ver más resultados en la Evaluación de vulnerabilidades).  

1. Seleccione **Examinar** para obtener los resultados de evaluación de la vulnerabilidad más recientes. Este proceso tardará unos minutos mientras la evaluación de vulnerabilidad examina la base de datos.

1. Cada riesgo de seguridad tiene un nivel de riesgo (alto, medio o bajo) e información adicional. Las reglas establecidas se basan en pruebas comparativas proporcionadas por el [Center for Internet Security](https://www.cisecurity.org/benchmark/microsoft_sql_server/?azure-portal=true). En la pestaña **Resultados**, seleccione una vulnerabilidad. Toma nota del **Id.** de la vulnerabilidad, por ejemplo **VA1143** (si aparece).

1. En función de la comprobación de seguridad, habrá vistas alternativas y recomendaciones. Revise la información que se proporciona. Para esta comprobación de seguridad, puede seleccionar el botón **Agregar todos los resultados como línea de base** y, a continuación, seleccionar **Sí** para establecer la línea de base. Ahora que se ha establecido una línea base, esta comprobación de seguridad generará errores en los análisis futuros en lo que los resultados sean diferentes de los de la línea base. Seleccione la **X** de la parte superior derecha para cerrar el panel de la regla específica.  

1. Vamos a volver a ejecutar **Examinar** para confirmar que la vulnerabilidad seleccionada se muestra ahora como comprobación de seguridad *Aprobada*.

    Si selecciona la comprobación de seguridad superada anteriormente, debería poder ver la línea de base que ha configurado. En caso de que se produzca algún cambio en el futuro, los exámenes de evaluación de vulnerabilidad lo detectarán y la comprobación de seguridad generará errores.  

## Advanced Threat Protection

1. Seleccione la **X** en la parte superior derecha para cerrar el panel Evaluación de vulnerabilidad y vuelva al panel **Microsoft Defender for Cloud** correspondiente a su base de datos. En **Alertas e incidentes de seguridad**, no debería ver ningún elemento. Esto quiere decir que **Advanced Threat Protection** no ha detectado ninguna incidencia. Advanced Threat Protection detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.  

    > &#128221; Lo habitual es que en esta fase no vea ninguna alerta de seguridad. En el paso siguiente, ejecutará una prueba que desencadenará una alerta para que pueda revisar los resultados en Advanced Threat Protection.  

    Se puede usar Advanced Threat Protection para identificar amenazas y enviar alertas cuando se sospeche que se está produciendo cualquiera de los eventos siguientes:  

    - Inyección de código SQL
    - Vulnerabilidad por inyección de código SQL
    - Filtración de datos
    - Acción insegura
    - Fuerza bruta
    - Inicio de sesión anómalo de cliente

    En esta sección, aprenderá cómo se puede desencadenar una alerta de inyección de código SQL a través de SSMS. Las alertas de inyección de código SQL están destinadas a aplicaciones personalizadas, no a herramientas estándar como SSMS. Por lo tanto, para desencadenar una alerta a través de SSMS como prueba de una inyección de código SQL, es necesario "establecer" el valor de **Nombre de aplicación**, que es una propiedad de conexión para los clientes que se conectan a SQL Server o a Azure SQL.

1. Desde la máquina virtual del laboratorio o desde tu máquina local, si no se te proporcionó ninguna, abre SQL Server Management Studio (SSMS). En el cuadro de diálogo Conectar con el servidor, pega el nombre del servidor de Azure SQL Database e inicia sesión con las siguientes credenciales:

    - **Nombre del servidor:**&lt;_pega el nombre del servidor de Azure SQL Database aquí_&gt;
    - **Autenticación:** autenticación de SQL Server
    - **Inicio de sesión de administrador del servidor:** tu inicio de sesión como administrador del servidor de Azure SQL Database
    - **Contraseña:** tu contraseña como administrador del servidor de Azure SQL Database

1. Seleccione **Conectar**.

1. En SSMS, seleccione **Archivo** > **Nuevo** > **Consulta de motor de base de datos** para crear una consulta con una nueva conexión.  

1. En la ventana de inicio de sesión principal, inicia sesión en la base de datos **AdventureWorksLT** como lo harías normalmente, con la autenticación de SQL y el nombre y las credenciales de administrador de Azure SQL Server. Pero antes de conectarte, selecciona **Opciones >>** > **Propiedades de conexión**. Escribe **AdventureWorksLT** en la opción **Conectar a la base de datos**.  

1. Seleccione la pestaña **Parámetros de conexión adicionales** e inserte la siguiente cadena de conexión en el cuadro de texto:  

    ```sql
    Application Name=webappname
    ```

1. Seleccione **Conectar**.  

1. En la nueva ventana de consulta, pegue la siguiente consulta y seleccione **Ejecutar**:  

    ```sql
    SELECT * FROM sys.databases WHERE database_id like '' or 1 = 1 --' and family = 'test1';
    ```

1. En Azure Portal, ve a la base de datos **AdventureWorksLT**. En el panel de la izquierda, en **Seguridad**, seleccione **Microsoft Defender for Cloud**.

1. En **Alertas e incidentes de seguridad**, selecciona **Buscar alertas en este recurso en Microsoft Defender for Cloud**.  

1. Ahora puede ver las alertas de seguridad en general.  

1. Seleccione **Potential SQL injection** (Posible inyección de código SQL) para mostrar alertas más específicas y recibir pasos de investigación.

1. Selecciona **Ver detalles completos** para mostrar los detalles de la alerta.

1. En la pestaña **Detalles de alerta**, ten en cuenta que se muestra la *Declaración de vulnerabilidad*. Esta es la instrucción SQL que se ejecutó para desencadenar la alerta. También fue la instrucción SQL que se ejecutó en SSMS. Además, ten en cuenta que la **Aplicación cliente** se muestra como **webappname**. Este es el nombre que especificaste en la cadena de conexión en SSMS.

1. Como paso de limpieza, considera la posibilidad de cerrar todos los editores de consultas en SSMS y de quitar todas las conexiones para no desencadenar accidentalmente alertas adicionales en los ejercicios siguientes.

## Habilitación de la clasificación de datos

1. En la hoja principal de Azure SQL Server, desplázate a la sección **Configuración**, selecciona **Bases de datos SQL** y después selecciona el nombre de la base de datos **AdventureWorksLT**.

1. En la hoja principal de la base de datos **AdventureWorksLT**, desplázate a la sección **Seguridad** y selecciona **Detección y clasificación de datos**.

1. En la pantalla **Detección y clasificación de datos**, verás un mensaje informativo que indica que **se está utilizando actualmente la política de protección de información de SQL y se han encontrado 15 columnas con recomendaciones de clasificación**. Seleccione este vínculo.

1. En la siguiente pantalla de **Data Discovery and Classification** (Detección y clasificación de datos), active la casilla situada junto a **Seleccionar todo**, seleccione **Accepted selected recommendations** (Recomendaciones seleccionadas aceptadas) y, después, seleccione **Guardar** para guardar las clasificaciones en la base de datos.

1. De vuelta a la pantalla **Detección y clasificación de datos**, observa que quince columnas se clasificaron correctamente en cinco tablas diferentes. Revisa el *Tipo de información* y la *Etiqueta de confidencialidad* para cada una de las columnas.

## Configuración de la clasificación y el enmascaramiento de datos

1. En Azure Portal, ve a tu instancia **AdventureWorksLT** de Azure SQL Database (no al servidor lógico).

1. En el panel de la izquierda, en **Seguridad**, seleccione **Clasificación y detección de datos**.  

1. En la tabla Cliente del esquema SalesLT, *Clasificación y detección de datos* ha identificado `FirstName` y `LastName` para ser clasificadas, pero no `MiddleName`. Usa las listas desplegables para agregarla ahora. Selecciona **Nombre** para el *Tipo de información* y **RGPD confidencial** para la *Etiqueta de confidencialidad* y luego selecciona **Agregar clasificación**.  

1. Seleccione **Guardar**.

1. Confirme que la clasificación se ha agregado correctamente; para ello, consulte la pestaña **Información general** y confirme que `MiddleName` se muestra en la lista de columnas clasificadas en el esquema SalesLT.

1. En el panel de la izquierda, puede seleccionar **Información general** para volver a la información general de la base de datos.  

   La característica Enmascaramiento dinámico de datos (DDM) está disponible en Azure SQL y en SQL Server. DDM limita la exposición de los datos enmascarando los datos confidenciales para los usuarios sin privilegios en el nivel de servidor de SQL Server en lugar de en el nivel de aplicación, donde tienes que programar esos tipos de reglas. Azure SQL le recomienda elementos para enmascarar, o bien, puede agregar máscaras manualmente.

   En los pasos siguientes, enmascarará las columnas `FirstName`, `MiddleName` y `LastName` que ha revisado en el paso anterior.  

1. En Azure Portal, vaya a la base de datos de Azure SQL. En el panel de la izquierda, en **Seguridad**, seleccione **Enmascaramiento dinámico de datos** y, a continuación, seleccione **Agregar máscara**.  

1. En las listas desplegables, seleccione el esquema **SalesLT**, la tabla **Customer** y la columna **FirstName**. Después, puede revisar las opciones de enmascaramiento, pero la opción predeterminada es adecuada para este escenario. Haga clic en **Agregar** para agregar la regla de enmascaramiento.  

1. Repita el paso anterior para las columnas **MiddleName** y **LastName** de esa tabla.  

    Ahora tiene tres reglas de enmascaramiento.  

1. Seleccione **Guardar**.

    > &#128221; Ten en cuenta que si el nombre de Azure SQL Server no se compone solo de letras minúsculas, números y guiones, este paso producirá un error y no podrás continuar con las secciones de enmascaramiento de datos.

1. En el panel de la izquierda, puede seleccionar **Información general** para volver a la información general de la base de datos.

## Recuperación de datos clasificados y enmascarados

Seguidamente, simulará que alguien consulta las columnas clasificadas y explorará la característica Enmascaramiento dinámico de datos en acción.

1. Abre SQL Server Management StudioSQL (SSMS) y conéctate a Azure SQL Server y abre una nueva ventana de consulta.

1. Haz clic con el botón derecho en la base de datos **AdventureWorksLT** y selecciona **Nueva consulta**.  

1. Ejecute la siguiente consulta para devolver los datos clasificados y, en algunos casos, columnas marcadas para su enmascaramiento. Seleccione **Ejecutar** para ejecutar la consulta.

    ```sql
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    ```

    Debería obtener como resultado los 10 primeros nombres, sin aplicación de enmascaramiento. ¿Por qué? Porque es el administrador de este servidor lógico de Azure SQL Database.  

1. En la consulta siguiente, creará un nuevo usuario y ejecutará la consulta anterior como ese usuario. También usará `EXECUTE AS` para suplantar a `Bob`. Cuando se ejecuta una instrucción `EXECUTE AS`, el contexto de ejecución de la sesión se cambia al inicio de sesión o usuario. Esto significa que los permisos se comprueban con el inicio de sesión o usuario y no con la persona que ejecuta el comando `EXECUTE AS` (en este caso, usted). Seguidamente, se utiliza `REVERT` para detener la suplantación del inicio de sesión o usuario.  

    Es posible que reconozca los primeros elementos de los comandos siguientes, ya que son una repetición de un ejercicio anterior. Cree una nueva consulta con los siguientes comandos y, a continuación, seleccione **Ejecutar** para ejecutar la consulta y observar los resultados.

    ```sql
    -- Create a new SQL user and give them a password
    CREATE USER Bob WITH PASSWORD = 'c0mpl3xPassword!';

    -- Until you run the following two lines, Bob has no access to read or write data
    ALTER ROLE db_datareader ADD MEMBER Bob;
    ALTER ROLE db_datawriter ADD MEMBER Bob;

    -- Execute as our new, low-privilege user, Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;
    ```

    Ahora debería obtener como resultado los 10 primeros nombres, pero con aplicación de enmascaramiento. A Bob no se le ha concedido acceso al formato sin máscara de estos datos.  

    ¿Qué ocurre si, por alguna razón, Bob necesita acceder a los nombres y obtiene permiso para hacerlo?  

    Puede actualizar los usuarios excluidos del enmascaramiento en Azure Portal desde el panel **Enmascaramiento dinámico de datos** en **Seguridad**, pero también puede hacerlo mediante T-SQL.

1. Haga clic con el botón derecho en la base de datos **AdventureWorksLT** y selecciona **Nueva consulta**, luego escribe la siguiente consulta para permitir que Bob consulte los resultados de los nombres sin enmascaramiento. Seleccione **Ejecutar** para ejecutar la consulta.

    ```sql
    GRANT UNMASK TO Bob;  
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Los resultados deben incluir los nombres completos.  

1. También se pueden quitar los privilegios de desenmascaramiento de un usuario y confirmar la acción con los siguientes comandos T-SQL en una nueva consulta:  

    ```sql
    -- Remove unmasking privilege
    REVOKE UNMASK TO Bob;  

    -- Execute as Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Los resultados deben incluir los nombres enmascarados.  

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

En este ejercicio, has mejorado la seguridad de una base de datos de Azure SQL mediante la habilitación de Microsoft Defender for SQL. También has creado columnas clasificadas en función de las recomendaciones de Azure Portal.
