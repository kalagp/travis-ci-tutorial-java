pipeline {
    agent {
        label 'maven-builder'
    }
    tools {
        maven 'linux-maven-3.3.9'
        jdk 'linux-jdk1.8.0_102'
    }
    stages {
        stage('Verify') {
            steps {
                sh "mvn compile"
            }
        }
    }
}
