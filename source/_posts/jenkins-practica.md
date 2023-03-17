---
title: IC/DC con Jenkins
Categoría: Aplicaciones web
---

![Jenkins](/images/jenkins-logo.png)




# Práctica: IC/DC con Jenkins

## Ejercicio 1: Construcción de una imagen docker


```
pipeline {
    environment {
        IMAGEN = "evanticks/django_tutorial"
        LOGIN = 'USER_DOCKERHUB'
    }
    agent none
    stages {
        stage("Desarrollo") {
            agent {
                docker { image "python:3"
                args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'main',url:'https://github.com/Evanticks/jenkins-django.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'python3 manage.py test'
                    }
                }

            }
        }
        stage("Construccion") {
            agent any
            stages {
                stage('CloneAnfitrion') {
                    steps {
                        git branch:'main',url:'https://github.com/Evanticks/docker-django.git'
                    }
                }
                stage('BuildImage') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:latest"
                        }
                    }
                }
                stage('UploadImage') {
                    steps {
                        script {
                            docker.withRegistry( '', LOGIN ) {
                                newApp.push()
                            }
                        }
                    }
                }
                stage('RemoveImage') {
                    steps {
                        sh "docker rmi $IMAGEN:latest"
                    }
                }
            }
        }           
    }
    post {
        always {
            mail to: 'antonio@gonzalonazareno.org',
            subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
    }
}
```






Una captura de pantalla donde se vea la salida de un build que se ha ejecutado de manera correcta.

![Jenkins](/images/practica-jenkins-1.png)
![Jenkins](/images/practica-jenkins-3.png)
Una captura de pantalla de tu cuenta de Docker Hub donde se vea la imagen subida de último build.

![Jenkins](/images/practica-jenkins-6.png)

Introduce un fallo en el Dockerfile y muestra la salida del build donde se produce el error.

![Jenkins](/images/practica-jenkins-1-1.png)

![Jenkins](/images/practica-jenkins-2-2.png)

Entrega la URL del repositorio para ver el Jenkinsfile.

https://github.com/Evanticks/jenkins-django

Pantallazo con el correo que has recibido de la ejecución del pipeline.

![Jenkins](/images/practica-jenkins-4.png)


## Ejercicio 2: Despliegue de la aplicación


```
pipeline {
    environment {
        IMAGEN = "evanticks/django_tutorial"
        LOGIN = 'USER_DOCKERHUB'
    }
    agent none
    stages {
        stage("Desarrollo") {
            agent {
                docker { image "python:3"
                args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'main',url:'https://github.com/Evanticks/jenkins-django.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'python3 manage.py test'
                    }
                }

            }
        }
        stage("Construccion") {
            agent any
            stages {
                stage('CloneAnfitrion') {
                    steps {
                        git branch:'main',url:'https://github.com/Evanticks/docker-django.git'
                    }
                }
                stage('BuildImage') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:latest"
                        }
                    }
                }
                stage('UploadImage') {
                    steps {
                        script {
                            docker.withRegistry( '', LOGIN ) {
                                newApp.push()
                            }
                        }
                    }
                }
                stage('RemoveImage') {
                    steps {
                        sh "docker rmi $IMAGEN:latest"
                    }
                }
                stage ('SSH') {
                    steps{
                        sshagent(credentials : ['SSH_ROOT']) {
                            sh 'ssh -o StrictHostKeyChecking=no shinji@evangelion.entrebytes.org docker rmi -f $IMAGEN:latest'
                            sh 'ssh -o StrictHostKeyChecking=no shinji@evangelion.entrebytes.org wget https://raw.githubusercontent.com/Evanticks/docker-django/main/docker-compose.yaml -O docker-compose.yaml'
                            sh 'ssh -o StrictHostKeyChecking=no shinji@evangelion.entrebytes.org docker-compose up -d --force-recreate'
                        }
                    }
                }
            }
        }           
    }
    post {
        always {
            mail to: 'antonio@antonio.gonzalonazareno.org',
            subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
    }
}

```



El contenido del fichero Jenkinsfile.

Está arriba.

Las credenciales que has guardado en Jenkins.

![Jenkins](/images/practica-jenkins-5.png)

Demuestra al profesor como se realiza la IC/DC completo.