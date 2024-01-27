pipeline {
    agent any
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scaner'
        IMAGE_TAG= "$BUILD_NUMBER"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '9deec83f-f312-44a0-aa36-226885ab73f1', poll: false, url: 'https://github.com/DumindaAWS/shopping-cart.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'dp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-scaner'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=web-cicd \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=web-cicd '''
               }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build & Push to dockerhub') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'e8846a3d-b55e-4f62-bab1-5feca7051966', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart4 -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart4 duminda/shopping-cart4:$IMAGE_TAG"
                        sh "docker push duminda/shopping-cart4:$IMAGE_TAG"
                    }
                }
            }
        }
        
        
    }
}
