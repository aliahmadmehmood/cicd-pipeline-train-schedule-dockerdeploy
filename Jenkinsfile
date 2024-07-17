pipeline {
    agent any
    environment {
        JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.el7_9.x86_64"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }
    }
    post {
        failure {
            script {
                // Notify GitHub of build failure
                def commitStatus = [
                    context: 'Jenkins Build',
                    description: 'Build failed',
                    state: 'failure'
                ]
                githubNotify(commitStatus)
            }
        }
    }
}
