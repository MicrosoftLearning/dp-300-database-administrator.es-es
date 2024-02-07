---
lab:
  title: 'Laboratorio: Aprovisionamiento de SQL Server en una máquina virtual de Azure'
  module: Plan and Implement Data Platform Resources
---

# Aprovisionamiento de un servidor SQL Server en una máquina virtual de Azure

**Tiempo estimado**: 20 minutos

Los alumnos explorarán Azure Portal y lo usarán para crear una máquina virtual de Azure con SQL Server 2019 instalado. Conectarse a una máquina virtual a través del Protocolo de escritorio remoto (RDP).

Es administrador de bases de datos para AdventureWorks. Debe crear un entorno de prueba para usarlo en una prueba de concepto. La prueba de concepto usará SQL Server en una máquina virtual de Azure. Debe configurar la máquina virtual, restaurar la base de datos y consultarla para asegurarse de que está disponible.

## Implementación de SQL Server en una máquina virtual de Azure

1. En la máquina virtual del laboratorio, inicie una sesión del explorador y vaya a [https://portal.azure.com](https://portal.azure.com/)e inicie sesión con la cuenta Microsoft asociada a la suscripción de Azure.

    ![Imagen 1](../images/dp-300-module-01-lab-01.png)

1. Localice la barra de búsqueda en la parte superior de la página. Busque Azure SQL. Seleccione el resultado de la búsqueda de Azure SQL que aparece en los resultados, debajo de Servicios.

    ![Imagen 9](../images/dp-300-module-01-lab-09.png)

1. En la hoja **Azure Cosmos DB**, seleccione **+ Crear**.

    ![Imagen 10](../images/dp-300-module-01-lab-10.png)

1. En la hoja Seleccione una opción de implementación de SQL, abra el cuadro desplegable que aparece en Máquinas virtuales de SQL. Seleccione **Free SQL Server License: SQL 2019 Developer on Windows Server 2019** (Licencia gratuita de SQL Server: SQL 2019 Developer en Windows Server 2019). Seleccione **Crear**.

    ![Imagen 11](../images/dp-300-module-01-lab-11.png)

1. En la página Crear una máquina virtual, escriba los siguientes datos:

    - **Suscripción**: &lt;Su suscripción&gt;
    - Grupo de recursos: su grupo de recursos.
    - Nombre de la máquina virtual: azureSQLserverVM.
    - **Región:** &lt;la región local, igual que la región seleccionada para el grupo de recursos.&gt;
    - **Opciones de disponibilidad**: no se requiere redundancia de la infraestructura.
    - **Licencia gratuita: SQL Server 2017 Developer en Windows Server 2016**
    - **Instancia de acceso puntual de Azure:** no (desactivada)
    - Standard D2S V3 (2 vCPU, 8 GiB de memoria) Es posible que tenga que seleccionar el vínculo "Ver todos los tamaños" para ver esta opción).
    - Nombre de usuario de la cuenta de administrador: labadmin.
    - **Administración contraseña de cuentaistrator:** pwd! DP300lab01 (o su propia contraseña que cumpla los criterios)
    - **Selección de puertos de entrada** = RDP (3389)
    - ¿Quiere usar una licencia de Windows Server existente?

    Anote el nombre de usuario y la contraseña para más adelante.

    ![Imagen 12](../images/dp-300-module-01-lab-12.png)

1. Vaya a la pestaña **Discos** y revise la configuración.

    ![Imagen 13](../images/dp-300-module-01-lab-13.png)

1. Vaya a la pestaña **Redes** y revise la configuración.

    ![Imagen 14](../images/dp-300-module-01-lab-14.png)

1. Vaya a la pestaña **Redes** y revise la configuración.

    ![Imagen 15](../images/dp-300-module-01-lab-15.png)

    Compruebe que **La opción Habilitar auto_shutdown** está desactivada.

1. Vaya a la pestaña **Opciones avanzadas** y revise la configuración.

    ![Imagen 16](../images/dp-300-module-01-lab-16.png)

1. Vaya a la pestaña **Configuración de SQL Server** y revise la configuración.

    ![Imagen 17](../images/dp-300-module-01-lab-17.png)

    También puede configurar el almacenamiento de la máquina virtual con SQL Server en esta pantalla. De forma predeterminada, la plantilla de máquina virtual de Azure con SQL Server crea un disco prémium con almacenamiento en caché de lectura para los datos, un disco prémium sin almacenamiento en caché para el registro de transacciones y usa el disco SSD local (D:\ en Windows) para tempdb.

1. Seleccione el botón **Revisar y crear**. Seleccione **Crear**.

    ![Imagen 18](../images/dp-300-module-01-lab-18.png)

1. En la hoja de información general de la implementación, espere hasta que esta finalice. La máquina virtual tarda aproximadamente 5-10 minutos en implementarse. Una vez finalizada la implementación, seleccione **Ir al recurso**.

    La implementación puede tardar varios minutos en completarse.

    ![Imagen 19](../images/dp-300-module-01-lab-19.png)

1. En la página de información general de la máquina virtual, desplácese por las opciones de menú del recurso para revisar lo que está disponible.

    ![Imagen 20](../images/dp-300-module-01-lab-20.png)

## Conexión a SQL Server en una máquina virtual de Azure

1. En la página de información general de la máquina virtual, seleccione el botón **Conectar** y, luego, **RDP**.

    ![Imagen 21](../images/dp-300-module-01-lab-21.png)

1. En la pestaña RDP, seleccione Descargar archivo RDP.

    ![Imagen 22](../images/dp-300-module-01-lab-22.png)

    **Nota:** Si ve que no se cumple** el requisito previo del puerto de error**. Asegúrese de seleccionar el vínculo para agregar una regla de grupo de seguridad de red de entrada con el puerto de destino mencionado en el *campo Número* de puerto.

    ![Imagen 22_1](../images/dp-300-module-01-lab-22_1.png)

1. Abra el archivo RDP que descarga. Cuando aparezca un cuadro de diálogo en el que se le pregunte si desea conectarse, seleccione **Conectar**.

    ![Imagen 23](../images/dp-300-module-01-lab-23.png)

1. Escriba el nombre de usuario y la contraseña seleccionados durante el proceso de aprovisionamiento de máquinas virtuales. Después, seleccione **Aceptar**.

    ![Imagen 24](../images/dp-300-module-01-lab-24.png)

1. Cuando aparezca el **cuadro de diálogo Conectar ion** de Escritorio remoto en el que se le pregunta si desea conectarse, seleccione **Sí**.

    ![Imagen 26](../images/dp-300-module-01-lab-26.png)

1. Seleccione el botón Inicio de Windows y escriba SSMS. Seleccione **Microsoft SQL Server Management Studio 18** en la lista.  

    ![Imagen 34](../images/dp-300-module-01-lab-34.png)

1. Cuando se abra SSMS, observe que el **cuadro de diálogo Conectar al** servidor se rellenará previamente con el nombre de instancia predeterminado. Seleccione **Conectar**.

    ![Imagen 35](../images/dp-300-module-01-lab-35.png)

Azure Portal proporciona herramientas eficaces para administrar un servidor SQL Server hospedado en una máquina virtual. Estas herramientas incluyen el control de la aplicación de revisiones automatizada, copias de seguridad automatizadas y una forma sencilla de configurar la alta disponibilidad.
