pipeline {
    agent any
    
    environment {
        SONAR_HOST = 'http://130.213.10.92:9000'
        APP_NAME = 'keyboard-app'
        DEPLOY_PATH = '/var/www/keyboard-app'
        NGINX_SERVER = '130.131.27.80'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Clonando repositorio...'
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Verificando archivos...'
                sh '''
                    ls -la
                    if [ -f "package.json" ]; then
                        npm install
                    fi
                '''
            }
        }
        
        stage('Code Quality Analysis') {
            steps {
                echo 'Ejecutando análisis con SonarQube...'
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=${APP_NAME} \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=${SONAR_HOST} \
                                -Dsonar.javascript.file.suffixes=.js \
                                -Dsonar.css.file.suffixes=.css \
                                -Dsonar.exclusions=**/node_modules/**,**/*.map
                        """
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo 'Esperando resultado de Quality Gate...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build') {
            steps {
                echo 'Preparando archivos para deploy...'
                sh '''
                    mkdir -p dist
                    cp index.html dist/
                    cp script.js dist/
                    cp -r css dist/
                '''
            }
        }
        
        stage('Deploy to Nginx') {
            steps {
                echo 'Desplegando aplicación en Nginx...'
                sshagent(['nginx-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no adminuser@${NGINX_SERVER} '
                            sudo mkdir -p ${DEPLOY_PATH}
                            sudo chown -R adminuser:adminuser ${DEPLOY_PATH}
                        '
                        scp -r dist/* adminuser@${NGINX_SERVER}:${DEPLOY_PATH}/
                        ssh adminuser@${NGINX_SERVER} '
                            sudo docker exec nginx nginx -s reload
                        '
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline ejecutado exitosamente!'
            echo "🌐 Aplicación disponible en: http://${NGINX_SERVER}/keyboard/"
        }
        failure {
            echo '❌ Pipeline falló. Revisa los logs.'
        }
        always {
            cleanWs()
        }
    }
}