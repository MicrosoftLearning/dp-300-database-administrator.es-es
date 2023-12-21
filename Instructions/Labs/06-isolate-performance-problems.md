---
lab:
  title: 'Laboratorio 6: aislamiento de problemas de rendimiento mediante supervisión'
  module: Monitor and optimize operational resources in Azure SQL
---

# Aislamiento de problemas de rendimiento mediante supervisión

**Tiempo estimado: 30 minutos**

Los alumnos tomarán la información adquirida en las lecciones para definir los resultados de un proyecto de transformación digital dentro de AdventureWorks. Al examinar Azure Portal, así como otras herramientas, los alumnos determinarán cómo usar herramientas para identificar y resolver problemas relacionados con el rendimiento.

Le han contratado como administrador de bases de datos para identificar problemas relacionados con el rendimiento y proporcionar soluciones viables para resolver los problemas detectados. Debe usar Azure Portal para identificar los problemas de rendimiento y sugerir métodos para resolverlos.

**Nota:** estos ejercicios piden copiar y pegar código T-SQL. Comprueba que el código se ha copiado correctamente antes de ejecutar el código.

## Revisión del uso de la CPU en Azure Portal

1. En la máquina virtual del laboratorio, inicia una sesión del explorador y desplázate a [https://portal.azure.com](https://portal.azure.com/). Conéctate al Portal con el **Nombre de usuario** y la **Contraseña** de Azure proporcionados en la pestaña **Recursos** de esta máquina virtual de laboratorio.

    ![Imagen 1](../images/dp-300-module-01-lab-01.png)

1. En Azure Portal, busca "Servidores SQL" en el cuadro de búsqueda de la parte superior y después haz clic en **Servidores SQL** en la lista de opciones.

    ![Captura de pantalla de una descripción de publicación de redes sociales generada automáticamente](../images/dp-300-module-04-lab-1.png)

1. Selecciona el nombre **dp300-lab-XXXXXXXX** que se va a llevar a la página de detalles (es posible que tenga un grupo de recursos y una ubicación diferentes asignados para el servidor SQL).

    ![Captura de pantalla de una descripción de publicación de redes sociales generada automáticamente](../images/dp-300-module-04-lab-2.png)

1. En la hoja principal de Azure SQL Server, desplázate a la sección **Configuración**, selecciona **Bases de datos SQL** y después selecciona el nombre de la base de datos.

    ![Captura de pantalla en la que se muestra la selección de la base de datos AdventureWOrksLT](../images/dp-300-module-05-lab-04.png)

1. En la página principal de la base de datos, selecciona **Establecer firewall de servidor**.

    ![Captura de pantalla que muestra la selección de Establecer el firewall del servidor](../images/dp-300-module-06-lab-01.png)

1. En la página **Redes**, selecciona **+ Agregar la dirección IPv4 del cliente (su dirección IP)** y después selecciona **Guardar**.

    ![Captura de pantalla que muestra la selección de Agregar IP de cliente](../images/dp-300-module-06-lab-02.png)

1. En la navegación que hay encima de **Redes**, selecciona el vínculo que comienza con **AdventureWorksLT**.

    ![Captura de pantalla que muestra la selección de AdventureWorks](../images/dp-300-module-06-lab-03.png)

1. En la barra de navegación izquierda, seleccione **Editor de consultas (versión preliminar)**.

    ![Captura de pantalla que muestra la selección del vínculo del editor de consultas (versión preliminar)](../images/dp-300-module-06-lab-04.png)

    **Nota:** esta característica se encuentra en versión preliminar.

1. En **Contraseña**, escribe ** P@ssw0rd01** y selecciona **Aceptar**.

    ![Captura de pantalla que muestra las propiedades de conexión del editor de consultas.](../images/dp-300-module-06-lab-05.png)

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

    ![Captura de pantalla que muestra la consulta](../images/dp-300-module-06-lab-06.png)

1. Espere a que se complete la consulta.

1. En la hoja de la base de datos **AdventureWorksLT**, selecciona el icono **Métricas** en la sección **Supervisión**.

    ![Captura de pantalla que muestra la selección del icono Métricas](../images/dp-300-module-06-lab-07.png)

1. Cambia la opción de menú **Métricas** para reflejar el **porcentaje de CPU** y después selecciona una **Agregación** de **Promedio**. Se mostrará el porcentaje promedio de CPU para el período de tiempo determinado.

    ![Captura de pantalla que muestra el porcentaje de CPU](../images/dp-300-module-06-lab-08.png)

1. Observa el promedio de CPU a lo largo del tiempo. Es posible que los resultados sean ligeramente diferentes. Como alternativa, puedes ejecutar la consulta varias veces para obtener resultados más sustanciales.

    ![Captura de pantalla que muestra la agregación de promedio](../images/dp-300-module-06-lab-09.png)

## Identificación de consultas de consumo de CPU alto

1. Busca el icono de **Información de rendimiento de consultas** en la sección **Rendimiento inteligente** de la hoja de la base de datos **AdventureWorksLT**.

    ![Captura de pantalla que muestra el icono de Información de rendimiento de consultas](../images/dp-300-module-06-lab-10.png)

1. Seleccione **Restablecer la configuración**.

    ![Captura de pantalla que muestra la opción Restablecer la configuración](../images/dp-300-module-06-lab-11.png)

1. Haga clic en la consulta en la cuadrícula situada debajo del gráfico. Si no ve ninguna consulta, espere 2 minutos y seleccione **Actualizar**.

    **Nota:** es posible que tenga una duración y un identificador de consulta diferentes. Si ves más de una consulta, haz clic en cada una para observar los resultados.

    ![Captura de pantalla que muestra la agregación de promedio](../images/dp-300-module-06-lab-12.png)

Para esta consulta, puedes ver que la duración total fue superior a un minuto y que se ejecutó aproximadamente 10 000 veces.

En este ejercicio, has aprendido a explorar los recursos del servidor para una instancia de Azure SQL Database e identificar posibles problemas de rendimiento de las consultas a través de Información de rendimiento de consultas.
