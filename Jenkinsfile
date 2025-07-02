pipeline {
    agent { label 'Build-Jenkins-Slave' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_id')
    }

    stages {
        stage('SCM_Checkout') {
            steps {
                echo 'Perform SCM Checkout'
                git 'https://github.com/sweta912/star-agile-health-care.git'
            }
        }

        stage('Application Build') {
            steps {
                echo 'Perform Application Build'
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Perform Docker Build'
                sh "docker build -t sweta912sharma/healthcare-app:${BUILD_NUMBER} ."
                sh "docker tag sweta912sharma/healthcare-app:${BUILD_NUMBER} sweta912sharma/healthcare-app:latest"
                sh 'docker image ls'
            }
        }

        stage('Login to Dockerhub') {
            steps {
                echo 'Login to DockerHub'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Publish the Image to Dockerhub') {
            steps {
                echo 'Publish to DockerHub'
                sh "docker push sweta912sharma/healthcare-app:${BUILD_NUMBER}"
                sh "docker push sweta912sharma/healthcare-app:latest"
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'Kubernetes_Master',
                                transfers: [
                                    sshTransfer(
                                        cleanRemote: false,
                                        excludes: '',
                                        execCommand: 'kubectl apply -f kdeploy.yml',
                                        execTimeout: 120000,
                                        flatten: false,
                                        makeEmptyDirs: false,
                                        noDefaultExcludes: false,
                                        patternSeparator: '[, ]+',
                                        remoteDirectory: '.',
                                        remoteDirectorySDF: false,
                                        removePrefix: '',
                                        sourceFiles: 'kdeploy.yml'
                                    )
                                ],
                                usePromotionTimestamp: false,
                                useWorkspaceInPromotion: false,
                                verbose: false
                            )
                        ]
                    )
                }
            }

            post {
                success {
                    echo 'Send mail on successful deployment'
                    mail bcc: 'sweta912sharma@gmail.com',
                         body: "Please go to ${BUILD_URL} and verify the build.",
                         cc: 'sweta912sharma@gmail.com',
                         subject: 'Healthcare App Deployment Status from Jenkins',
                         to: 'sweta912sharma@gmail.com'
                }

                failure {
                    echo 'Send mail on failure'
                    mail bcc: 'sweta912sharma@gmail.com',
                         body: "Please go to ${BUILD_URL} and verify the build.",
                         cc: 'sweta912sharma@gmail.com',
                         subject: 'Healthcare App Deployment Failed',
                         to: 'sweta912sharma@gmail.com'
                }

                unstable {
                    echo 'Send mail on unstable'
                    mail bcc: 'sweta912sharma@gmail.com',
                         body: "Please go to ${BUILD_URL} and verify the build.",
                         cc: 'sweta912sharma@gmail.com',
                         subject: 'Healthcare App Build Unstable',
                         to: 'sweta912sharma@gmail.com'
                }
            }
        }
    }
}
