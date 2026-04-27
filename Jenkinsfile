pipeline {
    agent any

    environment {
        IMAGE_NAME   = "insta-devops-project"
        IMAGE_TAG    = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "manish07648/insta-devops-project"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Code Clone') {
            steps {
                git branch: 'main',
                credentialsId: 'githubcreds',
                url: 'https://github.com/manish07648/insta-devops-project.git'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                echo "Running OWASP Scan..."
                dependencyCheck odcInstallation: 'owasp', additionalArguments: '--scan .'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('sonar-server') {
                        withCredentials([string(credentialsId: 'sonartoken', variable: 'SONAR_TOKEN')]) {
                            sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=insta-project \
                            -Dsonar.projectName=insta-project \
                            -Dsonar.sources=. \
                            -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "Building Docker Containers..."
                sh 'docker compose build'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo "Running Trivy Scan..."
                sh 'trivy fs .'
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhubcreds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker tag insta-nginx $DOCKER_IMAGE:latest || true
                    docker tag insta-nginx $DOCKER_IMAGE:$IMAGE_TAG || true

                    docker push $DOCKER_IMAGE:latest || true
                    docker push $DOCKER_IMAGE:$IMAGE_TAG || true
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                echo "Deploying Application..."
                sh '''
                docker compose down || true
                docker compose up -d --build
                '''
            }
        }

        stage('Cleanup') {
            steps {
                echo "Cleaning Docker Images..."
                sh 'docker image prune -f'
            }
        }
    }

    post {

        success {
            echo "Pipeline Completed Successfully"
        }

        failure {
            echo "Pipeline Failed"
        }

        always {
            cleanWs()
        }
    }
}
