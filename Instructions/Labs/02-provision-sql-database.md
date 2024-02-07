---
lab:
  title: 'Laboratorio: Aprovisionamiento de Azure SQL Database'
  module: Plan and Implement Data Platform Resources
---

# Aprovisionamiento de una base de datos de Azure SQL

**Tiempo estimado**: 20 minutos

Los alumnos configurarán los recursos básicos necesarios para implementar una instancia de Azure SQL Database con un punto de conexión de red virtual. Conectar ivity to the SQL Database will be validated using Azure Data Studio from the lab VM.

Como administrador de bases de datos, configurará una nueva base de datos SQL, incluido un punto de conexión de red virtual para aumentar y simplificar la seguridad de la implementación. Azure Data Studio se usará para evaluar el uso de un bloc de notas de SQL para consultar datos y revisar los resultados.

## Navegue hasta Azure Portal.

1. En la máquina virtual del laboratorio, inicie una sesión del explorador y vaya a [https://portal.azure.com](https://portal.azure.com/). Conectar al Portal mediante Azure **Nombre de usuario** y **contraseña** proporcionados en la **pestaña Recursos** de esta máquina virtual de laboratorio.

    ![Imagen 1](../images/dp-300-module-01-lab-01.png)

1. En Azure Portal, busque "grupos de recursos" en el cuadro de búsqueda de la parte superior y, a continuación, seleccione **Grupos** de recursos en la lista de opciones.

    ![Imagen 1](../images/dp-300-module-02-lab-45.png)

1. En la **página Grupo** de recursos, compruebe el grupo de recursos que aparece (debería empezar con *contoso-rg*), anote la **ubicación** asignada al grupo de recursos, ya que la usará en el ejercicio siguiente.

    **Nota:** Es posible que tenga asignada una ubicación diferente.

    ![Imagen 1](../images/dp-300-module-02-lab-46.png)

## Creación de una red virtual

1. En el menú de navegación izquierdo de Azure Portal, seleccione Inicio.  

    ![Imagen 2](../images/dp-300-module-02-lab-01_1.png)

1. En el panel de navegación izquierdo, haga clic en **Redes**.  

1. Haga clic en **Crear** para abrir la página **Crear una red virtual (clásica).** En la pestaña **Datos básicos**, complete la siguiente información:

    - **Suscripción**: &lt;Su suscripción&gt;
    - **Grupo de recursos:** a partir de *contoso-rg*
    - **Nombre:** lab02-vnet
    - **Ubicación:** seleccione la misma región en la que se creó el grupo de recursos.

1. Haga clic en **Revisar y crear**, revise la configuración de la nueva red virtual y, a continuación, haga clic en **Crear**.

1. Configure el intervalo IP de la red virtual para el punto de conexión de azure SQL Database; para ello, vaya a la red virtual creada y, en el **panel Configuración**, haga clic en **Subredes**.

1. En Nombre de subred, haga clic en el vínculo predeterminado. Tenga en cuenta que el **intervalo** de direcciones de subred que ve podría ser diferente.

1. En el control flotante Editar subred de la derecha, expanda la lista desplegable Servicios y marque Microsoft.Sql. Seleccione **Guardar**.

## Aprovisionamiento de una base de datos de Azure SQL

1. Busque Base de datos SQL en el cuadro de búsqueda de la parte superior y, después, seleccione Base de datos SQL en la lista de opciones.

    ![Imagen 5](../images/dp-300-module-02-lab-10.png)

1. En la página **Bases de datos SQL**, seleccione **+Crear**.

    ![Imagen 6](../images/dp-300-module-02-lab-10_1.png)

1. En la **página Crear base de datos** SQL, seleccione las siguientes opciones en la **pestaña Aspectos básicos** y, a continuación, haga clic en **Siguiente: Redes**.

    - **Suscripción**: &lt;Su suscripción&gt;
    - **Grupo de recursos:** a partir de *contoso-rg*
    - **Nombre de la base de datos**: AdventureWorksLT.
    - **Servidor:** haga clic en **El vínculo Crear nuevo** . Se abrirá la página del servidor de bases de datos. Proporcione los detalles del servidor como se indica a continuación:
        - Nombre del servidor: azuresql-server- sus iniciales (en minúsculas) (un nombre de servidor debe ser único globalmente)
        - **Ubicación:** &lt;la región local, igual que la región seleccionada para el grupo de recursos; de lo contrario, puede producir un error.&gt;
        - **Método de autenticación**: use la autenticación de SQL.
        - Inicio de sesión del administrador del servidor: dwdbadmin.
        - **Contraseña**: dp300P@ssword.
        - **Confirmar contraseña ***dp300P@ssword

        La **página Crear servidor de SQL Database** debe ser similar a la siguiente. Después, haga clic en **Aceptar**.

        ![Imagen 7](../images/dp-300-module-02-lab-11.png)

    -  De vuelta a la **página Crear base de datos** SQL, asegúrese de que **Desea usar el grupo elástico?** está establecido **en No**.
    -  En la opción **Proceso y almacenamiento**, seleccione **Configurar base de datos**. En la página Configurar **, en **la **lista desplegable Nivel de servicio**, seleccione **Básico** y, a continuación **, Aplicar**.

    **Nota:** Anote este nombre de servidor y su información de inicio de sesión. Lo usará en pasos posteriores.

1. Para la **opción Redundancia** de almacenamiento de copia de seguridad, mantenga el valor predeterminado: **Almacenamiento** de copia de seguridad con redundancia geográfica.

1. Haga clic en **Siguiente: Redes**.

1. En la **pestaña Redes**, en **la opción Network Conectar ivity (Red Conectar ivity**), haga clic en el **botón de radio Punto de conexión** privado.

    ![Imagen 8](../images/dp-300-module-02-lab-14.png)

1. Posteriormente, seleccione el vínculo Agregar un punto de conexión privado en Puntos de conexión privados

    ![Imagen 9](../images/dp-300-module-02-lab-15.png)

1. Complete el panel derecho Crear punto de conexión** privado de la **siguiente manera:

    - **Suscripción**: &lt;Su suscripción&gt;
    - **Grupo de recursos:** a partir de *contoso-rg*
    - **Ubicación:** &lt;la región local, igual que la región seleccionada para el grupo de recursos; de lo contrario, puede producir un error.&gt;
    - **Nombre:** DP-300-SQL-Endpoint
    - Recurso secundario de destino: SqlServer
    - **Virtual Network (VNET)**
    - **Subred:** lab02-vnet/default (10.x.0.0/24)
    - **Integrar con la zona DNS privada**: sí
    - **DNS privado zona: mantenga el valor predeterminado.**
    - Revise la configuración y, luego, haga clic en **Aceptar**.  

    ![Imagen 10](../images/dp-300-module-02-lab-16.png)

1. El nuevo punto de conexión se mostrará en la página **Puntos de conexión** .

    ![Imagen 11](../images/dp-300-module-02-lab-17.png)

1. Haga clic en **Siguiente: Seguridad** y, a continuación, en **Siguiente: Configuración** adicional.  

1. En la pestaña **Configuración adicional**, seleccione **Ejemplo** en **Usar datos existentes**. Seleccione **Aceptar** si se muestra un mensaje emergente para la base de datos de ejemplo.

    ![Imagen 12](../images/dp-300-module-02-lab-18.png)

1. Haga clic en **Revisar y crear**.

1. Revise la configuración y haga clic en **Crear**.

1. Una vez finalizada la implementación, haga clic en **Ir al recurso**.

## Acceso a Azure SQL Database.

1. En la **página base de datos** SQL, seleccione la **sección Información general** y, a continuación, seleccione el vínculo para el nombre del servidor en la sección superior:

    ![Imagen 13](../images/dp-300-module-02-lab-19.png)

1. En la hoja de navegación servidores SQL Server, seleccione **Redes en la **sección Seguridad****.

    ![Imagen 14](../images/dp-300-module-02-lab-20.png)

1. En la **pestaña Acceso** público, seleccione **Redes seleccionadas** y, a continuación, active la **propiedad Permitir que los servicios y recursos de Azure accedan a esta propiedad de servidor** . Haga clic en **Save**(Guardar).

    ![Imagen 15](../images/dp-300-module-02-lab-21.png)

## Conexión a Azure SQL Database mediante Azure Data Studio

1. Inicie Azure Data Studio desde la máquina virtual del laboratorio.

    - Es posible que vea este elemento emergente en el inicio inicial de Azure Data Studio. Si lo recibe, haga clic en **Sí (recomendado)**  

        ![Imagen 16](../images/dp-300-module-02-lab-22.png)

1. Cuando se abra Azure Data Studio, haga clic en el **botón Conectar ions** en la esquina superior izquierda y, a continuación **, en Agregar Conectar ion**.

    ![Imagen 17](../images/dp-300-module-02-lab-25.png)

1. En la barra lateral **Conexión**, rellene la sección **Detalles de conexión** con información de conexión para conectarse a la base de datos AdventureWorksLT creada en los pasos anteriores.

    - Tipo de conexión: **Microsoft SQL Server**
    - En Nombre de SQL Server, escriba el nombre del servidor SQL que ha creado antes. Por ejemplo: **dp300-lab-xxxxxxxx.database.windows.net** (donde "xxxxxxxx" es un número ramdom)
    - Tipo de autenticación: **Inicio de sesión SQL**
    - Nombre de usuario: **dp300admin**
    - Contraseña: **dp300P@ssword!**
    - Expanda la lista desplegable Base de datos para seleccionar **AdventureWorksLT** 
        - **NOTA:** Es posible que se le pida que agregue una regla de firewall que permita el acceso IP de cliente a este servidor. Si se le pide que agregue una regla de firewall, haga clic en **Agregar cuenta** e inicie sesión en su cuenta de Azure. En **la pantalla Crear nueva regla** de firewall, haga clic en **Aceptar**.

        ![Imagen 18](../images/dp-300-module-02-lab-26.png)

        Como alternativa, puede crear manualmente una regla de firewall para el servidor SQL Server en Azure Portal; para ello, vaya a su servidor SQL Server, seleccione **Redes** y, a continuación, seleccione **+ Agregar la dirección IPv4 del cliente (su dirección IP).**

        ![Imagen 18](../images/dp-300-module-02-lab-47.png)

    De nuevo en la barra lateral Conectar ion, siga rellenando los detalles de conexión:  

    - El grupo de servidores permanecerá en el **&lt;valor predeterminado&gt;**.
    - El valor de Nombre (opcional) se puede rellenar con un nombre descriptivo de la base de datos, si se quiere.
    - Revise la configuración y haga clic en **Conectar**  

    ![Imagen 19](../images/dp-300-module-02-lab-27.png)

1. ADS se conectará a la base de datos y mostrará información básica acerca de la base de datos, incluida una lista parcial de objetos.

    ![Imagen 20](../images/dp-300-module-02-lab-28.png)

## Consulta de Azure SQL Database con un cuaderno de SQL

1. En Azure Data Studio, conectado a la base de datos AdventureWorksLT de este laboratorio, haga clic en el **botón Nuevo cuaderno** .

    ![Imagen 21](../images/dp-300-module-02-lab-29.png)

1. Haga clic en el **vínculo +Texto** para agregar un nuevo cuadro de texto en el cuaderno.  

    ![Imagen 22](../images/dp-300-module-02-lab-30.png)

Dentro del cuaderno, puede insertar texto sin formato para explicar las consultas o los conjuntos de resultados.

1. Escriba el texto **Diez principales clientes por subtotal de pedido**.

    ![Captura de pantalla de un teléfono móvil

Descripción generada automáticamente](../images/dp-300-module-02-lab-31.png)

1. Haga clic en el **botón + Celda** y, a continuación **, en Celda** de código para agregar una nueva celda de código al final del cuaderno.  

    ![Imagen 23](../images/dp-300-module-02-lab-32.png)

5. Pegue la siguiente instrucción SQL en la nueva celda.

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

1. Seleccione el círculo azul con la flecha para ejecutar la consulta. Observe de qué forma los resultados se incluyen en la celda con la consulta.

1. Haga clic en el **botón + Texto** para agregar una nueva celda de texto.

1. Escriba el texto **Top Ten Ordered Product Categories (Primeras diez categorías** de productos ordenados), por lo que es negrita si lo desea.

1. Vuelva a seleccionar **+ Código** para agregar una nueva celda y pegue la siguiente instrucción SQL en la celda

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

1. Seleccione el círculo azul con la flecha para ejecutar la consulta.

1. Para ejecutar todas las celdas del cuaderno y presentar los resultados, seleccione el botón **Ejecutar celdas** en la barra de herramientas.

    ![Imagen 17](../images/dp-300-module-02-lab-33.png)

1. En Azure Data Studio, guarde el cuaderno en el menú Archivo (guardar o guardar como) en la **ruta de acceso de C:\Labfiles\Deploy Azure SQL Database** (cree la estructura de carpetas si no existe). (Asegúrese de que la extensión de archivo es ).

1. Cierre la pestaña del cuaderno desde azure Data Studio. En el menú Archivo, seleccione Abrir archivo y abra el cuaderno que acaba de guardar. Observe que los resultados de la consulta se han guardado junto con las consultas en el cuaderno.

En este ejercicio, ha visto cómo implementar una instancia de Azure SQL Database con un punto de conexión de red virtual. También pudo conectarse a SQL Database que ha creado mediante SQL Server Management Studio.
