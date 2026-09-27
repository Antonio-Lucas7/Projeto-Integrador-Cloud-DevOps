# Projeto Integrador — Cloud & DevOps

## Integrantes

- Antonio Lucas Florêncio da Silva
- Matheus Souza Talon

## Objetivo

Este projeto foi desenvolvido para aplicar, na prática, conceitos de Cloud Computing e DevOps, envolvendo desenvolvimento, versionamento, containerização, publicação em nuvem, configuração de DNS, HTTPS, CI/CD e monitoramento.

A aplicação consiste em uma página web simples publicada em uma máquina virtual na Oracle Cloud, utilizando Docker e Nginx.

## Tecnologias utilizadas

- HTML5
- Nginx
- Docker
- Docker Compose
- Git
- GitHub
- GitHub Actions
- Oracle Cloud Infrastructure (OCI)
- DuckDNS
- Certbot / Let's Encrypt
- Uptime Kuma
- Ubuntu 20.04 LTS

## Arquitetura do projeto

O projeto utiliza a seguinte arquitetura:

```
Usuário
   │
   ▼
Internet
   │
   ▼
DuckDNS
   │
   ▼
Oracle Cloud
   │
   ▼
Máquina Virtual Ubuntu
   │
   ├── Nginx + Docker
   │      ├── HTTP :80
   │      └── HTTPS :443
   │
   └── Uptime Kuma
          └── Monitoramento :3001
```

## Fluxo de acesso

   1. O usuário acessa o domínio configurado no DuckDNS.
   2. O DNS direciona o domínio para o IP público da máquina virtual.
   3. A Oracle Cloud recebe a requisição nas portas 80 ou 443.
   4. O Nginx recebe a requisição dentro do container Docker.
   5. O acesso HTTP é redirecionado para HTTPS.
   6. O Nginx entrega a página HTML da aplicação.
   7. O Uptime Kuma monitora a disponibilidade da aplicação.

## Estrutura de pastas

   Projeto-Integrador-Cloud-DevOps/
   ├── .github/
   │   └── workflows/
   │       └── deploy.yml
   ├── nginx/
   │   └── default.conf
   ├── site/
   │   └── index.html
   ├── .gitignore
   ├── Dockerfile
   ├── docker-compose.yaml
   └── README.md

## Descrição dos principais arquivos

   - site/index.html — página principal da aplicação.
   - Dockerfile — define a imagem Docker da aplicação.
   - docker-compose.yaml — configura os containers da aplicação e do monitoramento.
   - nginx/default.conf — configura o Nginx, HTTP, HTTPS e certificado SSL/TLS.
   - .github/workflows/deploy.yml — define a pipeline de CI/CD.
   - .gitignore — impede o versionamento de arquivos sensíveis ou desnecessários.
   - README.md — documentação do projeto.


# Docker

   A aplicação foi containerizada utilizando Docker e Nginx.


## Dockerfile

   O arquivo Dockerfile utiliza a imagem oficial do Nginx:
      FROM nginx:alpine
      COPY site /usr/share/nginx/html

   A aplicação HTML é copiada para o diretório padrão utilizado pelo Nginx dentro do container.

## Docker Compose

   O docker-compose.yaml é responsável por executar os serviços do projeto:
   - web: aplicação web utilizando Nginx.
   - monitor: monitoramento utilizando Uptime Kuma.
   O serviço web disponibiliza as portas:
   - 80 — HTTP
   - 443 — HTTPS
   O Uptime Kuma utiliza a porta 3001, porém ela está vinculada somente ao endereço local da máquina:
      127.0.0.1:3001

   Isso impede que o painel de monitoramento seja acessado diretamente pela Internet.
   Executando com Docker Compose
   Para iniciar os serviços:
      docker compose up -d --build

   Para verificar os containers em execução:
      docker ps

   Para interromper os serviços:
      docker compose down

## Containers utilizados

   projeto-devops
   └── Nginx
       ├── HTTP :80
       └── HTTPS :443

   uptime-kuma
   └── Monitoramento :3001

# Cloud Computing

   A aplicação está hospedada na Oracle Cloud Infrastructure (OCI).

   Infraestrutura utilizada:
   - Provedor: Oracle Cloud Infrastructure (OCI)
   - Sistema operacional: Ubuntu 20.04.6 LTS
   - Máquina virtual: Compute Instance
   - IP público: IP público da VM Oracle Cloud
   - Rede: VCN projeto-devops
   - Subnet: 10.0.0.0/24
   - Internet Gateway: igw-projeto-devops

   Portas utilizadas:
      Porta	|  Protocolo  |  Finalidade
      22	   |     TCP	  |   Acesso SSH à máquina virtual
      80	   |     TCP     |   Acesso HTTP
      443	|     TCP	  |   Acesso HTTPS


   A porta 3001 utilizada pelo Uptime Kuma não é exposta diretamente à Internet. O serviço está vinculado ao endereço local:
      127.0.0.1:3001

# DNS

   O projeto utiliza o DuckDNS para fornecer um domínio público para a aplicação.
   Domínio utilizado
   projeto-devops-antonio.duckdns.org

