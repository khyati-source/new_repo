
@Library("my_library") _

pipeline{
    agent any
    stages{
        stage("Checkout"){
            steps {
                script { 
                    echo "========executing A========"
                    def config = [ 
                            url: 'https://github.com/khyati-source/node_branch.git',
                            branch: 'main',
                            credentialsId: 'my-github-token'
                        ]
                    gitCheckout(config)
                    sh '''
                        pwd 
                        ls -lrt
                    '''
                }
            }
        }

        stage ("Code Qualitiy Sonar Check") {
            steps {
                withSonarQubeEnv('sonar_server') {
                    sh '''
                        sonar-scanner  
                    '''
                }
            }
        }

        stage ("Check Qualitiy Gate Result") {
            steps {
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
            }
        }

        stage('Vulnerability Scan') {
            steps {
                sh '''
                    wget https://raw.githubusercontent.com/aquasecurity/trivy/refs/heads/main/contrib/html.tpl
                    trivy fs --format template --template "@html.tpl" -o trivy_report.html .
                    ls trivy_report.html
                '''

                publishHTML (target : [
                            reportName: 'Trivy Vulnerability Report',
                            reportDir: '.',
                            reportFiles: 'trivy_report.html',
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            ])

            }
        }

        stage('Docker Build, Trivy scan & Push using plugin') {
            environment {
                IMAGE_NAME = 'khyati8/node-backend'
                IMAGE_TAG = 'latest'
                DOCKERFILE_SRC = 'Dockerfile'
                DOCKER_CREDS = 'dockerhub-creds-id'
            }
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_CREDS}") {
                        appImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}", "-f ${DOCKERFILE_SRC} .")
                        appImage.push()
                    }

                    sh '''
                        trivy image --severity CRITICAL --format template --template "@html.tpl" -o trivy_image_scan_report.html "$IMAGE_NAME:$IMAGE_TAG"
                        ls trivy_image_scan_report.html
                    '''

                    publishHTML (target : [
                        reportName: 'Trivy Vulnerability Image Report',
                        reportDir: '.',
                        reportFiles: 'trivy_image_scan_report.html',
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                    ])
                }
            }
        }

        // stage('Build and Push Docker Image using CLI') {
        //     environment {
        //         IMAGE_NAME = 'khyati8/node-backend'
        //         IMAGE_TAG = 'latest'
        //         DOCKERFILE_SRC = 'Dockerfile'
        //     }
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'dockerhub-creds-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_TOKEN')]) {
        //             sh '''
        //                 docker build -t "$IMAGE_NAME:$IMAGE_TAG" -f "$DOCKERFILE_SRC" .
        //                 echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin https://index.docker.io/v1/
        //                 docker push "$IMAGE_NAME:$IMAGE_TAG"
        //             '''
        //         }
                
        //     }
        // }
    }
    post {
        always {
            cleanWs()
        }
    }
}
