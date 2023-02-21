# Probar nuestra app

Para probar nuesta app debemos de hacer request para esto podemos usar insomnia.rest

## Introduccion a Docker

### Que es un contenedor Docker

Docker por lo general corre en linux el cual es un motor que puede correr contenedores el cual dicho contenedor no corre una maquina virtual por lo que hay solo un software interactuando con el hardware

-El contenedor no necesita tener el codigo de un sistema operativo
-Para correr Docker localmente podemos usar una aplicacion llamada Docker desktop
-Un contenedor DOcker corre todo excepto el kernel entonce el contenedor corre todas la aplicaciones
-Podemos usar docker images que es una recopilacionde librerias, dependecias, tools y todo lo de mas exepto el Kernel

#### El archivo docker

En un archivo docker encontramos:

        -Una definicion de la imagen docker
        -Usamos dockerfiles para crear imagenes
        -Podemos usar una imagen para correr varios contenedores

Para crear una imagen podemos usar el siguiente comando debemos estar dentro de la carpeta donde tenemos nuestro archivo docker

        docker build -t rest-apis-flask-python .

Para comenzar a correr un contenedor se usa el siguiente comando:

        docker run -p 5000:5005 rest-apis-flask-python

## Correr aplicacion en Docker con autoreload

Para correr la aplicacion Flask dentro de Docker necesitamos:

        -Crear el documento Dockerfile con la siguiente informacion:

                FROM python:3.10
                EXPOSE 5000
                WORKDIR /app
                COPY requirements.txt .
                RUN pip install -r requirements.txt
                COPY . . 
                CMD ["flask", "run", "--host", "0.0.0.0"]

        -Crear nuestra imagen Docker con el comando:

                `docker build -t flask-smorest-api .`
        
        -Crear nuestro contenedor con el comando:

                `docker run -d -p 5005:5000 -w /app -v "$(pwd):/app" flask-smorest-api`

## Como usar Bluprints y MethodView en Flas

## Escribir marshmallow schemas para nuestra API

En esta ocasion usamos marshmallow schemas para hacer la validacion de los datos que recibimos. Para crear los schemas necesitamos:

        -Crear un archivo llamado schema.py
        -Importar de marshmallow la clase Schema y fields
        -Definir una clase que nuestro molde dicha clase se hereda a Schema
        -Dentro de la clare definimos los parametros derivados del nombre de la columna
        NOTA: En los direntes metodos que usamos para nuestra aplicacion en ocaciones necesitamos menos informacion para esto debemos crear un nuevo schema que se adapte a la necesidad

### Como aplicar la validacio con marshmallow

Una vez creada nuestro schema lo que queremo es emplear dicho schema haciendo una validacion de nuestra data para esto haremos lo siguiente:

        -Vamos a nuestro metodo e importamos los schemas que necesitamo en el recurso
        -A nuestro metodo le declaramos el decorador @blueprint.arguments(NombreSchema)
        -A nuestra funcion del metodo le declaramos que recibe informacion del objeto que se la data validada
        Nota: Lo que pasa aqui es que el cliente manda la informacion al recurso el validador de marshmallow valida dicha informacion y en caso de ser correcta regresa la informacion a nuestra funcion del metodo

### Decorar respuestas con Flask-Smorest

Decorar las respuestas nos ayuda a mejorar la forma en que vamos a regresar la informacion al cliente, que status code se va regresar y la documentacion de nuestra respues para esto en nuetro metodos hacemos lo siguiente:

        -Le aplicamos el decorador @bluprintname.response(statuscode, SchemaName)
        -El orden de los decoradores importa por lo que el decorador respose debe ir el la posicion mas profunda de la funcion
        -Si queremos regreasar multiples objetos debemos de usar @bluprintname.response(statuscode, SchemaName(many=True))

## Por que usar SQLAlchemy

Se usa para poder hacer consultas de una manera mas sencilla a nuestra base de datos como quiera podemos usar sentencias nativas de SQL pero al ser mas practico de usar pues ahorra lineas de codigo usamos sql alchemy

### Modelo simple de SQLAchemy

