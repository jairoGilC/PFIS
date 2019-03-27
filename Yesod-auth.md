# Yesod Auth

La siguiente guía está basada en el siguiente [tutorial](https://pbrisbin.com/posts/writing_json_apis_with_yesod/) y su lectura es necesaria de manera previa para trabajar la siguiente guía.  

Por defecto el scaffolding de Yesod provee dos mecanismos de autenticación OpenId y un Dummy Login que funciona como ejemplo ilustrativo de registro. En esta guía veremos cómo eliminar ambos y remplazarlo por Google OAuth2. Para ello y como paso previo se debe crear una cuenta de desarrollador y obtener las correspondientes credenciales. Para ello pueden ingresas en el siguiente [enlace](https://developers.google.com/identity/protocols/OAuth2) con una guía de cómo hacerlo. 

Al momento de crear las credenciales es importante tener en cuenta:

1) En la pestaña credenciales *credencials* Seleccionamos el Id que vamos a usar de la lista *OAuth 2.0 client IDs* seleccionamos editar y en la opción *Authorized redirect URIs* añadimos la ruta de nuestro proyecto `http://localhost:3000/auth/page/google/callback`
2) En la pestaña *dashboard* en la parte superior seleccionamos *Enable Google+ API*

## Autenticación

### Importación de librerias

en `stack.yaml`

    extra-deps: [yesod-auth-oauth2-0.6.1.0,hoauth2-1.8.4,uri-bytestring-aeson-0.1.0.7]

en `package.yaml`

    - yesod-auth-oauth2

en `Foundation.hs`

    import Yesod.Auth.OAuth2.Google

### configuración 

en `Foundation.hs` añadir las credenciales de Goodle Auth, ademas ajustar la instancia de `YesodAuth` como se muestra a continucación

    -- Replace with Google client ID.
    clientId :: Text
    clientId = "111111111111-dskdsn12shgm1dcjtqtqftiv26ea1m4.apps.googleusercontent.com"

    -- Replace with Google secret ID.
    clientSecret :: Text
    clientSecret = "3k4njkj23n2k3n23n223n2k3"  
---
    instance YesodAuth App where
        type AuthId App = UserId
        
      ...
      
      authPlugins :: App -> [AuthPlugin App]
      authPlugins app = [oauth2GoogleScoped ["email", "profile"] clientId clientSecret]
      
Al cargar el proyeto si ingresamos en Login comprobamos la unica opción de login disponible es `Login via google` si realizamos el proceso de login podemos ver ademas que se crea un nuevo usario asociado a la cuenta de google usada para autenticarse.


## Autorización

La autorización eta determinada por los permisos que debe tener un usuario para poder realizar una acción especifica. Para lograrlo vamos a definir los permisos disponibles dentro de nuestra aplicación. Para ello ingresamos en `src/Models.hs` y definimos la clase que usaremos para definir nuestros permisos 


    data Privileges =
      PrvDemoOne         -- ^ what can be demo one...
      | PrvDemoTwo       -- ^ what can be demo two...
      deriving (Show,Read,Eq)

    derivePersistField "Privileges"
    
A continuación debemos ajustar la entidad de usuario a fin de poder definir los privilegios que un usuario tiene en la aplicación.


    User
        ident Text
        password Text Maybe
        UniqueUser ident
        perms [Privileges]
        deriving Typeable
        
Adicionalmente será necesario ajustar el archivo `Foundation.hs` para añadir el nuevo atributo al momento de crear el usuario 

    --Esta funcion ya existe en Foundation.hs simplemente hace falta añadir el nuevo atributo dentro la sintaxis de registro
    authenticate creds = liftHandler $ runDB $ do
        x <- getBy $ UniqueUser $ credsIdent creds
        case x of
            Just (Entity uid _) -> return $ Authenticated uid
            Nothing -> Authenticated <$> insert User
                { userIdent = credsIdent creds
                , userPassword = Nothing
                , userPerms = [] --New required line
                }
                
*Nota:* En caso de ya tener usuarios en la aplicación la migración para crear el nuevo campo puede generar errores, lo más practico es borrar todos los registros de la tabla `user` manualmente 

Finalmente, al igual que la función `isAuthenticated` debemos definir una función que nos ayude a validar si el usuario tiene o no los permisos suficientes para acceder a la ruta especificada. Para ello definimos la función `authorizedForPrivileges`


    authorizedForPrivileges :: [Privileges] -> Handler AuthResult
    authorizedForPrivileges perms = do
        mu <- maybeAuth
        return $ case mu of
         Nothing -> Unauthorized "You must login to access this page"
         Just u@(Entity userId user) ->
           if hasPrivileges u perms
                then Authorized
                else Unauthorized "Not enought priviledges"

Modificamos la autenticación de cualquiera de las rutas de nuestro aplicativo pasando como parametro la lista de privilegios requeridos para acceder:

        isAuthorized (DemoJsonR _) _ = authorizedForPrivileges [PrvDemoOne]


