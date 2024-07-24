pipeline{

    agent any 

    tools{
        maven 'maven3'
        jdk   'jdk17'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages{
        stage('Git Checkout'){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/BHarish07/Mission.git'
            }
        }
        stage('Compile'){
            steps{
                sh "mvn compile"
            }
        }

        stage('Test'){
            steps{
                sh "mvn test -DskipTests=true"
            }
        }

        stage('Trivy Scan Filesystem'){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html . "
            }
        }

        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('sonar') {
                 sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Mission -Dsonar.projectName=Mission \
                 -Dsonar.java.binaries=target '''
                }
            }
        }

        stage('Build'){
            steps{
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh "mvn package -DskipTests=true"
                }
            }
        }

        stage('Deploy Artifacts to nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh "mvn deploy -DskipTests=true"
                }
            }       
        }

        stage('Build & Tag Docker Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-auth', toolName: 'docker') {
                         sh "docker build -t harishbalike/mission:latest . "
                         
                     }
                    
                }
            }
        }

        stage('Trivy Image scan'){
            steps{
                sh "trivy image --format table -o trivy-image-report.html harishbalike/mission:latest"
            }
        }

         stage('Push Docker Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-auth', toolName: 'docker') {
                         sh "docker push harishbalike/mission:latest "
                         sh "docker run -d --name mission -p 8081:8080 harishbalike/mission:latest"
                         
                     }
                    
                }
            }
        }

        stage('Deploy to K8s'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'demo-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://53669D9303D62E2397D1EF3EF997D091.gr7.ap-south-1.eks.amazonaws.com') {
                   sh "kubectl apply -f manifest.yaml"
                }
            }
        }


    }

} 