Para crear nuevos modelos:

        - Dentro de nuestro archivo creamos una intancia llamada db a partir de la clase SQLAchemy (db = SQLAlchemy())
        - Creamos una carpeta donde declaramos nuestros modemos
        - Importamos dentro de nuestro archivo del modelo el objeto db
        - Creamos una clase que sera nuestro modelo la cual hereda a db.Model (class ItemModel(db.Model))
        - Dentro de la clase declaramos el nombre de la tabla con __name__ = 
        - Declaramos las columas de la clase (id = db.Column(db.Integer, primary_key=True))

### Relacion uno a muchos en SQLAlchemy

Para crear una relacion de uno a muchos necesitamos tener dos tablas por ejemplo una tienda puede tener muchos articulos

        -Para esto dentro de nuestra tabla que tendra varios articulos declaramos una FereignKey que hace referencia al id de la otra tabla "store_id = db.Column(db.Integer, db.ForeignKet(store.id), unique=False, nullable=False)"
        - Creamos la relacion para traer el objeto de la otra tabla con " store = db.relationship("StoreModel", back_populates="items") " lo anterior hara que la variable store traera la informacion de la tienda con el store_id del item

La vetaja es que vamso a evitar que creemos un articulo con un id de una tienda que no existe

### Configurar SQLAchemy

Para cargar las configuracionde de SQLAchemy a nuestra aplicacion dentro de del documento app.py creamos las siguientes variables:

                app.config["SQLALCHEMY_DATABASE_URI"] = db_url or os.getenv("DATABASE_URL", "sqlite:///data.db")
                app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
                db.init_app(app)

                api = Api(app)

                @app.before_first_request
                def create_tables():
                        db.create_all()

### Insertar modelos en SQLAlchemy

Para insertar un objeto a nuestra base de datos debemos crear un objeto a partir de nuestro modelos ejemplo:

        item = ItemModel(id=1, name="name", ...)
        db.session.add(item)
        db.session.commit()

### Seleccionar informacion de la base de datos o devolver 404

FlaskSQLAlchemy tiene una funcion integrada para devolver el objeto que se busca o devolver un error 404 para esto se debe de usar el siguiente metodo:

        item = ItemModel.query.get_or_404(item_id)
        return item

### Actualizar datos con SQLAlchemy

Para actualizar registros en nuestra base de datos con SQLAlchemy necesitamos:

### Eliminar registros de la base de datos

Para eliminar un registro de la base de datos a partir de su id:

        store = StoreModel.query.get_or_404(store_id) #Se comprueba que exista el elemento
        db.session.delete(store)
        db.session.commit()

### Eliminar modelos relacionado en cascada

Cuando eliminamos una tienda que tiene varios articulos en nuetra base de datos lo que queremos es que se borrer dichos articulos en consecuencia para esto usamos esta funcion esto va depende de la funcion de nuestra aplicacion para esto en nuestro modelo hacemos lo siguiente:

        class StoreModel(db.Model):
                ...
                items = db.relationship("ItemModel", back_populates="store", lazy="dynamic", cascade="all, delete")

## Relacion many to many

Para una relacion de many-to-many un modelo no puede tener un simple valor como llave foranea (si lo tuviera seria one-to-many). Por otra parte nosotros creamos una tabla secundaria que tiene en cada fila una tagID y un itemID

La tabla secundaria lo que nos dice es que cierta tag que pertenece a la tabla tags pertence a un item que pertence a la tabla items ejemplo:

                id      tag_id          item_id
                1          2               5
                2          1               4
                3          1               3
                4          4               5

el item 5 tiene las tags 2 y 4
el item 4 y 3 tiene la etiqueta 1

Se debe de crear un modelo para esta tabla secundaria y conectar dicho modelo con las tablas de tags e items

## Autenticacion con Flask-JWT

La forma en que nos autenticamos en internet generalmente es a traves de toquen los cuales se generan al momento de que un cliente accede a un sitio de login entonces el usuario provee a nuestra API generamente de un usuario y una contrasennia, la API valida la informacion y le asigna dicho token el cual puende ser usado para acceder a sitios o ruta que requieren uso de dicha autenticacion.

Generalidades:
        -Los tokens no duran para siempre pues seria un problema de seguridad
        -El tiempo de vida mas corto es mas seguro pero es mas molesto para los usuarios
        -Podemos refrescar el token para mejorar la experiencia de usuario

