pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('del-docker-hub-auth')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout(time: 60, unit: 'MINUTES')
        timestamps()
    }

    stages {
        stage('Maven Test') {
            agent {
                docker { image 'maven:3.8.5-openjdk-18' }
            }
            steps {
                sh '''
                    cd orders
                    mvn test
                '''
            }
        }

        stage('SonarQube analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:5.0.1'
                }
            }
            environment {
                CI = 'true'
                scannerHome = '/opt/sonar-scanner'
            }
            steps {
                withSonarQubeEnv('Sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Build-image-api') {
            steps {
                script {
                    sh '''
                    cd ${WORKSPACE}/orders
                    docker build -t devopseasylearning/s7rosine-do-it-yourself-orders-api:api-${BUILD_NUMBER} .
                    '''
                }
            }
        }
        
        stage('Build-image-db') {
            steps {
                script {
                    sh '''
                    cd ${WORKSPACE}/orders
                    docker build -t devopseasylearning/s7rosine-do-it-yourself-orders-db:db-${BUILD_NUMBER} -f Dockerfile-db .
                    ls -l
                    '''
                }
            }
        }

        stage('Push orders-image') {
            when {
                expression { env.GIT_BRANCH == 'origin/main' }
            }
            steps {
                sh '''
                id
                docker push devopseasylearning/s7rosine-do-it-yourself-orders-api:api-${BUILD_NUMBER}
                docker push devopseasylearning/s7rosine-do-it-yourself-orders-db:db-${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            slackSend (
                channel: '#development-alerts',
                color: 'good',
                message: "SUCCESSFUL: Application s7rosine-do-it-yourself-orders Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        unstable {
            slackSend (
                channel: '#development-alerts',
                color: 'warning',
                message: "UNSTABLE: Application s7rosine-do-it-yourself-orders Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        failure {
            slackSend (
                channel: '#development-alerts',
                color: '#FF0000',
                message: "FAILURE: Application s7rosine-do-it-yourself-orders Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        cleanup {
            deleteDir()
        }
    }
}
