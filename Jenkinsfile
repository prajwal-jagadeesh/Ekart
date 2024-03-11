pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prajwal-jagadeesh/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('OWASP DC') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker') {
                       sh "docker build -t prajwaldevops01/ekart:latest -f docker/Dockerfile ."
                   }
               }
                
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy image prajwaldevops01/ekart:latest > trivy-report.txt"
            }
        }
        
        stage('Docker Push') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker') {
                       sh "docker push prajwaldevops01/ekart:latest"
                   }
               }
                
            }
        }
        
        stage('Deploy to EKS') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://BD6902D96B792731B9B2FA27F9FFFE73.gr7.ap-south-1.eks.amazonaws.com') {
                    sh "kubectl apply -f deploymentservice.yml -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
