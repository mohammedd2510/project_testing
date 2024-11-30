pipeline {
  agent {
   docker {
      image 'mosama25/dynamic_jenkins_agent:v1.0'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
    environment {
        DOCKER_FRONTEND_IMAGE = "public.ecr.aws/i5a7b8h3/nti-project-frontend:v${BUILD_NUMBER}.0"
        ECR_NAME ="public.ecr.aws/i5a7b8h3/nti-project-frontend"
        DOCKER_LOGIN_CREDS = credentials('docker_credentials')// Replace with your Jenkins credentials ID
        GITHUB_CREDS = credentials('github')
        GITHUB_REPO = "project_testing"
        GITHUB_BRANCH = "feature/frontend"
        DEPLOYMENT_FILE = "project-manifests/frontend_deployment.yml"
    }

  stages {
    stage('Checkout') {
      steps {
        echo 'fetching repo'
        git branch: "${GITHUB_BRANCH}", url: 'https://github.com/mohammedd2510/project_testing.git'
      }
    }
     stage('Build Docker Images') {
            steps {
                echo 'Building Docker image...'
                sh '''
                docker build -t $DOCKER_FRONTEND_IMAGE ./frontend
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
                docker push $DOCKER_FRONTEND_IMAGE
                '''
            }
        }
      stage('Update Deployment File') {
         steps {
            withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                sh '''
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s|\\(image: $ECR_NAME:\\)[^ ]*|\\1v${BUILD_NUMBER}.0|g" $DEPLOYMENT_FILE
                    git config --global --add safe.directory $WORKSPACE
                    git add ./$DEPLOYMENT_FILE
                    git commit -m "Update Frontend deployment image to version ${BUILD_NUMBER}"
                    git push origin ${GITHUB_BRANCH}
                '''
            }
        }
    }
    stage('Make Git Pull Request'){
        steps{
            sh '''
            curl -X POST \
                -H "Authorization: token $GITHUB_CREDS_PSW" \
                -H "Accept: application/vnd.github.v3+json" \
                -d '{"title":"Amazing new feature","body":"Please pull this in!","head":"${GITHUB_BRANCH}","base":"main"}' \
                     https://api.github.com/repos/$GITHUB_CREDS_USR/$GITHUB_REPO/pulls
            '''
        }
    }
   
  }
 post {
        success {
                slackSend(
                    color: 'good', 
                    message: "*Build Successful*\nProject: ${env.JOB_NAME}\nBuild: #${BUILD_NUMBER}\nStatus: Image is built successfully, and the pull request is made successfully\nFMore info at: ${BUILD_URL}.",
                    channel: '#nti-graduation-project'
                )
        }

        failure {
                slackSend(
                    color: 'danger', 
                    message: "*Build Failed*\nProject: ${env.JOB_NAME}\nBuild: #${BUILD_NUMBER}\nStatus: Build failed.\nMore info at: ${BUILD_URL}",
                    channel: '#nti-graduation-project'
                )
        }
    }
}