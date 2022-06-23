# Curso de Flask

## Fundamentos de Flask

### Hello World Flask

Debemos de crear nuestro entorno virtual e instalar Flask con el comando `pip install flask`.

En nuestro archivo main.py hacemos:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello world Flask'

```

Y con esto podemos encender el servidor. En Unix es `export FLASK_APP=main.py` y en Windows es `set FLASK_APP=main.py`. Pero en la PowerShell es `$env:FLASK_APP="main.py"`

Y finalmente, hacemos `flask run`.

### Debugging en Flask

Podemos activar el modo debug de 2 formas:

1. Con el comando `export FLASK_DEBUG=1` o `SET FLASK_DEBUG=1`
2. En el código de python, hacer:

```python
if __name__ == '__main__':
    app.run(debug=True)
```

Y con esto, desde la terminal ejecutamos el archivo de Python.

### Request & response

Importamos request y lo usamos para, por ejemplo, mostrar la IP del usuario:

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def hello():
    user_ip = request.remote_addr
    return f'Tu dirección IP es {user_ip}'

```

### Ciclos de Request y Response

Podemos crear y obtener cookies desde el browser de la siguiente manera:

```python
from flask import Flask, request, make_response, redirect

app = Flask(__name__)

@app.route('/')
def index():
    user_ip = request.remote_addr

    response = make_response(redirect('/hello'))
    response.set_cookie('user_ip', user_ip)

    return response

@app.route('/hello')
def hello():
    user_ip = request.cookies.get('user_ip')
    return f'Tu dirección IP, (después de haberla obtenida con una cookie 🍪) es {user_ip}'

```

## Uso de templates y archivos estáticos

### Templates con Jinja 2

Los templates son archivos de HTML que hacen un render de la información.

Debemos de hacer en nuestro main.py:

```python
from flask import Flask, request, make_response, redirect, render_template

app = Flask(__name__)

@app.route('/')
def index():
    user_ip = request.remote_addr

    response = make_response(redirect('/hello'))
    response.set_cookie('user_ip', user_ip)

    return response

@app.route('/hello')
def hello():
    user_ip = request.cookies.get('user_ip')
    return render_template('hello.html', user_ip=user_ip)

```

Y creamos un archivo HTML donde mandamos nuestra variable, con el mismo nombre:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Curso de Flask</title>
</head>
<body>
    <h1>Tu dirección IP, (después de haberla obtenida con una cookie 🍪) es {{user_ip}}</h1>
