pipeline {
  agent {
    docker {
      image 'mosama25/dynamic_jenkins_agent:v1.0'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
    environment {
        DOCKER_FRONTEND = 'public.ecr.aws/i5a7b8h3/nti-project-frontend:${BUILD_NUMBER}'
        DOCKER_BACKEND = 'public.ecr.aws/i5a7b8h3/nti-project-backend:${BUILD_NUMBER}'
        DOCKER_LOGIN_CREDS = credentials('docker_credentials')// Replace with your Jenkins credentials ID
    }

  stages {
    stage('Checkout') {
      steps {
        echo 'fetching repo'
        git branch: 'main', url: 'https://github.com/mohammedd2510/project_testing.git'
      }
    }
     stage('Build Docker Images') {
            steps {
                echo 'Building Docker image...'
                sh '''
                docker build -t $DOCKER_FRONTEND ./frontend
                docker build -t $DOCKER_BACKEND ./backend
                '''
            }
        }
        stage('Login to Docker Hub') {
            steps {
                echo 'Logging into Docker Hub...'
                sh '''
                    echo "$DOCKER_LOGIN_CREDS_PSW" | docker login -u $DOCKER_LOGIN_CREDS_USR --password-stdin public.ecr.aws/i5a7b8h3
                    
                '''
            }
        }
        stage('Push Docker Images') {
            steps {
                echo 'Pushing Docker image...'
                sh '''
                docker push $DOCKER_FRONTEND
                docker push $DOCKER_BACKEND
                '''
            }
        }
  }
}
