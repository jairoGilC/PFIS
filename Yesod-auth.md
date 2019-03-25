# Yesod Auth

La siguiente guía está basada en el siguiente [tutorial](https://pbrisbin.com/posts/writing_json_apis_with_yesod/) y su lectura es necesaria de manera previa para trabajar la siguiente guía.  

Por defecto el scaffolding de Yesod provee dos mecanismos de autenticación OpenId y un Dummy Login que funciona como ejemplo ilustrativo de registro. En esta guía veremos cómo eliminar ambos y remplazarlo por Google OAuth2. Para ello y como paso previo se debe crear una cuenta de desarrollador y obtener las correspondientes credenciales. Para ello pueden ingresas en el siguiente [enlace](https://developers.google.com/identity/protocols/OAuth2) con una guía de cómo hacerlo. 

Al momento de crear las credenciales es importante tener en cuenta:

1) En la pestaña credenciales *credencials* Seleccionamos el Id que vamos a usar de la lista *OAuth 2.0 client IDs* seleccionamos editar y en la opción *Authorized redirect URIs* añadimos la ruta de nuestro proyecto `http://localhost:3000/auth/page/google/callback`
2) En la pestaña *dashboard* en la parte superior seleccionamos *Enable Google+ API*

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
