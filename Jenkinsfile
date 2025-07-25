pipeline {
  agent any

  environment {
    DOCKER_COMPOSE_PATH = './docker-compose.yml'
    BLUE_PORT = '8080'
    GREEN_PORT = '8081'
  }

  stages {
    stage('Clone Repository') {
      steps {
        git branch: 'main', url: 'https://github.com/Thaanees-RM/multi-service-app-main.git'
      }
    }

    stage('Build Docker Images') {
      steps {
        sh 'docker-compose build'
      }
    }

    stage('Deploy to Green') {
      steps {
        echo 'Starting Green environment on port 8081...'
        sh 'docker-compose -f docker-compose.yml -p green up -d'
      }
    }

    stage('Health Check') {
      steps {
        script {
          def retries = 5
          def healthy = false
          for (int i = 0; i < retries; i++) {
            def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${GREEN_PORT}", returnStdout: true)
            if (status == '200') {
              healthy = true
              break
            }
            sleep 5
          }
          if (!healthy) {
            error('Green deployment failed health check')
          }
        }
      }
    }

    stage('Switch Traffic') {
      steps {
        echo 'Switching Nginx to point to GREEN version...'
        sh '''
          sed -i 's/8080/8081/g' proxy/nginx.conf
          docker-compose -f docker-compose.yml restart proxy
        '''
      }
    }

    stage('Stop Old Version') {
      steps {
        echo 'Stopping BLUE environment...'
        sh 'docker-compose -p blue down || true'
      }
    }
  }

  post {
    failure {
      echo 'Pipeline failed.'
    }
    success {
      echo 'Blue-Green deployment successful.'
    }
  }
}
