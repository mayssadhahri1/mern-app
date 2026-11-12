pipeline {
  agent any
  options { skipDefaultCheckout(true) }
  triggers { pollSCM('H/5 * * * *') }
  environment {
    IMAGE_SERVER = 'mayssadhahri1/mern-server'
    IMAGE_CLIENT = 'mayssadhahri1/mern-client'
    TRIVY_IMAGE  = 'aquasec/trivy:0.76.2'
    TRIVY_EXIT_CODE = '0'
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/mayssadhahri1/mern-app.git'
      }
    }
    stage('SERVER : build, scan, push') {
      when {
        anyOf {
          changeset 'server/**'
          changeset 'Jenkinsfile'
          expression { env.BUILD_NUMBER == '1' }
        }
      }
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub',
          usernameVariable: 'DH_USER',
          passwordVariable: 'DH_PASS'
        )]) {
          sh '''
            set -eu
            docker build -t "$IMAGE_SERVER:$BUILD_NUMBER" server
            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
              "$TRIVY_IMAGE" image "$IMAGE_SERVER:$BUILD_NUMBER" > trivy_server.txt
            cat trivy_server.txt
            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
              "$TRIVY_IMAGE" image --severity HIGH,CRITICAL --exit-code "$TRIVY_EXIT_CODE" \
              "$IMAGE_SERVER:$BUILD_NUMBER"
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push "$IMAGE_SERVER:$BUILD_NUMBER"
          '''
        }
      }
    }
    stage('CLIENT : build, scan, push') {
      when {
        anyOf {
          changeset 'client/**'
          changeset 'Jenkinsfile'
          expression { env.BUILD_NUMBER == '1' }
        }
      }
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub',
          usernameVariable: 'DH_USER',
          passwordVariable: 'DH_PASS'
        )]) {
          sh '''
            set -eu
            docker build -t "$IMAGE_CLIENT:$BUILD_NUMBER" client
            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
              "$TRIVY_IMAGE" image "$IMAGE_CLIENT:$BUILD_NUMBER" > trivy_client.txt
            cat trivy_client.txt
            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
              "$TRIVY_IMAGE" image --severity HIGH,CRITICAL --exit-code "$TRIVY_EXIT_CODE" \
              "$IMAGE_CLIENT:$BUILD_NUMBER"
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push "$IMAGE_CLIENT:$BUILD_NUMBER"
          '''
        }
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'trivy_*.txt', allowEmptyArchive: true
      sh '''
        docker logout || true
        docker image rm "$IMAGE_SERVER:$BUILD_NUMBER" \
                        "$IMAGE_CLIENT:$BUILD_NUMBER" 2>/dev/null || true
      '''
    }
  }
}