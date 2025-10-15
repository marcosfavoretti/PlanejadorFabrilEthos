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
                        url: 'https://github.com/marcosfavoretti/PlanejadorFabril.git', // URL do seu repositório principal
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
                // O comando 'docker-compose up' com '--build' força a reconstrução das imagens se houver mudanças.
                // '-d' (detached) faz com que os contêineres rodem em segundo plano.
                // Usamos o seu docker-compose.yml original, que é a prática recomendada.
                sh 'docker-compose up -d --build'
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