# Manual de instalacion de la aplicacion web


## Decisiones de proyecto
|Elemento|Decision|Version|Justificacion|
|--------|-------|--------|-------------|
|Servidor web|Apache|2|Sencillo de usar, popular|
|Base de datos|MYSQL|8|Experiencia previa, popular|
|Lenguaje de servidor|Python|3|Muy interesante para ASIR, uso extendido|
|Framework|Flask|3|Sencillo de usar, pensado expecificamente para web/formularios, sesiones|
|Control de versiones|Git|2|Muy extendido|
|Documentacion|Markdown|-|Muy utilizado con git|

## Que hace un servidor web
Recibe solicitudes http de clientes y responde con la pagina web solicitada al navegador

## Proceso de INSTALACION / puesta en marcha

1. Actualizar el sistema
```bash
sudo apt update 
sudo apt upgrade 
```
2. Instalar git
```bash
sudo apt install git 
```
3. Instalar VS Code + Plugins
```
MarkDown all in one 
Python
```
4. Instalar git gui
` sudo apt install git-gui `
. Instalar apache2
` sudo apt install apache2 `
5. Cambiar permisos carpeta /var/www/html
` sudo chown -R $USER:$USER /var/www/html`
`sudo chmod -R u=rwX,go=rX /var/www/html`
6. Archivo apache
![Alt Text](FOTOS/conf.archivo.apache.png)
7. Configurar el archivo creado como default
   
    ` sudo a2dissite 000-default.conf  `

    ` sudo a2ensite incidencias.iago.ies.teis `  

    ` systemctl reload apache2 `
8. Instalar mysql server
` sudo apt install mysql-server `
9. Configurar sql
```sql
create database incidencias;
create user 'incidencias@localhost' identified by 'incidencias';
flush privileges;
grant all privileges on incidencias.* to 'incidencias'@'localhost';
select user, host from mysql.user;
-- Crear tabla
create table registro (
    id int auto_increment primary key,
    aula varchar(30),
    descripcion text,
    usuario varchar(20),
    estado varchar(30)
     );
-- Introducir datos
insert into registro (aula, descripcion, usuario, estado) values ('Taller1', 'Pc 24 no arranca', 'ifpereira', 'ABIERTA'),('Taller1','Cae monitor','ifpereira','ABIERTA');
```
10. Instalar paquetes necesarios de python y activar (entorno virtual)
```bash
sudo apt install python3 python3-pip python3-venv ý
python3 -m venv venv
source venv/bin/activate
```
11. Instalar flask, conector de bases de datos, comprobar y guardar las dependencias
```bash
pip install flask
pip install mysql-connector-python
pip list
pip freeze > requirements.txt
```

# Manual instalacion aplicacion
## Configurar git hub
1. Crear repositorio local
```bash
git init
git add .
git commit -m "Añadido base de datos, con usuario ty tabla de registro de incidencias e instrucciones de git/github"
```
2. Crear cuenta github, crear repositorio en github
3. Conectar repositorio local con remoto
```bash
git remote add origin url-repositorio
git branch -M main
git push -u origin main
```
4. Instalar el proyecto en otro equipo 
```bash
git clone url-repositorio 
```
5. Descargar cambios hechos en otro equipo
```bash
git pull
```

## Rutina de trabajo con Flask (venv)

Al empezar:
```bash
cd /var/www/incidencias.iago.ies.teis
source venv/bin/activate
python app.py # Lanzar app
```
Al terminar
```bash
# ctrl+c parar app
desactivate # salir del entorno
```

## Nuestra primera aplicacion Python/Flask
   
1. Creamos un fichero app.py
``` python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def inicio():
    return "<h1>Incidencias IES Teis</h1>"

if __name__ == "__main__":
    app.run(debug=True)
```
2. Ejecutamos
```bash
python3 app.py
```

- Esto crea un servidor web alternativo levantado en localhost en el puerto 5000. Se podria poner apache de intermediario utilizando un proxy
- Habria que modificar etc/apache2/sites-available/incidencias.iago.ies.teis.conf añadiendo esto dentro del VirtualHost:
```
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
```
 
- Y activar los modulos
```bash
sudo a2enmod proxy
sudo a2enmod proxy_http 
sudo systemctl restart apache2
```
1. Comprobamos abriendo http://incidencias.iago.ies.teis:5000

## Migracion del formulario a Python/Flask

1. Creamos una carpeta templates y movemos ahi nuestro index.html
2. Modificamos app.py:
``` python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def inicio():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```
3. Comprobamos abriendo http://incidencias.iago.ies.teis:5000  Hemos conseguido que ahora el formulario lo devuelva Flask

## Recibir los datos del formulario
1. Modificamos el archivo .py para recibir los datos del formulario
``` python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route("/")
def inicio():from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def inicio():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
    return render_template("index.html")

@app.route("/incidencia", methods=["POST"])
def crear_incidencia():

    aula = request.form["aula"]
    usuario = request.form["usuario"]
    descripcion = request.form["descripcion"]

    print("Aula:" + aula)
    print("Usuario:" + usuario)
    print("Descripcion:" + descripcion)

    return "Incidencia recibida"

if __name__ == "__main__":
    app.run(debug=True)
```

2. Modificar el index.html (cambiar el form)
```html
    <form action="/incidencia" method="post">
      <label for="aula">Aula:</label>
      <input type="text" id="aula" name="aula">
      
      <br><br>

      <label for="usuario">Usuario</label>
      <input type="text" id="usuario" name="usuario">

      <br><br>

      <label for="descripcion">Descripción:</label>
      <textarea id="descripcion" name="descripcion"></textarea>

      <br><br>

      <input type="submit" value="Enviar">
    </form>
```
3. Probamos a rellenar el formulario y fijarnos en que los datos salen en la terminal y nos devuelve incidencia recibida

## Introducir los datos en la BD