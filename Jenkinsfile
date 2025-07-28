pipeline {
  agent any

  environment {
    DOCKER_COMPOSE_COMMON = './docker-compose.yml'
    DOCKER_COMPOSE_BLUE = './blue/docker-compose.blue.yml'
    DOCKER_COMPOSE_GREEN = './green/docker-compose.green.yml'
    BLUE_PORT = '8080'
    GREEN_PORT = '8081'
    PROXY_SERVICE = 'proxy'
    MAX_HEALTH_CHECK_RETRIES = 5
    HEALTH_CHECK_INTERVAL = 5
  }

  stages {
    stage('Deploy Green') {
      steps {
        echo '🟢 Deploying green version...'
        sh """
          docker-compose -f ${DOCKER_COMPOSE_COMMON} -f ${DOCKER_COMPOSE_GREEN} -p green up -d --build
        """
      }
    }

    stage('Health Check') {
      steps {
        echo '🔍 Waiting for green service to become healthy...'
        script {
          def healthy = false
          for (int i = 0; i < MAX_HEALTH_CHECK_RETRIES.toInteger(); i++) {
            def status = sh (
              script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${GREEN_PORT}/health || true",
              returnStdout: true
            ).trim()
            echo "Attempt ${i+1}: Green service status = ${status}"
            if (status == '200') {
              healthy = true
              break
            }
            sleep(HEALTH_CHECK_INTERVAL.toInteger())
          }
          if (!healthy) {
            error("❌ Green service failed health check.")
          }
        }
      }
    }

    stage('Switch Traffic') {
      steps {
        echo '🔁 Switching traffic to green...'
        sh """
          docker exec ${PROXY_SERVICE} sh -c "echo 'Switching to GREEN on port ${GREEN_PORT}'"
          docker exec ${PROXY_SERVICE} sh -c "sed -i 's/:${BLUE_PORT}/:${GREEN_PORT}/' /etc/nginx/conf.d/default.conf"
          docker exec ${PROXY_SERVICE} nginx -s reload
        """
      }
    }

    stage('Clean Up Blue') {
      steps {
        echo '🧹 Shutting down blue deployment...'
        sh """
          docker-compose -f ${DOCKER_COMPOSE_COMMON} -f ${DOCKER_COMPOSE_BLUE} -p blue down
        """
      }
    }
  }
}