### Configurar JWT

1.- Para comenzar a trabajar con JWT necesitamos instalar las librerias "flask-jwt-extended"
2.- Agregar "from flask_jwt_extended import JWTManager" a nuestra aplicacion
3.- Establecer una secretkey: "app.config["JWT_SECRET_KEY"] = "braulio"" esto se usa para firmar los jwt y evitar que el usuario genere la llave en otro lugar pretendiendo que la genero nuestra API
        Para generar nuestra SECRET_KEY podemos usar un generador random usando el modulo secretes
        "app.config["JWT_SECRET_KEY"] = secrets.SystemRandom().getrandbits(128)"
4.- Instaciar jwt = JWTManager(app)

Para trabajar con la autenticacion debemos de crear una tabla que almacene nuestra informacion de usuario para esto creamos un model llamado usuario con sus repectivas columna y tambien creamos su respectivo Schema

## Como agregar un register endpoint a la REST API

Creamos un archivo "user" en la carpeta resources
Importamos from passlib.hash import pbkdf2_sha256 para hashear las contrasennas
Se crean dos rutas una para registrar un nuevo usuario y la otra para ver o borrar usuarios segun su id

## Agregar un endpoint para hacer login

Importamos from flask_jwt_extended import create_access_token para enviarlo al cliente en caso de se verifique el usuario y contra
Creamo una ruta nueva llamada "/login" dentro del recurso user
dentro de la ruta creamos un metodo post el cual verifica que usuario exista y validamos que el hash coicida con la contrasena

## Proteger los endpoints requiriendo un JWT

Si el usuario no hace login y no envia el token de acceso de esta forma no podremos crear nuevos items

Para hacer lo anterior importamos from flask_jwt_extended import jwt_required al archivo donde tenemos nuestros recursos de items
Despues decoramos la funcion del metodo que sera restringido con "@jwt_required()"
Para probar nuestro codigo en insomnia vamos a la peticion post de item para agregar un item y en la seccion del campo agregamos un nuevo campo con la siguiente informacion:

        Authorization           Bearer Token-generado

### Manejando mensajes de error de JWT

Para manejar lo mesajes de errores se deben de agrgar funciones decoradas con su respectivo error a nuestra app esto se hace justo despues de instancian nuestra clase JWTManager(app)

## Reclamaciones y autorizaciones de JWT

Las reclamaciones no se usan muy seguido pero se usan para agregar informacion extra a nustro JWT token esto tambien se agrega en forma de funcion decorada a nuetra aplicacion jsut despues donde se instancio JWTManager(app) ejemplo

@jwt.additional_claims_loader
def add_claims_to_jwt(identity):
        if identity == 1:
                return {"is_admin": True}
        return {"is_admin": False}

En este caso se utilizo las reclamaciones para verificar que un usuario sea o no sea admin al momento de eliminar un item la misma validacion se podria hacer directamente en el recurso delete ya dependera del desarrollo de la aplicacion determinar cual es la mas conveniente

## Como hacer un logout en la REST API

Se crea un archivo llamado blocklist.py la cual almacena la informacion de los tokens usados
Creamos las siguiente funciones dentro de nuestra app
    @jwt.token_in_blocklist_loader
    def chek_if_token_in_blocklist(jwt_header, jwt_payload):
        return jwt_payload["jwt"] in BLOCKLIST

    @jwt.revoked_token_loader
    def revoked_token_callback(jwt_header, jwt_payload):
        return (
            jsonify(
                {"description": "The token has been revoked.", "error": "toke_revoked"}
            ),
            401,
        )
Creamos una ruta en los recursos de users para hacer el logout

Se recomienda usar una base de datos para almacenar los token usados ya que si usamos simplemente una estructura de datos de python para almacenar los tokens esta no sera persistente por lo que se borrara si restablecemos la aplicacion

## Mantener la sesion con insomnia

Al momento de hacer request con insominia no queremo estar copiando y pegando nuestro JWT token por cada enpoint que tenemos en la aplicacion para esto debemos de hacer lo siguiente:

En la zona de header escribimos:

                Authorization           Bearer crt + space

