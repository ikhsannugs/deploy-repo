properties([pipelineTriggers([githubPush()])]) 

pipeline {

  agent any
    stages{
      stage('Test Code With Sonarqube') {
        steps {
          script {
            def scannerHome = tool 'sonarscanner';
              withSonarQubeEnv("sonarserver") {
              sh "${tool("sonarscanner")}/bin/sonar-scanner \
              -Dsonar.projectKey=webapps-${BRANCH_NAME} \
              -Dsonar.sources=."
            }
          }
        }
      }
      stage('Build with Docker') {
        steps {
          sh "docker build -f Dockerfile -t ${DOCKER_REPO}/webapps:${BRANCH_NAME}-${BUILD_NUMBER} ."
        }
      }
      stage('Publish Docker Image') {
        steps {
          sh "docker push ${DOCKER_REPO}/webapps:${BRANCH_NAME}-${BUILD_NUMBER}"
          sh "docker image rm -f ${DOCKER_REPO}/webapps:${BRANCH_NAME}-${BUILD_NUMBER}"
        }
      }
      stage('Deploy to Kubernetes') {
        when {
           changelog 'deployment'
        }
        input {
          message "Should we continue?"
            ok "Yes, we should."
            parameters {
              string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who are you?')
            }
        }
        steps {
          script {
            if ( env.GIT_BRANCH == 'stagging' ) {
              sh "wget https://raw.githubusercontent.com/ikhsannugs/deploy-repo/master/deploy-webapps.yaml"
              sh "sed -i 's/ENV/${BRANCH_NAME}/g' deploy-webapps.yaml"
              sh "sed -i 's/NO/${BUILD_NUMBER}/g' deploy-webapps.yaml"
              sh "kubectl apply -f deploy-webapps.yaml"
            }
            else if ( env.GIT_BRANCH == 'master' ) {
              sh "wget https://raw.githubusercontent.com/ikhsannugs/deploy-repo/master/deploy-webapps.yaml"
              sh "sed -i 's/ENV/${BRANCH_NAME}/g' deploy-webapps.yaml"
              sh "sed -i 's/NO/${BUILD_NUMBER}/g' deploy-webapps.yaml"
              sh "kubectl apply -f deploy-webapps.yaml"
            }
          }
        }
      }
    }
    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir()
        }
        success {
            echo 'I succeeded!'
        }
        failure {
            echo 'I failed :('
        }
    }
}
