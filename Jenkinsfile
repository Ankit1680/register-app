pipeline { 
    agent { label 'Jenkins-Agent' } 
    tools { 
        jdk 'java17' 
        maven 'maven3' 
    } 
    environment { 
        APP_NAME = "register-app-pipeline" 
            RELEASE = "1.0.0" 
            DOCKER_USER = "ankit2849" 
            DOCKER_PASS = "docker-cred" 
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}" 
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    } 
    stages{ 
        stage("Cleanup Workspace"){ 
                steps { 
                cleanWs() 
                } 
        } 
 
        stage("Checkout from SCM"){ 
                steps { 
                    git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/Ankit1680/register-app' 
                } 
        } 
 
        stage("Build Application"){ 
            steps { 
                sh "mvn clean package" 
            } 
 
       } 
 
       stage("Test Application"){ 
           steps { 
                 sh "mvn test" 
           } 
       } 
 
       stage("SonarQube Analysis"){ 
           steps { 
            script { 
          withSonarQubeEnv(credentialsId: 'sonar-token') { 
                sh "mvn sonar:sonar" 
          } 
            }  
           } 
       } 
 
       stage("Quality Gate"){ 
           steps { 
               script { 
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }  
            } 
 
        } 
 
        stage("Build & Push Docker Image") { 
            steps { 
                script { 
                    docker.withRegistry('',DOCKER_PASS) { 
                        docker_image = docker.build "${IMAGE_NAME}" 
                    } 
 
                    docker.withRegistry('',DOCKER_PASS) { 
                        docker_image.push("${IMAGE_TAG}") 
                        docker_image.push('latest') 
                    } 
                } 
            } 
 
       } 
 
       stage("Trivy Scan") { 
           steps { 
               script { 
             sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image 
                    ankit2849/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity 
                    HIGH,CRITICAL --format table') 
               } 
           } 
       } 
 
       stage ('Cleanup Artifacts') { 
           steps { 
               script { 
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}" 
                    sh "docker rmi ${IMAGE_NAME}:latest" 
               } 
          } 
       } 
 
    //    stage("Trigger CD Pipeline") { 
    //         steps { 
    //             script { 
    //                 sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: 
// no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 
// 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-235-27-140.ap-south
// 1.compute.amazonaws.com:8080/job/register-app-cd
// pipeline/buildWithParameters?token=gitops-token'" 
    //             } 
    //         } 
    //    } 
    } 
 
//     post { 
//        failure { 
//              emailext body: '''${SCRIPT, template="groovy-html.template"}''',  
//                       subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed",  
//                       mimeType: 'text/html',to: "avishwakarma8855@gmail.com" 
//       } 
//       success { 
//             emailext body: '''${SCRIPT, template="groovy-html.template"}''',  
//                      subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful",  
//                      mimeType: 'text/html',to: "avishwakarma8855@gmail.com" 
//       }       
//    } 
}