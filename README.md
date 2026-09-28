\# Práctica 5 – Despliegue de una Aplicación Web con Docker Compose



\## Servicios Convergentes



\### Descripción



En esta práctica se realiza el despliegue de una aplicación web completa utilizando \*\*Docker y Docker Compose\*\*.



La aplicación está compuesta por tres servicios principales:



\- \*\*MongoDB\*\*: base de datos.

\- \*\*Backend APPSC\*\*: API desarrollada con Node.js, Express y Mongoose.

\- \*\*Frontend FAPPSC\*\*: interfaz desarrollada con React y Vite.



Docker Compose permite iniciar los tres servicios de manera conjunta y establecer una red interna para que puedan comunicarse entre ellos.



\---



\# 1. Arquitectura de la aplicación



La arquitectura utilizada es la siguiente:



```text

Usuario / Navegador

&#x20;       |

&#x20;       | http://localhost:5173

&#x20;       v

+-----------------------+

|       Frontend        |

|     React + Vite      |

|       Puerto 5173     |

+-----------------------+

&#x20;       |

&#x20;       | /api

&#x20;       | Proxy Vite

&#x20;       v

+-----------------------+

|        Backend        |

|   Node.js + Express   |

|       Puerto 3000     |

+-----------------------+

&#x20;       |

&#x20;       | mongodb://mongo:27017

&#x20;       v

+-----------------------+

|       MongoDB         |

|       Puerto 27017    |

+-----------------------+

```



Los tres servicios se encuentran conectados mediante una red interna de Docker llamada:



```text

app\_network

```



Dentro de esta red los contenedores pueden comunicarse utilizando el nombre de cada servicio.



Por ejemplo:



```text

frontend -> backend:3000

backend  -> mongo:27017

```



\---



\# 2. Repositorios utilizados



\## Backend – APPSC



Repositorio:



```text

https://github.com/Isabella-55/convergentes-2.git

```



Contiene:



\- Backend Node.js.

\- Express.

\- Mongoose.

\- Autenticación mediante JWT.

\- Gestión de usuarios.

\- Gestión de notas.

\- Dockerfile del backend.



\---



\## Frontend – FAPPSC



Repositorio:



```text

https://github.com/Isabella-55/convergentes-front.git

```



Contiene:



\- React.

\- Vite.

\- Axios.

\- React Router.

\- Registro de usuarios.

\- Inicio de sesión.

\- Gestión de notas.

\- Dockerfile del frontend.



\---



\# 3. Requisitos previos



Antes de iniciar el despliegue se debe tener instalado:



\- Git.

\- Docker.

\- Docker Compose.



En Windows se recomienda utilizar \*\*Docker Desktop\*\* con WSL 2.



Para comprobar las instalaciones se pueden ejecutar los siguientes comandos:



```bash

git --version

docker --version

docker compose version

```



Ejemplo:



```text

git version 2.x

Docker version 29.x

Docker Compose version v5.x

```



También se puede comprobar que Docker funciona correctamente mediante:



```bash

docker run hello-world

```



Si la instalación funciona debe aparecer:



```text

Hello from Docker!

```



\---



\# 4. Crear la carpeta del proyecto



Crear una carpeta donde se alojarán el backend, frontend y archivo Docker Compose.



Ejemplo:



```bash

mkdir Proyecto

cd Proyecto

```



La estructura inicial será:



```text

Proyecto/

```



\---



\# 5. Clonar el backend



Dentro de la carpeta `Proyecto` ejecutar:



```bash

git clone https://github.com/Isabella-55/convergentes-2.git APPSC

```



Esto descargará el backend dentro de una carpeta llamada:



```text

APPSC

```



\---



\# 6. Clonar el frontend



Desde la misma carpeta `Proyecto` ejecutar:



```bash

git clone https://github.com/Isabella-55/convergentes-front.git FAPPSC

```



Esto descargará el frontend dentro de:



```text

FAPPSC

```



Después de clonar ambos repositorios la estructura debe ser:



```text

Proyecto/

├── APPSC/

└── FAPPSC/

```



\---



\# 7. Dockerfile del backend



El repositorio del backend ya incluye el archivo:



```text

APPSC/Dockerfile

```



Su configuración es:



```dockerfile

FROM node:18



WORKDIR /usr/src/app



COPY package\*.json ./



RUN npm install



COPY . .



EXPOSE 3000



CMD \["node", "src/index.js"]

```



Este archivo realiza las siguientes acciones:



1\. Utiliza Node.js 18.

2\. Crea `/usr/src/app` como directorio de trabajo.

