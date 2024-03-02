---
lab:
  title: 'Laboratorio 1: Aprovisionamiento de SQL Server en una máquina virtual de Azure'
  module: Plan and Implement Data Platform Resources
---

# Aprovisionamiento de un servidor SQL Server en una máquina virtual de Azure

**Tiempo estimado**: 30 minutos

Los alumnos explorarán Azure Portal y lo usarán para crear una máquina virtual de Azure con SQL Server 2019 instalado. A continuación, se conectarán a la máquina virtual a través del Protocolo de escritorio remoto.

Es administrador de bases de datos para AdventureWorks. Debe crear un entorno de prueba para usarlo en una prueba de concepto. La prueba de concepto usará SQL Server en una máquina virtual de Azure y una copia de seguridad de la base de datos AdventureWorksDW. Debe configurar la máquina virtual, restaurar la base de datos y consultarla para asegurarse de que está disponible.

## Implementación de SQL Server en una máquina virtual de Azure

1. En la máquina virtual del laboratorio, inicie una sesión del explorador, vaya a [https://portal.azure.com](https://portal.azure.com/) e inicie sesión con la cuenta Microsoft asociada a la suscripción de Azure.

    ![Imagen 1](../images/dp-300-module-01-lab-01.png)

1. Localice la barra de búsqueda en la parte superior de la página. Busque **Azure SQL**. Seleccione el resultado de la búsqueda de **Azure SQL** que aparece en los resultados, debajo de **Servicios**.

    ![Imagen 9](../images/dp-300-module-01-lab-09.png)

1. En la hoja **Azure SQL**, seleccione **Crear**.

    ![Imagen 10](../images/dp-300-module-01-lab-10.png)

1. En la hoja **Seleccione una opción de implementación de SQL**, abra el cuadro desplegable que aparece en **Máquinas virtuales de SQL**. Seleccione la opción etiquetada **Free SQL Server License: SQL 2019 Developer on Windows Server 2022** (Licencia gratuita de SQL Server: SQL 2019 Developer en Windows Server 2022). Seleccione **Crear**.

    ![Imagen 11](../images/dp-300-module-01-lab-11.png)

1. En la página **Crear una máquina virtual**, escriba los siguientes datos:

    - **Suscripción**: &lt;Su suscripción&gt;
    - **Grupo de recursos:** &lt;su grupo de recursos&gt;.
    - **Nombre de la máquina virtual:** azureSQLserverVM
    - **Región:** &lt;la región local, igual que la región seleccionada para el grupo de recursos&gt;
    - **Opciones de disponibilidad:** No se necesita redundancia de la infraestructura
    - **Imagen:** Licencia de SQL Server gratis: SQL 2019 Developer on Windows Server 2022 - Gen1 (Licencia gratuita de SQL Server: SQL 2019 Developer en Windows Server 2022 - Gen1)
    - **Instancia de Azure Spot:** No (desactivado)
    - **Size:** Estándar *D2s_v3* (2 vCPU, 8 GiB de memoria). Es posible que tenga que seleccionar el vínculo "Ver todos los tamaños" para ver esta opción)
    - **Nombre de usuario de la cuenta de administrador:** sqladmin
    - **Contraseña de la cuenta de administrador:** pwd!DP300lab01 (o su propia contraseña que cumpla los criterios)
    - **Seleccionar puertos de entrada:** RDP (3389)
    - **¿Quiere usar una licencia de Windows Server existente?:** No (desactivado)

    Anote el nombre de usuario y la contraseña para más adelante.

    ![Imagen 12](../images/dp-300-module-01-lab-12.png)

1. Vaya a la pestaña **Discos** y revise la configuración.

    ![Imagen 13](../images/dp-300-module-01-lab-13.png)

1. Vaya a la pestaña **Redes** y revise la configuración.

    ![Imagen 14](../images/dp-300-module-01-lab-14.png)

1. Vaya a la pestaña **Redes** y revise la configuración.

    ![Imagen 15](../images/dp-300-module-01-lab-15.png)

    Compruebe que la opción **Habilitar auto_shutdown** esté desactivada.

1. Vaya a la pestaña **Opciones avanzadas** y revise la configuración.

    ![Imagen 16](../images/dp-300-module-01-lab-16.png)

1. Vaya a la pestaña **Configuración de SQL Server** y revise la configuración.

    ![Imagen 17](../images/dp-300-module-01-lab-17.png)

    **Nota:** también puede configurar el almacenamiento de la máquina virtual con SQL Server en esta pantalla. De forma predeterminada, las plantillas de máquina virtual de Azure con SQL Server crean un disco prémium con almacenamiento en caché de lectura para los datos, un disco prémium sin almacenamiento en caché para el registro de transacciones y usa el disco SSD local (D:\ en Windows) para tempdb.

1. Seleccione el botón **Revisar y crear**. Seleccione **Crear**.

    ![Imagen 18](../images/dp-300-module-01-lab-18.png)

1. En la hoja de información general de la implementación, espere hasta que esta finalice. La máquina virtual tarda aproximadamente 5-10 minutos en implementarse. Una vez finalizada la implementación, seleccione **Ir al recurso**.

    **Nota:** La implementación puede tardar varios minutos en completarse.

    ![Imagen 19](../images/dp-300-module-01-lab-19.png)

1. En la página **Información general** de la máquina virtual, explore las opciones de menú de este recurso para revisar lo que está disponible.

    ![Imagen 20](../images/dp-300-module-01-lab-20.png)

## Conexión a SQL Server en una máquina virtual de Azure

1. En la página **Información general** de la máquina virtual, seleccione el botón **Conectar** y elija RDP.

    ![Imagen 21](../images/dp-300-module-01-lab-21.png)

1. En la pestaña RDP, seleccione el botón **Descargar archivo RDP**.

    ![Imagen 22](../images/dp-300-module-01-lab-22.png)

    **Nota:** Si observa el error **Requisito previo del puerto no cumplido**. Asegúrese de seleccionar el vínculo para agregar una regla de grupo de seguridad de red de entrada con el puerto de destino mencionado en el campo *Número de puerto*.

    ![Imagen 22_1](../images/dp-300-module-01-lab-22_1.png)

1. Abra el archivo RDP que acaba de descargar. Cuando aparezca un cuadro de diálogo en el que se le pregunte si desea conectarse, seleccione **Conectar**.

    ![Imagen 23](../images/dp-300-module-01-lab-23.png)

1. Escriba el nombre de usuario y la contraseña seleccionados durante el proceso de aprovisionamiento de máquinas virtuales. A continuación, seleccione **Aceptar**.

    ![Imagen 24](../images/dp-300-module-01-lab-24.png)

1. Cuando aparezca el cuadro de diálogo** Conexión a Escritorio remoto** en el que se le pregunta si desea conectarse, seleccione **Sí**.

    ![Imagen 26](../images/dp-300-module-01-lab-26.png)

1. Seleccione el botón Inicio de Windows y escriba SSMS. Seleccione **Microsoft SQL Server Management Studio** en la lista.  

1. Cuando se abra SSMS, observe que el cuadro de diálogo **Conectar al servidor** se rellenará previamente con el nombre de instancia predeterminado. Seleccione **Conectar**.

    ![Imagen 35](../images/dp-300-module-01-lab-35.png)

Azure Portal proporciona herramientas eficaces para administrar un servidor SQL Server hospedado en una máquina virtual. Estas herramientas incluyen el control de la aplicación de revisiones automatizada, copias de seguridad automatizadas y una forma sencilla de configurar la alta disponibilidad.
