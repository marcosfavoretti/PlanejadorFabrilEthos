pipeline {
    agent any

    environment {
        REPO = 'https://github.com/marcosfavoretti/PlanejadorFabril.git'
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: REPO,
                        credentialsId: 'github-ssh-key' // Garanta que esta credencial existe
                    ]]
                ])
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('planejamento-ethos') {
                    sh 'docker build -t planejamentoapi .'
                }
                dir('PlanejamentoEthosPortal') {
                    sh 'docker build -t planejamentoportal .'
                }
            }
        }

        stage('Run Services with Docker Compose') {
            steps {
                script {
                    writeFile file: 'docker-compose.yml', text: '''
                version: '3.8'
                services:
                backend:
                    image: planejamentoapi
                    build:
                    context: ./planejamento-ethos
                    ports:
                    - "${PORT}:${PORT}"
                    environment:
                    -TZ=${TZ}
                    -SECRET=${SECRET}
                    -EXPIREHOURS=${EXPIREHOURS}
                    -TZ=${TZ}
                    -CHECKPOINT_RANGE=${CHECKPOINT_RANGE}
                    -PORT=${PORT}
                    -BATELADAMAX=${BATELADAMAX}
                    -BD=${BD}
                    -ORACLEHOST=${ORACLEHOST}
                    -ORACLEPORT=${ORACLEPORT}
                    -ORACLEUSER=${ORACLEUSER}
                    -ORACLEPASSWORD=${ORACLEPASSWORD}
                    -ORACLESID=${ORACLESID}
                    -SQLREPO=${SQLREPO}
                    -ESTRUTURA_SERVICE=${ESTRUTURA_SERVICE}
                    -SQLUSER=${SQLUSER}
                    -SQLSENHA=${SQLSENHA}
                    -SQLHOST=${SQLHOST}
                    -SQLDATABASE=${SQLDATABASE}
                    -SQLPORT=${SQLPORT}
                    restart: always
                    
                frontend:
                    image: planejamentoportal
                    build:
                    context: ./PlanejamentoEthosPortal
                    ports:
                    - "8083:80"
                    restart: always
                '''
                    sh "docker-compose up -d --build"
                }
            }
        }
    }

}
