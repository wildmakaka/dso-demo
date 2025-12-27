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

    stage('Static Analysis') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }

        stage('SCA') {
            steps {
                container('maven:3.8.6-openjdk-11') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'mvn org.owasp:dependency-check-maven:check'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts(
                        allowEmptyArchive: true,
                        artifacts: 'target/dependency-check-report.html',
                        fingerprint: true,
                        onlyIfSuccessful: true
                        )
                    // dependencyCheckPublisher pattern: 'report.xml'
                }
            }
        }

        stage('OSS License Checker') {
            steps {
                container('licensefinder') {
                    sh 'ls -al'
                    sh '''#!/bin/bash --login
                        /bin/bash --login
                        rvm use default
                        gem install license_finder
                        license_finder
                    '''
                }
            }
        }

      }
    }
    
    stage('Package') {
      steps {
        container('maven') {
          sh 'mvn package -DskipTests'
        }
      }
    }
    stage('Build and Push Docker Image') {
      steps {
        container('kaniko') {
          sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/webmakaka/dso-demo'
        }
      }
    }
  }
}
