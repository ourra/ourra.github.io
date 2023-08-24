Cómo ejecutar OpenMSX desde el editor Kate
==========================================

[Kate](https://kate-editor.org) es un editor de texto bastante ligero que viene con el entorno de escritorio [KDE](https://kde.org) de Linux, y que está disponible tanto para esta plataforma como para Windows y Mac.

Se puede utilizar para desarrollar proyectos de software, ya que cuenta con las funcionaliades apropiadas para ello como son el resaltado de sintaxis, explorador de objetos del código, posibilidad de lanzar compiladores, o la integración con Git. En este sentido no se diferencia mucho de otros editores como Sublime, o VS Code.

En mi próximo proyecto en MSX BASIC voy a utilizarlo para escribir en él el código y ejecutarlo directamente en el emulador [OpenMSX](https://openmsx.org).

Para ello, es necesario configurar algunas cosas que se explican a continuación. Los nombres y menús corresponden a la versión en "español de España":
1. Lo primero de todo será crear un _shell script_ y guardarlo con un nombre (por ejemplo `lanzarmsx.sh`), que será utilizado por Kate para ejecutar el emulador. El contenido es este:
   ```bash
   #!/bin/bash
   
   [ -z "$1" ] && echo "Error: Falta parametro" && exit

   fich="$1"

   midir=$(dirname $fich)

   cp $fich AUTOEXEC.BAS
   
   # Convertir retornos de carro a formato de DOS
   todos AUTOEXEC.BAS

   # Fichero TCL para configurar OpenMSX
   echo "
     message \"Ejecutado desde $PWD\"
     set throttle off     
     after realtime 0.25 \"set throttle on\"
   " > start.tcl

   # Ejecutar OpenMSX
   openmsx -diska $midir -script start.tcl

2. A grandes rasgos, el script lee como parámetro un nombre de fichero que le pasa Kate (por ejemplo un programa en BASIC), lo copia con el nombre de AUTOEXEC.BAS y lo convierte a formato DOS con la utilidad `todos` (del paquete [tofrodos](https://repology.org/project/tofrodos), que deberá estar instalado en el sistema Linux). A continuación crea un fichero TCL para configurar OpenMSX de forma que se ejecute a la máxima velocidad durante 0.25 segundos y luego vuelva a la velocidad normal (es posible que tengas que ajustar manualmente este valor según tus preferencias). Finalmente ejecuta el OpenMSX indicando que monte como disco el directorio donde está nuestro programa y que se configure según el fichero TCL.
   
3. Ahora configuraremos Kate para que utilice el script anterior al pulsar un botón. Para ello ir al menú _Preferencias_ -> _Configurar Kate_ y luego en el panel lateral a _Herramientas externas_
4. Presionar el botón _Añadir_ y seleccionar _Añadir herramienta..._
5. Se abrirá una ventana con varios campos para rellenar. Configurar los siguientes:
  * **Nombre**: _Lanzar OpenMSX_    (o lo que queramos; es lo que se mostrará en un botón de la barra de herramientas)
  * **Ejecutable**: Seleccionar el script que hemos creado en el paso 1
  * **Argumentos**: Escribir lo siguiente: `%{Document:FilePath}`
  * **Guardar**: Elegir _Documento actual_
  * El resto de opciones pueden dejarse en blanco o como están por defecto.
6. Pulsar el botón de _Aceptar_ para cerrar las ventanas.
7. Para mostrar el botón en la barra de herramientas, ir de nuevo al menú _Preferencias_ -> _Configurar las barras de herramientas_
8. Seleccionar _Barra de herramientas principal <externaltools>_, seleccionar la que hemos creado en el paso 5 (_Lanzar OpenMSX) y pasarla de _Acciones disponibles_ a _Acciones actuales_
9. Pulsar el botón de _Aceptar_ para cerrar las ventanas.

Si todo ha ido bien, a partir de ahora cada vez que presionemos el botón de _Lanzar OpenMSX_, el fichero en BASIC que estemos editando se ejecutará inmediatamente en el emulador y podremos ver si funciona correctamente o si tiene algún error como el temido _Syntax error_. Cuando lo hayamos terminado de comprobarlo, se puede cerrar la ventana del OpenMSX y seguir trabajando en el fichero que teníamos abierto en Kate.
