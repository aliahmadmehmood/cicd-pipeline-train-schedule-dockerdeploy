pipeline {
    agent any
        environment {
        JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.el7_9.x86_64"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Verify Java Version') {
            steps {
                sh 'java -version'
                sh 'echo $JAVA_HOME'
                sh 'echo $PATH'
            }
        }
        stage('Print Environment Variables') {
            steps {
                sh 'env'
            }
        }
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.el7_9.x86_64'
                sh 'export PATH=$JAVA_HOME/bin:$PATH'
                sh './gradlew build --no-daemon --info'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("aliahmadmehmood/train-schedule")
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
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
    }   
}

stage ('DeployToProduction') {
    when {
        branch 'master'
    }
    steps {
        input 'Deploy to Production'
        milestone(1)
        withCredentials ([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
            script {
                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker pull aliahmadmehmood/train-schedule:${env.BUILD_NUMBER}\""
                try {
                   sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker stop train-schedule\""
                   sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker rm train-schedule\""
                } catch (err) {
                    echo: 'caught error: $err'
                }
                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker run --restart always --name train-schedule -p 8080:8080 -d <DOCKER_HUB_USERNAME>/train-schedule:${env.BUILD_NUMBER}\""
            }
        }
    }
}
