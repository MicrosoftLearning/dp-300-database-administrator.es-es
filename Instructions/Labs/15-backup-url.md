---
lab:
  title: 'Laboratorio 15: Realización de una copia de seguridad en URL y restaurar desde URL'
  module: Plan and implement a high availability and disaster recovery solution
---

# Copia de seguridad en URL

**Tiempo estimado: 30 minutos**

Como administrador de bases de datos de AdventureWorks, debes realizar una copia de seguridad de una base de datos en una dirección URL en Azure y restaurarla desde Azure Blob Storage después de que se haya producido un error humano.

## Configura el entorno

Si la máquina virtual de laboratorio se proporcionó y está preconfigurada, deberías encontrar los archivos de laboratorio listos en la carpeta **C:\LabFiles**. *Dedica un momento a comprobar si los archivos ya están allí, y omite esta sección*. Sin embargo, si usas tu propia máquina o faltan los archivos para laboratorio, deberás clonarlos desde *GitHub* para continuar.

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, inicia una sesión de Visual Studio Code.

1. Abre la paleta de comandos (CTRL+Mayús+P) y escribe **Git: Clone**. Selecciona la opción **Git: Clone**.

1. Pega la siguiente dirección URL en el campo **Repository URL** y selecciona **Enter**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Guarda el repositorio en la carpeta **C:\LabFiles** en la máquina virtual del laboratorio o en tu máquina local si no se te proporcionó ninguna (crea la carpeta si no existe).

## Restauración de la base de datos

Si ya tienes restaurada la base de datos **AdventureWorks2017**, puedes omitir esta sección.

1. Desde la máquina virtual del laboratorio o la máquina local si no se proporcionó una, inicia una sesión de SQL Server Management Studio (SSMS).

1. Cuando se abra SSMS, aparecerá de forma predeterminada el cuadro de diálogo **Conectar al servidor**. Elige la instancia predeterminada y selecciona **Conectar**. Es posible que tengas que activar la casilla **Certificado de servidor de confianza**.

    > &#128221; Ten en cuenta que si usas tu propia instancia de SQL Server, deberás conectarte a ella mediante el nombre y las credenciales de la instancia de servidor adecuadas.

1. Seleccione la carpeta **Bases de datos** y, a continuación, **Nueva consulta**.

1. En la ventana Nueva consulta, copie y pegue T-SQL. Ejecute la consulta para restaurar la base de datos.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; Debes tener una carpeta denominada **C:\LabFiles**. Si no tienes esta carpeta, créala o especifica otra ubicación para la base de datos y los archivos de copia de seguridad.

1. En la pestaña **Mensajes**, deberías ver un mensaje que indica que la base de datos se restauró correctamente.

## Configuración de la copia de seguridad en una dirección URL

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, inicia una sesión de Visual Studio Code.

1. Abre el repositorio clonado en **C:\LabFiles\dp-300-database-administrator**.

1. Haz clic con el botón derecho del ratón en la carpeta **Allfiles** y selecciona **Abrir en terminal integrado**. Se abrirá una ventana de terminal en la ubicación correcta.

1. En el terminal, escribe lo siguiente y presiona **Entrar**.

    ```bash
    az login
    ```

1. Se te solicitará que abras un explorador y que introduzcas un código. Sigue las instrucciones para iniciar sesión en la cuenta de Azure.

1. *Si ya tienes un grupo de recursos, omite este paso*. Si no tienes ningún grupo de recursos, crea uno ejecutando el siguiente comando en el terminal. Reemplaza *contoso-rgXXX######* por un nombre único para el grupo de recursos. El nombre debe ser único en Azure. Reemplaza la ubicación (-l) por la ubicación del grupo de recursos.

    ```bash
    az group create -n "contoso-rglod#######" -l eastus2
    ```

    Reemplaza **######** por algunos caracteres aleatorios.

1. En el terminal, escribe lo siguiente y presiona **Entrar** para crear una cuenta de almacenamiento. Asegúrate de usar un nombre único para la cuenta de almacenamiento. *El nombre deben tener entre 3 y 24 caracteres, y solo pueden contener números y letras minúsculas*. Reemplaza *########* por 8 caracteres numéricos aleatorios. El nombre debe ser único en Azure. Reemplaza contoso-rgXXX###### por el nombre del grupo de recursos que desees. Por último, reemplaza la ubicación (-l) por la ubicación del grupo de recursos.

    ```bash
    az storage account create -n "dp300bckupstrg########" -g "contoso-rgXXX########" --kind StorageV2 -l eastus2
    ```

