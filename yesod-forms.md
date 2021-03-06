# Formas en Yesod

##  Sobre las formas en Yesod

Buscan resolver los principales problemas asociados a formularios web

* Validaciones
* Mapear los String ingresados en el formulario a tipos de datos en Haskell
* Generar código HTML 

## Formas aplicativas

Es el tipo de forma mas común. Incluye algunas propiedades para validaciones y manejo de mensajes de error

    carAForm :: AForm Handler Car
    carAForm = Car
        <$> areq textField "Model" Nothing
        <*> areq intField "Year" Nothing

**Nota:** Los tipos estan simplificados con el proposito de entender mejor su comportamiento. El detalle real de cada tipo se puede ver en [Hackage](http://hackage.haskell.org/package/yesod-form-1.6.4/docs/Yesod-Form-Functions.html) 

> areq :: Field m a -> FieldSettings site -> Maybe a -> AForm m a

Field m a             ==> Tipo de campo  
FieldSettings site    ==> Identificador (intenacionalización)  
Maybe a               ==> Valor por defecto   

Una alternativa para manejar el valor por defecto de los campos es pasar un opcional del tipo de dato y mapear cada valor sobre el formulario

    carAForm :: Maybe Car -> AForm Handler Car
    carAForm mcar = Car
        <$> areq textField "Model" (carModel <$> mcar)
        <*> areq intField  "Year"  (carYear  <$> mcar)
        
Al igual que `areq` existe `aopt` que se utiliza para campos opcionales, manteniendo sin embargo la misma sintaxis.

### Validaciones

Existen dos funciones principales para relizar validaciones `check`, `checkBool` y `checkM`. La definicion de cada una seria 

> checkBool :: Monad m => (a -> Bool) -> msg -> Field m a -> Field m a

> check :: Monad m => (a -> Either msg a) -> Field m a -> Field m a

> checkM :: Monad m => (a -> m (Either msg a)) -> Field m a -> Field m a

La principal diferencia entre `check` y `checkM` es que este ultimo permite permite validaciones con funciones monadicas. Es decir que el tipo de retorno tiene contexto. Por ejemplo:

    carAForm :: Maybe Car -> AForm Handler Car
    carAForm mcar = Car
        <$> areq textField    "Model" (carModel <$> mcar)
        <*> areq carYearField "Year"  (carYear  <$> mcar)
        <*> aopt textField    "Color" (carColor <$> mcar)
      where
        carYearField = checkBool (>= 1990) errorMessage intField

---
    carAForm :: Maybe Car -> AForm Handler Car
    carAForm mcar = Car
        <$> areq textField    "Model" (carModel <$> mcar)
        <*> areq carYearField "Year"  (carYear  <$> mcar)
        <*> aopt textField    "Color" (carColor <$> mcar)
      where
        carYearField = checkM inPast $ checkBool (>= 1990) errorMessage intField
        inPast y = do
            thisYear <- liftIO getCurrentYear
            return $ if y <= thisYear
                then Right y
                else Left ("You have a time machine!" :: Text)

    getCurrentYear :: IO Int
    getCurrentYear = do
        now <- getCurrentTime
        let today = utctDay now
        let (year, _, _) = toGregorian today
        return $ fromInteger year
        
### Selectores

Existen dos funciones muy utiles `selectFieldList` `radioField` para hacer listas y radio buttons segun se requiera. En cualquier caso su uso se explica en el siguiente ejemplo

**Nota:** Este ejemplo trae la data a mostrar directamente de base de datos. Sin embargo no es mandatorio. Un ejemplo alternativo mucho mas simble se puede encontrar e  [YesodBook](https://www.yesodweb.com/book/forms#forms_more_sophisticated_fields)

    carAForm :: Maybe Car -> AForm Handler Car
    carAForm mcar = Car
        <$> areq textField "Model" (carModel <$> mcar)
        <*> areq carYearField "Year" (carYear <$> mcar)
        <*> aopt (selectFieldList colors) "Color" (carColor <$> mcar)
      where
        colors :: [(Text, Color)]
        colors = [("Red", Red), ("Blue", Blue), ("Gray", Gray), ("Black", Black)]
        
---
    carAForm :: Maybe Car -> AForm Handler Car
    carAForm mcar = Car
        <$> areq textField "Model" (carModel <$> mcar)
        <*> areq carYearField "Year" (carYear <$> mcar)
        <*> aopt (selectFieldList colors) "Color" (carColor <$> mcar)
      where colors :: Handler (OptionList CityId)
            colors = do
               entities <- runDB $ selectList [] [Asc ColorName]
               optionsPairs $ colorPair <$> entities
            colorPair color = (cityName $ entityVal color, entityKey color)
            
### Ejecutando formas

existen varias formas de ejecutar formas. Sin embargo nos vamos a concentrar en dos en particular 

`generateFormPost` Genera el HTML para mostrar el formulario

`runFormPost` Permite ejecutar el formulario para enviar los parametros

    getDemoNewR ::  Handler Html 
    getDemoNewR = do
               (widget, encoding) <- generateFormPost $ renderBootstrap3 BootstrapBasicForm $ demoForm Nothing
               defaultLayout $ do
                    let actionR = DemoNewR
                    $(widgetFile "Demo/DemoCreate")

    postDemoNewR :: Handler Html
    postDemoNewR = do
                    ((result,widget), encoding) <- runFormPost $ renderBootstrap3 BootstrapBasicForm $ demoForm  Nothing
                    case result of
                         FormSuccess demo -> do 
                                     _ <- runDB $ insert demo
                                     redirect HomeR
                         _ -> defaultLayout $ do 
                            let actionR = DemoNewR
                            $(widgetFile "Demo/DemoCreate")
  
  ### Parametros de forma
  
Existen ciertos atributos de una entidad que no necesariamente deben venir del usuario, por ejemplo el id del usuario, para estos casos una forma puede recibir uno o mas parámetros.

    carAForm :: UserId ->  Maybe Car -> AForm Handler Car
    carAForm uid mcar = Car
        <$> areq textField    "Model" (carModel <$> mcar)
        <*> areq carYearField "Year"  (carYear  <$> mcar)
        <*> aopt textField    "Color" (carColor <$> mcar)
        <*> pure uid
        
  ### Apendice
  
  Relaciones foráneas en Persistence
  
    Question
        questionText Text
        exam         ExamId 
    Exam
        name Text
        description Text
  
  ## Ejercicio 

![ ](https://i.pinimg.com/originals/83/c4/cb/83c4cb42bd400eb585c3ce03b582ef99.png)


Basado en el ejerció realizado anteriormente para construir un juego de cartas tipo TCG (Magic – Yu-Gi-Oh - hearthstone) construir un aplicativo web en Yesod que permita a los jugadores construir mazos.

Tenga en cuenta que:

* Los campos deben tener su correspondiente validación para garantizar que no se creen cartas que no tengan sentido (de un tipo invalido, atributo inválidos…)
* Debe construir un modelo de datos consistente que aplique para el sistema diseñado en el ejercicio anterior
