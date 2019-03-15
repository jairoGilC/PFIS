
# Persistent

##  Sobre Persistent

De la misma forma que los `Forms` buscan resolver la comunicación entre la data ingresada por el usuario y los tipos seguros en Haskell `Persistent` busca resolver la comunicación entre dichos tipos y bases de datos 

* Database-agnostic: Poder comunicarse con diferentes DBMS aislando dicha configuración. 
* Uso de tipos seguros para la persistencia  
* Migraciones automáticas para un rápido desarrollo 

**Nota** Persistent posee un tremendo limitante en lo que acceso de datos se refiere ya que no soporta SQL Joins para resolver esto existen dos alternativas `RawSQL` `Esqueleto` 

## models

    Exam
        name Text
        description Text
        
    Question
        questionText Text
        exam         ExamId
        
    PossibleAnswer
        answer Text
        is_correct Bool       	
        justification Text Maybe
        question QuestionId

> justification Text Maybe 

Por defecto los atributos definidos son `NOTNULL` podemos usar el tipo `Maybe` para indicar que un campo es `nullable` 

>exam ExamId

Estable una relación foránea entre ambas entidades. La gran ventaja de Persistent comparada con otros ORM es que las relaciones son de tipo seguro de la misma manera que los identificadores de tal modo que si intentamos por ejemplo obtener un elemento `get` con un índice de una entidad que no corresponda el programa fallara en compilación 

### Tipos de datos disponibles para los atributos

Haskell	   |   PostgreSQL         |  MySQL            |  MongoDB      |  SQLite
-----------|----------------------|-------------------|---------------|---------
Text	   |  VARCHAR             |  TEXT             | String        |  VARCHAR
ByteString |  BYTEA               |  BLOB             | BinData       |  BLOB
Int        |  INT8                |  BIGINT(20)       | NumberLong    |  INTEGER
Double     |  DOUBLE PRECISION    |  DOUBLE           | Double        |  REAL
Rational   |  NUMERIC(22, 12)     |  DECIMAL(32,20)   | *Unsupported* |  NUMERIC(32,20)
Bool       |  BOOLEAN             |  TINYINT(1)       | Boolean       |  BOOLEAN
Day        |  DATE                |  DATE             | NumberLong    |  DATE
TimeOfDay  |  TIME                |  TIME\*\*         | *Unsupported* |  TIME
UTCTime\*  |  TIMESTAMP           |  DATETIME\*\*     | Date          |  TIMESTAMP
        
### Persistent typeclass

> PersistField    ===> tipos de datos de SQL INTEGER, VARCHAR....

> PersistValue    ===> Columnas de SQL

> PersistEntity   ===> Tablas SQL

## Migraciones

El detalle sobre como las migraciones funcionan en `Persistent` se puede encontrar en el YesodBook. Sin embargo. Es importante tener en cuenta que si bien las migraciones autogeneradas son bastante útiles a la hora de programar. Suponen un tremendo riesgo en entornos productivos.  

Por otro lado para el caso del ` scaffolding` de Yesod la configuración de las migraciones se puede encontrar en el archivo `Application.hs` 

Las migraciones automaticas que `Persistent` puede realizar son:

* Cambio de un tipo de dato (de Int a Text)
* Adición de un nuevo campo
* Cambio de notnull a null de un campo
* Adición de una nueva table

Las migraciones automaticas que `Persistent` **NO** puede realizar son:

* Cambio de nombres (En su lugar crear un nuevo campo sin borrar el que ya existia)
* Eliminar campos

## Uniqueness

    User
        username Text
        UniqueUsername username
        
    Person
        firstName String
        lastName String
        age Int
        PersonName firstName lastName

Inician con letra mayuscula pues son un constructor de datos. Es posible reaizar ademas una conbinacíon de campos para definir el constrain como se muestra en el ejemplo `Person`

**Nota:** Una restricción para campos únicos presente únicamente en `Persistent` impide definir constraints `unique` sobre campos nullable esto es así porque en campos nulos en Haskell y en SQL pueden comportarse diferente. Por ejemplo, en PostgreSQL `NULL == NULL == FALSE` mientras que en Haskell `Nothing == Nothing == True` 

## Selectores

* `get`: Encuentra un elemento por ID 

        michaelId <- insert $ Person "Michael" $ Just 26
        michael <- get michaelId

* `getBy` Encuentra un elemente a parti de un valor concreto

        maybePerson <- getBy $ PersonName "Michael" "Snoyman"
        
* `selectList` retorna una lista de elementos basado en un criterio de busqueda

        people <- selectList [PersonAge >. 25, PersonAge <=. 30] []

* `selectFirst` devuelve el primer elemento que coincida con un criterio de busqueda

        people <- selectFirst [PersonAge >. 25, PersonAge <=. 30] []
        
* `selectKeys` devuelve una lista de llaves basado en un criterio de busqueda

        people <- selectKeys [PersonAge >. 25, PersonAge <=. 30] []
        
### Operadores

La lista de operadores disponibles y su equivalente en SQL son:

