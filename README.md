El objetivo de {auth0}es implementar un esquema de autenticación para Shiny usando OAuth Apps a través del servicio freemium Auth0 .

Instalación
Puede instalar {auth0}desde CRAN con:

install.packages ( " auth0 " )
También puede instalar la versión de desarrollo de github con:

# Install.packages ( "DevTools") 
telecontroles :: install_github ( " curso-r / auth0 " )
Tutorial
Para crear su aplicación brillante autenticada, debe seguir los cinco pasos a continuación.

Paso 1: crea una cuenta Auth0
Ir a auth0.com
Haga clic en "Registrarse"
Puede crear una cuenta con una combinación de nombre de usuario y contraseña, o registrándose con sus cuentas de GitHub o Google.
Paso 2: crea una aplicación Auth0
Después de iniciar sesión en Auth0, verá una página como esta:



Haga clic en "+ Crear aplicación"
Dale un nombre a tu aplicación
Seleccione "Aplicaciones web normales" y haga clic en "Crear"
Paso 3: configura tu aplicación
Vaya a Configuración en su aplicación seleccionada. Debería ver una página como esta:


Agregue http://localhost:8080a las "URL de devolución de llamada permitidas", "Orígenes web permitidos" y "URL de cierre de sesión permitidas".
Puede cambiar http://localhost:8080a otro puerto.
Agregue el servidor remoto donde va a implementar su aplicación brillante en los mismos cuadros.
Solo asegúrese de que estas direcciones sean correctas. Si está colocando su aplicación dentro de una carpeta (por ejemplo, https://johndoe.shinyapps.io/fooBar ), no incluya la carpeta ( fooBar) en "Orígenes web permitidos".
Clic en Guardar"
¡Ahora vayamos a R!

Paso 4: crea tu aplicación brillante y llena el _auth0.ymlarchivo
Cree un archivo de configuración para su aplicación brillante llamando a auth0::use_auth0():
auth0 :: use_auth0 ()
Puede establecer el directorio donde se creará este archivo utilizando el path=parámetro. Consulte ?auth0::use_auth0para obtener más detalles.
Su _auth0.ymlarchivo debería ser así:
Nombre : myApp 
remote_url : ' '
 auth0_config :
   api_url : ! expr paste0 ( 'https: //', Sys.getenv ( "AUTH0_USER"), '.auth0.com') 
  credenciales :
     clave : ! expr Sys.getenv ( "AUTH0_KEY ") 
    secreto : ! expr Sys.getenv (" AUTH0_SECRET ")
Ejecute usethis::edit_r_environ()y agregue estas tres variables de entorno:
AUTH0_USER=johndoe
AUTH0_KEY=5wugt0W...
AUTH0_SECRET=rcaJ0p8...
Así es como identifica a cada uno de ellos (vea la imagen a continuación):

AUTH0_USER es su nombre de usuario, que se encuentra en la esquina superior del sitio.
AUTH0_KEY es su ID de cliente, que se puede copiar desde dentro de la página de la aplicación.
AUTH0_SECRET es su secreto de cliente, que se puede copiar desde la página de la aplicación.


Más sobre las variables de entorno aquí . También puede completar esta información directamente en el _auth0.ymlarchivo (ver más abajo). Si lo hace, no olvide guardar el _auth0.ymlarchivo después de editarlo.

Guarde y reinicie su sesión .
Escriba una aplicación brillante simple en un app.Rarchivo, como este:
biblioteca ( brillante )

ui  <- fluidPage (
  fluidRow (plotOutput ( " plot " ))
)
  
servidor  <-  función ( entrada , salida , sesión ) {
   salida $ plot  <- renderPlot ({
    plot ( 1 : 10 )
  })
}

# ¡ Tenga en cuenta que aquí estamos usando una versión diferente de shinyApp! 
auth0 :: shinyAppAuth0 ( ui , servidor )
Nota : Si desea utilizar una ruta diferente al auth0archivo de configuración, puede pasarla shinyAppAuth0()o configurar la auth0_config_fileopción ejecutando options(auth0_config_file = "path/to/file").

Paso 5: ¡Corre!
Puedes probar tu aplicación ejecutándose

opciones ( shiny.port  =  8080 )
 shiny :: runApp ( " aplicación / directorio / " )
Si todo está bien, debe ser reenviado a una página de inicio de sesión y, después de iniciar sesión o registrarse, será redirigido a su aplicación.

Si está ejecutando su aplicación en un servidor remoto como shinyapps.io o su propio servidor, y si su aplicación está en una subcarpeta del host (como https://johndoe.shinyapps.io/fooBar ), debe incluir su control remoto URL en el remote_urlparámetro del _auth0.ymlarchivo.

También puede forzar el {auth0}uso de la configuración de URL local options(auth0_local = TRUE). Esto puede resultar útil si está ejecutando una aplicación dentro de un contenedor Docker.

Variables de entorno y múltiples aplicaciones Auth0
Si está usando {auth0}solo una aplicación brillante o está ejecutando muchas aplicaciones para la misma base de datos de usuario, el flujo de trabajo recomendado es usar las variables de entorno AUTH0_KEYy AUTH0_SECRET.

Sin embargo, si está ejecutando muchas aplicaciones brillantes y desea utilizar diferentes configuraciones de inicio de sesión, debe crear muchas aplicaciones Auth0. Por lo tanto, tendrá muchas ID de cliente y secretos de cliente para usar. En este caso, las variables de entorno global serán improductivas porque deberá cambiarlas cada vez que cambie la aplicación que está desarrollando.

Hay dos opciones en este caso:

(Recomendado) Agregue variables de entorno dentro del repositorio de su aplicación, usando usethis::edit_r_environ("project").
(No recomendado) Agregue el ID de cliente y el secreto directamente en el archivo _auth0.yml:
La mejor opción en este caso es simplemente agregar el ID de cliente y el secreto directamente en el _auth0.ymlarchivo:

name : myApp 
remote_url : ' '
 auth0_config :
   api_url : https: // <USERNAME> .auth0.com 
  credenciales :
     clave : <CLIENT_ID> 
    secreto : <CLIENT_SECRET>
Ejemplo:

name : myApp 
remote_url : ' '
 auth0_config :
   api_url : https://johndoe.auth0.com 
  credenciales :
     clave : cetQp0e7bdTNGrkrHpuF8gObMVl8vu 
    secreto : C6GHFa22mfliojqPyKP_5K0ml4TituWr6-If
Aunque es posible, la última opción es menos segura y, por lo tanto, no se recomienda porque es fácil olvidar las contraseñas allí y enviarlas a repositorios públicos, por ejemplo.

ui.R/server.R
Para que {auth0}funcione con un marco ui.R/ server.R, deberá ajustar su uiobjeto / función con auth0_ui()y su serverfunción con auth0_server(). Aquí hay un pequeño ejemplo de trabajo:

ui.R
biblioteca ( brillante )
biblioteca ( auth0 )

auth0_ui (fluidPage (logoutButton ()))
servidor.R
biblioteca ( auth0 )

auth0_server ( función ( entrada , salida , sesión ) {})
{auth0}intentará encontrar el _auth0.ymlutilizando la misma estrategia que el app.Rmarco: primero desde options(auth0_config_file = "path/to/file")y luego arreglando "./_auth0.yml". Ambos auth0_ui()y auth0_server()tienen un info=parámetro donde puede pasar la ruta del _auth0.ymlarchivo o el objeto devuelto por la auth0_info()función.

Parámetro de audiencia
Para autorizar a un cliente a realizar llamadas a la API contra un servidor remoto, la solicitud de autorización debe incluir un audienceparámetro ( documentación de Auth0 ).

Para hacer esto {auth0}, agregue un audienceparámetro a la auth0_config sección de su _auth0.ymlarchivo. Por ejemplo:

Nombre : myApp 
remote_url : ' '
 auth0_config :
   api_url : ! expr paste0 ( 'https: //', Sys.getenv ( "AUTH0_USER"), '.auth0.com') 
  de la audiencia : https://example.com/api 
  credenciales :
     clave : ! expr Sys.getenv ( "AUTH0_KEY") 
    secreto : ! expr Sys.getenv ( "AUTH0_SECRET")
Cuando audiencese incluye un parámetro en la solicitud, el token de acceso devuelto por Auth0 será un token de acceso JWT en lugar de un token de acceso opaco. El cliente debe incluir el token de acceso con las solicitudes de API para autenticar las solicitudes.

Limitaciones de RStudio
Debido a que RStudio está especializado en aplicaciones brillantes estándar, algunas funciones no funcionan como se esperaba cuando se usan {auth0}. Los principales problemas son que debe ejecutar la aplicación en un navegador real, como Chrome o Firefox. Si usa RStudio Viewer o ejecuta la aplicación en una ventana de RStudio, la aplicación mostrará una página en blanco y no funcionará.

Si está utilizando una versión inferior a 1.2 en RStudio, es posible que el botón "Ejecutar aplicación" no aparezca en la esquina derecha del script app.R. Eso es porque RStudio busca el término "shinyApp (" en el código para identificar una aplicación brillante.

Marcadores
Desde v0.2.0, auth0admite marcadores de estado de shiny, pero debido a problemas de análisis de URL, los marcadores solo funcionan con el almacenamiento del servidor. Para activar esta función, debe llamar a la aplicación con las siguientes líneas en su app.Rarchivo:

enableBookmarking ( tienda  =  " servidor " )
shinyAppAuth0 ( interfaz de usuario , servidor )
También tenga en cuenta que Auth0 agrega codey statea los parámetros de consulta de URL.

Esta solución funciona normalmente en ui.R/ server.Rframework.

Gestionar usuarios
Puede administrar el acceso de los usuarios desde el panel Usuarios en Auth0. Para crear un usuario, haga clic en "+ Crear usuarios".

También puede utilizar muchos proveedores de OAuth diferentes como Google, Facebook, Github, etc. Para configurarlos, vaya a la pestaña Conexiones .

En un futuro cercano, nuestro plan es implementar la API de Auth0 en R para que pueda administrar su aplicación usando R.

Información registrada
Después de que un usuario inicia sesión, es posible acceder a la información del usuario actual utilizando el session$userData$auth0_infoobjeto reactivo. Se puede acceder al token Auth0 usando session$userData$auth0_credentials. He aquí un pequeño ejemplo:

biblioteca ( brillante )
biblioteca ( auth0 )

# UI simple con información de usuario 
ui  <- fluidPage (
  verbatimTextOutput ( " user_info " )
  verbatimTextOutput ( " credential_info " )
)

servidor  <-  función ( entrada , salida , sesión ) {

  # imprimir información de usuario 
  salida $ user_info  <- renderPrint ({
     sesión $ userData $ auth0_info
  })
  
  salida $ credential_info  <- renderPrint ({
     sesión $ userData $ auth0_credentials
  })

}

shinyAppAuth0 ( interfaz de usuario , servidor )
Debería ver los objetos que contienen la información de usuario y credencial.

Información de usuario

$sub
[1] "auth0|5c06a3aa119c392e85234f"

$nickname
[1] "jtrecenti"

$name
[1] "jtrecenti@email.com"

$picture
[1] "https://s.gravatar.com/avatar/1f344274fc21315479d2f2147b9d8614?s=480&r=pg&d=https%3A%2F%2Fcdn.auth0.com%2Favatars%2Fjt.png"

$updated_at
[1] "2019-02-13T10:33:06.141Z"
Tenga en cuenta que el subcampo es único y se puede utilizar para muchos propósitos, como crear aplicaciones personalizadas para diferentes usuarios.

Información de la credencial (abreviada)

$access_token
[1] "y5Yv..."

$id_token
[1] "eyJ0..."

$scope
[1] "openid profile"

$expires_in
[1] 86400

$token_type
[1] "Bearer"
Se id_tokenpuede usar con aplicaciones que requieren un Authorization encabezado con cada solicitud web.

Información registrada y ui.R / server.R
Si está ejecutando {auth0}utilizando ui.R/server.Rframework y desea acceder a la información registrada, deberá usar la misma auth0_info()función de objeto devuelto en ambos auth0_ui()y auth0_server().

Esto es posible usando el global.Rarchivo. Por ejemplo:

global.R
a0_info  <-  auth0 :: auth0_info ()
ui.R
biblioteca ( brillante )
biblioteca ( auth0 )

auth0_ui (fluidPage (), info  =  a0_info )
servidor.R
biblioteca ( auth0 )

auth0_server ( función ( entrada , salida , sesión ) {

  observar({ 
    imprimir ( sesión $ userData $ auth0_info )
  })
  
}, info  =  a0_info )
Cerrar sesión
Puede agregar un botón de cierre de sesión a su aplicación usando logoutButton().

biblioteca ( brillante )
biblioteca ( auth0 )

# UI simple con botón de cierre de sesión 
ui  <- fluidPage (logoutButton ())
 server  <-  function ( input , output , session ) {}
shinyAppAuth0 ( interfaz de usuario , servidor )
Costos
Auth0 es un servicio freemium. La cuenta gratuita le permite tener hasta 7000 conexiones en un mes y dos tipos de conexiones sociales. Puedes consultar todos los planos aquí .

Descargo de responsabilidad
Este paquete no es proporcionado ni respaldado por Auth0 Inc. Úselo bajo su propio riesgo.

Además, NO soy un experto en seguridad y, como señaló Bob Rudis , agregar la palabra "seguro" a algo tiene amplias implicaciones de eficacia e integridad. Entonces, este paquete puede estar mintiendo cuando dice que es seguro.

Si es un experto en seguridad y le gustó la idea de este paquete, considere probarlo. Estaremos muy, muy agradecidos por cualquier ayuda.

Mapa vial
{auth0} 0.2.0
[heavy_check_mark] Elimina la necesidad de direcciones URL locales y remotas en el config_file.
[heavy_check_mark] Resuelva el problema de los marcadores y los parámetros de URL (Problema # 22).
[heavy_check_mark] shinyAppDirAuth0()función para funcionar como shiny::shinyAppDir()(Problema # 21).
[heavy_check_mark] Soporte para ui.R/ server.Rapps.
{auth0} 0.3.0
Implemente {auth0}funciones de API para administrar usuarios y opciones de inicio de sesión a través de R.
[heavy_check_mark] Pegatina hexagonal.
