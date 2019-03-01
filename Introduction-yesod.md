# Yesod primeros pasos

##  Configuración ambiente de desarrollo


 1) Ejecutar en la terminal los siguientes comandos

 	    wget -qO- https://get.haskellstack.org/ | sh

2) Instalar las dependencias del motor de base de datos

	    sudo apt-get install -y libmysqlclient-dev    
	    sudo apt-get install libpcre3 libpcre3-dev    //Para Mysql
	
	    sudo apt-get install -y libpq-dev			  //Para postgreSQL

2.1) Para mac el comando equivalente seria

	    brew install postgresql			  //Para postgreSQL

3) Crear un nuevo scaffolding

	    stack new my-project yesod-sqlite && cd my-project  //Para Sqlite
	    stack new my-project yesod-mysql && cd my-project  //Para MySQl
	    stack new my-project yesod-postgres && cd my-project  //Para postgreSQL

4) acceder a la carpeta creada `my-proyect` y ejecutar los siguientes comandos

	    stack build yesod-bin cabal-install --install-ghc
	    stack build

5) subir la aplicación en ambiente de desarrollo con el comando

	    stack exec -- yesod devel


6) Otros comandos

	    stack exec -- cabal update //Actualizar dependencias
        
        
##  Acciones basicas

###  models

Luego de generar el proyecto y compilarlo podemos ingresar a la ruta http://localhost:3000 para verificar el proyecto generado.

A continuación ingresamos en `my-proyect\config\models`. En este archivo permite crear dos cosas por un lado las tablas (del motor de base de datos seleccionado) de nuestro proyecto y por otro las entidades de dominio sobre las cuales podemos acceder y manipular dichas tablas. 

Como ejemplo vamos a definir una nueva tabla de prueba 

    Demo
        fieldOne Text 
        fieldTwo Int
        fieldThree Bool
	
Al ingresarlo si compilamos nuevamente nuestro proyecto (enter en el terminal) recibiremos un input similar al siguiente:

	Migrating: CREATE TABLE "demo"("id" INTEGER PRIMARY KEY,"field_one" VARCHAR NOT NULL,"field_two" INTEGER NOT NULL,"field_three" BOOLEAN NOT NULL)
	Devel application launched: http://localhost:3000
	
Una de las grandes ventajas de `Persistent` el ORM aplicado en Yesod es la autogeneración de migraciones basado en cambios sobre el archivo `modeles` sin requerir (en principio) realizar querys a mano


###  routes

Las rutas se definen en `my-proyect\config\routes`. Una ruta se compone por: /pathDeLaRuta NombreRecurso HTTP-METODOS. Como ejemplo podemos definir la siguiente ruta

	/demoNew DemoNewR GET POST

donde `demoNew` es el path para acceder desde la url `DemoNewR` es el constructor para el tipo de dato asociado a la URL (urls de tipo seguro). Por convención estas siempre inician con letra mayuscula y terminana con un `R` mayuscula. `GET POST` son los servicios REST que se implementaran para esa ruta en especifico

###  Handlers

1) Las capas de controller, service, repository en Yesod están por defecto encapsuladas en los Handlers. Para definir los servicios que declaramos en las rutas vamos a crear un nuevo handler `Demo.hs` dentro del directorio `src\Handler`

Adicionalmente es necesario especificar el handler que creamos en los archivos `my-proyect.cabal` e imporarlos en `src\Appication.hs`

2) definimos los import basicos dentro de nuestro handler incluyendo sus extenciones 

		{-# LANGUAGE NoImplicitPrelude #-}
		{-# LANGUAGE OverloadedStrings #-}
		{-# LANGUAGE TemplateHaskell #-}
		{-# LANGUAGE MultiParamTypeClasses #-}
		{-# LANGUAGE TypeFamilies #-}
		module Handler.Demo where

		import Import
		import Yesod.Form.Bootstrap3 (BootstrapFormLayout (..), renderBootstrap3)
		

###  Forms

Existen tres tipos de formas en Yesod se utilizan para interactuar con los forms desde el lado del front y mapear la data a tipos definidos. Para este ejemplo usaremos una forma aplicativa como se muestra a continuación 

	--Aform From Entity Demo
	demoForm :: Maybe Demo -> AForm Handler Demo
	demoForm   demo = Demo 
			<$> areq textField "fieldone" (demoFieldOne <$> demo)
			<*> areq intField "fieldTwo" (demoFieldTwo <$> demo) 
			<*> areq boolField "fieldThree" (demoFieldThree <$> demo) 

### Shakespearean Templates

Shakespearean es otro de los DLS definidos en Yesod. Ofrece una sintaxis muy similiar a la de HTML pero ofrece funciones adicionales para manipular facilmente la data que se procesa en los handlers. Vamos a definir un formulario en el cual podamos capturar la data necesario para construir un `Demo` 


	<div .container>
	    <form method=post action=@{actionR} encType=#{encoding}>
		 <div .row>
		     ^{widget}   
		    <div .row .clearfix .visible-xs-block>
		    <button .btn .btn-success> 
		       <span .glyphicon .glyphicon-floppy-saved>
		       default text create

Guardamos este archivo como `DemoCreate.hamlet` en `static/template/Demo` 

## Servicios

Acorde a lo que especificamos en la ruta debemos especificar los servicios de GET y POST para la entidad creada

	--CRUD 
	--Create
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
			    			     
Con lo anterior si compilamos nuevamente nuestro proyecto y navegamos hasta http://localhost:3000/demoNew recibiremos el siguiente mensaje

	src/Foundation.hs:(163,5)-(172,45): Non-exhaustive patterns in function isAuthorized
	
Esto se debe a que por defecto el scaffolding de Yesod incluye un sistema de autorización para las rutas. Para solucionar esto vamos al archivo `src/Foundation.hs` y modificamos como se muestra a continuación 

    isAuthorized DemoNewR  _ = return Authorized


