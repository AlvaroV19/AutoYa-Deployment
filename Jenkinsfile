pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
  }

  environment {
    // Archivos específicos del entorno MAIN
    COMPOSE_FILE = 'docker-compose.main.yml'
    ENV_FILE     = '.env.main'

    // Repositorios externos
    FRONTEND_REPO = 'https://github.com/AlvaroV19/AutoYa-Frontend.git'
    BACKEND_REPO  = 'https://github.com/AlvaroV19/AutoYa-Backend.git'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        echo "🌀 Branch actual: ${env.BRANCH_NAME}"
      }
    }

    stage('Clone Frontend & Backend') {
      steps {
        echo "📥 Clonando repos de Frontend y Backend para MAIN..."

        sh """
          rm -rf frontend backend

          git clone ${FRONTEND_REPO} frontend
          git clone ${BACKEND_REPO} backend

          echo '✅ Repositorios clonados en carpetas frontend/ y backend/'
        """
      }
    }

    stage('Load ENV Variables') {
      steps {
        sh """
          echo "📥 Exportando variables desde ${ENV_FILE}..."
          export \$(grep -v '^#' ${ENV_FILE} | xargs)
          echo "🔧 Variables cargadas correctamente (solo para este step)."
        """
      }
    }

    stage('Build images') {
      steps {
        sh """
          echo "🚧 Construyendo imágenes Docker para MAIN..."

          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} build --pull --parallel
        """
      }
    }

    stage('Clean previous environment') {
      steps {
        sh """
          echo "🧹 Eliminando entorno MAIN previo..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} down -v --remove-orphans || true
        """
      }
    }

    stage('Deploy') {
      steps {
        sh """
          echo "🚀 Desplegando entorno MAIN (producción)..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} up -d --build
        """
      }
    }
  }

  post {
    success {
      echo "✅ Deploy MAIN exitoso en la rama: ${env.BRANCH_NAME}"
    }
    failure {
      echo "❌ Deploy MAIN falló en la rama: ${env.BRANCH_NAME}"
    }
  }
}
