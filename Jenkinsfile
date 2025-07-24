pipeline {
  agent any

  environment {
    COMPOSE_FILE = 'docker-compose.yml'
  }

  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/Thaanees-RM/multi-service-app-main.git'
      }
    }

    stage('Build Containers') {
      steps {
        sh 'docker compose build'
      }
    }

    stage('Run Containers') {
      steps {
        sh 'docker compose up -d'
      }
    }

    stage('Health Check') {
      steps {
        sh 'curl -f http://localhost || exit 1'
      }
    }

    stage('Cleanup Unused') {
      steps {
        sh 'docker system prune -f'
      }
    }
  }

  post {
    success {
      echo '✅ Deployed successfully!'
    }
    failure {
      echo '❌ Deployment failed!'
    }
  }
}
