pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      // УБЕРИТЕ ЭТУ СТРОКУ: defaultContainer 'maven'
      idleMinutes 1
    }
  }
  
  stages {
    stage('Build') {
      steps {
        container('maven') {  // Явно указываем контейнер
          sh 'mvn compile'
        }
      }
    }
    
    stage('Test') {
      steps {
        container('maven') {  // Явно указываем контейнер
          sh 'mvn test'
        }
      }
    }
    
    stage('Create Jarfile') {
      steps {
        container('maven') {  // Явно указываем контейнер
          sh 'mvn package -DskipTests'
        }
      }
    }
    
    stage('Docker Build and Push') {
      steps {
        container('kaniko') {  // Теперь это будет работать
          script {
            // Проверяем что мы в контейнере kaniko
            sh 'echo "Building Docker image..."'
            
            // Собираем с тегом
            sh '''
              /kaniko/executor \
                -f Dockerfile \
                -c . \
                --destination=docker.io/webmakaka/dso-demo:latest
            '''
          }
        }
      }
    }
  }
}
