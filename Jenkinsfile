pipeline {
  agent any

  environment {
    SONAR_HOST_URL = 'http://130.213.10.92:9000'
    SONAR_TOKEN    = credentials('SONAR_TOKEN')
  }

  stages {
    stage('Checkout') {
      steps {
        echo "Clonando repositorio..."
        checkout scm
      }
    }

    stage('SonarQube analysis') {
      steps {
        echo "Ejecutando análisis estático con SonarQube..."
        // Ejecuta sonar-scanner (debe estar en PATH del agente Jenkins)
        script {
            def scannerHome = tool 'SonarScanner'
            withSonarQubeEnv('SonarQube') {
                sh """
                ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=keyboard-app \
                    -Dsonar.projectName="Keyboard App" \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_TOKEN} \
                    -Dsonar.exclusions=**/node_modules/**,**/*.map,**/dist/**
                """
            }
        }
      }
    }

    stage('Build static files') {
      steps {
        echo "Preparando archivos para despliegue..."
        sh '''
          set -euxo pipefail
          rm -rf dist && mkdir -p dist
          cp -v index.html dist/
          [ -f script.js ] && cp -v script.js dist/
          [ -d css ] && cp -rv css dist/
        '''
      }
    }

    stage('Deploy to nginx (via SSH password)') {
      when { branch 'main' }
      steps {
        echo "Desplegando en servidor Nginx (${env.SONAR_HOST_URL})..."
        script {
          // Usa credencial tipo "Username with password"
          withCredentials([usernamePassword(credentialsId: 'jenkins-pass-nginx', passwordVariable: 'SSH_PASS', usernameVariable: 'SSH_USER')]) {
            // Creamos un script temporal para evitar problemas de escape
            sh '''cat > deploy.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail

# Empaqueta el contenido actual
git archive --format=tar.gz -o teclado_site.tar.gz HEAD

# Asegura sshpass en el agente
if ! command -v sshpass >/dev/null 2>&1; then
  echo "Instalando sshpass..."
  if [ -f /etc/debian_version ]; then
    sudo apt-get update && sudo apt-get install -y sshpass
  else
    echo "No se pudo instalar sshpass automáticamente. Instálalo manualmente." >&2
    exit 1
  fi
fi

# Copia al servidor remoto y despliega
sshpass -p "$SSH_PASS" scp -o StrictHostKeyChecking=no teclado_site.tar.gz "$SSH_USER@130.131.27.80:/tmp/teclado_site.tar.gz"

sshpass -p "$SSH_PASS" ssh -o StrictHostKeyChecking=no "$SSH_USER@130.131.27.80" <<'REMOTE'
  set -euxo pipefail
  DEPLOY_PATH="/var/www/keyboard-app"
  sudo mkdir -p $DEPLOY_PATH
  sudo rm -rf $DEPLOY_PATH/*
  sudo tar xzf /tmp/teclado_site.tar.gz -C $DEPLOY_PATH
  sudo chown -R www-data:www-data $DEPLOY_PATH
  sudo systemctl reload nginx
REMOTE

rm -f teclado_site.tar.gz
SH

chmod +x deploy.sh
bash ./deploy.sh
rm -f deploy.sh
'''
          }
        }
      }
    }
  }

  post {
    success {
      echo '✅ Pipeline completado correctamente.'
      echo '🌐 App disponible en: http://130.131.27.80/keyboard/'
    }
    failure {
      echo '❌ Pipeline falló. Revisa los logs de Jenkins.'
    }
  }
}