| Operador      | Equivalente   | Ejemplo     |  Notas       |
| ------------- | ------------- |-------------|------------- |
|               | AND           | `[PersonAge >. 25, PersonAge <=. 30]` |Cada operación en la lista se relacióna con un `AND` por lo tanto no existe un operador especifico para el `AND` |
| `.\|\|`         | OR            | `([PersonAge >. 25] \|\|. [PersonAge <=. 30])` | |
| `/<-.`          | IN            | `[PersonFirstName /<-. ["Adam", "Bonny"]` | |
| `/<-.`          | NOT IN        | `[PersonFirstName /<-. ["Adam", "Bonny"]` | |
| `>.`, `>=.` `<=.`, `<=.`        |> >= < <=       | `[PersonAge >. 25]` | |


### Opciones 

El segundo parametro de los selectores en una lista de opciones, las cueales pueden ser:

| función       | Equivalente   | Ejemplo     |  
| ------------- | ------------- |-------------|
| `Asc`         | ASC           | `[Asc PersonLastName]` |
| `Desc`        | DESC          | `[Desc PersonLastName]` |
| `LimitTo`     | LIMIT         | `[LimitTo 100]` |
| `OffsetBy`    | OFFSET        | `[OffsetBy $ (pageNumber - 1) * resultsPerPage]` |

### Manipulación de datos

Antes de exponer las operaciones disponibles para la manipulación de datos es importante aclar que en el contexto de Persistent los identificadores están separados de los tipos concretos. Esto es así por practicidad en las operaciones de Insert (al momento de inserta el ID no existe y deberían definirse los id de las entidades como `Maybe` para poder realizar la función de insert) 

* `insert`: inserta un nuevo registro

        personId <- insert $ Person "Michael" "Snoyman" 26

* `update` recibe un identificador una lista de parametros a modificar y modifica dichos parametros, las operaciones disponibles para este segundo parametro son: `+=.`, `-=.`, `*=.`, `/=.`, `=.`

        personId <- insert $ Person "Michael" "Snoyman" 26
        update personId [PersonAge =. 27]
        
* `updateWhere` similar a `update` pero recibe una lista adicional de condiciones para ejectar el update en lugar de un identificador

        updateWhere [PersonFirstName ==. "Michael"] [PersonAge *=. 2]

* `replace` recibe un identificar un tipo concreto y remplaza cada valor por el nuevo tipo

        personId <- insert $ Person "Michael" "Snoyman" 26
        replace personId $ Person "John" "Doe" 20
        
* `delete` Borrado por Id

        personId <- insert $ Person "Michael" "Snoyman" 26
        delete personId
        
* `deleteBy` Borrado por constraints

        deleteBy $ PersonName "Michael" "Snoyman"

* `deleteWhere` Borrado basado en condiciones

        deleteWhere [PersonFirstName ==. "Michael"]
        
## Atributos

Cualquier campo en Persisten se compone como lo hemos visto hasta el momento de, un nombre un tipo y una lista de atributos. Ahora vamos a ver en detalle la lista de atributos  

En el siguiente [Enlace](https://github.com/yesodweb/persistent/edit/master/docs/Persistent-entity-syntax.md) se encuentra disponbiel la lista completa de atributos que podemos incluir en cualquier campo.


## Campos personalizados 

**Nota:** No es una buena práctica utilizar campos enumerables pues representan lógica de negocio oculta a nivel de base de datos. Sin embargo, gracias al sistema de tipos seguros que Persistent emplea esto no se aplica ya que el enumerable está definido y controlado a nivel de aplicación 

     Person
        name String
        employment Employment
        
-----
     data Employment = Employed | Unemployed | Retired
        deriving (Show, Read, Eq)
    derivePersistField "Employment"

## RawSQL

RawSQL es una alternativa que permite realizar query nativo de SQL es particularmente útil cuando se quiere resolver el problema de realizar JOINS entre tablas que no es soportado por Persistent. Su defecto es que no emplea tipos seguros con lo cual podremos encontrar errores en tiempo de ejecución sin mencionar lo críptico de los mensajes que podríamos obtener. En cualquier caso, es una alternativa necesaria en múltiples circunstancias.

    Author
        name Text
    Blog
        author AuthorId
        title Text
        content Html
---
    getExampleR :: Handler Html
    getExampleR = do
        blogs <- runDB $ rawSql
            "SELECT ??, ?? \
            \FROM blog INNER JOIN author \
            \ON blog.author=author.id"
            []
        
        
# Ejercicio

Basado en el ejercicio anterior de cartas. Realizar un buscardo de cartas que permita encontrar cartas basado en cualquier atrbutoincluyendo la desripción de la carta.


## Apendice

    getDemoSearchR ::  Handler Html 
    getDemoSearchR = do
               (widget, encoding) <- generateFormPost $ renderBootstrap3 BootstrapBasicForm $ demoForm Nothing
               defaultLayout $ do
                    let actionR = DemoSearchR
                    $(widgetFile "Demo/DemoSearch")

    postDemoSearchR :: Handler Html
    postDemoSearchR = do
                    ((result,widget), encoding) <- runFormPost $ renderBootstrap3 BootstrapBasicForm $ demoForm  Nothing
                    case result of
                         FormSuccess _ -> do 
                                     demos <- runDB $ selectList [] []
                                     defaultLayout $ do
                                        $(widgetFile "Demo/DemoList")
                         _ -> defaultLayout $ do 
                            let actionR = DemoSearchR
                            $(widgetFile "Demo/DemoCreate")