## Fluxo do DNS

   Usuário
      │
      ▼
   projeto-devops-antonio.duckdns.org
      │
      ▼
   DuckDNS
      │
      ▼
   IP público da VM Oracle Cloud
      │
      ▼
   Máquina Virtual
      │
      ▼
   Nginx
      │
      ▼
   Aplicação Docker

   Dessa forma, o usuário não precisa acessar diretamente o endereço IP da máquina virtual, podendo utilizar o domínio configurado.

# HTTPS

   A aplicação utiliza HTTPS para proteger a comunicação entre o usuário e o servidor.
   O certificado SSL/TLS foi obtido utilizando o Let's Encrypt, através do Certbot.
   Certificado
   O certificado foi configurado para o domínio:
   projeto-devops-antonio.duckdns.org

   Os arquivos utilizados pelo Nginx são:
      /etc/letsencrypt/live/projeto-devops-antonio.duckdns.org/fullchain.pem
      /etc/letsencrypt/live/projeto-devops-antonio.duckdns.org/privkey.pem

   Os certificados são disponibilizados ao container Nginx através do Docker Compose.
   Redirecionamento HTTP → HTTPS
   O Nginx está configurado para redirecionar automaticamente as requisições HTTP para HTTPS.
      HTTP :80
         │
         ▼
      Nginx
         │
         ▼
      HTTPS :443
         │
         ▼
      Aplicação

## Aplicação

   Ao acessar:
      http://projeto-devops-antonio.duckdns.org

   o usuário é redirecionado para:
      https://projeto-devops-antonio.duckdns.org

## Validação

   O funcionamento do HTTPS foi validado diretamente na máquina virtual utilizando:
      curl -I https://projeto-devops-antonio.duckdns.org

   O servidor retornou uma resposta:
      HTTP/1.1 200 OK

   confirmando que a aplicação estava acessível através de HTTPS.

# CI/CD

   O projeto utiliza GitHub Actions para automatizar o processo de integração contínua e implantação contínua (CI/CD).
   A pipeline é executada automaticamente sempre que ocorre um push na branch main.

## Fluxo da pipeline

   Git Push
      │
      ▼
   GitHub Actions
      │
      ▼
   Validação do Docker Compose
      │
      ▼
   Build da imagem Docker
      │
      ▼
   Deploy via SSH
      │
      ▼
   Oracle Cloud
      │
      ▼
   Docker Compose
      │
      ▼
   Aplicação atualizada

## Etapas

   A pipeline possui duas etapas principais:
   1. Build e validação
   A primeira etapa:
   - baixa o código do repositório;
   - valida a configuração do Docker Compose;
   - realiza o build da imagem Docker.
   Com isso, erros básicos de configuração podem ser identificados antes da implantação.

   2. Deploy
   Após a conclusão da etapa de validação, a segunda etapa realiza o deploy na máquina virtual da Oracle Cloud.
   O processo:
      1. estabelece uma conexão SSH com a máquina virtual;
      2. acessa o diretório do projeto;
      3. executa git pull para obter a versão mais recente;
      4. executa docker compose up -d --build;
      5. remove imagens Docker não utilizadas.

## Workflow

   O arquivo responsável pela pipeline está localizado em:
      .github/workflows/deploy.yml

   As credenciais utilizadas no processo de deploy são armazenadas como GitHub Secrets e não ficam disponíveis no código-fonte.
   Os secrets utilizados são:
      VM_HOST
      VM_USER
      VM_SSH_KEY

# Monitoramento

   O projeto utiliza o Uptime Kuma para monitorar a disponibilidade da aplicação.
   O Uptime Kuma é executado em um container Docker separado do container da aplicação.

## Monitoramento da aplicação

   Foi configurado um monitor do tipo HTTP(s) para verificar o endereço:
      https://projeto-devops-antonio.duckdns.org

   O monitor realiza verificações periódicas da aplicação e informa se o serviço está disponível.
   A frequência configurada para verificação é de:
      60 segundos

   Configuração
   O serviço de monitoramento utiliza a porta 3001, porém o acesso foi restringido ao próprio servidor:
      127.0.0.1:3001

## Acesso ao painel

   Como a porta 3001 está vinculada somente ao endereço local da máquina virtual, o painel pode ser acessado através de um túnel SSH.
   Exemplo:
      ssh -i ~/.ssh/sua-chave.key -L 3001:127.0.0.1:3001 ubuntu@IP_DA_VM
   Após estabelecer a conexão, o painel pode ser acessado localmente através de:
      http://localhost:3001

# Containers

   A infraestrutura possui os seguintes containers principais:
   projeto-devops
   └── Nginx
       └── Aplicação web

   uptime-kuma
   └── Monitoramento da aplicação

   O monitoramento permite verificar continuamente se a aplicação está respondendo através de HTTPS.

# Segurança

   Foram adotadas medidas básicas de segurança para reduzir a exposição desnecessária da infraestrutura.

