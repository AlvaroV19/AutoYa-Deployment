pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
  }

  environment {
    // El Jenkinsfile detecta automáticamente la rama
    BRANCH_NAME = "${env.BRANCH_NAME}"

    // Selección dinámica según entorno
    COMPOSE_FILE = "docker-compose.${BRANCH_NAME}.yml"
    ENV_FILE     = ".env.${BRANCH_NAME}"

    // Repos externos
    FRONTEND_REPO = "https://github.com/AlvaroV19/AutoYa-Frontend.git"
    BACKEND_REPO  = "https://github.com/AlvaroV19/AutoYa-Backend.git"
  }

  stages {

    stage('Checkout Deploy Repo') {
      steps {
        checkout scm
        echo "🌀 Rama actual del deploy: ${BRANCH_NAME}"
      }
    }

    stage('Clone Frontend & Backend') {
      steps {
        echo "📥 Clonando Frontend y Backend en la rama: ${BRANCH_NAME}"

        sh """
          rm -rf frontend backend

          git clone --branch ${BRANCH_NAME} --single-branch ${FRONTEND_REPO} frontend
          git clone --branch ${BRANCH_NAME} --single-branch ${BACKEND_REPO} backend
        """
      }
    }

    stage('Build images') {
      steps {
        sh """
          echo "🔍 Obteniendo VITE_API_URL desde ${ENV_FILE}..."
          API_URL=\$(grep VITE_API_URL ${ENV_FILE} | cut -d '=' -f2-)
          echo "🌐 API_URL=\$API_URL"

          echo "🚧 Construyendo imagen del FRONTEND..."
          docker build -t autoya-frontend --build-arg VITE_API_URL=\$API_URL -f frontend/Dockerfile frontend

          echo "🚧 Construyendo servicios BACKEND..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} build --pull --parallel
        """
      }
    }

    stage('Deploy') {
      steps {
        sh """
          echo "🛑 Deteniendo entorno ${BRANCH_NAME}..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} down -v || true

          echo "🚀 Levantando entorno ${BRANCH_NAME}..."
          docker compose -f ${COMPOSE_FILE} --env-file ${ENV_FILE} up -d --build
        """
      }
    }
  }

  post {
    success {
      echo "✅ Deploy successful on branch: ${BRANCH_NAME}"
    }
    failure {
      echo "❌ Deploy failed on branch: ${BRANCH_NAME}"
    }
  }
}
