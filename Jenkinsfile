pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("mohamedbenighil/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_cred') {
                        echo '******************************'
                        //echo "COMMIT ID is : ${GIT_COMMIT[0..7]}"
                        app.push("${env.GIT_COMMIT[0..7]}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production ?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'production_server_creds', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker pull mohamedbenighil/train-schedule:${env.GIT_COMMIT[0..7]}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker run --restart always --name train-schedule -p 8080:8080 -d mohamedbenighil/train-schedule:${env.GIT_COMMIT[0..7]}\""
                    }
                }
            }
        }      
    }
}
