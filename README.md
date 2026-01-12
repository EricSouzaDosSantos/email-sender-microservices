# Email Sender App

Aplicação de envio de e-mails desenvolvida com arquitetura limpa (Clean Architecture), utilizando Spring Boot, RabbitMQ para processamento assíncrono e observabilidade completa.

##  Arquitetura

A aplicação segue os princípios da **Arquitetura Limpa**, com separação clara de responsabilidades:

- **Domain**: Entidades e regras de negócio
- **Application**: Casos de uso e portas (interfaces)
- **Infrastructure**: Implementações concretas (adapters)
- **Interfaces**: DTOs e contratos de API

## Funcionalidades

- Validações no domínio (e-mail inválido, campos obrigatórios)
- Tratamento de exceções global
- Envio assíncrono com fila (RabbitMQ)
- Observabilidade (logs estruturados, métricas, Actuator)
- Dockerização completa

## Pré-requisitos

- Java 17+
- Maven 3.9+
- Docker e Docker Compose (para execução com containers)
- RabbitMQ (ou usar via Docker Compose)

## Configuração

### Variáveis de Ambiente

Configure as seguintes variáveis de ambiente para o envio de e-mails:

```bash
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=seu-email@gmail.com
MAIL_PASSWORD=sua-senha-app
```

### Executando Localmente

1. Inicie o RabbitMQ:
```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.13-management-alpine
```

2. Execute a aplicação:
```bash
mvn spring-boot:run
```

### Executando com Docker Compose

```bash
docker-compose up --build
```

A aplicação estará disponível em `http://localhost:8080`

## Endpoints

### Enviar E-mail

```http
POST /emails
Content-Type: application/json

{
  "to": "destinatario@example.com",
  "subject": "Assunto do e-mail",
  "body": "Corpo do e-mail"
}
```

### Health Check

```http
GET /actuator/health
```

### Métricas Prometheus

```http
GET /actuator/prometheus
```

## Observabilidade

### Logs

Os logs são estruturados e incluem:
- Request ID para rastreamento
- Timestamp formatado
- Nível de log
- Contexto da requisição

### Métricas

A aplicação expõe as seguintes métricas:
- `email.requests`: Total de requisições recebidas
- `email.processed`: Total de e-mails processados com sucesso
- `email.errors`: Total de erros ao processar e-mails
- `email.request.duration`: Tempo de processamento das requisições

### Actuator

Endpoints disponíveis:
- `/actuator/health`: Status de saúde da aplicação
- `/actuator/info`: Informações da aplicação
- `/actuator/metrics`: Lista de métricas disponíveis
- `/actuator/prometheus`: Métricas no formato Prometheus

## Docker

### Build da Imagem

```bash
docker build -t email-sender-app .
```

### Executar Container

```bash
docker run -p 8080:8080 \
  -e RABBITMQ_HOST=localhost \
  -e MAIL_USERNAME=seu-email@gmail.com \
  -e MAIL_PASSWORD=sua-senha \
  email-sender-app
```

## Testes

```bash
mvn test
```

## Estrutura do Projeto


### `application/` — camada de orquestração de casos de uso
Responsabilidade: implementar a lógica de aplicação que orquestra entidades e portas (interfaces). Contém contratos que definem o que a aplicação faz e implementações dos casos de uso.
- `application/port/in/` — Portas de entrada  
  Descreve os contratos que adaptadores de entrada (controllers, listeners) chamam.  
- `application/port/out/` — Portas de saída  
  Interfaces que representam dependências externas usadas pelos casos de uso (envio de e‑mail, persistência, enfileiramento).  
- `application/usecase/` — Implementações dos casos de uso  
  Implementações concretas que realizam a orquestração e regras de aplicação.  

### `domain/` — núcleo do negócio
Responsabilidade: modelos do domínio, validações e regras puras do negócio, independentes de frameworks.
- `domain/model/` — Entidades e objetos de valor  
- `domain/exception/` — Exceções e validações de domínio  

### `infrastructure/` — implementações tecnológicas
Responsabilidade: detalhes concretos de infraestrutura e integração com frameworks/serviços externos.
- `infrastructure/adapters/in/` — Adaptadores de entrada  
  Convertem requisições externas em chamadas às portas de entrada.  
- `infrastructure/adapters/out/` — Adaptadores de saída  
  Implementações das portas de saída como clientes SMTP, repositórios, produtores/consumidores de fila.  
- `infrastructure/configuration/` — Configurações do Spring  
  Beans e configuração de integrações (RabbitMQ, Mail, métricas).  
- `infrastructure/web/` — Componentes web transversais  
  Interceptors, filtros, configuração de logs e métricas HTTP.  

### `interfaces/` — DTOs e contratos de API
Responsabilidade: modelos de entrada/saída expostos pela API e mapeamentos entre DTOs e entidades do domínio.
- `interfaces/dtos/` — DTOs de request/response  

## Observações de organização
- Os casos de uso (camada `application`) dependem de interfaces nas `port/out` e não de implementações concretas.
- Implementações concretas ficam em `infrastructure/` e implementam as portas definidas em `application/port/out/`.
- Validações e regras puras devem ficar em `domain/` para manter o núcleo testável e independente de frameworks.

## Validações

A aplicação valida:
- Formato de e-mail (regex)
- Campos obrigatórios (to, subject, body)
- Tamanho máximo dos campos (subject: 200, body: 5000)

## Tecnologias

- Spring Boot 4.0.1
- Spring AMQP (RabbitMQ)
- Spring Boot Actuator
- Micrometer (métricas)
- Jakarta Validation
- SLF4J + Logback (logs)