3\. Copia los archivos de dependencias.

4\. Ejecuta `npm install`.

5\. Copia el código del backend.

6\. Expone el puerto `3000`.

7\. Ejecuta el servidor mediante:



```bash

node src/index.js

```



\---



\# 8. Configuración de MongoDB en el backend



La conexión del backend utiliza la variable de entorno:



```text

MONGO\_URI

```



El archivo:



```text

APPSC/src/db.js

```



permite obtener la dirección de MongoDB mediante:



```javascript

process.env.MONGO\_URI

```



Dentro de Docker Compose la conexión utilizada será:



```text

mongodb://admin:password123@mongo:27017/appdb?authSource=admin

```



Es importante utilizar:



```text

mongo

```



y no:



```text

localhost

```



porque `mongo` corresponde al nombre del servicio de MongoDB dentro de la red de Docker.



\---



\# 9. Dockerfile del frontend



El frontend contiene:



```text

FAPPSC/Dockerfile

```



con la siguiente configuración:



```dockerfile

FROM node:18



WORKDIR /usr/src/app



COPY package\*.json ./



RUN npm install



COPY . .



EXPOSE 5173



CMD \["npm", "run", "dev", "--", "--host"]

```



Esto permite iniciar Vite dentro del contenedor y hacerlo accesible desde fuera de Docker.



\---



\# 10. Comunicación frontend-backend



El frontend utiliza Axios con la siguiente URL base:



```javascript

baseURL: "/api"

```



Las peticiones realizadas por React son de este tipo:



```text

/api/login

/api/register

/api/getNotes

/api/getNote

```



Vite utiliza un proxy para enviar estas solicitudes hacia el backend.



En:



```text

FAPPSC/vite.config.js

```



se encuentra:



```javascript

proxy: {

&#x20; "/api": {

&#x20;   target: "http://backend:3000",

&#x20;   changeOrigin: true,

&#x20;   secure: false,

&#x20; },

},

```



El nombre:



```text

backend

```



corresponde al servicio definido dentro de Docker Compose.



\---



\# 11. Crear el archivo docker-compose.yml



En la carpeta raíz `Proyecto` se debe crear:



```text

docker-compose.yml

```



El archivo utilizado es:



```yaml

services:

&#x20; mongo:

&#x20;   image: mongo:6

&#x20;   container\_name: mongo\_container

&#x20;   restart: always

&#x20;   environment:

&#x20;     MONGO\_INITDB\_ROOT\_USERNAME: admin

&#x20;     MONGO\_INITDB\_ROOT\_PASSWORD: password123

&#x20;   ports:

&#x20;     - "27017:27017"

&#x20;   networks:

&#x20;     - app\_network



&#x20; backend:

&#x20;   build: ./APPSC

&#x20;   container\_name: backend\_container

&#x20;   restart: always

&#x20;   command: \["node", "src/index.js"]

&#x20;   volumes:

&#x20;     - ./APPSC:/usr/src/app

&#x20;     - /usr/src/app/node\_modules

&#x20;   working\_dir: /usr/src/app

&#x20;   environment:

&#x20;     MONGO\_URI: "mongodb://admin:password123@mongo:27017/appdb?authSource=admin"

&#x20;   ports:

&#x20;     - "3000:3000"

&#x20;   depends\_on:

&#x20;     - mongo

&#x20;   networks:

&#x20;     - app\_network



&#x20; frontend:

&#x20;   build: ./FAPPSC

&#x20;   container\_name: frontend\_container

&#x20;   restart: always

&#x20;   command: \["npm", "run", "dev", "--", "--host"]

&#x20;   volumes:

&#x20;     - ./FAPPSC:/usr/src/app

&#x20;     - /usr/src/app/node\_modules

&#x20;   working\_dir: /usr/src/app

&#x20;   ports:

&#x20;     - "5173:5173"

&#x20;   depends\_on:

&#x20;     - backend

&#x20;   networks:

&#x20;     - app\_network



networks:

&#x20; app\_network:

&#x20;   driver: bridge

```



La estructura final del proyecto debe ser:



```text

Proyecto/

│

├── docker-compose.yml

│

├── APPSC/

│   ├── Dockerfile

│   ├── package.json

│   ├── package-lock.json

│   └── src/

│

└── FAPPSC/

&#x20;   ├── Dockerfile

&#x20;   ├── package.json

&#x20;   ├── package-lock.json

&#x20;   ├── vite.config.js

&#x20;   └── src/

```



\---



\# 12. Verificar el archivo Docker Compose



Antes de iniciar los contenedores se recomienda verificar la configuración mediante:



