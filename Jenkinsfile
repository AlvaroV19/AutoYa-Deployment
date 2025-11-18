pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
  }

  environment {
    // Archivos específicos del entorno QA
    COMPOSE_FILE = 'docker-compose.qa.yml'
    ENV_FILE = '.env.qa'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        echo "🌀 Branch actual: ${env.BRANCH_NAME}"
      }
    }

    stage('Load ENV Variables') {
      steps {
        sh """
          echo "📥 Exportando variables desde ${ENV_FILE}..."
          export \$(grep -v '^#' ${ENV_FILE} | xargs)
          echo "🔧 Variables cargadas correctamente."
        """
      }
    }

    stage('Build images') {
      steps {
        sh """
          echo "🚧 Construyendo imágenes Docker para QA..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} build --pull --parallel
        """
      }
    }

    stage('Clean previous environment') {
      steps {
        sh """
          echo "🧹 Eliminando entorno QA previo..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} down -v --remove-orphans || true
        """
      }
    }

    stage('Deploy') {
      steps {
        sh """
          echo "🚀 Desplegando entorno QA..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} up -d --build
        """
      }
    }
  }

  post {
    success {
      echo "✅ Deploy QA exitoso en la rama: ${env.BRANCH_NAME}"
    }
    failure {
      echo "❌ Deploy QA falló en la rama: ${env.BRANCH_NAME}"
    }
  }
}
