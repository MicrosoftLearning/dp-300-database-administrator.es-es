---
lab:
  title: 'Laboratorio: Realizar una copia de seguridad en URL y restaurar desde URL'
  module: Plan and implement a high availability and disaster recovery solution
---

# Copia de seguridad en URL

**Tiempo estimado**: 20 minutos

Como administrador de bases de datos de Wide World Importers, debe realizar una copia de seguridad de una base de datos en una dirección URL en Azure y restaurarla después de que se haya producido un error humano.

## Restauración de una base de datos

1. Descargue el archivo de copia de seguridad de la base de datos ubicado en ****https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** C:\LabFiles\HADR** en la máquina virtual del laboratorio (cree la estructura de carpetas si no existe).

    ![Imagen 03](../images/dp-300-module-15-lab-00.png)

1. Seleccione el botón Inicio de Windows y escriba SSMS. Seleccione **Microsoft SQL Server Management Studio 18** en la lista.  

    ![Imagen 10](../images/dp-300-module-01-lab-34.png)

1. Cuando se abra SSMS, observe que el **cuadro de diálogo Conectar al** servidor se rellenará previamente con el nombre de instancia predeterminado. Seleccione **Conectar**.

    ![Imagen 02](../images/dp-300-module-07-lab-01.png)

1. Seleccione la **carpeta Bases de datos** y, a continuación **, Nueva consulta**.

    ![Imagen 03](../images/dp-300-module-07-lab-04.png)

