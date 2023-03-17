---
title: Introducción a Jenkins
Categoría: Aplicaciones web
---

![Jenkins](/images/jenkins-intro.png)

# Introducción a Jenkins

## Taller 1: Corrector ortográfico de documentos markdown (test)

La URL del tu repositorio GitHub.

https://github.com/Evanticks/ic-diccionario

El contenido de la tu fichero Jenkinfile.

```
pipeline {
    agent {
        docker { image 'debian'
        args '-u root:root'
        }
    }
    stages {
        stage('Clone') {
            steps {
                git branch:'master',url:'https://github.com/Evanticks/ic-diccionario.git'
            }
        }
        stage('Install') {
            steps {
                sh 'apt-get update && apt-get install -y aspell-es ' 
            }
        }
        stage('Test')
        {
            steps {
                sh '''
                export LC_ALL=C.UTF-8
                OUTPUT=`cat doc/*.md | aspell list -d es -p ./.aspell.es.pws`; if [ -n "$OUTPUT" ]; then echo $OUTPUT; exit 1; fi'''
            }
        }
    }
    post {
         always {
          mail to: 'root@antonio.gonzalonazareno.org',
          subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
          body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
      }
}
```


Una captura de pantalla donde se vea la configuración del disparador del pipeline.


![Jenkins](/images/jenkins-taller-1-1.png)


Una captura de un correo electrónico recibido sin ningún error, y otro con algún error en al ejecución del pipeline.

![Jenkins](/images/jenkins-taller-1-2.png)

Aquí podemos ver que al volver a construir el pipeline, nos ha enviado un correo electrónico indicando que el pipeline ha fallado.

![Jenkins](/images/jenkins-taller-1-3.png)


Y eso es debido a que he puesto a drede un error ortográfico:

![Jenkins](/images/jenkins-taller-1-4.png)


## Taller 2: Comprobación de HTML5 válido y despliegue en surge.sh (test y deploy)

La URL del tu repositorio GitHub.

https://github.com/Evanticks/ic-html5

El contenido de la tu fichero Jenkinfile.

```
pipeline {
    environment {
        TOKEN = credentials('SURGE_TOKEN')
      }
    agent {
        docker { image 'josedom24/debian-npm'
        args '-u root:root'
        }
    }
    stages {
        stage('Clone') {
            steps {
                git branch:'master',url:'https://github.com/Evanticks/ic-html5.git'
            }
        }
        
        stage('Install surge')
        {
            steps {
                sh 'npm install -g surge'
            }
        }
        stage('Deploy')
        {
            steps{
                sh 'surge ./_build/ antoniomarchan.surge.sh --token $TOKEN'
            }
        }
        
    }
}
```

Captura de pantalla donde se vea donde has creado las credenciales necesarias.

![Jenkins](/images/jenkins-taller-2-1.png)

Explica la configuración necesaria y una prueba de funcionamiento para que se dispare el pipeline cuando hagamos un push al repositorio.


Primero exponemos el servicio de Jenkins a una url pública gracias a ngrok como explico en https://dit.gonzalonazareno.org/redmine/boards/16/topics/1016

Luego hacemos las credenciales para que Jenkins pueda publicar a través del token de surge.sh

Luego crearemos un webhook ingresando la url pública de ngrok.

Y ya con eso a la hora de hacer un push, Jenkins se encargará de construir el pipelinem, tardará 1 minuto.

![Jenkins](/images/jenkins-taller-2-2.gif)


## Taller 3: Integración continua de aplicación django (Test)


Una captura de pantalla donde se vea la salida de un build que se ha ejecutado de manera correcta.

![Jenkins](/images/jenkins-taller-3-3.png)

Modifica el código de la aplicación para que se produzca un fallo en el código. *Recuerda que para hacer fallar un test, no hay que tocar el fichero test.py. Los test no se pasan porque al modificar el código de la aplicación se dejan de cumplir las condiciones indicadas en los test. Recuerda no elegir en el que hemos visto en este taller:mensaje *“No polls are available” **.

![Jenkins](/images/jenkins-taller-3-4.png)


Una captura de pantalla donde se vea la salida de un build que se ha ejecutado con errores de testeo.


![Jenkins](/images/jenkins-taller-3-5.png)