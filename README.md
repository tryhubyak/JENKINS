# JENKINS
## Despliegue de aplicaciones web 2ºDAW 

Practica 6, configuración de Jenkins y pipeline para despliegue de una web.

# Proceso de instalación 
Agregamos dockerfile y docker-compose a la carpeta con la que vamos a trabajar:
![image](https://github.com/tryhubyak/jenkins/assets/145651101/46bbf5b9-a5dc-420f-8fe3-e3f58b83cad0)

ejecutamos el dockerfile:
```
docker build -t jenkins .
```
![image](https://github.com/tryhubyak/jenkins/assets/145651101/ffb7189f-0b70-4697-bf8e-901700499e5c)
```
docker-compose up -d
```
![image](https://github.com/tryhubyak/jenkins/assets/145651101/ab925e60-ff02-4f2f-8737-e0acd9ed8d58)

Ahora se accede a localhost:8080

![image](https://github.com/tryhubyak/jenkins/assets/145651101/1d117717-fba9-4d1b-956c-a641cf2360f6)

buscamos la contraseña accediendo al contenedor y buscando la ruta que muestra debajo de Unlock Jenkins

![image](https://github.com/tryhubyak/jenkins/assets/145651101/a4139498-2e68-41b2-9f52-2dc073ea8dd7)

ahora una vez dentro le damos a install suggested plugins y esperamos

![image](https://github.com/tryhubyak/jenkins/assets/145651101/d3e91fe4-c598-4d15-b9c4-7465e2601d01)

Una vez instalado todo configuramos el usuario

![image](https://github.com/tryhubyak/jenkins/assets/145651101/d6836870-6039-49bb-ab4a-a7c2b3d15c73)

![image](https://github.com/tryhubyak/jenkins/assets/145651101/5aa1e46a-01c0-40c6-83a1-883a804f13e7)

![image](https://github.com/tryhubyak/jenkins/assets/145651101/84bd1b0f-a9c2-4d78-b566-d0f11aed28f5)

Con eso terminamos la configuración y podemos empezar a usar jenkins

![image](https://github.com/tryhubyak/jenkins/assets/145651101/2437c372-c409-4442-b2c7-77e6fdbee287)

## Pipeline
Creamos el proyecto pipeline:

![image](https://github.com/tryhubyak/jenkins/assets/145651101/4deb97e1-f7c3-4626-8511-5e07c3e14f00)

Creamos el Dockerfile con la imagen de apache

![image](https://github.com/tryhubyak/jenkins/assets/145651101/c746ddeb-33d5-420d-a77c-1d680ca27270)

Añadimos el script con las stages que tienen que pasar para que se despliegue nuestro servicio

```
pipeline {
    agent any
    stages {
        stage('Clean') {
            steps {
                sh 'rm -rf *'
            }
        }
    
        stage('Clone') {
            steps {
                sh 'git clone https://github.com/tryhubyak/jenkins'
            }
        }
        
        stage('Build') {
            steps {
                sh 'cd jenkins/Jenkins && docker build -t tryhubyak/practica6:latest .'
            }
        }
        
        stage('Login') {
            steps {
                sh 'cd jenkins/Jenkins && cat password.txt | docker login --username tryhubyak --password-stdin'
            }
        }
        
        stage('Push') {
            steps {
                sh 'docker push tryhubyak/practica6:latest'
            }
        }
        
        stage('Deploy') {
            steps {
                 sh 'cd jenkins/Jenkins && ssh -i sever-veronika.pem ubuntu@ec2-3-248-223-119.eu-west-1.compute.amazonaws.com sh deploy.sh'
            }}
}}
```
En el intento 39 lo único que falla es el deploy.

![image](https://github.com/tryhubyak/jenkins/assets/145651101/58b5f08e-dc9d-45ae-9f1b-38d8262ad803)

# Problemas durante la instalación y resoluciones aplicadas
- Todos los problemas los fui viendo por los logs en los intentos de construir una vez ingresado el pipeline.
- Primero me dio error al poner mal la ruta en el 3 paso, la cambié a la correcta y este fallo fue solucionado. Luego me dio error de denegación de permisos en el mismo paso, dicho error se soluciono con sudo chmod 666 /var/run/docker.sock.
- También el archivo de password.txt no estaba creado, había que crearlo y añadirlo. 
- Tuve un error relacionado con el archivo pem, lo sustitui y ya no tuvo problema.
- Este es el ultimo log, en el deploy

![image](https://github.com/tryhubyak/jenkins/assets/145651101/57bac880-ea5d-4e28-97b3-9bc7bef02c09)


