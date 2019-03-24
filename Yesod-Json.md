# JSON services in Yesod

La siguiente guía está basada en el siguiente [tutorial](https://pbrisbin.com/posts/writing_json_apis_with_yesod/) y el capitulo  [JSON web services](https://www.yesodweb.com/book/json-web-service) del Yesod Book. Ambas lecturas son necesarias de manera previa para trabajar la siguiente guía.  

## Servicios JSON

Aunque Yesod es un framework fullStack y la respuesta de los servicios está pensada para ser de tipo HTTP. Es posible responder un JSON con el fin de por ejemplo desarrollar una arquitectura front y back separada.  

### FromJSON y ToJSON 

El primer paso a la hora de recibir o responder servicios JSON es especificar la manera como transformamos una entidad (por ejemplo, una entidad de nuestro modelo de datos) a un objeto de tipo JSON y viceversa. Para esto Yesod define dos clases `FromJSON` ` ToJSON `. Por ejemplo, para la entidad `Demo` que se define como se muestra a continuación esta sería la instancia para ambas clases 
*Nota:* La instancia de las clases debe ser incluida en `src/Models.hs`

    Demo
        fieldOne Text 
        fieldTwo Int
        fieldThree Bool
---
    -- { "id": 1, "fieldOne": "example", "fieldTwo": 1, "fieldThree": true }
    instance ToJSON (Entity Demo) where
        toJSON (Entity demoId demo) = object
            [ "id"         .= (String $ toPathPiece demoId)
            , "fieldOne"   .= demoFieldOne demo
            , "fieldTwo"   .= demoFieldTwo demo
            , "fieldThree" .= demoFieldThree demo
            ]

    instance FromJSON Demo where
            parseJSON (Object demo) = Demo
                <$> demo .: "fieldOne"
                <*> demo .: "fieldTwo"
                <*> demo .: "fieldThree"

          parseJSON _ = mzero
          
### Routes

Vamos a definir 5 acciones básicas listar objetos, crear editarlos, encontrar un elemento por su id y eliminarlos. Para ellos definimos las rutas correspondientes para cada servicio.

    /demosJson         DemosJsonR GET POST
    /demoJson/#DemoId  DemoJsonR  GET PUT DELETE
    
### Handler

    getDemosJsonR :: Handler Value 
    getDemosJsonR = do
        demos <- runDB $ selectList [] [] :: Handler [Entity Demo]

        return $ object ["demos" .= demos]

    postDemosJsonR :: Handler Value 
    postDemosJsonR = do
        demo <- requireJsonBody :: Handler Demo
        _    <- runDB $ insert demo

        sendResponseStatus status201 ("CREATED" :: Text)

    getDemoJsonR :: DemoId -> Handler Value 
    getDemoJsonR demoId = do
        demo <- runDB $ get404 demoId

        return $ object ["demo" .= (Entity demoId demo)]

    putDemoJsonR :: DemoId -> Handler Value 
    putDemoJsonR demoId = do
        demo <- requireJsonBody :: Handler Demo

        runDB $ replace demoId demo

        sendResponseStatus status200 ("UPDATED" :: Text)

    deleteDemoJsonR :: DemoId -> Handler Value 
    deleteDemoJsonR demoId = do
        runDB $ delete demoId

        sendResponseStatus status200 ("DELETED" :: Text)


## Ejercicio

Basado en el ejerció realizado anteriormente para construir un juego de cartas tipo TCG (Magic – Yu-Gi-Oh - hearthstone) construir las 5 operaciones basicas para la entidad `Card` (o el equivalente en su proyecto).





