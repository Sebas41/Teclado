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
            echo 'Preparando archivos para despliegue...'
            sh '''
                #!/bin/bash
                set -eux
                mkdir -p dist
                cp index.html dist/
                cp script.js dist/
                cp -r css dist/
            '''
        }
    }

    stage('Deploy to nginx (via SSH password)') {
      when {
        anyOf {
            branch 'main'
            expression { env.GIT_BRANCH == 'origin/main' }
            expression { env.GIT_BRANCH == 'main' }
        }
      }
      steps {
        echo "Desplegando en servidor Nginx (${env.SONAR_HOST_URL})..."
        script {
          // Usa credencial tipo "Username with password"
          withCredentials([usernamePassword(credentialsId: 'jenkins-pass-nginx', passwordVariable: 'SSH_PASS', usernameVariable: 'SSH_USER')]) {
            // Creamos un script temporal para evitar problemas de escape
            sh '''cat > deploy.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail

# Empaqueta el contenido del directorio dist
cd dist
tar czf ../teclado_site.tar.gz *
cd ..

# Instala sshpass si no existe (sin sudo)
if ! command -v sshpass >/dev/null 2>&1; then
  echo "ERROR: sshpass no está instalado en Jenkins"
  echo "Por favor, instálalo en el contenedor de Jenkins"
  exit 1
fi

# Copia al servidor remoto
sshpass -p "$SSH_PASS" scp -o StrictHostKeyChecking=no teclado_site.tar.gz "$SSH_USER@130.131.27.80:/tmp/teclado_site.tar.gz"

# Despliega en el servidor remoto
sshpass -p "$SSH_PASS" ssh -o StrictHostKeyChecking=no "$SSH_USER@130.131.27.80" <<'REMOTE'
  set -euxo pipefail
  DEPLOY_PATH="/var/www/keyboard-app"
  sudo mkdir -p $DEPLOY_PATH
  sudo rm -rf $DEPLOY_PATH/*
  sudo tar xzf /tmp/teclado_site.tar.gz -C $DEPLOY_PATH
  sudo chown -R www-data:www-data $DEPLOY_PATH
  sudo systemctl reload nginx
  rm -f /tmp/teclado_site.tar.gz
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