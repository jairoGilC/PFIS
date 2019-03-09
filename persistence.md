
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
//OJO AQUI EXPLICAR EL MANEJO DE LLAVES FORANEAS EN PERSISTENCE

### Tipos de datos disponibles para los atributos

    data PersistValue
        = PersistText Text
        | PersistByteString ByteString
        | PersistInt64 Int64
        | PersistDouble Double
        | PersistRational Rational
        | PersistBool Bool
        | PersistDay Day
        | PersistTimeOfDay TimeOfDay
        | PersistUTCTime UTCTime
        | PersistNull
        | PersistList [PersistValue]
        | PersistMap [(Text, PersistValue)]
        | PersistObjectId ByteString
        | PersistDbSpecific ByteString
        
### Persistent typeclass

> PersistField    ===> tipos de datos de SQL INTEGER, VARCHAR....

> PersistValue    ===> Columnas de SQL

> PersistEntity   ===> Tablas SQL
