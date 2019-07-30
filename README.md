# Utilizar Nexus y repositorio privado para deploy y "cache" de packages

https://blog.sonatype.com/using-nexus-3-as-your-repository-part-2-npm-packages

### Docker de Nexus

docker run -d -p 8081:8081 -p 8082:8082 -p 8083:8083 --name my-nexus sonatype/nexus3:3.0.0

Para generar una carpeta que mapee los volúmenes de Nexus a la computadora anfitriona: -v /opt/my-nexus-data:/nexus-data

Para acceder a la administración: admin/admin123

Luego de correr ese comando verificar que levante correctamente el servicio ingresando a la página: 
http://localhost:8081

Ingresar al área de administración

Dentro de la "tuerca" se pueden agregar respositorios, vamos a agregar 3 del tipo npm: 

- hosted: para nuestros packages (npm-private)
- proxy: para "cachear" los packages de npm (npm-proxy)
- group: para agrupar todos los otros registry (npm-group)

Se puede definir un "blob store" (y es recomendable) por cada uno de los repositorios, de manera que luego se encuentren en carpetas diferentes (dentro de /nexus-data)

# Configuración de npm

## Install packages 

Se podría configurar npm de forma global con "npm addUser" sin embargo para distribuir más fácilmente esta información se puede agregar un archivo ".npmrc" en la carpeta raíz del o los proyectos con el siguiente contenido:

registry=[URL-NPM-GROUP]
_auth=[HASH-USER]

Para los valores por defecto queda como lo siguiente:

registry=http://localhost:8081/repository/npm-group/
_auth=YWRtaW46YWRtaW4xMjM=

Si utilizamos otro usuario o contraseña se calcula el hash con el siguiente comando: 

echo -n 'myuser:mypassword' | openssl base64  // Ej: echo -n 'admin:admin123' | openssl base64

Se puede chequear que este funcionando 

Una vez que tenemos ese paso completo ya Nexus nos sirve como "cache" de respositorios. 

## Publish

Para completar el proceso, en toda máquina que necesitemos publicar nuestros componentes podremos agregar la siguiente configuración:

Dentro del ".npmrc"

email=any@email.com


Y dentro del package.json se debe configurar el registry del repository a donde publicaremos nuestro "package", con lo siguiente:

  "publishConfig": {
    "registry": "http://localhost:8081/repository/npm-private/"
  } 

Y cambiando esto: 

  "private": false,

  (de true a false)

Una vez hecho ese paso podemos hacer "yarn build" o "npm run build" y luego "npm publish". Deberíamos ver que entre los assets del respository "npm-private" aparece nuestro package disponible
