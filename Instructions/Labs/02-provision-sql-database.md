---
lab:
  title: 'Laboratorio 2: Aprovisionamiento de Azure SQL Database'
  module: Plan and Implement Data Platform Resources
---

# Aprovisionamiento de Azure SQL Database

**Tiempo estimado**: 40 minutos

Los alumnos configurarán los recursos básicos necesarios para implementar Azure SQL Database con un punto de conexión de red virtual. La conectividad con SQL Database se validará mediante SQL Server Management Studio desde la máquina virtual del laboratorio, si está disponible, o desde la configuración de tu máquina local.

Como administrador de base de datos de AdventureWorks, configurarás una nueva base de datos SQL, incluido un punto de conexión de red virtual para aumentar y simplificar la seguridad de la implementación. SQL Server Management Studio se usará para evaluar el uso de un cuaderno de SQL para consultar datos y retener los resultados.

## Ve a Azure Portal

1. En la máquina virtual del laboratorio, si se te proporciona una, o, de lo contrario, en tu máquina local, abre una ventana del explorador.

1. Ve a Azure Portal en [https://portal.azure.com](https://portal.azure.com/). Inicia sesión en Azure Portal con tu cuenta de Azure o con las credenciales proporcionadas, si dispones de ellas.

1. En Azure Portal, busca *grupos de recursos* en el cuadro de búsqueda de la parte superior y selecciona **Grupos de recursos** en la lista de opciones.

1. En la página **Grupo de recursos**, si se proporciona, selecciona el grupo de recursos que comienza por *contoso-rg*. Si este grupo de recursos no existe, crea un nuevo grupo de recursos denominado *contoso-rg* en la región local o usa un grupo de recursos existente y toma nota de la región en la que está.

## Creación de una red virtual

1. En la página principal de Azure Portal, selecciona el menú izquierdo.  

1. En el panel de navegación izquierdo, haz clic en **Redes virtuales**  

1. Haz clic en **+ Crear** para abrir la página **Crear una red virtual**. En la pestaña **Datos básicos**, completa la siguiente información:

    - **Suscripción**: &lt;tu suscripción&gt;
    - **Grupo de recursos:** que empieza en *DP300* o el grupo de recursos que seleccionó anteriormente
    - **Nombre:** lab02-vnet
    - **Región:** selecciona la misma región en la que se creó el grupo de recursos

1. Selecciona **Revisión + crear**, revisa la configuración de la nueva red virtual y después selecciona **Crear**.

## Disposición de una base de datos de Azure SQL en Azure Portal

1. En Azure Portal, busca *Bases de datos SQL* en el cuadro de búsqueda de la parte superior y después selecciona **Bases de datos SQL** en la lista de opciones.

1. En la hoja **Bases de datos SQL**, selecciona **+ Crear**.

1. En la página **Crear base de datos SQL**, selecciona las siguientes opciones en la pestaña **Aspectos básicos** y, después, haz clic en **Siguiente: Redes**.

    - **Suscripción**: &lt;tu suscripción&gt;
    - **Grupo de recursos:** que empieza en *DP300* o el grupo de recursos que seleccionó anteriormente
    - **Nombre de base de datos**: AdventureWorksLT
    - **Servidor:** selecciona el vínculo **Crear nuevo**. Se abrirá la página del **servidor de SQL Database**. Proporciona los detalles del servidor como se indica a continuación:
        - **Nombre del servidor:** dp300-lab-&lt;tus iniciales (minúsculas)&gt; y, si es necesario, un número aleatorio de 5 dígitos (el nombre del servidor debe ser único globalmente)
        - **Ubicación:**&lt;la región local, igual que la región seleccionada para el grupo de recursos; de lo contrario, se puede producir un error&gt;
        - **Método de autenticación**: usa la autenticación de SQL
        - **Inicio de sesión del administrador del servidor:** dp300admin
        - **Contraseña:** selecciona una contraseña compleja y toma nota de ella.
        - **Confirmar contraseña:** selecciona la misma contraseña que seleccionaste anteriormente.
    - Selecciona **Aceptar** para volver a la página **Crear base de datos SQL**.
    - Deja la opción **¿Quieres usar un grupo elástico?** establecida en **No**.
    - **Entorno de la carga de trabajo**: desarrollo
    - En la opción **Proceso y almacenamiento**, selecciona el vínculo **Configurar base de datos**. En la página **Configurar**, en el elemento desplegable **Nivel de servicio**, selecciona **Básico** y, a continuación, **Aplicar**.

1. Para la opción **Redundancia de almacenamiento de Backup**, mantén el valor predeterminado: **Almacenamiento de copia de seguridad con redundancia local**.

1. Y, después, selecciona **Siguiente: redes**.

1. En la pestaña **Redes**, en la opción **Conectividad de red**, selecciona el botón de radio **Punto de conexión privado**.

1. Después, selecciona el vínculo **+ Agregar un punto de conexión privado** en la opción **Puntos de conexión privados**.

1. Completa el panel derecho **Crear punto de conexión privado** de la siguiente manera:

    - **Suscripción**: &lt;tu suscripción&gt;
    - **Grupo de recursos:** que empieza en *DP300* o el grupo de recursos que seleccionó anteriormente
    - **Ubicación:**&lt;la región local, igual que la región seleccionada para el grupo de recursos; de lo contrario, se puede producir un error&gt;
    - **Nombre:** DP-300-SQL-Endpoint
    - **Recurso secundario de destino:** SqlServer
    - **Red virtual:** lab02-vnet
    - **Subred:** lab02-vnet/default (10.x.0.0/24)
    - **Integrar con la zona DNS privada**: sí
    - **Zona DNS privada:** mantén el valor predeterminado
    - Revisa la configuración y, luego, selecciona **Aceptar**.  

1. El nuevo punto de conexión se mostrará en la lista **Puntos de conexión privados**.

1. Selecciona **Siguiente: Seguridad** y, después, **Siguiente: Configuración adicional**.  

1. En la página **Configuración adicional**, selecciona **Ejemplo** en la opción **Usar datos existentes**. Selecciona **Aceptar** si se muestra un mensaje emergente para la base de datos de ejemplo.

1. Seleccione **Revisar + crear**.

1. Revise la configuración antes de seleccionar **Crear**.

1. Una vez finalizada la implementación, seleccione **Ir al recurso**.

## Habilitar el acceso a Azure SQL Database

1. En la página **SQL Database**, selecciona la sección **Información general** y luego selecciona el vínculo para el nombre del servidor en la sección superior:

1. En la hoja de navegación servidores SQL, selecciona **Redes** en la sección **Seguridad**.

1. En la pestaña **Acceso público**, seleccione **Redes seleccionadas**.

1. Selecciona **+ Agregar la dirección IPv4 del cliente**. Esto agregará una regla de firewall para permitir que la dirección IP actual acceda al servidor SQL Server.

1. Activa la propiedad **Permitir que los servicios y recursos de Azure accedan a este servidor**.

1. Seleccione **Guardar**.

---

## Conectarse a Azure SQL Database mediante SQL Server Management Studio

1. En Azure Portal, selecciona las **Bases de datos SQL**, en el panel de navegación de la izquierda. Y después selecciona la base de datos **AdventureWorksLT**.

1. En la página **Información general**, copia el valor **Nombre de servidor**.

1. Inicia SQL Server Management Studio desde la máquina virtual del laboratorio, si se te proporciona, o desde tu máquina local, si no es así.

1. En el cuadro de diálogo **Conectar con el servidor**, pega el valor **Nombre del servidor**, copiado desde Azure Portal.

1. En la lista desplegable **Autenticación**, selecciona **Autenticación de SQL Server**.

1. En el campo **Inicio de sesión**, escribe **dp300admin**.

1. En el campo **Contraseña** , escribe la contraseña seleccionada durante la creación del servidor de SQL Server.

1. Seleccione **Conectar**.

1. Abre SQL Server Management Studio y conéctate a tu servidor de Azure SQL Database. Puedes expandir el servidor y luego el nodo **Bases de datos** para ver la base de datos *AdventureWorksLT*.

## Consultar Azure SQL Database mediante SQL Server Management Studio

1. En SQL Server Management Studio, haz clic con el botón derecho en la base de datos *AdventureWorksLT* y selecciona **Nueva consulta**.

1. Pega la instrucción SQL siguiente en la ventana de consulta:

    ```sql
    SELECT TOP 10 cust.[CustomerID], 
        cust.[CompanyName], 
        SUM(sohead.[SubTotal]) as OverallOrderSubTotal
    FROM [SalesLT].[Customer] cust
        INNER JOIN [SalesLT].[SalesOrderHeader] sohead
             ON sohead.[CustomerID] = cust.[CustomerID]
    GROUP BY cust.[CustomerID], cust.[CompanyName]
    ORDER BY [OverallOrderSubTotal] DESC
    ```

1. Selecciona el botón **Ejecutar** de la barra de herramientas para ejecutar la consulta.

1. En el panel **Resultados**, revisa los resultados de la consulta.

1. Haz clic con el botón derecho en la base de datos *AdventureWorksLT* y selecciona **Nueva consulta**.

1. Pega la instrucción SQL siguiente en la ventana de consulta:

    ```sql
    SELECT TOP 10 cat.[Name] AS ProductCategory, 
        SUM(detail.[OrderQty]) AS OrderedQuantity
    FROM salesLT.[ProductCategory] cat
        INNER JOIN [SalesLT].[Product] prod
            ON prod.[ProductCategoryID] = cat.[ProductCategoryID]
        INNER JOIN [SalesLT].[SalesOrderDetail] detail
            ON detail.[ProductID] = prod.[ProductID]
    GROUP BY cat.[name]
    ORDER BY [OrderedQuantity] DESC
    ```

1. Selecciona el botón **Ejecutar** de la barra de herramientas para ejecutar la consulta.

1. En el panel **Resultados**, revisa los resultados de la consulta.

1. Cierre SQL Server Management Studio. Si se te solicita guardar los cambios, selecciona **No**.

---

## Recursos de limpieza

Si no usa la máquina virtual para ningún otro propósito, puedes limpiar los recursos que creaste en este laboratorio.

### Eliminar el grupo de recursos

Si creaste un nuevo grupo de recursos para este laboratorio, puedes eliminar el grupo de recursos para quitar todos los recursos creados en este laboratorio.

1. En Azure Portal, selecciona **Grupos de recursos** en el panel de navegación izquierdo o busca **Grupos de recursos** en la barra de búsqueda y selecciónalo desde los resultados.

1. Ve al grupo de recursos que creaste para este laboratorio. El grupo de recursos contendrá la máquina virtual y otros recursos creados en este laboratorio.

1. Seleccione **Eliminar grupo de recursos** del menú superior.

1. En el panel **Eliminar un grupo de recursos**, escribe el nombre del grupo de recursos para confirmarlo y selecciona **Eliminar**.

1. Espera a que se elimine el grupo de recursos.

1. Cierra Azure Portal.

### Eliminar solo los recursos del laboratorio

Si no creaste ningún grupo de recursos nuevo para este laboratorio y deseas dejar intacto el grupo de recursos y sus recursos anteriores, aún puedes eliminar los recursos creados en este laboratorio.

1. En Azure Portal, selecciona **Grupos de recursos** en el panel de navegación izquierdo o busca **Grupos de recursos** en la barra de búsqueda y selecciónalo desde los resultados.

1. Ve al grupo de recursos que creaste para este laboratorio. El grupo de recursos contendrá la máquina virtual y otros recursos creados en este laboratorio.

1. Selecciona todos los recursos con el prefijo del nombre de SQL Server que especificaste anteriormente en el laboratorio. Además, selecciona la red virtual y la zona DNS privada que creaste.

1. Seleccione **Eliminar** en el menú superior.

1. En el cuadro de diálogo **Eliminar recursos**, escribe **delete** y selecciona **Eliminar**.

1. Para confirmar la eliminación, escribe el nombre del recurso y selecciona **Eliminar**.

1. Espera a que se eliminen los recursos.

1. Cierra Azure Portal.

---

Completaste correctamente este laboratorio.

En este ejercicio, has visto cómo implementar una Azure SQL Database con un punto de conexión de red virtual. También has podido conectarte a la base de datos SQL que has creado mediante SQL Server Management Studio.
