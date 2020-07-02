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
                script {
                    app = docker.build('anh3h/train-schedule')
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push to docker registry') {
            steps {
                script {
                    docker.withRegistry('http://registry.hub.docker.com', 'docker_key') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push('latest')
                    }
                }
            }
        }
        stage('Proceed deploy to production') {
            steps {
                input 'Do you want to proceed with the deployment to production?'
            }
        }
        stage('Deploy to production') {
            steps {
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    script {
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull anh3h/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$PASSWORD' -V ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$PASSWORD' -V ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo 'Caught error: $err'
                        }
                        sh "sshpass -p '$PASSWORD' -V ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d anh3h/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