</body>
</html>
```

### Estructuras de control

Podemos hacer uso de `if` y `for` dentro de las templates:

````python
from flask import Flask, request, make_response, redirect, render_template

app = Flask(__name__)

todos = ['TODO 1', 'TODO 2', 'TODO 3']

@app.route('/')
def index():
    user_ip = request.remote_addr

    response = make_response(redirect('/hello'))
    response.set_cookie('user_ip', user_ip)

    return response

@app.route('/hello')
def hello():
    user_ip = request.cookies.get('user_ip')
    context = {
        'user_ip': user_ip,
        'todos': todos
    }
    return render_template('hello.html', **context)

````

````html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Curso de Flask</title>
</head>
<body>
{% if user_ip %}
    <h1>Tu dirección IP, (después de haberla obtenida con una cookie 🍪) es {{user_ip}}</h1>
{% else %}
    <a href="{{ url_for('index') }}">Ir a inicio</a>
{% endif %}

<ul>
    {% for todo in todos %}
        <li>{{ todo }} | Índice del elemento -> {{ loop.index }}</li>
    {% endfor %}
</ul>


</body>
</html>
````

### Herencia de templates

Podemos exportar e importar templates.

Creamos un archivo 'base.html':

````html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} Curso Flask | {% endblock %}</title>
</head>
<body>
{% block content %}
{% endblock %}
</body>
</html>
````

El contenido será hello.html:

````html
{% extends 'base.html' %}
{% import 'macros.html' as macros %}

{% block title %}
    {{ super() }}
    Bienvenido
{% endblock %}

{% block content %}
    {% if user_ip %}
        <h1>Tu dirección IP, (después de haberla obtenida con una cookie 🍪) es {{user_ip}}</h1>
    {% else %}
        <a href="{{ url_for('index') }}">Ir a inicio</a>
    {% endif %}

    <ul>
        {% for todo in todos %}
            {{ macros.render_todo(todo) }} | índice -> {{ loop.index }}
        {% endfor %}
    </ul>
{% endblock %}

````

Podemos crear archivos que tendrán código que será usado muchas veces, comunmente llamados macros;

````html
{% macro render_todo(todo) %}
  <li>{{ todo }}</li>
{% endmacro %}
````

### Include y links

Creamos un archivo navbar.html:

````html
<nav>
    <ul>
        <li><a href="{{ url_for('index') }}">Ir a inicio</a></li>
        <li><a href="https://midiaenunosminutos.com" target="_blank">Ir a mi blog</a></li>
    </ul>
</nav>
````

Y lo incluimos en base.html:

````html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} Curso Flask | {% endblock %}</title>
</head>
<body>
    <header>
        {% include 'navbar.html' %}
    </header>
    {% block content %}
    {% endblock %}
</body>
</html>
````

### Uso de archivos estáticos: imágenes

Creamos un directorio 'static' donde vivirán las imágenes y algunos archivos de css.

Insertamos la imagen en el navbar:

```html
<nav>
    <ul>
        <img
            src="{{ url_for('static', filename='images/bocetoRock.png') }}"
            alt="Boceto Rock"
        />
        <li><a href="{{ url_for('index') }}">Ir a inicio</a></li>
        <li><a href="https://midiaenunosminutos.com" target="_blank">Ir a mi blog</a></li>
    </ul>
</nav>
```

Y hacemos referencia al archivo de CSS:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} Curso Flask | {% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
</head>
<body>
    <header>
        {% include 'navbar.html' %}
    </header>
    {% block content %}
    {% endblock %}
</body>
</html>
```

### Configurar páginas de error

Creamos nuevas funciones en el archivo main.py:

```python
@app.errorhandler(404)
def not_found(error):
    return render_template('404.html', error=error)


@app.errorhandler(500)
def server_error(error):
    return render_template('500.html', error=error)

```

Para el 404:

```html
{% extends 'base.html' %}
{% block title %}
  {{ super() }}
  404
{% endblock %}

{% block content %}
  <h1>Lo sentimos, no encontramos lo que buscabas</h1>
  <p>{{ error }}</p>
{% endblock %}
```

Para el 500:

```html
{% extends 'base.html' %}
{% block title %}
  {{ super() }}
  500
{% endblock %}

{% block content %}
  <h1>Lo sentimos, no tenemos servicio en estos momentos. Por favor intenta más tarde.</h1>
  <p>{{ error }}</p>
{% endblock %}
```

## Extensiones de Flask

### Flask Bootstrap

