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
        stage('BuildImage'){
            when {
                branch 'master'
            }
            steps{
                script {
                    app = docker.build("whitenoise1213/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                }
                }
                
                milestone(1)
            }
      
        }
        stage('PushToRegistry') {
            when {
                branch 'master'
            }
            steps{
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
                milestone(2)
            }
        }
        stage('DeployToProduction') {
            when {
                 branch 'master'
            }
            steps {
                input 'Deploy to Production'
                 milestone(3)
                withCredentials ([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker pull whitenoise1213/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker stop train-schedule\""
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker rm train-schedule\""
                        }  catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker run --restart always --name train-schedule -p 8080:8080 -d <DOCKER_HUB_USERNAME>/train-schedule:${env.BUILD_NUMBER}\""
            }
        }
    }

 }
    }}
