# Yesod primeros pasos

###  Configuración ambiente de desarrollo


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
        
        
###  Acciones basicas

1) Luego de generar el proyecto y compilarlo podemos ingresar a la ruta http://localhost:3000 para verificar el proyecto generado.

A continuación ingresamos en `my-proyect\config\models`. En este archivo permite crear dos cosas por un lado las tablas (del motor de base de datos seleccionado) de nuestro proyecto y por otro las entidades de dominio sobre las cuales podemos acceder y manipular dichas tablas. 

Como ejemplo vamos a definir una nueva tabla de prueba 

    Demo
        fieldOne Text 
        fieldTwo Int
        fieldThree Bool



