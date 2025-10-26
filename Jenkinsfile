pipeline {
  agent any

  environment {
    IMAGE = "kibojago/demo-app"
    TAG = "latest"
    DOCKER_CRED = "docker-hub"
    KUBECONFIG_CRED = "kubeconfig-dev"
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy-jenkins1"
  }

  stages {

    stage('Checkout Source Code') {
      steps {
        echo "📦 Checking out source code..."
        git url: 'https://github.com/perdi77/studycase-jenkins.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "🛠️ Building Docker image ${IMAGE}:${TAG}..."
          sh """
            docker build -t ${IMAGE}:${TAG} .
          """
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKER_CRED}",
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          script {
            echo "📤 Pushing image to Docker Hub..."
            sh """
              echo "$PASS" | docker login -u "$USER" --password-stdin
              docker push ${IMAGE}:${TAG}
              docker logout
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes (Helm)') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG')]) {
          script {
            echo "🚀 Deploying to Kubernetes..."
            sh """
              helm upgrade --install ${HELM_RELEASE} ./helm-chart \
                --kubeconfig $KUBECONFIG \
                --set image.repository=${IMAGE} \
                --set image.tag=${TAG} \
                --namespace ${NAMESPACE} --create-namespace
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline Sukses: Aplikasi berhasil dibuild, dipush, dan dideploy ke Kubernetes!"
    }
    failure {
      echo "❌ Pipeline Gagal: Periksa log build untuk melihat penyebab error."
    }
  }
}
