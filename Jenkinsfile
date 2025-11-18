pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
  }

  environment {
    // Selección dinámica de archivos según ambiente
    COMPOSE_FILE = "docker-compose.dev.yml"
    ENV_FILE = ".env.dev"
    BRANCH_NAME = "dev"
  }

  stages {
    stage('Checkout') {
      steps {
        // Jenkins Multibranch ya hace el checkout automáticamente,
        // pero esto garantiza que tengamos la última versión
        checkout scm
        echo "🌀 Branch actual: ${env.BRANCH_NAME}"
      }
    }

    stage('Build images') {
      steps {
        sh """
          # Extrae la URL de API del .env correspondiente
          API_URL=\$(grep VITE_API_URL \$ENV_FILE | cut -d '=' -f2-)
          echo "API_URL=\$API_URL"
          # Build frontend con la variable correcta
          docker build -t autoya-frontend --build-arg VITE_API_URL=\$API_URL -f frontend/Dockerfile frontend
          # Build otros servicios normalmente
          docker compose -f \$COMPOSE_FILE --env-file \$ENV_FILE build --pull --parallel
        """
      }
    }

    stage('Deploy') {
      steps {
        sh """
          docker compose -f \$COMPOSE_FILE --env-file \$ENV_FILE down -v || true
          docker compose -f \$COMPOSE_FILE --env-file \$ENV_FILE up -d --build
        """
      }
    }
  }

  post {
    success {
      echo "✅ Deploy successful on branch: ${env.BRANCH_NAME}"
    }
    failure {
      echo "❌ Deploy failed on branch: ${env.BRANCH_NAME}"
    }
  }
}
