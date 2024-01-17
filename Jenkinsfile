pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'ekart', url: 'https://github.com/srinivasreddyjangam/JENKINS-PROJECTS.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('SonarQube Analysis') {
            steps {
               withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART  -Dsonar.projectName=EKART \
                -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        } 
        stage('Build') {
            steps {
               sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Deploy To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Docker Build and tag image') {
            steps {
            // This step should not normally be used in your script. Consult the inline help for details.
            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
            sh "docker build -t srinivasjangam/ekart:latest -f docker/Dockerfile ."
                  }
            }
        }
        stage('Trivy scan') {
            steps {
                sh "trivy image srinivasjangam/ekart:latest > trivy-report.txt "
                }
         }
        stage('Docker push image') {
            steps {
            // This step should not normally be used in your script. Consult the inline help for details.
            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
            sh "docker push srinivasjangam/ekart:latest"
            }
            }
        }
        stage('Kubernetes deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.83.75:6443') {
                sh "kubectl apply -f deploymentservice.yml -n webapps"
                sh "kubectl get svc -n webapps"
                    }
                }
         }
    }
}
