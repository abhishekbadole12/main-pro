// pipeline {
//     agent any

//     tools {
//         jdk "jdk21"
//         maven "maven3"
//     }

//     environment {
//         SCANNER_HOME = tool 'sonar-scanner'
//     }
    
//     stages {
//         stage('Git Checkout') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/abhishekbadole12/main-pro.git'
//             }
//         }
//         stage('Compile') {
//             steps {
//                 sh "mvn compile"
//             }
//         }
//         stage('Trivy FS') {
//             steps {
//                 sh "trivy fs . --format table -o fs.html"
//             }
//         }
//         stage('SonarQube Analysis') {
//             steps {
//                 withSonarQubeEnv('sonarqubeServer') {
//                     sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=blogginapp -Dsonar.projectKey=blogginapp \
//                           -Dsonar.java.binaries=target'''
//                 }
//             }
//         }
//         stage('Build') {
//             steps {
//                 sh "mvn package"
//             }
//         }
//         stage('Publish Artifacts') {
//             steps {
//                 withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk21', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
//                         sh "mvn deploy"
//                 }
//             }
//         }
//        stage('Docker Build & Tag') {
//             steps {
//                 script {
//                     withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
//                         sh """
//                             docker build --no-cache \
//                             -t abhishekbadole12/gab-blogging-app:${BUILD_NUMBER} .
//                         """
//                     }
//                 }
//             }
//         }
//         stage('Trivy Image Scan') {
//             steps {
//                 sh "trivy image --format table -o image.html abhishekbadole12/gab-blogging-app:latest"
//             }
//         }
//         stage('Docker Push Image') {
//             steps {
//                 script{
//                 withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
//                     sh "docker push abhishekbadole12/gab-blogging-app"
//                 }
//                 }
//             }
//         }
//         stage('K8s Deploy') {
//             steps {
//                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', serverUrl: 'https://08D790A58E638F573FAF7D697D694185.yl4.ap-southeast-1.eks.amazonaws.com']]) {
//                     sh "kubectl apply -f deployment-service.yml"
//                     sleep 20
//                 }
//             }
//         }
//         stage('Verify Deployment') {
//             steps {
//                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', serverUrl: 'https://08D790A58E638F573FAF7D697D694185.yl4.ap-southeast-1.eks.amazonaws.com']]) {
//                     sh "kubectl get pods"
//                     sh "kubectl get service"
//                 }
//             }
//         }
        
//     }  // Closing stages

//     post {
//         always {
//             script {
//                 // Get job name, build number, and pipeline status
//                 def jobName = env.JOB_NAME
//                 def buildNumber = env.BUILD_NUMBER
//                 def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
//                 pipelineStatus = pipelineStatus.toUpperCase()
                
//                 // Set the banner color based on the status
//                 def bannerColor = pipelineStatus == 'SUCCESS' ? 'green' : 'red'

//                 // HTML body for the email
//                 def body = """
//                 <body>
//                     <div style="border: 2px solid ${bannerColor}; padding: 10px;">
//                         <h3 style="color: ${bannerColor};">
//                             Pipeline Status: ${pipelineStatus}
//                         </h3>
//                         <p>Job: ${jobName}</p>
//                         <p>Build Number: ${buildNumber}</p>
//                         <p>Status: ${pipelineStatus}</p>
//                     </div>
//                 </body>
//                 """

//                 // Send email notification
//                 emailext(
//                     subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
//                     body: body,
//                     to: 'abhishekbadole12@gmail.com',
//                     from: 'jenkins@example.com',
//                     replyTo: 'jenkins@example.com',
//                     mimeType: 'text/html'
//                 )
//             }
//         }
//     }
// }  // Closing pipeline

pipeline {
    agent any

    tools {
        jdk "jdk21"
        maven "maven3"
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'abhishekbadole12/gab-blogging-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/abhishekbadole12/main-pro.git'

                sh '''
                    echo "======================================"
                    echo "Latest Git Commit:"
                    git log -1 --oneline
                    echo "======================================"
                '''
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Trivy FS') {
            steps {
                sh "trivy fs . --format table -o fs.html"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqubeServer') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=blogginapp \
                        -Dsonar.projectKey=blogginapp \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Publish Artifacts') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'global-settings',
                    jdk: 'jdk21',
                    maven: 'maven3',
                    mavenSettingsConfig: '',
                    traceability: true
                ) {
                    sh "mvn deploy"
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: 'docker-cred',
                        url: 'https://index.docker.io/v1/'
                    ) {
                        sh """
                            echo "Building Docker image:"
                            echo "${DOCKER_IMAGE}:${IMAGE_TAG}"

                            docker build --no-cache \
                                -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                        """
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh """
                    trivy image \
                    --format table \
                    -o image.html \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}
                """
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: 'docker-cred',
                        url: 'https://index.docker.io/v1/'
                    ) {
                        sh """
                            docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('K8s Deploy') {
            steps {
                withKubeCredentials(
                    kubectlCredentials: [[
                        caCertificate: '',
                        clusterName: 'devopsshack-cluster',
                        contextName: '',
                        credentialsId: 'k8-cred',
                        namespace: 'webapps',
                        serverUrl: 'https://08D790A58E638F573FAF7D697D694185.yl4.ap-southeast-1.eks.amazonaws.com'
                    ]]
                ) {
                    sh """
                        echo "Deploying image:"
                        echo "${DOCKER_IMAGE}:${IMAGE_TAG}"

                        kubectl apply -f deployment-service.yml

                       kubectl set image deployment/bloggingapp-deployment \
                        bloggingapp=${DOCKER_IMAGE}:${IMAGE_TAG} \
                        -n webapps

                        kubectl rollout status deployment/bloggingapp-deployment \
                            -n webapps \
                            --timeout=180s
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeCredentials(
                    kubectlCredentials: [[
                        caCertificate: '',
                        clusterName: 'devopsshack-cluster',
                        contextName: '',
                        credentialsId: 'k8-cred',
                        namespace: 'webapps',
                        serverUrl: 'https://08D790A58E638F573FAF7D697D694185.yl4.ap-southeast-1.eks.amazonaws.com'
                    ]]
                ) {
                    sh """
                        echo "======================================"
                        echo "Pods:"
                        kubectl get pods -n webapps

                        echo "======================================"
                        echo "Deployment:"
                        kubectl get deployment bloggingapp-deployment -n webapps

                        echo "======================================"
                        echo "Current Image:"
                        kubectl get deployment bloggingapp-deployment \
                            -n webapps \
                            -o=jsonpath='{.spec.template.spec.containers[*].image}'

                        echo ""
                        echo "======================================"
                        echo "Services:"
                        kubectl get service -n webapps
                    """
                }
            }
        }
    }

    post {
        always {
            script {

                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'

                pipelineStatus = pipelineStatus.toUpperCase()

                def bannerColor =
                    pipelineStatus == 'SUCCESS' ? 'green' : 'red'

                def body = """
                <body>
                    <div style="border: 2px solid ${bannerColor}; padding: 10px;">
                        <h3 style="color: ${bannerColor};">
                            Pipeline Status: ${pipelineStatus}
                        </h3>

                        <p>Job: ${jobName}</p>
                        <p>Build Number: ${buildNumber}</p>
                        <p>Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}</p>
                        <p>Status: ${pipelineStatus}</p>
                    </div>
                </body>
                """

                emailext(
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
                    body: body,
                    to: 'abhishekbadole12@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html'
                )
            }
        }
    }
}