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
                                                 
                     }
                    
                }
            }
        }

        stage('Deploy to K8s'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'demo-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://53669D9303D62E2397D1EF3EF997D091.gr7.ap-south-1.eks.amazonaws.com') {
                   sh "kubectl apply -f manifest.yaml"
                   sleep 60
                }
            }
        }

        stage('Verify Deployment'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'demo-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://53669D9303D62E2397D1EF3EF997D091.gr7.ap-south-1.eks.amazonaws.com') {
                   sh "kubectl get pods -n webapps"
                   sh "kubectl get svc -n webapps"
                }
            }
        }


    }

    post{
        always{
            script{
                def jobName = env.JOB_NAME 
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """
                   <html>
                   <body>
                   <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                   <h2>${jobName} - Build $ ${buildNumber}</h2>
                   <div style="background-color: ${bannerColor}; padding: 10px;">
                   <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                   </div>
                   <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                   </div>
                   </body>
                   </html>
                   """
                   emailext(
                    subject: "${jobName} - Bukd ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body:body,
                    to: 'harishbalike1995@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                   )
            }
        }
    }
    

} 