1. En la ventana Nueva consulta, copie y pegue la instrucción T-SQL siguiente. Ejecute la consulta para restaurar la base de datos.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\HADR\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\HADR\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\HADR\AdventureWorks2017_log.ldf';
    ```

    **Nota:** El nombre y la ruta de acceso del archivo de copia de seguridad de la base de datos deben coincidir con lo que descargó en el paso 1; de lo contrario, se producirá un error en el comando.

1. Debería ver un mensaje correcto una vez completada la restauración.

    ![Imagen 03](../images/dp-300-module-07-lab-05.png)

## Configuración de la copia de seguridad en una dirección URL

1. En la máquina virtual del laboratorio, inicie una sesión del explorador y vaya a [https://portal.azure.com](https://portal.azure.com/). Conectar al Portal mediante Azure **Nombre de usuario** y **contraseña** proporcionados en la **pestaña Recursos** de esta máquina virtual de laboratorio.

    ![Captura de pantalla de la página de inicio de sesión en Azure](../images/dp-300-module-01-lab-01.png)

1. Abra una solicitud de **Cloud Shell** seleccionando el icono que se muestra a continuación.

    ![Captura de pantalla de Azure Portal en la que se muestra el icono de Cloud Shell.](../images/dp-300-module-15-lab-01.png)

1. Si aún no ha usado una instancia de Cloud Shell, en la mitad inferior del portal podrá ver un mensaje que le da la bienvenida a Azure Cloud Shell. Seleccione **Bash**.

    ![Captura de pantalla de la página principal de Cloud Shell en Azure Portal.](../images/dp-300-module-15-lab-02.png)

1. Si no ha usado previamente una instancia de Cloud Shell, debe proporcionarle almacenamiento. Seleccione **Mostrar configuración** avanzada (puede tener una suscripción diferente asignada).

    ![Captura de pantalla de Azure Portal en la pantalla de creación de Cloud Shell.](../images/dp-300-module-15-lab-03.png)

1. Use el **Grupo de recursos** existente y especifique nuevos nombres para la **Cuenta de almacenamiento** y el **Recurso compartido de archivos**, tal y como se muestra en el siguiente cuadro de diálogo. Tome nota del nombre del **grupo de recursos**. Debería empezar con *contoso-rg*. Después, seleccione **Create storage** (Crear almacenamiento).

    El nombre de la cuenta de almacenamiento debe ser único y todo en minúsculas, sin caracteres especiales. Especifique un nombre único.

    ![Captura de pantalla de la creación de una cuenta de almacenamiento desde Azure Portal](../images/dp-300-module-15-lab-04.png)

1. Una vez que haya finalizado, verá un mensaje similar al siguiente. Compruebe que la esquina superior izquierda de la pantalla de Cloud Shell muestra **Bash**.

    ![Captura de pantalla de Cloud Shell en Azure Portal.](../images/dp-300-module-15-lab-05.png)

1. Cree una nueva cuenta de almacenamiento desde la CLI mediante la ejecución del siguiente comando en Cloud Shell. Use el nombre del grupo de recursos que comienza por **DP-300-HADR** y que anotó anteriormente.

    > [!NOTE]
    > Cambie el nombre del grupo de recursos (**parámetro-g** ) y proporcione un nombre de cuenta de almacenamiento único (**-n** parámetro).

    ```bash
    az storage account create -n "dp300backupstorage1234" -g "contoso-rglod23149951" --kind StorageV2 -l eastus2
    ```

    ![Captura de pantalla del proceso de creación de una cuenta de almacenamiento en Azure Portal](../images/dp-300-module-15-lab-16.png)

1. A continuación, obtendrá las claves de la cuenta, que deberá usar en los pasos siguientes. Ejecute el código siguiente en Cloud Shell mediante el nombre único de la cuenta de almacenamiento:

    ```bash
    az storage account keys list -g contoso-rglod23149951 -n dp300backupstorage1234
    ```

    La clave de cuenta se mostrará en los resultados del comando anterior. Asegúrese de usar el mismo nombre (después de **-n**) y el grupo de recursos (después de **-g**) que usó en el comando anterior. Copie el valor devuelto para **Key1** (sin las comillas dobles), tal como se muestra aquí:

    ![Captura de pantalla de Azure Portal en la que se muestra la cuenta de almacenamiento.](../images/dp-300-module-15-lab-06.png)

1. Al realizar la copia de seguridad de una base de datos de SQL Server en una dirección URL, verá que usa una cuenta de almacenamiento y un contenedor dentro de ella. En este paso, creará un contenedor específicamente para el almacenamiento de copia de seguridad. Para ello, ejecute los comandos siguientes.

    ```bash
    az storage container create --name "backups" --account-name "dp300backupstorage1234" --account-key "storage_key" --fail-on-exist
    ```

    Donde **dp300storage** es el nombre de la cuenta de almacenamiento que usó al crearla y **storage_key** es la clave generada anteriormente. La salida debe devolver el valor **true**.

    ![Captura de pantalla de la salida de la creación del contenedor.](../images/dp-300-module-15-lab-07.png)

1. Para comprobar mejor las copias de seguridad del contenedor que se han creado, ejecute:

    ```bash
    az storage container list --account-name "dp300backupstorage1234" --account-key "storage_key"
    ```

    Donde **dp300storage** es el nombre de la cuenta de almacenamiento que usó al crearla y **storage_key** es la clave generada anteriormente. La salida debe devolver algo parecido a lo siguiente:

    ![Captura de pantalla del registro de eventos de contenedor.](../images/dp-300-module-15-lab-08.png)

1. Por seguridad, es necesario tener una Firma de acceso compartido (SAS) en el nivel de contenedor. Esto puede realizarse a través de Cloud Shell o PowerShell. Ejecute lo siguiente:

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300backupstorage1234" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    Donde **dp300storage** es el nombre de la cuenta de almacenamiento que creó anteriormente, **storage_key** es la clave generada anteriormente y **date_in_the_future** es una hora posterior a la de ahora. **date_in_the_future** debe estar en formato UTC. Un ejemplo es **2021-12-31T00:00Z**, que indica que la fecha de expiración es la medianoche del 31 de diciembre de 2020.

    La salida debe devolver algo parecido a lo siguiente. Copie toda la Firma de acceso compartido y péguela en el **Bloc de notas**, ya que la usará en la siguiente tarea:

    ![Captura de pantalla de la página Firma de acceso compartido](../images/dp-300-module-15-lab-09.png)

## Crear una credencial

Ahora que la funcionalidad ya está configurada, puede generar un archivo de copia de seguridad como un blob en Azure.

1. Inicie **SQL Server Management Studio (SSMS)**.

1. Se le pedirá que se conecte a la instancia de SQL Server. ‎Asegúrese de que la opción **Autenticación de Windows** esté seleccionada y, a continuación, seleccione **Conectar**.

1. Seleccione **Nueva consulta**.

1. Cree la credencial que se usará para obtener acceso al almacenamiento en la nube con el siguiente código de Transact-SQL. Rellene los valores adecuados y seleccione **Ejecutar**.

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

    Ambos resultados de **dp300storage** son el nombre de la cuenta de almacenamiento creada anteriormente y **sas_token** es el valor generado al final de la tarea anterior.

    `'se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=csig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D'`

1. Puede comprobar si la credencial se creó correctamente; para ello, vaya a **Security -> Credentials (Credenciales** ) en El explorador de objetos.

    ![Captura de pantalla de la credencial en SSMS.](../images/dp-300-module-15-lab-17.png)

1. Si no ha escrito bien algún dato y necesita volver a crear la credencial, puede quitarla con el siguiente comando, pero asegúrese de cambiar el nombre de la cuenta de almacenamiento:

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## Copia de seguridad en URL

1. Realice una copia de seguridad de la base de datos WideWorldImporters en Azure con el siguiente comando en Transact-SQL:

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    Donde **<storage_account_name>** es el nombre de la cuenta de almacenamiento único que se usó. La salida debe devolver algo parecido a lo siguiente.

    ![Captura de pantalla de la página de copia de seguridad.](../images/dp-300-module-15-lab-18.png)

    Si algo se configura incorrectamente, verá un mensaje de error similar al siguiente:

    ![Captura de pantalla de la página de copia de seguridad.](../images/dp-300-module-15-lab-10.png)

    Si se produce un error, compruebe que no escribió incorrectamente nada y que todo se haya creado correctamente.

## Validación de la copia de seguridad mediante la CLI de Azure

Para ver si el archivo se encuentra realmente en Azure, puede usar Explorador de Storage (versión preliminar) o Azure Cloud Shell.

1. Abra un explorador web y vaya a [https://portal.azure.com](https://portal.azure.com/). Conectar al Portal mediante Azure **Nombre de usuario** y **contraseña** proporcionados en la **pestaña Recursos** de esta máquina virtual de laboratorio.

1. Use Azure Cloud Shell para ejecutar el siguiente comando de la CLI de Azure:

    ```bash
    az storage blob list -c "backups" --account-name "dp300backupstorage1234" --account-key "storage_key" --output table
    ```

    Asegúrese de usar el mismo nombre de cuenta de almacenamiento único (después de **--account-name**) y la clave de cuenta (después de la **clave --account-key**) que usó en los comandos anteriores.

    ![Captura de pantalla de la copia de seguridad en el contenedor.](../images/dp-300-module-15-lab-19.png)

    Podemos confirmar que el archivo de copia de seguridad se generó correctamente.

## Validación de la copia de seguridad mediante la CLI de Azure y el Explorador de Storage

1. Para usar el Explorador de Storage (versión preliminar), vaya a la Página principal de Azure Portal y seleccione **Cuentas de almacenamiento**.

    ![Captura de pantalla que muestra la selección de una cuenta de almacenamiento.](../images/dp-300-module-15-lab-11.png)

1. Seleccione el nombre de la cuenta de almacenamiento único que creó para las copias de seguridad.

1. En el panel de navegación izquierdo, seleccione **Explorador de Storage (versión preliminar)**. Expanda **Contenedores de blobs**.

    ![Captura de pantalla que muestra el archivo de copia de seguridad en la cuenta de almacenamiento.](../images/dp-300-module-15-lab-12.png)

1. Seleccione **Copias de seguridad**.

    ![Captura de pantalla que muestra el archivo de copia de seguridad en la cuenta de almacenamiento.](../images/dp-300-module-15-lab-13.png)

1. Tenga en cuenta que el archivo de copia de seguridad se almacena en el contenedor.

    ![Captura de pantalla que muestra el archivo de copia de seguridad en el explorador de almacenamiento.](../images/dp-300-module-15-lab-14.png)

## Restauración desde URL

En esta tarea verá cómo restaurar una base de datos.

1. En **SQL Server Management Studio (SSMS),** seleccione **Nueva consulta** y pegue y ejecute la consulta siguiente.

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

    ![Captura de pantalla que muestra el nombre del cliente antes de ejecutar la actualización.](../images/dp-300-module-15-lab-21.png)

1. Ejecute este comando para cambiar el nombre de ese cliente.

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. Vuelva a ejecutar el **paso 2** para comprobar que se haya cambiado el nombre. Ahora, imagine que alguien ha cambiado miles o millones de filas sin una cláusula WHERE, o bien mediante la cláusula WHERE equivocada. Una de las soluciones implica restaurar la base de datos a partir de la última copia de seguridad disponible.

    ![Captura de pantalla que muestra el nombre del cliente después de ejecutar la actualización.](../images/dp-300-module-15-lab-15.png)

1. Para restaurar la base de datos y volver a un punto anterior al cambio realizado en el paso 3, ejecute lo siguiente.

    **Nota:** **SET SINGLE_USER sintaxis WITH ROLLBACK IMMEDIATE** , todas las transacciones abiertas se revertirán. Esto puede impedir que se produzca un error en la restauración debido a conexiones activas.

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

    Donde **<storage_account_name>** es el nombre de cuenta de almacenamiento único que creó.

    La salida debería ser similar a esta:

    ![Captura de pantalla que muestra la base de datos de restauración a partir de la dirección URL que se ejecuta.](../images/dp-300-module-15-lab-20.png)

1. Vuelva a ejecutar el **paso 2** para comprobar que se han restaurado los datos.

    ![Captura de pantalla que muestra la columna con el valor correcto.](../images/dp-300-module-15-lab-21.png)

Es importante entender los componentes y la interacción entre ellos para realizar una copia de seguridad o una restauración desde el servicio Microsoft Azure Blob Storage.

Ahora ya puede realizar la copia de seguridad de una base de datos en una dirección URL de Azure y, si es necesario, restaurarla.
