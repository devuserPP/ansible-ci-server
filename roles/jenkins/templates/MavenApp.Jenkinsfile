pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                ranches: [[name: '*/master']], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [], 
                submoduleCfg: [], 
                userRemoteConfigs: [[url: 'https://github.com/devuserPP/simple-java-maven-app']]])
                }

                        }
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
                sh './jenkins/scripts/deliver.sh' 
            }
        }
    }
}



