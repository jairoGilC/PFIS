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
	
Al ingresarlo si compilamos nuevamente nuestro proyecto (oprimir cualquier tecla en el terminal) recibiremos un input similar al siguiente:

	Migrating: CREATE TABLE "demo"("id" INTEGER PRIMARY KEY,"field_one" VARCHAR NOT NULL,"field_two" INTEGER NOT NULL,"field_three" BOOLEAN NOT NULL)
	Devel application launched: http://localhost:3000
	
Una de las grandes ventajas de `Persistent` el ORM aplicado en Yesod es la autogeneración de migraciones basado en cambios sobre el archivo `modeles` sin requerir (en principio) realizar querys a mano


###  routes

Las rutas se definen en `my-proyect\config\routes`. Una ruta se compone por: /pathDeLaRuta NombreRecurso HTTP-METODOS. Como ejemplo podemos definir la siguiente ruta

	/demoNew DemoNewR GET POST

donde `demoNew` es el path para acceder desde la url `DemoNewR` es el constructor para el tipo de dato asociado a la URL (urls de tipo seguro). Por convención estas siempre inician con letra mayuscula y terminana con un `R` mayuscula. `GET POST` son los servicios REST que se implementaran para esa ruta en especifico

###  Handlers