Usaremos [Bootstrap-flask](https://bootstrap-flask.readthedocs.io/en/stable/)

### Configuración de Flask

Para configurar el entorno en desarrollo hacemos: `export FLASK_ENV=development`

Podemos guardar una cookie y encriptarla con session (esto es más seguro).

```python
from flask import Flask, request, make_response, redirect, render_template, session
from flask_bootstrap import Bootstrap5

app = Flask(__name__)
bootstrap = Bootstrap5(app)

app.config['SECRET_KEY'] = 'SUPER SECRETO'  # ! mala práctica

todos = ['Comprar café', 'Hacer ejercicio', 'Grabar video']


@app.route('/')
def index():
    user_ip = request.remote_addr
    response = make_response(redirect('/hello'))
    session['user_ip'] = user_ip
    return response


@app.route('/hello')
def hello():
    user_ip = session.get('user_ip')
    context = {
        'user_ip': user_ip,
        'todos': todos
    }
    return render_template('hello.html', **context)


@app.errorhandler(404)
def not_found(error):
    return render_template('404.html', error=error)


@app.errorhandler(500)
def server_error(error):
    return render_template('500.html', error=error)

```

### Implementación de Boostrap-Flask y Flask-WTF

``{% import 'bootstrap/wtf.html' as wtf %}``

### Uso de método POST en Flask-WTF

Podemos indicar qué métodos se usarán en las rutas.

````python
from flask import Flask, request, make_response, redirect, render_template, session, url_for
from flask_bootstrap import Bootstrap
from flask_wtf import FlaskForm
from wtforms.fields import StringField, PasswordField, SubmitField
from wtforms.validators import DataRequired

app = Flask(__name__)
bootstrap = Bootstrap(app)

app.config['SECRET_KEY'] = 'SUPER SECRETO'


todos = ['Comprar cafe', 'Enviar solicitud de compra', 'Entregar video a productor ']


class LoginForm(FlaskForm):
    username = StringField('Nombre de usuario', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    submit = SubmitField('Enviar')


@app.errorhandler(404)
def not_found(error):
    return render_template('404.html', error=error)


@app.errorhandler(500)
def server_error(error):
    return render_template('500.html', error=error)


@app.route('/')
def index():
    user_ip = request.remote_addr

    response = make_response(redirect('/hello'))
    session['user_ip'] = user_ip

    return response


@app.route('/hello', methods=['GET', 'POST'])
def hello():
    user_ip = session.get('user_ip')
    login_form = LoginForm()
    username = session.get('username')

    context = {
        'user_ip': user_ip,
        'todos': todos,
        'login_form': login_form,
        'username': username
    }

    if login_form.validate_on_submit():
        username = login_form.username.data
        session['username'] = username

        return redirect(url_for('index'))

    return render_template('hello.html', **context)

````

Y mostramos la variable en el template

````html
{% extends 'base.html' %}
{% import 'macros.html' as macros %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block title %}
    {{ super() }}
    Bienvenido
{% endblock %}

{% block content %}
    {% if username %}
        <h1>Bienvenido {{ username | capitalize }}</h1>
    {% endif %}
    {% if user_ip %}
        <h3>Hello World Platzi, tu IP es {{ user_ip }}</h3>
    {% else %}
        <a href="{{ url_for('index') }}">Ir a inicio</a>
    {% endif %}

    <div class="container">
        <!--<form action="{{ url_for('hello')}}" method="POST">-->
            <!--{{ login_form.username }}-->
            <!--{{ login_form.username.label }}-->
        <!--</form>-->
        {{ wtf.quick_form(login_form) }}
    </div>

    <ul>
        {% for todo in todos %}
            {{ macros.render_todo(todo) }}
        {% endfor %}
    </ul>
{% endblock %}
````

### Desplegar Flashes (mensajes emergentes)

Importamos ``flash`` para mostrar alertas en Flask:

````python
from flask import Flask, request, make_response, redirect, render_template, session, url_for, flash
from flask_bootstrap import Bootstrap
from flask_wtf import FlaskForm
from wtforms.fields import StringField, PasswordField, SubmitField
from wtforms.validators import DataRequired

app = Flask(__name__)
bootstrap = Bootstrap(app)

app.config['SECRET_KEY'] = 'SUPER SECRETO'


todos = ['Comprar cafe', 'Enviar solicitud de compra', 'Entregar video a productor ']


class LoginForm(FlaskForm):
    username = StringField('Nombre de usuario', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    submit = SubmitField('Enviar')


@app.errorhandler(404)
def not_found(error):
    return render_template('404.html', error=error)


@app.errorhandler(500)
def server_error(error):
    return render_template('500.html', error=error)


@app.route('/')
def index():
    user_ip = request.remote_addr

    response = make_response(redirect('/hello'))
    session['user_ip'] = user_ip

    return response


@app.route('/hello', methods=['GET', 'POST'])
def hello():
    user_ip = session.get('user_ip')
    login_form = LoginForm()
    username = session.get('username')

    context = {
        'user_ip': user_ip,
        'todos': todos,
        'login_form': login_form,
        'username': username
    }

    if login_form.validate_on_submit():
        username = login_form.username.data
        session['username'] = username

        flash('Nombre de usuario registrado con éxito')

        return redirect(url_for('index'))

    return render_template('hello.html', **context)

````

Y mostramos la alerta (que se puede cerrar) en base.html:

````html
{% extends 'bootstrap/base.html' %}

{% block head %}
    {{ super() }}

    <title>
        {% block title %}Flask Platzi |{% endblock %}
    </title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/main.css')}}">
{% endblock %}

{% block body %}
    {% block navbar %}
        {% include 'navbar.html' %}
    {% endblock %}

    {% for message in get_flashed_messages() %}
        <div class="alert alert-success alert-dismissible">
            <button type="button" data-dismiss="alert" class="close">
                X
            </button>
            {{ message }}
        </div>
    {% endfor %}

    {% block content %}{% endblock %}
    {% block scripts %}
        {{ super() }}
    {% endblock %}
{% endblock %}
````

### Pruebas básicas con Flask-testing

Usaremos [Flask Testing](https://pythonhosted.org/Flask-Testing/)

Instalamos ``pip install flask-testing``

Importamos ``unittest``

Y hacemos esto en main.py:

````python
import unittest

@app.cli.command()
def test():
    tests = unittest.TestLoader().discover('tests')
    unittest.TextTestRunner().run(tests)
````

Creamos una carpeta 'tests'. Para correr los tests hacemos ``flask test``.

Creamos un archivo donde ejecutaremos pruebas, tiene que iniciar con 'test' el nombre del archivo.

````python
from flask_testing import TestCase
from flask import current_app, url_for

from main import app


class MainTest(TestCase):
    def create_app(self):
        app.config['TESTING'] = True
        app.config['WTF_CSRF_ENABLED'] = False  # CSRF Protection -> https://flask-wtf.readthedocs.io/en/0.15.x/csrf/
        return app

    def test_app_exists(self):
        self.assertIsNotNone(current_app)


    def test_app_in_test_mode(self):
        self.assertTrue(current_app.config['TESTING'])


    def test_index_redirects(self):
        response = self.client.get(url_for('index'))
        self.assertRedirects(response, url_for('hello'))


    def test_hello_get(self):
        response = self.client.get(url_for('hello'))
        self.assert200(response)


    def test_hello_post(self):
        fake_form = {
            'username': 'fake',
            'password': 'fake-password'
        }
        response = self.client.post(url_for('hello'), data=fake_form)
        self.assertRedirects(response, url_for('index'))

````

## Proyecto

### App Factory

Organizamos un poco mejor nuestros archivos.

### Uso de Blueprints

Un blueprit es como una pequeña aplicación de Flask, con rutas, vistas y templates. Debe ser importada dentro de una app de Flask para funcionar.

Usaremos [Signals](https://flask.palletsprojects.com/en/2.1.x/signals/) instalando [Blinker](https://pypi.org/project/blinker/)

### Blueprints 2

### Base de datos y App Engine con Flask

Flask no tiene un ORM definido por defecto. Podemos utilizar el que nosotros querramos. Existe [SQL Alchemy](https://www.sqlalchemy.org/).

[Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)

### Configuración de Google Cloud SDK

[Artículo en Platzi](https://platzi.com/clases/1540-flask/18834-configuracion-de-google-cloud-sdk/)

### Configuración de proyecto en Google Cloud Platform

Conectar con nuestro servidor local => `gcloud auth application-default login`

### Implementación de Firestore

Instalaremos [Flask Firebase Admin](https://pypi.org/project/flask-firebase-admin/) con ``pip install flask-firebase-admin``.

### Autenticación de usuarios: Login

Usaremos [Flask-Login](https://flask-login.readthedocs.io/en/latest/) que se instala con ``pip install flask-login``.

### Autenticación de usuarios: Logout

### Signup

### Agregar tareas

### Eliminar tareas

### Editar tareas

### Deploy a producción con App Engine

Usamos [Google App Engine](https://cloud.google.com/appengine?hl=es) para subir nuestro proyecto a producción. Hay que tomar en cuenta que nos pedirá una tarjeta bancaria.

## Examen

[] ¿Cuál es la función correcta para crear un link interno a una ruta específica?: url_for()
[] ¿Qué variable hay que crear en la terminal para activar el debugger y reloader?: ``FLASK_DEBUG=1``
[] ¿Cómo se llama el archivo de configuración de AppEngine?: app.yaml
[] ¿Para qué nos sirve un Blueprint?: Para modularizar la aplicación, son un patrón de rutas, funciones y templates que nos permiten crear secciones de la aplicación
[] ¿Cómo se llama el directorio donde Flask busca archivos estáticos por defecto?: static
[] ¿Para qué utilizamos @login_manager.user_loader?: Para proteger la ruta decorada y solo permitir acceso a usuarios autenticados 💢
[] ¿Cuál es el decorador para crear una función para manejar errores?: ``@app.errorhandler(error)``
[] ¿Para qué sirve Flask?: Todas
[] ¿Cómo debemos guardar un password del usuario?: Nunca en el texto original
[] ¿Qué debes conocer para comenzar con Flask?: Conocimientos básicos de Python, pip y virtualenv
[] ¿Cómo debemos cuidar o manejar nuestro SECRET_KEY de producción?: No debe estar disponible en nuestro repositorio
[] Nombre de la variable que Flask expone para acceder a la información de la petición del usuario: current_app 💢
[] ¿Qué es un flash?: Un mensaje que presentar información al usuario sobre la acción que acaba de realizar
[] ¿Con qué comando prendemos el servidor local?: ``flask run``
[] ¿Cuál es la sintaxis correcta para representar una variable?: ``{{ variable }}``
[] ¿Cuál es la sintaxis correcta para iniciar un bloque condicional?: ``{%``
[] Una aplicación web utiliza el internet y un __ para comunicarse con el servidor: Navegador Web
[] ¿Qué tipo de base de datos es Firestore?: No SQL Orientada a Documentos
[] Después de crear un nuevo Blueprint, ¿cómo lo integramos en la aplicación?: Llamamos la función ``app.register_blueprint()`` y pasamos nuestra instancia de Blueprint como parámetro
[] ¿Cuál es el template inicial que tenemos que extender en Bootstrap?: ``'bootstrap/base.html'``
[] Para desplegar una forma y encriptar la sesiones, debemos de declarar esta variable en app.config: SECRET_KEY
[] Flask-Login requiere la implementación de una clase UserModel con propiedades específicas: Verdadero
[] Nombre del método que tenemos que implementar en una nueva instancia de flask_testing.TestCase: ``create_app``
[] ¿Qué variable hay que declarar en la terminal para prender el servidor de Flask?: ``FLASK_APP=main.py``
[] ¿A qué nos referimos con microframework?: Un framework que no cuenta inicialmente con funcionalidades específicas, como ORM o autenticación
[] Sintaxis correcta para declarar una ruta dinámica "users" que recibe "user_id" como parámetro: ``/users/<user_id>``
[] ¿Cuál es el comando que agregamos después de instalar GCloud SDK?: ```gcloud```
[] ¿Con qué comando creamos una nueva instancia de Flask?: ``app = Flask(__name__)``
[] ¿Cuál es la variable que expone flask_wtf.FlaskForm para validar formas cuando son enviadas y qué tipo de variable es?: validate_on_submit, boolean
[] Variable que usamos para detectar si el usuario está firmado. Disponible en cualquier template: current_user.is_authenticated 