pipeline {
    agent any
    stages {
        stage('Build') { 
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') {
            steps {
                script {
                    // Obtiene el mensaje del último commit
                    def commitMsg = sh(script: 'git log -1 --pretty=%%B', returnStdout: true).trim()
                    if (commitMsg.contains('DO_NOT_DELIVER')) {
                        echo "El commit contiene DO_NOT_DELIVER. Saltando Deliver."
                    } else {
                    // Construye la imagen Docker usando el Dockerfile
                    def pom = readMavenPom(file: 'pom.xml')
                    def pom_version = pom.version
                    sh "docker build --build-arg VERSION=${pom_version} -t my-app:${pom_version} ."

                    // Elimina el contenedor si ya existe
                    sh 'docker rm -f my-app-container || exit 0'


                    // Corre un contenedor basado en esa imagen y mapea el puerto 8080
                    sh "docker run -d --name my-app-container my-app:${pom_version}"
                    }
                }
            }
        }
    }
}
