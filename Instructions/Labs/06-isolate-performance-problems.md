---
lab:
  title: 'Laboratorio 6: aislamiento de problemas de rendimiento mediante supervisión'
  module: Monitor and optimize operational resources in Azure SQL
---

# Aislamiento de problemas de rendimiento mediante supervisión

**Tiempo estimado: 30 minutos**

Los alumnos tomarán la información adquirida en las lecciones para definir los resultados de un proyecto de transformación digital dentro de AdventureWorksLT. Al examinar Azure Portal, así como otras herramientas, los alumnos determinarán cómo usar herramientas para identificar y resolver problemas relacionados con el rendimiento.

Le han contratado como administrador de bases de datos para identificar problemas relacionados con el rendimiento y proporcionar soluciones viables para resolver los problemas detectados. Debe usar Azure Portal para identificar los problemas de rendimiento y sugerir métodos para resolverlos.

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

## Revisión del uso de la CPU en Azure Portal

1. Desde la máquina virtual del laboratorio, o la máquina local si no te se proporcionó ninguna, inicia una sesión del explorador y ve a [https://portal.azure.com](https://portal.azure.com/). Conéctate al portal usando tus credenciales de Azure.

1. En Azure Portal, busca *Servidores SQL* en el cuadro de búsqueda de la parte superior y después haz clic en **Servidores SQL** en la lista de opciones.

1. Selecciona SQL Server **dp300-lab-xxxxxxxx**, donde *xxxxxxxx* es una cadena numérica aleatoria.

    > &#128221; Ten en cuenta que, si usas tu propio servidor de Azure SQL Server no creado por este laboratorio, debes seleccionar el nombre de ese servidor SQL Server.

1. En la página principal de Azure SQL Server, en **Seguridad**, selecciona **Redes**.

1. En la página **Redes**, comprueba si la dirección IP pública actual ya se ha agregado a la lista **Reglas de firewall**; si no es así, selecciona **+ Agregar la dirección IPv4 del cliente (tu dirección IP)** para agregarla y luego selecciona **Guardar**.

1. En la hoja principal de Azure SQL Server, desplázate a la sección **Configuración**, selecciona **Bases de datos SQL** y después selecciona el nombre de la base de datos **AdventureWorksLT**.

1. En la barra de navegación izquierda, seleccione **Editor de consultas (versión preliminar)**.

    **Nota:** esta característica se encuentra en versión preliminar.

1. Selecciona el nombre de usuario administrador de SQL Server y escribe la contraseña o las credenciales de Microsoft Entra, si las tienes asignadas, para conectarte a la base de datos.

1. En **Consulta 1**, escribe la siguiente consulta y selecciona **Ejecutar**:

    ```sql
    DECLARE @Counter INT 
    SET @Counter=1
    WHILE ( @Counter <= 10000)
    BEGIN
        SELECT 
             RTRIM(a.Firstname) + ' ' + RTRIM(a.LastName)
            , b.AddressLine1
            , b.AddressLine2
            , RTRIM(b.City) + ', ' + RTRIM(b.StateProvince) + '  ' + RTRIM(b.PostalCode)
            , CountryRegion
            FROM SalesLT.Customer a
            INNER JOIN SalesLT.CustomerAddress c 
                ON a.CustomerID = c.CustomerID
            RIGHT OUTER JOIN SalesLT.Address b
                ON b.AddressID = c.AddressID
        ORDER BY a.LastName ASC
        SET @Counter  = @Counter  + 1
    END
    ```

1. Espere a que se complete la consulta.

1. Vuelva a ejecutar la consulta *dos* veces más para generar cierta carga de CPU en la base de datos.

1. En la hoja de la base de datos **AdventureWorksLT**, selecciona el icono **Métricas** en la sección **Supervisión**.

    Si aparece el mensaje *Se descartarán los cambios no guardados*, selecciona **Aceptar**.

1. Cambia la opción de menú **Métricas** para reflejar el **porcentaje de CPU** y después selecciona una **Agregación** de **Promedio**. Se mostrará el porcentaje promedio de CPU para el período de tiempo determinado.

1. Observa el promedio de CPU a lo largo del tiempo. Deberías notar un pico en el uso de CPU al final del gráfico cuando se estaba ejecutando la consulta.

## Identificación de consultas de consumo de CPU alto

1. Busca el icono de **Información de rendimiento de consultas** en la sección **Rendimiento inteligente** de la hoja de la base de datos **AdventureWorksLT**.

1. Seleccione **Restablecer la configuración**.

1. Haz clic en la consulta en la cuadrícula situada debajo del gráfico. Si no ves la consulta que se ejecutó antes varias veces, espera de 2 a 5 minutos y selecciona **Actualizar**.

    > &#128221; Si aparece más de una consulta, selecciona cada una de ellas para observar los resultados. Ten en cuenta la gran cantidad de información que hay disponible para cada consulta.

1. Para la consulta que ejecutaste anteriormente, ten en cuenta que la duración total fue superior a un minuto y que se ejecutó alrededor de treinta miles de veces.

1. Al revisar el texto SQL de la página **Detalles de la consulta** frente a la consulta que ejecutaste, observarás que los **Detalles de la consulta** solo incluyen la instrucción **SELECT** y no el bucle **WHILE** u otra instrucción. Esto sucede porque **Información de rendimiento de consultas** se basa en los datos del **Almacén de consultas**, que solo realiza un seguimiento de las instrucciones del lenguaje de manipulación de datos (DML), como **SELECT, INSERT, UPDATE, DELETE, MERGE** y **BULK INSERT** al omitir las instrucciones del lenguaje de definición de datos (DDL).

No todos los problemas de rendimiento están relacionados con un uso elevado de CPU mediante una sola ejecución de consulta. En este caso, la consulta se ejecutó miles de veces, lo que también puede dar lugar a un uso elevado de la CPU.

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

En este ejercicio, has aprendido a explorar los recursos del servidor para una instancia de Azure SQL Database e identificar posibles problemas de rendimiento de las consultas a través de Información de rendimiento de consultas.
