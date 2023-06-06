<h1 align='center'>Tutorial Jenkins com Docker</h1>

<h3 align='justify'>Neste tutorial você aprenderá a como utilizar o Jenkins utilizando a ferramenta Docker e como utilizar Pipeline para construção de build numa aplicação ReactJS utilizando um agente Node em Docker</h3>

------------

<h4>Primeiros passos</h4>

<p align='justify'>Utilizaremos o Dockerfile para construir a imagem Docker. <strong>Mas o que é Dockerfile? </strong>Dockerfile é um arquivo de texto que contém uma série de instruções que definem como construir uma imagem Docker.</p>

    FROM jenkins/jenkins:latest-jdk11  # Usa a imagem mais recente do Jenkins com JDK 11
    USER root  # Altera para o usuário root para executar tarefas administrativas
    
    # Atualiza o repositório de pacotes e instala o pacote lsb-release
    RUN apt-get update && apt-get install -y lsb-release
    
    # Faz o download da chave GPG do Docker e a salva em /usr/share/keyrings/docker-archive-keyring.asc
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
        https://download.docker.com/linux/debian/gpg
    
    # Adiciona o repositório do Docker à lista de fontes de pacotes
    RUN echo "deb [arch=$(dpkg --print-architecture) \
        signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
        https://download.docker.com/linux/debian \
        $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    
    # Atualiza o repositório de pacotes novamente e instala o pacote docker-ce-cli
    RUN apt-get update && apt-get install -y docker-ce-cli
    
    # Cria um grupo chamado 'docker' e adiciona o usuário 'jenkins' a ele
    RUN groupadd docker && usermod -aG docker jenkins
    
    # Adiciona o usuário 'jenkins' ao grupo 'root'
    RUN usermod -aG root jenkins
    
    # Volta para o usuário 'jenkins'
    USER jenkins
    
    # Instala os plugins do Jenkins usando a ferramenta jenkins-plugin-cli
    RUN jenkins-plugin-cli --plugins "blueocean:1.25.8 docker-workflow:521.v1a_a_dd2073b_2e"

<p align='justify'>Com o Dockerfile montado, agora precisamos configurar o docker-compose.yml, Mas afinal, <strong>o que é docker-compose.yml?</strong> docker-compose.yml é usado para definir e configurar serviços Docker em um ambiente de múltiplos contêineres. Ele é usado em conjunto com o Docker Compose, que é uma ferramenta para orquestração de contêineres Docker.</p>

    version: '3'  # Versão do formato do arquivo docker-compose
    
    services:  # Define os serviços que serão executados
      jenkins:  # Nome do serviço
        container_name: jenkins  # Nome do contêiner que será criado
        build:  # Configuração de build do contêiner
          context: .  # Define o contexto do build, que é o diretório atual
        ports:
          - 8080:8080  # Mapeia a porta 8080 do host para a porta 8080 do contêiner
        volumes:
          - /home/mateus/Documentos/gcsitsi/jenkins/jenkins_home/:/var/jenkins_home
            # Mapeia o diretório local /home/mateus/Documentos/gcsitsi/jenkins/jenkins_home/
            # para o diretório /var/jenkins_home do contêiner
          - /var/run/docker.sock:/var/run/docker.sock
            # Mapeia o arquivo de soquete do Docker no host (/var/run/docker.sock)
            # para o mesmo local dentro do contêiner (/var/run/docker.sock)

<p>Com essas configurações feitas, agora será necessário criar o diretório <strong>jenkins_home</strong> dentro da raíz do projeto, nele será onde a mágica vai acontecer! 😆 </p>

------------
<h4>Comando para rodar o container docker</h4>
<p align='justify'><strong>O que faz o comando <code>docker compose up</code>?</strong> O Docker Compose é uma ferramenta que permite definir e gerenciar aplicativos Docker compostos por vários contêineres. O arquivo docker-compose.yml descreve a configuração dos serviços, incluindo as imagens dos contêineres, variáveis de ambiente, volumes, portas expostas e outras opções de configuração. Quando você executa o comando`docker-compose up`, o Docker Compose lê o arquivo docker-compose.yml, cria os contêineres necessários com as configurações especificadas e os inicia.</p> 

------------

<h4>No navegador, utilize o [localhost:8080](localhost:8080) para acessar o Jenkins</h4>  

- Próximos passos:
  - **Utilize a chave de acesso para desbloquear o Jenkins**
    - Onde encontro? Ela será mostrada no saída do terminal, você também poderá acessá-la navegando pelo caminho abaixo:
      - /var/lib/jenkins/secrets/initialAdminPassword
  - **Instale os plug-ins recomendados**
    - A instalação poderá demorar um pouco
  - **Crie um usuário administrador do Jenkins**
    - Atente-se para as informações preenchidas
  - **Finalizado todos esses passos, o Jenkins estará pronto para uso**

------------

<h4>Criando uma Pipeline com Build em uma aplicação ReactJS utilizando agente Node em Docker</h4>

- Próximos passos:
 - **Crie uma nova tarefa**
   - Adicione um nome e nas opções selecione **Pipeline**
 - **Nas configurações, mais abaixo em Pipeline:**
   - Selecione Pipeline script from SCM
   - Em SCM selecione GIT e insira o link do GIT
     - Atente-se, no final da URL é necessário que haja .git
        - Repositório que você pode utilizar como exemplo: https://github.com/rhavymaia/acomidadobebe-landingpage.git

 - **Em \_branch to build\_**
   - Mude de Master para Main
     - Isso é uma boa prática atualmente
 - **Salve e execute a Pipeline**
   - Neste repositório exemplo, contém o **Jenkinsfile** com um agente Node no Docker, isso se faz necessário para que a Pipeline execute o ReactJS, tendo em vista que o Jenkins possuí suporte apenas para Java

<h4>Configuração do Jenkinsfile</h4>

    pipeline {
        agent {
            docker {            
                // Define a imagem do Docker que será usada como ambiente de execução
                image 'node:20.2.0-alpine3.17'
                // Define os argumentos do contêiner Docker, mapeando a porta 3000 do contêiner para a porta 3000 do host
                args '-p 3000:3000' 
            }
        }
        stages {
            stage('Build') { 
                steps {
                    // Instala as dependências do projeto executando o comando 'npm install'
                    sh 'npm install' 
                    // Executa o comando de construção do projeto, com a variável de ambiente CI definida como 'false'
                    sh 'CI=false npm run build'
                }
            }
        }
    }


