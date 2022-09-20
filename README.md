# Lab2_Robotica_de_Desarrollo
## Instalación de linux

La instalación del sistema operativo Ubuntu 20.04 LTS se hizo en un equipo Acer Aspire e5 y se presentaron varias dificultades. A continuación se indica cómo podrían resolverse en futuras ocasiones.

La BIOS de los Acer Aspire puede ser un poco diferente. Antes de bootear es necesario deshabilitar el modo seguro y habilitar la opción de selección de booteable que corresponde a F12; para hacer ambos cambios es necesario que la bios tenga usuario administrador, lo que se hace asignando una contraseña.

Una vez se hagan los cambios mencionados, se podrá seleccionar la usb Booteable como medio de arranque y se podrá proceder con la instalación del sistema operativo. Anteriormente ya habíamos hecho la partición correspondiente al disco, sin embargo esta opción tambien está disponible en el instalador.

Posterior a la instalación no aparecía la opción de iniciar con Ubuntu. Primero se intentó reinstalar el sistema operativo, sin embargo esto no arregló los problemas. Se tenía un problema con el GRUB, así que nuevamente se usa el booteable. Se usa la función de probar el sistema operativo y desde la terminal se instala el reparador de GRUB.

## MATLAB
### Envío de mensajes

- Se da por sentado que se tiene instalado matlab con el toolbox de robotic systems. Primero se lanza una instancia de matlab, haciendo uso de la terminal e ingresando el siguiente comando: 
```bash
matlab 
```
Es importante que no se cierre esta terminal aun después de ejecutar el comando, dado que esto finalizará la sesión de matlab de manera automática y se perderá todo el trabajo.

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191157662-c175da9a-49a3-43f8-bf3e-12141838bfc7.png" width="350" />
</p>

- Después se usaron otras dos terminales: una para iniciar el nodo maestro con el comando
```bash
roscore
```
y otra para iniciar turtlesim con el comando 
```bash 
rosrun turtlesim turtlesim_node
```

- En ese momento se tienen tres terminales y dos ventanas abiertas. Una de nuestras terminales tiene dos pestañas, por eso no se tienen tres ventanas pero si tres terminales.

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191157945-f7aa56fe-4db7-4bf6-b1fc-6eefe9bd077a.png" width="350" />
</p>

- Ahora vamos iniciar con los comandos en matlab, para ello se crea un script o un live script segun se requiera y se ejecuta el siguiente código:

```bash
rosinit;                                                         %Conexion con nodo maestro
velPub = rospublisher(’/turtle1/cmd_vel’,’geometry_msgs/Twist’); %Creacion publicador
velMsg = rosmessage(velPub);                                     %Creacion de mensaje
velMsg.Linear.X = 1;                                             %Valor del mensaje
send(velPub,velMsg);                                             %Envio
pause(1)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191158625-f3679ff8-56c5-42b0-a7ad-c5d89cd98d92.png" width="350" />
</p>

Al ejecutar este script la tortuga debería avanzar a la derecha una unidad. Se obtiene el siguiente resultado:

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191158697-790a6268-ee2e-42cc-9096-d1cb1cef916f.png" width="350" />
</p>

### Captura de información

La captura de información se hace con una suscripción al tópico pose, esto se logra con la función rossubscriber con el siguiente procedimiento:
- Se usa el mismo script en el que se ejecutó el código de la anterior sesión, sin embargo si se usa un nuevo script primero se debe iniciar la conexión con el nodo maestro usando 
```bash
rosinit
```
- Ahora se usa el método rossubscriber, el cual posee dos parametros: el primero corresponde al nodo al que se suscribe, que en este caso es “/turtle1/pose”, y el segundo corresponde al tipo de mensaje que se recibirá “turtlesim/Pose”.
```bash
poseSub = rossubscriber('/turtle1/pose','turtlesim/Pose');
```
- Por último, el atributo LatestMessage del suscriptor nos permite ver la información de interés:
```bash
x = poseSub.LatestMessage.X
y = poseSub.LatestMessage.Y
theta = poseSub.LatestMessage.Theta
vel = poseSub.LatestMessage.LinearVelocity
omega = poseSub.LatestMessage.AngularVelocity
```
- Otra opción es usar la función receive, la que recibe como parametros el susbcriptor y un tiempo de timeout:
```bash
pose = receive(poseSub,10)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191158757-642e0f46-b873-45d8-ade2-21ed3a577179.png" width="350" />
</p>