1. A continuación, obtendrás las claves de la cuenta de almacenamiento, que deberás usar en los pasos siguientes. Ejecuta el código siguiente en el terminal mediante el nombre único de la cuenta de almacenamiento y grupo de recursos.

    ```bash
    az storage account keys list -g contoso-rgXXX######## -n dp300bckupstrg########
    ```

    La clave de cuenta se mostrará en los resultados del comando anterior. Asegúrate de usar el mismo nombre (después de **-n**) y el grupo de recursos (después de **-g**) que usaste en el comando anterior. Copia el valor devuelto para **Key1** (sin las comillas dobles).

1. La copia de seguridad de una base de datos de SQL Server en una dirección URL usa una cuenta de almacenamiento y un contenedor dentro de ella. En este paso, crearás un contenedor específicamente para el almacenamiento de copia de seguridad. Para ello, ejecuta los comandos siguientes.

    ```bash
    az storage container create --name "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --fail-on-exist
    ```

    Donde **dp300bckupstrg########** es el nombre de la cuenta de almacenamiento única que usaste al crearla y **storage_key** es la clave generada. La salida debe devolver el valor **true**.

1. Para comprobar que las copias de seguridad del contenedor se han creado correctamente, ejecuta:

    ```bash
    az storage container list --account-name "dp300bckupstrg########" --account-key "storage_key"
    ```

    Donde **dp300bckupstrg########** es el nombre de la cuenta de almacenamiento única que usaste al crearla y **storage_key** es la clave generada.

1. Por seguridad, es necesario tener una Firma de acceso compartido (SAS) en el nivel de contenedor. Ejecuta el comando siguiente en el terminal:

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    Donde **dp300bckupstrg########** es el nombre de la cuenta de almacenamiento único que usaste al crearla y **storage_key** es la clave generada y **date_in_the_future** es una hora posterior a la de ahora. **date_in_the_future** debe estar en formato UTC. Un ejemplo es **2025-12-31T00:00Z**, que indica que la fecha de expiración es la medianoche del 31 de diciembre de 2025.

    La salida debe reflejar algo parecido a lo siguiente. Copia toda la firma de acceso compartido y pégala en el **Bloc de notas**, ya que la usarás en la siguiente tarea.

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7IhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

## Crear credenciales

Ahora que la funcionalidad ya está configurada, puedes generar un archivo de copia de seguridad como un blob en la cuenta de Azure Storage.

1. Inicia **SQL Server Management Studio (SSMS)**.

1. Se te pedirá que te conectes a SQL Server. Asegúrate de que la opción **Autenticación de Windows** esté seleccionada y, a continuación, selecciona **Conectar**.

1. Selecciona **Nueva consulta**.

1. Crea la credencial que se usará para obtener acceso al almacenamiento en la nube con el siguiente código de Transact-SQL. Rellena los valores adecuados y selecciona **Ejecutar**.

    ```sql
    IF NOT EXISTS  
    (SELECT * 
        FROM sys.credentials  
        WHERE name = 'https://<storage_account_name>.blob.core.windows.net/backups')  
    BEGIN
        CREATE CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]
        WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<key_value>'
    END;
    GO  
    ```

    Ambos resultados de **<storage_account_name>** son el nombre de la cuenta de almacenamiento único creada anteriormente y **<key_value>** es el valor generado al final de la tarea anterior similar al siguiente:

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7IhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

1. Puedes comprobar si la credencial se creó correctamente; para ello, ve a **Seguridad -> Credenciales** en el explorador de objetos de SSMS.

1. Si no has escrito bien algún dato y necesitas volver a crear la credencial, puedes quitarla con el siguiente comando, pero asegúrate de cambiar el nombre de la cuenta de almacenamiento:

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## Copia de seguridad de base de datos en dirección URL

1. Con SSMS, realiza una copia de seguridad de la base de datos **AdventureWorks2017** en Azure con el siguiente comando en Transact-SQL:

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    Donde **<storage_account_name>** es el nombre de la cuenta de almacenamiento único que se usó para crearla. 

    Si se produce un error, comprueba que no escribiste incorrectamente nada al crear la credencial y que todo se haya creado correctamente.

