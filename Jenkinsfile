pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build docker image') {
            steps {
                app = docker.build('anh3h/train-schedule')
                app.inside {
                    sh 'echo $(curl localhost:8080)'
                }
            }
        }
        stage('Push to docker registry') {
            docker.withRegistry('http://registry.hub.docker.com', 'docker_key') {
                app.push("${env.BUILD_NUMBER}")
                app.push('latest')
            }
        }
    }
}
