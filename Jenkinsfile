pipeline {
    agent any

    environment {
        IMAGE_NAME = "insta-devops-project"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "manish07648/insta-devops-project"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Code Clone') {
            steps {
                echo "Cloning latest code..."
                git branch: 'main',
                credentialsId: 'githubcreds',
                url: 'https://github.com/manish07648/insta-devops-project.git'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                echo "Running OWASP scan..."
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo "Running SonarQube scan..."
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=insta-project \
                    -Dsonar.projectName=insta-project \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "Building containers..."
                sh 'docker compose build'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo "Scanning images..."
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
                echo "Deploying latest build..."
                sh '''
                docker compose down || true
                docker compose up -d --build
                '''
            }
        }

        stage('Cleanup') {
            steps {
                echo "Cleaning unused images..."
                sh 'docker image prune -f'
            }
        }
    }

    post {

        success {
            echo "Pipeline completed successfully"
        }

        failure {
            echo "Pipeline failed"
        }

        always {
            cleanWs()
        }
    }
}
