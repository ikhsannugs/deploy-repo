properties([pipelineTriggers([githubPush()])]) 

pipeline {

  agent  any
    stages {
      stage('TEST CODE') {
        parallel {
          stage('Code Quality Check via SonarQube') {
            steps {
              script {
              def scannerHome = tool 'sonarscanner';
                withSonarQubeEnv("sonarserver") {
                sh "${tool("sonarscanner")}/bin/sonar-scanner \
                -Dsonar.projectKey=android-login-${BRANCH_NAME} \
                -Dsonar.java.binaries=."
                }
              }
            }
          }
          stage('Unit & Integration Tests') {
            steps {
              script {
                try {
                  sh './gradlew clean test --no-daemon' //run a gradle task
                } 
                finally {
                  junit '**app/build/test-results/testReleaseUnitTest/*.xml'
                  junit '**app/build/test-results/testDebugUnitTest/*.xml'
                }
              }
            }
          }
        }
      }
      stage('Build Apps') {
        steps {
          sh './gradlew build'
        }
      }
      stage('Deploy Artifact') {
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
            if ( env.GIT_BRANCH == 'staging' ) {
              sh "mv ${WORKSPACE}/app/build/outputs/apk/release/*.apk ${WORKSPACE}/app/build/outputs/apk/release/Release-stag-${BUILD_NUMBER}.apk "
              sh "mv ${WORKSPACE}/app/build/outputs/apk/debug/*.apk ${WORKSPACE}/app/build/outputs/apk/debug/Debug-stag-${BUILD_NUMBER}.apk "
              sh "${HOME}/ossutil64 cp ${WORKSPACE}/app/build/outputs/apk/release/Release-stag-${BUILD_NUMBER}.apk oss://android-login-stagging/Release/"
              sh "${HOME}/ossutil64 cp ${WORKSPACE}/app/build/outputs/apk/debug/Debug-stag-${BUILD_NUMBER}.apk oss://android-login-stagging/Debug/"
            }
            else if ( env.GIT_BRANCH == 'master' ) {
              sh "mv ${WORKSPACE}/app/build/outputs/apk/release/*.apk ${WORKSPACE}/app/build/outputs/apk/release/Release-stag-${BUILD_NUMBER}.apk "
              sh "mv ${WORKSPACE}/app/build/outputs/apk/debug/*.apk ${WORKSPACE}/app/build/outputs/apk/debug/Debug-stag-${BUILD_NUMBER}.apk "
              sh "${HOME}/ossutil64 cp ${WORKSPACE}/app/build/outputs/apk/release/Release-stag-${BUILD_NUMBER}.apk oss://android-login-production/Release/"
              sh "${HOME}/ossutil64 cp ${WORKSPACE}/app/build/outputs/apk/debug/Debug-stag-${BUILD_NUMBER}.apk oss://android-login-production/Debug/"
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