```bash

docker compose config

```



Si la configuración es correcta Docker mostrará la configuración procesada sin presentar errores.



\---



\# 13. Verificar los puertos



Los servicios utilizan:



| Servicio | Puerto |

|----------|--------|

| MongoDB | 27017 |

| Backend | 3000 |

| Frontend | 5173 |



En Windows se puede verificar si estos puertos están ocupados mediante:



```cmd

netstat -ano | findstr :27017

netstat -ano | findstr :3000

netstat -ano | findstr :5173

```



Si no aparece ninguna salida, el puerto está disponible.



\---



\# 14. Posible conflicto con MongoDB instalado en Windows



Si MongoDB se encuentra instalado directamente en Windows, puede estar utilizando el puerto:



```text

27017

```



Esto impediría que Docker publique el mismo puerto.



Puede verificarse mediante:



```cmd

netstat -ano | findstr :27017

```



Si MongoDB está funcionando como servicio de Windows, puede detenerse temporalmente desde una terminal ejecutada como administrador:



```cmd

net stop MongoDB

```



Luego se puede comprobar nuevamente:



```cmd

netstat -ano | findstr :27017

```



Cuando se quiera volver a iniciar MongoDB local:



```cmd

net start MongoDB

```



\---



\# 15. Construir e iniciar la aplicación



Desde la carpeta donde se encuentra:



```text

docker-compose.yml

```



ejecutar:



```bash

docker compose up --build -d

```



La opción:



```text

\--build

```



indica que Docker debe construir las imágenes del backend y frontend.



La opción:



```text

\-d

```



ejecuta los contenedores en segundo plano.



Durante el primer despliegue Docker descargará las imágenes necesarias e instalará las dependencias.



\---



\# 16. Verificar los contenedores



Ejecutar:



```bash

docker ps

```



Deben aparecer tres contenedores:



```text

mongo\_container

backend\_container

frontend\_container

```



Con puertos similares a:



```text

mongo\_container       27017:27017

backend\_container      3000:3000

frontend\_container     5173:5173

```



También se puede utilizar:



```bash

docker compose ps

```



\---



\# 17. Verificar el backend



Para consultar los registros del backend:



```bash

docker logs backend\_container

```



Si el servicio funciona correctamente se debe observar:



```text

Server on port 3000

DB is connected

```



Esto confirma que:



1\. Node.js se encuentra funcionando.

2\. El servidor escucha en el puerto 3000.

3\. El backend logró conectarse al contenedor MongoDB.



Para visualizar solamente los últimos registros:



```bash

docker logs --tail 30 backend\_container

```



\---



\# 18. Verificar el frontend



Ejecutar:



```bash

docker logs frontend\_container

```



Debe aparecer una salida similar a:



```text

VITE ready



Local:   http://localhost:5173/

Network: http://172.x.x.x:5173/

```



Esto confirma que el servidor Vite se encuentra funcionando dentro del contenedor.



\---



\# 19. Acceder a la aplicación



Abrir un navegador web e ingresar:



```text

http://localhost:5173

```



La aplicación permitirá acceder a la interfaz desarrollada en React.



Si el servicio se ejecuta en otro equipo dentro de una red, también puede utilizarse la dirección IP de la máquina:



```text

http://IP\_MAQUINA:5173

```



\---



\# 20. Prueba funcional



Para comprobar el funcionamiento completo de la aplicación se puede realizar el siguiente procedimiento:



1\. Abrir:



```text

http://localhost:5173

```



2\. Registrar un usuario nuevo.



3\. Iniciar sesión.



4\. Acceder a la sección de notas.



5\. Crear una nota indicando título y descripción.



6\. Guardar la nota.



7\. Verificar que la nota aparezca en la aplicación.



Este proceso permite comprobar el recorrido:



```text

Frontend

&#x20;  |

&#x20;  v

Backend

&#x20;  |

&#x20;  v

MongoDB

```



Si la nota puede crearse y visualizarse correctamente, se confirma que los tres servicios se encuentran comunicados.



\---



\# 21. Autenticación



Las rutas protegidas de la aplicación utilizan JWT.



El frontend puede enviar el token mediante:



```text

Authorization: Bearer TOKEN

```



El backend valida el token antes de permitir operaciones protegidas, por ejemplo:



```text

GET /api/getNotes

POST /api/getNote

```



Cuando la autenticación es correcta estas peticiones deben responder correctamente.



Los registros del backend permiten comprobarlo:



```bash

docker logs --tail 30 backend\_container

```



Ejemplo:



```text

POST /api/login 200

GET /api/getNotes 200

POST /api/getNote 200

```