Lo anterior deplega una lista y seleccionamos Response -> Body Attribute

Nos mostrara un boton naranaja si hacemos click lo podremos cofigutar

        Request:
                [User] POST /login - authenticate use
        Filter (JSONPath or XPath)
                $.access_token
        Trugger Behavior
                When Expired

Tambien podemos crear una variable global:
{
"url": "http://127.0.0.1:5005",
"access_toke": "{% response 'body',"
}

De la anterior manera ahora podemos ir a los headers de cada ruta y justo despues de Bearer usar: {{access_token}}

## Refrescar token con Flask-JWT-Extended

Refrescar token es util cuando queremos mejorar la expericia del usuario, al usarlo podemos evitar que un usuario haga login cada vez que el token expira

Podemos en nuestra aplicacion tener endpoint que necesiten un token reciente y tambien podemos tener endpoint que no necesariamente usen un token fresco para esto podemos segmentar nuestra api, entonce cuando un endpoint no es importate no requeriomos un token fresco lo contrari ocurre para un end point importante

Para esto:

        1.- importamos from flask_jwt_extended import create_refresh_toke, get,jwt_identity
        2.- creamos un refresh_token al momento de hacer login, el refresh_token es algo que el usuario va usar cuando necesite refrescar su token por lo que al visitar el endpoint "url/refresh" se le pedira en el header el refresh token 
        3.- Creamos en los recurso de Users el MethodView para hacer refres "/refresh"
        4.- Una vez creado lo anterior podemos ir a nuestro diferentes end point y solo permitir a los endpoint importantes solictar un token fresco esto se hace de la siguiente manera: @jwt_required(fresh=True) lo anterior se puede hacer para cambiar la contrasena, eliminar cuenta y ese timpo de acciones importantes
        5.- Por ultimo agregamos la siguiente funcion a app.py para que maneje el error en caso de no tener un token fresco

                @jwt.needs_fresh_token_loader
                def token_not_fresh_callback(jwt_header, jwt_payload):
                        return (
                        jsonify(
                                {
                                "description": "The toke is not fresh.",
                                "error": "fresh_token_required",
                                }
                        ),
                        401
                                )

Cada vez que el token expira el usuario puede ir a "/refresh" para obtener un nuevo access_token

## Migraciones con Alembic y Flask-Migrate

Las migraciones no ayudan a hacer cambios en la estructura de nuetras tablas a lo largo de la vida de nuestra aplicacion para agregar flask migrate debemos seguir lo siguiente:

        1.- agregar pip install flask-migrate
        2.- importamos en app.py from flask_migrate import Migrate
        3.- instanciamos migrate = Migrate(app, db)

Para incializar la base de datos:
        1.- en la cosola de comandos:  flask db init esto crea una carpeta llamada migrations
        2.- creamos la primer migracion flask db migrate -m "Initial migration" crea una nueva version basada en los modelos que ya tenemos declarados en nuestra aplicacion
        3.- aplicamos los cambios con flask db upgrade

Cuando agregamos una nueva columna a una tabla ya existente y con datos flask-migrate agregara un valor por defecto a los registros anteriores con el valor de "null" esto puede ocasionar problemas en nuestra aplicacion lo que debemos hacer es dentro de nuestra version de migraciona agregar un query para actualizar la informacion aterior

op.execute("UPDATE invoices SET enable_dowloads = False")

## Taller de git

Git se usa para coolaborar con un equipo y almacenar cambios de nuestros archivos

        - git init      inicializa nuestro repositorio
        - git status    muestra como va el repositorio

Creamos una carpeta .gitignore para agregar las carpetas y archivos que no queremos que sean agregados a nuestro repositorio ejemplo dentro del archivo podemos incluir:

                        `/venv`
                        `/vscode`
                        `/__pycache__`
                        `data.db`
                        `*.pyc`
                        `.DS_Store`
                        `.venv`
                        

        - `git commit -m "" `             Crea un commit con un mesage
        - `git checkout -- app.py`        Regresa al utimo commit un archivo
        - `git restore app.py `           Regresa al utimo commit un archivo

### Branches and merging

Las branches o ramas son un nombre para un commit:

        - Una branch regularmente empieza con una branch llamada main o master
        - HEAD es similar a branch, es el actual commit activo

