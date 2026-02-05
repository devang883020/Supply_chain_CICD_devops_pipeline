pipeline {
    agent any

    environment {
        REGISTRY      = "13.201.12.133"
        PROJECT       = "phase1-project"
        IMAGE_NAME    = "nginx"
        IMAGE_TAG     = "${env.GIT_COMMIT.take(7)}"
        FULL_IMAGE    = "${REGISTRY}/${PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}"
        DOCKER_CREDS  = "harbor-creds"
        SONAR_ENV     = "sonarqube"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Scan (SAST)') {
    steps {
        script {
            def scannerHome = tool 'sonar-scanner'
            withSonarQubeEnv('sonarqube') {
                sh """
                  ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=phase1-secure-pipeline \
                    -Dsonar.projectName=phase1-secure-pipeline \
                    -Dsonar.sources=.
                """
            }
        }
    }
}


        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

       
        stage('Trivy FS Scan (Dependencies)') {
    steps {
        sh '''
          trivy fs \
            --severity HIGH,CRITICAL \
            --exit-code 1 \
            --no-progress \
            .
        '''
    }
}



        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                  docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}
                '''
            }
        }

        stage('Login to Harbor') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'HARBOR_USER',
                    passwordVariable: 'HARBOR_PASS'
                )]) {
                    sh '''
                      echo "$HARBOR_PASS" | docker login ${REGISTRY} \
                        -u "$HARBOR_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to Harbor') {
            steps {
                sh '''
                  docker push ${FULL_IMAGE}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Secure image pushed: ${FULL_IMAGE}"
        }
        failure {
            echo "❌ Pipeline blocked due to security/quality failure"
        }
    }
}
