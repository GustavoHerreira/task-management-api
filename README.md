# API de Gerenciamento de Tarefas com Deploy na Nuvem Azure
![.NET](https://img.shields.io/badge/.NET-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=c-sharp&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Microsoft Azure](https://img.shields.io/badge/Microsoft_Azure-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)


Este projeto apresenta uma solução completa de back-end para um sistema de gerenciamento de tarefas. Nascido como um desafio de projeto do Bootcamp .NET da DIO/GFT, ele evoluiu para uma aplicação totalmente funcional com deploy realizado na Microsoft Azure, demonstrando um ciclo completo de desenvolvimento, containerização e publicação na nuvem. A arquitetura final utiliza **`Infraestrutura como Serviço (IaaS)`** para a aplicação (API) e **`Plataforma como Serviço (PaaS)`** para o banco de dados PostgreSQL.

## 🚀 API em Produção!
A aplicação está online e totalmente funcional. Pode testá-la ao vivo através da documentação interativa do Swagger:
[Clique aqui para acessar a API na Azure](http://132.196.1.215/swagger/index.html#/Tarefa)
> Observação: A aplicação roda em uma infraestrutura do nível gratuito da Azure. A primeira requisição pode levar alguns segundos para "acordar" os recursos.

## 🏗️ Arquitetura da Solução na Nuvem

A solução foi projetada seguindo o princípio de "defesa em profundidade", utilizando múltiplos recursos da Azure para garantir segurança, escalabilidade e manutenibilidade.

**O fluxo da arquitetura é o seguinte:**
1.  **Firewall de Entrada (`Network Security Group`):** A primeira camada de segurança é o NSG associado à rede da VM. Ele atua como um firewall, permitindo o tráfego da `Internet` apenas na porta **HTTP (80)** e bloqueando todo o resto.
2.  **Servidor de Aplicação (`Azure VM - IaaS`):** Uma Máquina Virtual Linux (Ubuntu) recebe as requisições permitidas pelo NSG. Dentro dela, a **API .NET roda isolada em um contentor Docker**, garantindo um ambiente de execução consistente e portátil.
3.  **Rede Privada Segura (`Azure VNet`):** A comunicação entre a API e o Banco de Dados **não acontece pela internet pública**. Ambos os recursos estão localizados na mesma **Rede Virtual (VNet)**, comunicando-se de forma rápida e segura através da rede interna da Microsoft.
4.  **Banco de Dados Gerenciado (`Azure DB for PostgreSQL - PaaS`):** O banco de dados foi configurado com **Acesso Privado**. Isto significa que ele **não possui um endereço de IP público** e é totalmente inacessível pela internet. Ele só aceita ligações que se originam de dentro da sua VNet, como a da nossa VM, tornando a camada de dados extremamente segura.

## 📝 Contexto do Desafio Original (DIO Bootcamp)
O objetivo inicial do projeto era construir um sistema gerenciador de tarefas com um CRUD (Criar, Ler, Atualizar, Deletar) funcional.

Requisitos da API:
A classe principal de `Tarefa` deveria seguir o diagrama abaixo, e a API deveria expor endpoints para todas as operações de CRUD, além de buscas por título, data e status.

![Diagrama da classe Tarefa](diagrama.png)

**Endpoints Esperados:**

| Verbo  | Endpoint                | Parâmetro | Body          |
|--------|-------------------------|-----------|---------------|
| GET    | /Tarefa/{id}            | id        | N/A           |
| PUT    | /Tarefa/{id}            | id        | Schema Tarefa |
| DELETE | /Tarefa/{id}            | id        | N/A           |
| GET    | /Tarefa/ObterTodos      | N/A       | N/A           |
| GET    | /Tarefa/ObterPorTitulo  | titulo    | N/A           |
| GET    | /Tarefa/ObterPorData    | data      | N/A           |
| GET    | /Tarefa/ObterPorStatus  | status    | N/A           |
| POST   | /Tarefa                 | N/A       | Schema Tarefa |

Esse é o schema (model) de Tarefa, utilizado para passar para os métodos que exigirem

```json
{
  "id": 0,
  "titulo": "string",
  "descricao": "string",
  "data": "2022-06-08T01:31:07.056Z",
  "status": "Pendente"
}
```

## 🚀 A Jornada para a Nuvem: O Processo de Deploy
Após completar os requisitos de código, o projeto foi levado para o próximo nível com o deploy em um ambiente de produção real na Azure.

### ☁️ Parte 1: Provisionando a Infraestrutura (VM - IaaS)
Foi criada uma Máquina Virtual **Ubuntu Server 24.04 LTS** na região `Central US` para hospedar a aplicação. A segurança do acesso administrativo foi garantida pelo uso de Chaves SSH, e a firewall (NSG) foi configurada para permitir o tráfego público na porta HTTP (80).

### 🗄️ Parte 2: Configurando o Banco de Dados Gerenciado (DB - PaaS)
Para garantir a persistência segura dos dados, foi provisionado um Banco de Dados do Azure para PostgreSQL no modo "Servidor Flexível". A conexão foi configurada via Acesso Privado (Integração VNet), tornando o banco de dados inacessível a partir da internet pública e garantindo que apenas a VM da aplicação possa comunicar-se com ele através da rede interna da Azure.

### 📦 Parte 3: Containerização e Deploy Final (Docker)
Com a infraestrutura pronta, o deploy da aplicação foi realizado:

1. O código-fonte da API foi clonado do GitHub para a VM.
2. O Docker e o Docker Compose [via repositório oficial](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) foram instalados no servidor.
3. A aplicação foi iniciada com o comando docker compose, utilizando um ficheiro `docker-compose.azure.yml` específico para produção que:
   - Constrói a imagem Docker da API.
   - Expõe a porta 80.
   - Injeta a ConnectionString do banco de dados do Azure como uma variável de ambiente (mantendo os segredos fora do código-fonte).
  
## 🐳 Como Executar Localmente

Este projeto está preparado para rodar facilmente em um ambiente de desenvolvimento local usando Docker.

1. **Clone o repositório:**
```
git clone https://github.com/GustavoHerreira/todoapp-trilha-dotnet-api-desafio.git
cd todoapp-trilha-dotnet-api-desafio
```
2. **Crie o arquivo de ambiente:**
Crie um arquivo `.env` na raiz do projeto. Ele é essencial para configurar as credenciais do banco de dados local. Use o seguinte template:
```
# Credenciais para o banco de dados PostgreSQL local
POSTGRES_USER=root
POSTGRES_PASSWORD=root
POSTGRES_DB=TodoAppDb
```
3. **Execute o Docker Compose:**
Use o arquivo `docker-compose.dev.yml`, que subirá tanto a API quanto um contêiner com o banco de dados PostgreSQL.
```
docker compose -f docker-compose.dev.yml up --build
```
4. **Acesse a API:**
A documentação do Swagger estará disponível em `http://localhost:8080/swagger`.
