pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
      }
    }
    stage('Package') {
      stages {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
        
        stage('Docker Build and Push') {
          steps {
            container('kaniko') {
              // Только после успешного создания jar
              sh '''
                /kaniko/executor \
                  -f Dockerfile \
                  -c . \
                  --destination=docker.io/webmakaka/dso-demo
              '''
            }
          }
        }
      }
    }
  }
}