### Crean nuevas ramas o branches

`git branch new_branch`         -Crea una nueva branch
`git checkout new_branch`       -Cambia el head a new_branch
Dentro de la branch main
`git branch main`
`git merge new_branch`          -Hace un merge e identifica los cambios echos en las dos branches,
                                en casos de que las dos branches hayan echo cambios en el mismo archivo y las mismas lineas se crea un "merge coflict"

### Merge conflicts y como resolverlos

en casos de que las dos branches hayan echo cambios en el mismo archivo y las mismas lineas se crea un "merge coflict" al suceder esto en nuestro codigo aparecera unas leyendas las cuales hacen referencia a como era el HEAD antes de hacer el merge y otra etiqueta seprarada por signos de igual que muestra que es lo que viene de la otra rama.

Las acciones que podemos tomar en la situacion atererior son:

                - Mantener el codigo como antes, esto elimina nuestro codigo
                - Mantener el nuevo codigo, esto desecha el codigo existente es conveniente hablar primero antes tomar accion
                - Y por ultimo cambiar el codigo a lo que quramos y hacer commit, por lo que se elimanan los signos >>> y <<< y se cambia el codigo a lo queremos despues de hacer el merge

## Desplegando la aplicacion en linea

Para desplegar la aplicacione en linea:

        1.- Creat una cuenra en render.com
        2.- Seleccinar new web service
        3.- Conectamos nuestro git hub y seleccionamos el repositorio a desplegar
        4.- Asginamos un nombre, un enviroment en este caso Docker, la brach del repositorio, tipo de plan.
        5.- Creamos web service esto puede tardar ya que es un plan free

## Correr flask con gunicorn en Docker

Gunicron mejora el desempleno de nuestra aplicacion para eso:

        1.- Agregamos "gunicorn" a nuestro archivo requirements.txt
        2.- En docker file dejamos los siguientes comandos
                `FROM python:3.10`
                `WORKDIR /app`
                COPY requirements.txt .
                `RUN pip install --no-cache-dir --upgrade -r requirements.txt`
                `COPY . . `
                `CMD ["gunicorn", "--bind", "0.0.0.0:80", "app:create_app()"]`

## Desplegar base de datos PostgreSQL

Se recomienda usar ElephantSQL

        1.- creamos una cuenta
        2.- creamos una nueva instacia
        3.- asignamos nombre, region y plan
        4.- seleccionamos nuestra instancia
        5.- copiamos nuestro url

## Usar PostgreSQL de forma local y en produccion

Para usar PostgreSQL de forma local debemos:

        1.- Crear archivo ".env" dentro del proyecto y agregarlo a .gitignore y agregamos el url copiado de elephant y cambiando `postgre` por `postgresql` ejemplo:
                `DATABASE_URL=postgresql://xxxx`
        2.- Agregar psycopg2 a requirements.txt
        3.- Crear archivo ".env.example" para que se corparta en el repositorio
        4.- Importamos en app.py `from dotenv import load_dotenv`
        5.- En la funcion de app.py "create_app" agregamos `load_dotenv()`
        6.- Ejecutamos el comando en la terminal `flask db migrate` seguido de `flask db upgrade`
        7.- Creamos el archivo `docker-entrypoint.sh`
        8.- En el archivo "Dockerfile" sustituimos `CMD ["gunicorn", "--bind", "0.0.0.0:80", "app:create_app()"]` con `CMD ["/bin/bash", "docker-entrypoint.sh"]`
        9.- Hacemos commit y subios nuestro repositorio a github

Usar PostgreSQL para produccion

Una vez echo los cambios anteriores a nuestra aplicacion podemos desplegarla en produccion para esto:

        1.- Vamos a render.com
        2.- Vamos a elephant.com y copiamos el URL
        3.- En render en la parte de eviroment agregamos una variable llamada `DATABASE_URL` y pegamos el URL de la base de datos asefurandonos de cambiar "postgres" por "postgresql"
        4.- Guardamos y deveria cargar otro despliegue de la app
        5.- Listo ya podemos probar la aplicacion

Nota importante usar el comando `CMD ["/bin/bash", "docker-entrypoint.sh"]` en nuestro docker file

