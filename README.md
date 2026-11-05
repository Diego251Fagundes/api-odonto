# Agendamento Odontológico

API REST desenvolvida em contexto acadêmico, a partir do repositório disponibilizado pelo professor. A atividade contempla a preparação e publicação de uma aplicação Spring Boot, com banco de dados em nuvem, empacotamento Docker e verificações automatizadas no GitHub Actions.

O escopo implementado inclui gestão de dentistas e autenticação de usuários. Apesar do nome do projeto, esta versão não possui endpoints de agendamento de consultas.

## Acessos

| Recurso | Endereço |
| --- | --- |
| API publicada | [Render](https://agendamento-odontologico-aynx.onrender.com) |
| Verificação de saúde | [/actuator/health](https://agendamento-odontologico-aynx.onrender.com/actuator/health) |
| Integração contínua | [GitHub Actions](https://github.com/Diego251Fagundes/api-odonto/actions) |

> A aplicação é uma API, sem interface gráfica. O acesso à raiz `/` pode retornar **403** por exigir autenticação. Para verificar a disponibilidade, utilize `/actuator/health`. No plano gratuito do Render, o primeiro acesso após inatividade pode demorar.

## Funcionalidades

- Cadastro de dentistas com nome, e-mail, CRO, especialidade e endereço.
- Listagem paginada de dentistas ativos e consulta individual por ID.
- Atualização de dados e exclusão lógica, preservando o registro no banco.
- Validação dos dados recebidos pela API.
- Autenticação com login e senha e emissão de token JWT.
- Controle de acesso às rotas com Spring Security.
- Versionamento do banco com Flyway e monitoramento de saúde com Actuator.

## Tecnologias

| Tecnologia | Utilização |
| --- | --- |
| Java 21 e Spring Boot | Desenvolvimento e execução da API |
| Spring Data JPA e Hibernate | Persistência e acesso aos dados |
| Spring Security e JWT | Autenticação e autorização |
| Bean Validation | Validação dos dados de entrada |
| MySQL e Flyway | Banco relacional e migrations |
| Maven Wrapper | Compilação, testes e empacotamento |
| Docker | Construção da imagem e execução em contêiner |
| GitHub Actions | Testes e validação do build Docker |
| Aiven e Render | Hospedagem do banco e da aplicação |

## Organização do repositório

O projeto Maven está na subpasta `api-odonto`.

```text
.github/workflows/               # Workflow do GitHub Actions
api-odonto/
├── .mvn/wrapper/                # Configuração do Maven Wrapper
├── src/main/java/              # Controllers, DTOs, entidades e segurança
├── src/main/resources/
│   ├── application.properties  # Configurações da aplicação
│   └── db/migration/           # Scripts SQL versionados
├── src/test/java/              # Testes
├── .dockerignore
├── Dockerfile
└── pom.xml
```

## Configuração e execução local

### Pré-requisitos

- JDK 21 instalado e `JAVA_HOME` configurado.
- Acesso a um banco MySQL, como o serviço criado no Aiven.
- Docker Desktop em execução para os comandos Docker.

### Variáveis de ambiente

Configure as variáveis no terminal ou na configuração de execução da IDE:

| Variável | Descrição |
| --- | --- |
| `DB_URL` | URL JDBC do MySQL, por exemplo: `jdbc:mysql://SEU_HOST:SUA_PORTA/defaultdb?sslMode=REQUIRED` |
| `DB_USERNAME` | Usuário do banco |
| `DB_PASSWORD` | Senha do banco |
| `JWT_SECRET` | Chave secreta utilizada na assinatura e validação dos tokens |
| `PORT` | Porta HTTP opcional; o padrão local é `8080` |

Exemplo no PowerShell, substituindo os valores pelos dados do seu ambiente:

```powershell
$env:DB_URL = 'jdbc:mysql://SEU_HOST:SUA_PORTA/defaultdb?sslMode=REQUIRED'
$env:DB_USERNAME = 'SEU_USUARIO'
$env:DB_PASSWORD = 'SUA_SENHA'
$env:JWT_SECRET = 'SUA_CHAVE_JWT_ALEATORIA'
```

As variáveis definidas dessa forma valem para o terminal atual e seus processos. Arquivos `.env` ou `.env.*` são ignorados pelo Git e não são carregados automaticamente pela aplicação. Utilize somente dados fictícios neste laboratório.

### Iniciar a API

A partir da raiz do repositório:

```powershell
cd api-odonto
.\mvnw.cmd spring-boot:run
```

Na inicialização, o Flyway aplica as migrations pendentes ao banco configurado. O banco deve existir e estar acessível; as tabelas previstas nos scripts são criadas pelas migrations.

### Executar os testes

Na pasta que contém o `pom.xml`, com as variáveis configuradas:

```powershell
.\mvnw.cmd --batch-mode clean verify
```

O teste existente, `contextLoads`, verifica a inicialização do contexto Spring e utiliza o banco configurado. Ele não substitui testes funcionais de todas as rotas. Utilize um banco exclusivo de laboratório.

## Executar com Docker

Na pasta que contém o `Dockerfile` e o `pom.xml`:

```powershell
docker build -t agendamento-odontologico .
docker run --rm -p 8080:8080 -e PORT=8080 -e DB_URL -e DB_USERNAME -e DB_PASSWORD -e JWT_SECRET agendamento-odontologico
```

Os valores das quatro variáveis devem estar definidos no terminal antes de executar o contêiner. O Dockerfile utiliza duas etapas: Maven com JDK 21 para gerar o JAR e uma imagem com JRE 21 para executá-lo. A construção usa `-DskipTests`; os testes são executados separadamente.

## Endpoints

| Método | Rota | Finalidade | Acesso |
| --- | --- | --- | --- |
| `POST` | `/login` | Autenticar usuário e gerar JWT | Público |
| `POST` | `/dentista` | Cadastrar dentista | Autenticado |
| `GET` | `/dentista` | Listar dentistas ativos com paginação | Autenticado |
| `GET` | `/dentista/{id}` | Consultar dentista por ID | Autenticado |
| `PUT` | `/dentista` | Atualizar dados, informando o ID no corpo | Autenticado |
| `DELETE` | `/dentista/{id}` | Desativar dentista | Autenticado |
| `GET` | `/actuator/health` | Consultar a saúde da aplicação | Público |
| `GET` | `/actuator/health/liveness` | Consultar a verificação de liveness | Público |
| `GET` | `/actuator/health/readiness` | Consultar a verificação de readiness | Público |

O login recebe `login` e `senha` e retorna o campo `tokenJWT`. É necessário ter um usuário previamente cadastrado no banco, com senha armazenada em BCrypt; esta versão não possui endpoint de cadastro de usuários.

Nas rotas protegidas, envie o token no cabeçalho:

```http
Authorization: Bearer SEU_TOKEN_JWT
```

## Deploy e integração contínua

O deploy foi realizado conforme o roteiro da atividade:

1. **Aiven:** provisionamento do MySQL e utilização do banco `defaultdb`, com conexão criptografada.
2. **Aplicação:** configuração por variáveis de ambiente, migrations Flyway e endpoint de saúde.
3. **Docker:** criação e validação da imagem da API.
4. **GitHub Actions:** execução de testes, empacotamento Maven e construção da imagem a cada push na `main`, com opção de execução manual.
5. **Render:** publicação do serviço Docker e configuração do health check em `/actuator/health`.

O workflow está em `.github/workflows/agendamento-odontologico-cicd.yml`. Ele valida testes e build; não contém uma etapa que aciona o deploy no Render.

### Configuração utilizada no Render

| Campo | Valor |
| --- | --- |
| Runtime | Docker |
| Branch | `main` |
| Root Directory | `api-odonto` |
| Dockerfile Path | `./Dockerfile` |
| Docker Build Context | `.` |
| Health Check Path | `/actuator/health` |

As variáveis `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` e `JWT_SECRET` são cadastradas separadamente nos **Secrets do GitHub Actions** e no **ambiente do serviço Render**. Nenhuma credencial real deve ser adicionada ao código ou ao README.

## Contexto acadêmico

A atividade teve como objetivo praticar configuração de ambientes, persistência em nuvem, conteinerização, testes automatizados e publicação de uma API. O código-base foi fornecido pelo professor e preparado para execução e deploy durante o laboratório.
