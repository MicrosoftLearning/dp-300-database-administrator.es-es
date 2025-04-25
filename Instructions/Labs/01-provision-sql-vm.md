---
lab:
  title: 'Laboratorio 1: Aprovisionamiento de SQL Server en una máquina virtual de Azure'
  module: Plan and Implement Data Platform Resources
---

# Aprovisionamiento de un servidor SQL Server en una máquina virtual de Azure

**Tiempo estimado**: 30 minutos

Los alumnos explorarán Azure Portal y lo usarán para crear una máquina virtual de Azure con SQL Server 2022 instalado. A continuación, se conectarán a la máquina virtual a través del Protocolo de escritorio remoto.

Es administrador de bases de datos para AdventureWorks. Debe crear un entorno de prueba para usarlo en una prueba de concepto. La prueba de concepto usará SQL Server en una máquina virtual de Azure y una copia de seguridad de la base de datos AdventureWorksDW. Debe configurar la máquina virtual, restaurar la base de datos y consultarla para asegurarse de que está disponible.

## Implementación de SQL Server en una máquina virtual de Azure

1. En la máquina virtual del laboratorio, inicie una sesión del explorador, vaya a [https://portal.azure.com](https://portal.azure.com/) e inicie sesión con la cuenta Microsoft asociada a la suscripción de Azure.

1. Localice la barra de búsqueda en la parte superior de la página. Busque **Azure SQL**. Seleccione el resultado de la búsqueda de **Azure SQL** que aparece en los resultados, debajo de **Servicios**.

1. En la hoja **Azure SQL**, seleccione **Crear**.

1. En la hoja **Seleccione una opción de implementación de SQL**, abra el cuadro desplegable que aparece en **Máquinas virtuales de SQL**. Selecciona la opción etiquetada **Licencia gratuita de SQL Server: SQL 2022 Developer en Windows Server 2022**. Seleccione **Crear**.

1. En la página **Crear una máquina virtual**, escribe la siguiente información y *deja todas las demás opciones como valores predeterminados*:

    - **Suscripción**: &lt;Su suscripción&gt;
    - **Grupo de recursos:** &lt;su grupo de recursos&gt;.
    - **Nombre de la máquina virtual:** AzureSQLserverVM
    - **Región:**&lt; escoge la región local, igual que la región seleccionada, para el grupo de recursos.&gt;
    - **Opciones de disponibilidad:** No se necesita redundancia de la infraestructura
    - **Imagen:** Licencia gratuita de SQL Server: SQL Server 2022 Developer en Windows Server 2022 - Gen2
    - **Ejecución con descuento puntual de Azure:** No (sin marcar)
    - **Tamaño:** estándar *D2s_v5* (2 vCPU, 8 GiB de memoria). *Es posible que tengas que seleccionar el vínculo "Ver todos los tamaños" para ver esta opción).*
    - **Nombre de usuario de la cuenta de administrador:**&lt;elige un nombre para la cuenta de administrador.&gt;
    - **Contraseña de la cuenta de administrador:**&lt;elige una contraseña segura.&gt;
    - **Seleccionar puertos de entrada:** RDP (3389)
    - **¿Quiere usar una licencia de Windows Server existente?:** No (desactivado)

    > &#128221; Anota el nombre de usuario y la contraseña para más adelante.

1. Vaya a la pestaña **Discos** y revise la configuración.

1. Vaya a la pestaña **Redes** y revise la configuración.

1. Vaya a la pestaña **Redes** y revise la configuración.

    Compruebe que la opción **Habilitar auto_shutdown** esté desactivada.

1. Vaya a la pestaña **Opciones avanzadas** y revise la configuración.

1. Vaya a la pestaña **Configuración de SQL Server** y revise la configuración.

    > &#128221; Observa también que puedes configurar el almacenamiento de la máquina virtual con SQL Server en esta pantalla. De forma predeterminada, las plantillas de máquina virtual de Azure con SQL Server crean un disco prémium con almacenamiento en caché de lectura para los datos, un disco prémium sin almacenamiento en caché para el registro de transacciones y usa el disco SSD local (D:\ en Windows) para tempdb.

1. Seleccione el botón **Revisar y crear**. Seleccione **Crear**.

1. En la hoja de información general de la implementación, espere hasta que esta finalice. La máquina virtual tarda aproximadamente 5-10 minutos en implementarse. Una vez finalizada la implementación, seleccione **Ir al recurso**.

    > &#128221; Ten en cuenta que la implementación puede tardar varios minutos en completarse.

1. En la página **Información general** de la máquina virtual, explore las opciones de menú de este recurso para revisar lo que está disponible.

---

## Conexión a SQL Server en una máquina virtual de Azure

1. En la página **Información general** de la máquina virtual, selecciona el menú desplegable **Conectar** y luego selecciona **Conectar**.

1. En el panel Conectar, selecciona el botón **Descargar archivo RDP**.

    > &#128221; Si observas el error **Requisito previo de puerto no cumplido**. Asegúrese de seleccionar el vínculo para agregar una regla de grupo de seguridad de red de entrada con el puerto de destino mencionado en el campo *Número de puerto*.

1. Abra el archivo RDP que acaba de descargar. Cuando aparezca un cuadro de diálogo en el que se le pregunte si desea conectarse, seleccione **Conectar**.

1. Escriba el nombre de usuario y la contraseña seleccionados durante el proceso de aprovisionamiento de máquinas virtuales. Después, selecciona **Aceptar**.

1. Cuando aparezca el cuadro de diálogo** Conexión a Escritorio remoto** en el que se le pregunta si desea conectarse, seleccione **Sí**.

1. Selecciona la barra de búsqueda junto al botón Inicio de Windows y escribe SSMS. Seleccione **Microsoft SQL Server Management Studio** en la lista.  

1. Cuando se abra SSMS, observe que el cuadro de diálogo **Conectar al servidor** se rellenará previamente con el nombre de instancia predeterminado. Activa la opción **Confiar en el certificado de servidor** y después selecciona **Conectar**.

1. Para cerrar SSMS, selecciona la **X** en la esquina superior derecha.

1. Ahora ya puedes desconectarte de la máquina virtual para cerrar la sesión de RDP.

Azure Portal proporciona herramientas eficaces para administrar un servidor SQL Server hospedado en una máquina virtual. Estas herramientas incluyen el control de la aplicación de revisiones automatizada, copias de seguridad automatizadas y una forma sencilla de configurar la alta disponibilidad.

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

1. Selecciona todos los recursos con el prefijo del nombre de la máquina virtual que especificaste previamente en el laboratorio.

1. Seleccione **Eliminar** en el menú superior.

1. En el cuadro de diálogo **Eliminar recursos**, escribe **delete** y selecciona **Eliminar**.

1. Para confirmar la eliminación, escribe el nombre del recurso y selecciona **Eliminar**.

1. Espera a que se eliminen los recursos.

1. Cierra Azure Portal.

---

Completaste correctamente este laboratorio.