### Edición de la pose

Para la edición de los valores de la pose se usó rossvcclient. Esta función recibe el nombre del servicio que se utilizá, en este caso "/turtle1/teleport_absolute". Se crea un servicio para el cambio en la pose:
```bash
poseServ = rossvcclient("/turtle1/teleport_absolute");
```
Ahora se crea el mensaje que contiene la información que queremos editar:
```bash
pose = rosmessage(poseServ);
pose.X = 4;
pose.Y = 4;
pose.Theta = 3.14/2;
```
Ahora para enviar el mensaje y usar el servicio se usa call, la cual recibe el servicio, el mensaje y un time-out:
```bash
if isServerAvailable(poseServ)
    call(poseServ,pose,"Timeout",3)
else
    error("Service server not available on network")
end
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191158806-1e9f5227-9211-442d-90fd-79ec7f3c9024.png" width="350" />
</p>

### Finalización de conexión
Al hacer la busqueda, o simplemente correr rosinit nuevamente se indica que el comando para finalizar la conexión al nodo maestro es 
```bash
rosshtdown
```

## Python
### Instalación librerias de phyton
Instalar el instalador de paquetes:
```bash
sudo apt install python3-pip
```
Instalación de la librería pynput:
```bash
pip install pynput
```
### Edición archivo make
En la carpeta hello_turtle se debe editar el archivo CMakeLists.txt:

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191162817-ffd634de-448d-4d76-9112-16b5324c8120.png" width="350" />
</p>

En el apartado de catkin_install_python incluya el scrip creado myTeleopKey.py:

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191163934-840a752d-0131-43dd-9bb2-43505166fe0c.png" width="350" />
</p>

### Ejecución
Para la ejecución del programa en python se necesitan 3 consolas abiertas al mismo tiempo. En las dos primeras se ejecutarán los mismos comandos para iniciar el nodo maestro y turtlesim en el caso de matlab. En la tercer consola primero se navega hasta el workspace, puede hacerlo con click derecho desde el explorador con la opcion open in terminal:

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191163992-1d30cb54-ec0f-420c-989b-79c21cdeb232.png" width="350" />
</p>

Una vez ahí, si se tiene catkin tools se puede usar el comando:
```bash
catkin_make
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191164077-f0ebe9c5-eb9b-40c1-825d-bd17652706df.png" width="350" />
</p>

Si no se tiene el paquete, se usa el comando:
```bash
catkin build hello_turtle
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191164127-c8431211-1c21-41a9-b91c-2cb2405e3f98.png" width="350" />
</p>

> No se pueden utilizar ambos métodos de construcción al mismo tiempo. En caso de que requiera cambiarlo, elimine todas las carpetas que no sean src y construya con el comando que desee.

Una vez construido ingrese el siguiente comando:
```bash
source devel/setup.bash
```

Por último se lanza el script mediante el comando:
```bash
rosrun hello_turtle myTeleopKey.py
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191164420-36030a8a-4eb7-4ffa-a361-529d8e061553.png" width="350" />
</p>

### Controles
Se usaron las siguientes teclas para mover la tortuga desde el terminal:
- w: avanzar
- a: girar a la izquierda
- s: retroceder
- d: girar a la izquierda
- Espacio: girar la tortuga
- r: reiniciar la posición

### Resultado

<p align="center">
  <img src="https://user-images.githubusercontent.com/51519737/191164481-793b0d63-7d9a-492a-8521-4b862f08cbba.png" width="350" />
</p>

[YouTube video](https://youtu.be/qHCmZQftQYE)

https://user-images.githubusercontent.com/51519737/191165705-036b77ae-b0ea-4170-8f99-53c3723ec32c.mp4

## Integrantes
- Juan Andres Bueno Hortua
- Oscar Javier Manrique Merchan

## Profesores
- Ricardo Emiro Ramirez Heredia
- Jhoan Sebastian Rodriguez Rodriguez
