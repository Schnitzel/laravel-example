 pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }
  environment {
    DOCKER_CREDS = credentials('amazeeiojenkins-dockerhub-password')
    COMPOSE_PROJECT_NAME = "drupaltest-${BUILD_ID}"
  }
  stages {
    stage('Docker login') {
      steps {
        sh """
        docker login --username amazeeiojenkins --password $DOCKER_CREDS
        docker kill 997c23b75769 580d165166cd 58d5a129acb4
        docker ps | head
        docker ps  |grep 8000
        """
      }
    }
    stage('Docker Build') {
      steps {
        sh '''
        docker network create amazeeio-network || true
        docker-compose config -q
        docker-compose down
        docker-compose up -d --build "$@"
        '''
      }
    }
    stage('Waiting') {
      steps {
        sh """
        sleep 1s
        """
      }
    }
    stage('Verification') {
      steps {
        sh '''
        docker-compose exec -T blog php -r \"file_exists('.env') || copy('.env.example', '.env');\"
        docker-compose exec -T blog php artisan key:generate --ansi
        docker-compose exec -T blog php public/index.php
        docker-compose exec -T blog cat .env
        echo docker-compose exec -T blog ls -al storage/logs
        echo docker-compose exec -T blog cat storage/logs/laravel-2019-04-23.log
        docker-compose exec -T nginx curl http://nginx:8000 -v
        docker-compose down
        '''
      }
    }
  }
}
