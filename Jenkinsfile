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
        HEALTH_CHECK_INTERVAL = 5 // seconds
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: 'https://github.com/Thaanees-RM/multi-service-app-main.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh 'docker-compose -f $DOCKER_COMPOSE_COMMON build'
            }
        }

        stage('Determine Current Active Deployment') {
            steps {
                script {
                    // Simple heuristic: check which port is currently active in Nginx config
                    def nginxConf = readFile('proxy/nginx.conf')
                    if (nginxConf.contains(GREEN_PORT)) {
                        env.ACTIVE_ENV = 'green'
                        env.INACTIVE_ENV = 'blue'
                        env.INACTIVE_PORT = BLUE_PORT
                    } else {
                        env.ACTIVE_ENV = 'blue'
                        env.INACTIVE_ENV = 'green'
                        env.INACTIVE_PORT = GREEN_PORT
                    }
                    echo "Current active environment: ${env.ACTIVE_ENV}"
                    echo "Deploying to inactive environment: ${env.INACTIVE_ENV}"
                }
            }
        }

        stage('Deploy to Inactive Environment') {
            steps {
                script {
                    def composeFile = env.INACTIVE_ENV == 'blue' ? env.DOCKER_COMPOSE_BLUE : env.DOCKER_COMPOSE_GREEN
                    def projectName = env.INACTIVE_ENV

                    echo "Starting $projectName environment..."

                    // Bring up the inactive environment (detached)
                    sh "docker-compose -f $DOCKER_COMPOSE_COMMON -f $composeFile -p $projectName up -d"
                }
            }
        }

        stage('Health Check Inactive Environment') {
            steps {
                script {
                    def port = env.INACTIVE_PORT
                    def healthy = false
                    for (int i = 0; i < MAX_HEALTH_CHECK_RETRIES; i++) {
                        echo "Checking health on http://localhost:${port} (Attempt ${i+1})"
                        def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${port}", returnStdout: true).trim()
                        if (status == '200') {
                            echo "Health check passed."
                            healthy = true
                            break
                        }
                        sleep HEALTH_CHECK_INTERVAL
                    }
                    if (!healthy) {
                        error "Health check failed on http://localhost:${port}"
                    }
                }
            }
        }

        stage('Switch Traffic to Inactive Environment') {
            steps {
                script {
                    // Copy the appropriate nginx config for the new active env and restart proxy
                    def confFile = env.INACTIVE_ENV == 'blue' ? 'proxy/nginx.blue.conf' : 'proxy/nginx.green.conf'
                    echo "Switching Nginx traffic to $env.INACTIVE_ENV environment"

                    sh """
                      cp $confFile proxy/nginx.conf
                      docker-compose -f $DOCKER_COMPOSE_COMMON restart $PROXY_SERVICE
                    """
                }
            }
        }

        stage('Shutdown Old Environment') {
            steps {
                script {
                    def oldEnv = env.ACTIVE_ENV
                    echo "Stopping old $oldEnv environment"
                    sh "docker-compose -f $DOCKER_COMPOSE_COMMON -f ./docker-compose.${oldEnv}.yml -p $oldEnv down || true"
                }
            }
        }
    }

    post {
        success {
            echo 'Blue-Green deployment completed successfully.'
            // Add notification here (Slack, email etc) if desired
        }
        failure {
            echo 'Pipeline failed!'
            // Add failure notifications here
        }
    }
}
