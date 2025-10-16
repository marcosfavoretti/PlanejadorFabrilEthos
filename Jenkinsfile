// Jenkinsfile
pipeline {
    // 1. Agente de Execução
    // O agente (máquina) que executará o pipeline precisa ter Git, Docker e Docker Compose instalados.
    agent any

    // 2. Etapas (Stages) do Pipeline
    stages {
        // Etapa 1: Clonar Repositório e Submodules
        stage('Checkout') {
            steps {
                // Clona o repositório principal e, crucialmente, inicializa e atualiza os submodules (backend e frontend).
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/marcosfavoretti/PlanejadorFabrilEthos.git', // URL do seu repositório principal
                        credentialsId: 'github-ssh-key' // ID da credencial SSH configurada no Jenkins
                    ]],
                    extensions: [[$class: 'SubmoduleOption', disableSubmodules: false, recursiveSubmodules: true, reference: '', trackingSubmodules: false, parentCredentials: true]]
                ])
            }
        }

        // Etapa 2: Construir e Subir os Serviços com Docker Compose
        // Esta etapa utiliza o seu arquivo docker-compose.yml para construir as imagens e iniciar os contêineres.
        stage('Build and Run Docker Compose') {
            steps {
                // Carrega os dois arquivos secretos em variáveis de ambiente temporárias
                withCredentials([
                    file(credentialsId: 'ethos_planejamento_infra_env', variable: 'INFRA_ENV_FILE'),
                    file(credentialsId: 'ethos_planejador_frontend', variable: 'FRONTEND_ENV_FILE'),
                    file(credentialsId: 'ethos_planejador_backend', variable: 'BACKEND_ENV_FILE')
                ]) {
                    // O comando 'docker-compose' usa a flag '--env-file' para cada arquivo.
                    // As variáveis de ambos os arquivos serão carregadas.
                    sh 'docker-compose --env-file $INFRA_ENV_FILE --env-file $FRONTEND_ENV_FILE --env-file $BACKEND_ENV_FILE up -d --build'
                }
            }
        }
        
        // Etapa 3: Verificar a Saúde dos Serviços (Opcional, mas recomendado)
        // O seu docker-compose.yml já define um healthcheck para o serviço da API.
        // Podemos adicionar um passo para verificar se a API está saudável antes de finalizar.
        stage('Health Check') {
            steps {
                // Este script espera um pouco e depois verifica o status do contêiner da API.
                // Se o healthcheck falhar após algumas tentativas, o pipeline falhará.
                sh '''
                    echo "Aguardando a API ficar saudável..."
                    sleep 15
                    
                    HEALTH_STATUS=$(docker inspect --format='{{.State.Health.Status}}' planejamento_ethos_api)
                    
                    if [ "$HEALTH_STATUS" = "healthy" ]; then
                        echo "API está saudável!"
                    else
                        echo "Health check da API falhou. Status: $HEALTH_STATUS"
                        exit 1
                    fi
                '''
            }
        }
    }

}