## Firewall e portas

   A máquina virtual possui regras de entrada configuradas para permitir somente as portas necessárias:

      Porta	 |  Protocolo   |  Finalidade
      22	    |    TCP	    |    Acesso SSH
      80  	 |    TCP       |    HTTP
      443	 |    TCP	    |    HTTPS
   
   A porta 3001, utilizada pelo Uptime Kuma, não é liberada para acesso externo. O serviço está vinculado somente ao endereço:
      127.0.0.1:3001

# HTTPS

   A aplicação utiliza HTTPS através de certificado SSL/TLS emitido pelo Let's Encrypt.
   As requisições HTTP são redirecionadas para HTTPS pelo Nginx.
   Proteção de informações sensíveis
   Informações sensíveis não são armazenadas diretamente no código-fonte.
   O arquivo .gitignore impede o versionamento de arquivos como:
      .env
      *.key
      *.pem

   Além disso, as credenciais utilizadas pelo GitHub Actions são armazenadas através de GitHub Secrets.
   A chave privada SSH e os certificados SSL/TLS não são enviados para o repositório público.
   Instalação e execução
   Para executar o projeto, é necessário possuir:
      - Git
      - Docker
      - Docker Compose
   Clonar o repositório
   git clone git@github.com:Antonio-Lucas7/Projeto-Integrador-Cloud-DevOps.git

   Acessar o diretório:
      cd Projeto-Integrador-Cloud-DevOps

   Executar com Docker Compose
   Para construir a imagem e iniciar os serviços:
      docker compose up -d --build

   Verificar os containers:
      docker ps

   Para interromper os serviços:
      docker compose down

   Observação
   A configuração de HTTPS utiliza os certificados do Let's Encrypt armazenados no servidor de produção. Portanto, a configuração completa de HTTPS depende da existência dos    certificados no ambiente em que o projeto for executado.

# Deploy

   O deploy da aplicação é realizado automaticamente através da pipeline de CI/CD do GitHub Actions.
   O processo ocorre após um push na branch main.
   Processo de implantação
      Desenvolvimento
            │
            ▼
      Git Commit
            │
            ▼
      Git Push
            │
            ▼
      GitHub
            │
            ▼
      GitHub Actions
            │
            ├── Validação
            ├── Build
            └── Deploy via SSH
                    │
                    ▼
             Oracle Cloud
                    │
                    ▼
             Docker Compose
                    │
                    ▼
              Aplicação
   
   Durante o deploy, a máquina virtual executa:
      git pull origin main
   
   e posteriormente:
      docker compose up -d --build
   
   Dessa forma, a aplicação é reconstruída e reiniciada utilizando a versão mais recente disponível na branch main.
   Recuperação em caso de falha
   Em caso de falha na aplicação, os containers podem ser verificados através do comando:
      docker ps
   
   Os logs do container da aplicação podem ser consultados com:
      docker logs projeto-devops
   
   Para reiniciar os serviços:
      docker compose restart
   
   Caso seja necessário reconstruir completamente os containers:
      docker compose down
      docker compose up -d --build
   
   Também é possível verificar os logs do Uptime Kuma:
      docker logs uptime-kuma

   O Uptime Kuma auxilia na identificação de indisponibilidade da aplicação através do monitoramento periódico do endereço HTTPS.

# Versionamento e histórico

   O projeto utiliza Git para controle de versão e GitHub para armazenamento remoto do código-fonte.
   O repositório é público:
      https://github.com/Antonio-Lucas7/Projeto-Integrador-Cloud-DevOps

   Os commits foram organizados de acordo com as alterações realizadas durante o desenvolvimento.
   Exemplos de alterações registradas:
      - criação da estrutura inicial do projeto;
      - configuração do Docker;
      - configuração do CI/CD;
      - adição do monitoramento;
      - atualização dos integrantes;
      - configuração da infraestrutura de nuvem.
   O histórico completo das alterações pode ser consultado diretamente no repositório GitHub.

# Histórico de alterações

   Versão	Alteração
      1.0	Criação da estrutura inicial do projeto
      1.1	Adição da aplicação web
      1.2	Containerização com Docker
      1.3	Configuração da infraestrutura na Oracle Cloud
      1.4	Configuração do domínio DuckDNS
      1.5	Configuração do HTTPS com Let's Encrypt
      1.6	Implementação da pipeline CI/CD
      1.7	Implementação do monitoramento com Uptime Kuma
      1.8	Atualização dos integrantes e documentação


   O histórico detalhado das alterações pode ser consultado através dos commits do repositório Git.

# Acesso à aplicação

   Aplicação
   A aplicação está disponível publicamente através do endereço:
      https://projeto-devops-antonio.duckdns.org

   Repositório
   O código-fonte está disponível em:
      https://github.com/Antonio-Lucas7/Projeto-Integrador-Cloud-DevOps

# Infraestrutura

   A aplicação está hospedada em uma máquina virtual da Oracle Cloud Infrastructure (OCI), utilizando Docker e Nginx.
   O domínio público é fornecido pelo DuckDNS e o acesso HTTPS é protegido por certificado SSL/TLS do Let's Encrypt.