\---



\# 22. Detener la aplicación



Para detener y eliminar los contenedores creados por Docker Compose ejecutar:



```bash

docker compose down

```



Este comando detiene:



```text

frontend\_container

backend\_container

mongo\_container

```



\---



\# 23. Iniciar nuevamente la aplicación



Si las imágenes ya se encuentran construidas no es necesario utilizar nuevamente `--build`.



Se puede iniciar mediante:



```bash

docker compose up -d

```



Y comprobar:



```bash

docker ps

```



\---



\# 24. Reconstruir después de cambios



Si se realizan modificaciones importantes en los Dockerfile, dependencias o configuración puede reconstruirse el proyecto mediante:



```bash

docker compose down

docker compose up --build -d

```



\---



\# 25. Reiniciar un contenedor individual



Por ejemplo, para reiniciar únicamente el backend:



```bash

docker restart backend\_container

```



Para el frontend:



```bash

docker restart frontend\_container

```



Para MongoDB:



```bash

docker restart mongo\_container

```



\---



\# 26. Consultar registros



Backend:



```bash

docker logs backend\_container

```



Frontend:



```bash

docker logs frontend\_container

```



MongoDB:



```bash

docker logs mongo\_container

```



Para observar los registros en tiempo real:



```bash

docker logs -f backend\_container

```



Para salir de esta vista utilizar:



```text

Ctrl + C

```



\---



\# 27. Comandos principales



\## Construir e iniciar



```bash

docker compose up --build -d

```



\## Ver contenedores



```bash

docker ps

```



\## Ver estado del proyecto



```bash

docker compose ps

```



\## Ver logs del backend



```bash

docker logs backend\_container

```



\## Ver logs del frontend



```bash

docker logs frontend\_container

```



\## Ver logs de MongoDB



```bash

docker logs mongo\_container

```



\## Reiniciar backend



```bash

docker restart backend\_container

```



\## Detener el proyecto



```bash

docker compose down

```



\## Volver a iniciar



```bash

docker compose up -d

```



\---



\# 28. Solución de problemas



\## Error: puerto 27017 ocupado



Comprobar:



```cmd

netstat -ano | findstr :27017

```



Si corresponde al MongoDB instalado en Windows:



```cmd

net stop MongoDB

```



Después iniciar nuevamente Docker Compose:



```bash

docker compose up -d

```



\---



\## Error 401 al guardar o consultar notas



Consultar los logs:



```bash

docker logs --tail 30 backend\_container

```



Un error:



```text

POST /api/getNote 401

```



indica que el backend no está recibiendo o validando correctamente el token de autenticación.



La configuración actual permite recibir el token mediante:



```text

Authorization: Bearer TOKEN

```



Después de modificar el backend puede reiniciarse mediante:



```bash

docker restart backend\_container

```



\---



\## El frontend no inicia



Consultar:



```bash

docker logs frontend\_container

```



También comprobar:



```bash

docker ps

```



El contenedor:



```text

frontend\_container

```



debe encontrarse en estado:



```text

Up

```



\---



\## El backend no conecta con MongoDB



Ejecutar:



```bash

docker logs backend\_container

```



La conexión correcta debe mostrar:



```text

DB is connected

```



También se puede comprobar que MongoDB se encuentre iniciado:



```bash

docker ps

```



Debe aparecer:



```text

mongo\_container

```



El backend debe utilizar:



```text

mongo:27017

```



como dirección interna y no:



```text

localhost:27017

```



\---



\# 29. Resultado esperado



Al finalizar el despliegue deben encontrarse funcionando simultáneamente:



```text

MongoDB   -> puerto 27017

Backend   -> puerto 3000

Frontend  -> puerto 5173

```



El acceso principal se realiza mediante:



```text

http://localhost:5173

```



El usuario debe poder:



\- Registrarse.

\- Iniciar sesión.

\- Acceder a las rutas protegidas.

\- Crear notas.

\- Consultar notas.



Con esto se valida la comunicación completa entre:



```text

React/Vite -> Node.js/Express -> MongoDB

```



utilizando Docker Compose.



\---



\# 30. Tecnologías utilizadas



\- Docker

\- Docker Compose

\- MongoDB 6

\- Node.js 18

\- Express

\- Mongoose

\- React

\- Vite

\- Axios

\- JWT

\- Git

\- GitHub



\---



\## Autor



\*\*Isabella Ordoñez Ortega\*\*



Universidad del Cauca  

Facultad de Ingeniería Electrónica y Telecomunicaciones  

Servicios Convergentes  

2026