## Como enviar emails con Python y Mailgun

Mailgun es un servicio de entrega de correos que puede usar python para usarlo debemos:

        1.- Crear una cuenta https://www.mailgun.com/
        2.- Ir al dominio sandbox ya creafo y seleccionar API
        3.- Lo anterior te da un key y un URL
        4.- Seleccionamos el leguaje de programacion y nos muestra un ejemplo de integracion
        5.- Agregar la libreria requests a requirements.txt
        6.- Agregar `MAILGUN_API_KEY` Y `MAILGUN_DOMAIN` al archivp ".env"
        7.- Pegar el codigo de ejemplo en nuestro archivo user.py de recursos

## Enviar emails cuando el usuario se registra

Pra enviar email de confirmacion al registrar un usuario:

        1.- Agregamos el campo de email a el modelo "User"
        2.- Creamos la migracion
        3.- Creamo un nuevo Schema que contega el campo para email
        4.- Mandamos a llamar la funcion en el recurso de registro justo despues de agregar el usuario a nuestra base de datos
        5.- Dentro de mailgun agregamos el correo de prueba a la seccion de authorized receipents
        6.- Pobar la aplicacion

## Que es una fila de tareas y como usar la base de datos Redis

Una queue es una estructura de datos de la queue puede eliminar informacion, pero un punto clave es que cuando se remueve un dato de ella pasa la posicion a la siguiente pieza de informacion

Para emplear las queue necesitamos implemetar una base de datos Redis para esto:
        1.- Vamos a render.com
        2.- Creamos un nueva intacia tipo Redis asiganamos un name, region, noeviction for queues, free plan
        3.- Agregamos 0.0.0.0/0 a access control, guardamos
        4.- Copiamos Extenal Redis URL y lo agregamos al archivo ".env" REDIS_URL=xxxxx
        (Se puede usar redis localmente con docker pero se recomienda usar render pues es mas facil de usar)

## Como poblar y consumir la fila de tareas (task queues) con rq

Para consumir y crear nuestras queues necesitamos:

        1.- Agragar "rq" a "requirements.txt" que es una libreria para administrar redis y consumirla
        2.- Crear un archivo "tasks.py" en nuestra carpeta de proyecto
        3.- Movemos funciones a este archivo como la funcion send_simple_message
        4.- Creamos una nueva funcion que llama a la funcion "send_simple_message"
        5.- Importamos redis y `from rq import Queue` en "app.py"
        6.- Creamos la conexion dentro de la funcion "create_app"
        7.- Dentro de nuestro recurso que envia el correo (User) importamos `from flask import current_app`
        8.- Agregamos `current_app.queue.enqueue(send_user_registration_email, user.email, user.username)` donde queremos que ejecute la accion

## Como procesar las tareas de segundo plano con rq worker

Despues de haber agregado nuestra tarea a nuestra app los paso a seguir son:

        1.- Crear nuestra imagen docker `docker build -t rest-apis-flask-smorest-rq .`
        2.- En otra termianl correr el comando `docker run -w /app rest-apis-flask-smorest-rq sh -c "rq worker -u <insert your Redis url here> emails"`
        3.- Ejecutamos `docker run -p 5000:80`
        docker run -p 5000:5000 rest-apis-flask-smorest-rq sh -c "flask run --host 0.0.0.0"

## Como enviar correos HTML usando Mailgun y Python

Cada cliente de email tiene pequenas diferencias y soporta diferentes versiones de HTML y CSS por lo que mailgun cuenta con algunos templates para resolver estos problemas los templates los pidemos encontrar en: [github](https://github.com/mailgun/transactional-email-templates) para implementarlas:

        1.- Creamos una carpeta llamada templates/email y el archivo action.html
        2.- Copiamos el codigo de ejemplo de github y lo pegamos
        3.- Agregamos un parametro extra llamado html a nuestra funcion "send_simple_message"
        4.- Importamos jinja2 `import jinja2`
        5.- Declaramos `template_loader = jinja2.FileSystemLoader("templates")`  

## Como desplegar un background worker en render.com

        1.- Crear un archivo llamados "settings.py" donde viviran las configuraciones de rq
        2.-