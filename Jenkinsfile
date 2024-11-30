pipeline {
  agent {
   docker {
      image 'mosama25/dynamic_jenkins_agent:v1.0'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
    environment {
        DOCKER_FRONTEND = "public.ecr.aws/i5a7b8h3/nti-project-frontend:v${BUILD_NUMBER}.0"
        DOCKER_LOGIN_CREDS = credentials('docker_credentials')// Replace with your Jenkins credentials ID
        GITHUB_TOKEN = credentials('github')_PSW
        GITHUB_USERNAME = credentials('github')_USR
        GITHUB_REPO = "project_testing"
    }

  stages {
    stage('Checkout') {
      steps {
        echo 'fetching repo'
        git branch: 'feature/frontend', url: 'https://github.com/mohammedd2510/project_testing.git'
      }
    }
     stage('Build Docker Images') {
            steps {
                echo 'Building Docker image...'
                sh '''
                docker build -t $DOCKER_FRONTEND ./frontend
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
                '''
            }
        }
      stage('Update Deployment File') {
         steps {
            withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                sh '''
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s|\\(image: public.ecr.aws/i5a7b8h3/nti-project-frontend:\\)[^ ]*|\\1v${BUILD_NUMBER}.0|g" project-manifests/frontend_deployment.yml
                    git config --global --add safe.directory /var/lib/jenkins/workspace/test_frontend_feature_frontend
                    git add ./project-manifests/frontend_deployment.yml
                    git commit -m "Update Frontend deployment image to version ${BUILD_NUMBER}"
                    git push origin feature/frontend
                '''
            }
        }
    }
    stage('Make Git Pull Request'){
        steps{
            sh '''
            curl -X POST \
                -H "Authorization: token $GITHUB_TOKEN" \
                -H "Accept: application/vnd.github.v3+json" \
                -d '{"title":"Amazing new feature","body":"Please pull this in!","head":"feature/frontend","base":"main"}' \
                     https://api.github.com/repos/$GITHUB_USERNAME/$GITHUB_REPO/pulls
            '''
        }
    }
  }
}