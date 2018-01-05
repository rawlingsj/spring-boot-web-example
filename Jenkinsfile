pipeline {
  agent {
    label "jenkins-maven"
  }

  stages {
    stage('Build Release') {
      steps {
        container('maven') {
          sh "mvn clean deploy fabric8:build fabric8:push -Ddocker.push.registry=$JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT"
        }
      }
    }
    stage('Deploy Staging') {

// *** START - to be removed once CLI creates secret
      environment {
          JENKINS_X_BOT = credentials('jenkins-x-bot')
          JENKINS_X_BOT_USER = "${env.JENKINS_X_BOT_USR}"
          JENKINS_X_BOT_PASSWORD = "${env.JENKINS_X_BOT_PSW}"
      }
// *** END

      steps {
        dir ('./helm/spring-boot-web-example') {
          container('maven') {
            
// *** START - to be removed once CLI creates secret            
            sh """cat > ~/.netrc << EOL
machine github.com
       login $JENKINS_X_BOT_USER
       password $JENKINS_X_BOT_PASSWORD
EOL"""
// *** END

            sh 'make release'
            sh 'helm install . --namespace staging --name example-release'
          }
        }
      }
    }
  }
}