## Validación de la copia de seguridad mediante la CLI de Azure

Para ver si el archivo se encuentra realmente en Azure, puedes usar Explorador de Storage (versión preliminar) o Azure Cloud Shell.

1. En el terminal de Visual Studio Code, ejecuta este comando de la CLI de Azure:

    ```bash
    az storage blob list -c "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --output table
    ```

    Asegúrate de usar el mismo nombre de cuenta de almacenamiento único (después de **--account-name**) y la clave de cuenta (después de **--account-key**) que usaste en los comandos anteriores.

    Podemos confirmar que el archivo de copia de seguridad se generó correctamente.

## Validación de la copia de seguridad con el explorador de almacenamiento

1. En una ventana del explorador, ve a Azure Portal, y busca y selecciona **Cuentas de almacenamiento**.

1. Selecciona el nombre de la cuenta de almacenamiento único que creaste para las copias de seguridad.

1. En el panel de navegación izquierdo, selecciona **Explorador de almacenamiento**. Expande **Contenedores de blobs**.

1. Selecciona **Copias de seguridad**.

1. Ten en cuenta que el archivo de copia de seguridad se almacena en el contenedor.

## Restauración desde URL

En esta tarea te mostraremos cómo restaurar una base de datos desde Azure Blob Storage.

1. En **SQL Server Management Studio (SSMS),** selecciona **Nueva consulta** y pega y ejecuta la consulta siguiente.

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

1. Ejecuta este comando para cambiar el nombre de ese cliente.

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. Vuelve a ejecutar el **paso 1** para comprobar que se haya cambiado la dirección. Ahora, imagina que alguien ha cambiado miles o millones de filas sin una cláusula WHERE, o bien mediante la cláusula WHERE equivocada. Una de las soluciones implica restaurar la base de datos a partir de la última copia de seguridad disponible.

1. Para restaurar la base de datos y volver a un punto anterior al cambio del nombre del cliente por error, ejecuta lo siguiente.

    > &#128221; Con la sintaxis **SET SINGLE_USER WITH ROLLBACK IMMEDIATE**, las transacciones abiertas se revertirán. Esto puede impedir que se produzca un error en la restauración debido a conexiones activas.

    ```sql
    USE [master]
    GO

    ALTER DATABASE AdventureWorks2017 SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    GO

    RESTORE DATABASE AdventureWorks2017 
    FROM URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak'
    GO

    ALTER DATABASE AdventureWorks2017 SET MULTI_USER
    GO
    ```

    Donde **<storage_account_name>** es el nombre de cuenta de almacenamiento único que creaste.

1. Vuelve a ejecutar el **paso 1** para comprobar que se ha restaurado el nombre del cliente.

Es importante entender los componentes y la interacción entre ellos para realizar una copia de seguridad o una restauración desde el servicio Azure Blob Storage.

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

Si no usas la base de datos o los archivos de laboratorio para ningún otro propósito, puedes limpiar los objetos que creaste en este laboratorio.

### Eliminar la carpeta C:\LabFiles

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, abre **Explorador de archivos**.
1. Ve a **C:\\**.
1. Elimina la carpeta **C:\LabFiles**.

## Eliminar la base de datos AdventureWorks2017

1. Desde la máquina virtual del laboratorio o la máquina local si no se proporcionó una, inicia una sesión de SQL Server Management Studio (SSMS).
1. Cuando se abra SSMS, aparecerá de forma predeterminada el cuadro de diálogo **Conectar al servidor**. Elige la instancia predeterminada y selecciona **Conectar**. Es posible que tengas que activar la casilla **Certificado de servidor de confianza**.
1. En el **Explorador de objetos**, expande la carpeta **Bases de datos**.
1. Haz clic con el botón derecho en la base de datos **AdventureWorks2017** y selecciona **Eliminar**.
1. En el cuadro de diálogo **Eliminar objeto**, activa la casilla **Cerrar conexiones existentes**.
1. Seleccione **Aceptar**.

---

Completaste correctamente este laboratorio.

Ahora ya puedes realizar la copia de seguridad de una base de datos en una dirección URL de Azure y, si es necesario, restaurarla.
