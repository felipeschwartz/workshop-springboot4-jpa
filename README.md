# workshop-springboot4-jpa

API REST feita com **Spring Boot 4** + **Spring Data JPA (Hibernate)**.

O projeto está preparado para:

- rodar **localmente** com um profile de desenvolvimento
- rodar em **produção (Render)** usando **PostgreSQL** e **variáveis de ambiente** para configuração e segredos

## Requisitos

- **Java** (use a mesma versão que você usa no projeto e no deploy, quando possível)
- **Maven** (ou o **Maven Wrapper** incluído no repositório)

## Como rodar localmente

Na raiz do projeto (onde está o `pom.xml`):

### Usando Maven Wrapper (recomendado)
```bash
./mvnw spring-boot:run
```

No Windows (PowerShell), se preferir:
```powershell
mvn spring-boot:run
```

A aplicação normalmente sobe em:
- `http://localhost:8080`

## Profiles (dev e prod)

Este projeto usa profiles do Spring:

- `dev`: desenvolvimento local
- `prod`: produção, tudo via variáveis de ambiente (banco, JWT, porta etc.)

### Ativar um profile via terminal (PowerShell)
```powershell
$env:SPRING_PROFILES_ACTIVE="dev"
mvn spring-boot:run
```

Recomendação: evite deixar `spring.profiles.active=prod` fixo no `application.properties`, porque isso força produção até no seu PC, e vira a origem clássica de erro de datasource local.

## Produção (Render) com PostgreSQL

No profile `prod`, a aplicação lê configurações do banco por env vars:

- `SPRING_DATASOURCE_URL`
- `SPRING_DATASOURCE_USERNAME`
- `SPRING_DATASOURCE_PASSWORD`

E a porta do servidor deve respeitar o `PORT` do ambiente:

- `server.port=${PORT:8080}`

### Variáveis de ambiente (produção)

| Variável | Obrigatória | Observação |
|---|---:|---|
| `SPRING_PROFILES_ACTIVE` | sim | `prod` |
| `SPRING_DATASOURCE_URL` | sim | Precisa começar com `jdbc:` |
| `SPRING_DATASOURCE_USERNAME` | sim | Usuário do Postgres |
| `SPRING_DATASOURCE_PASSWORD` | sim | Senha do Postgres |
| `JWT_SECRET` | sim | Segredo para assinar tokens |
| `JWT_EXPIRATION` | sim | Ex.: `3600000` (ms) |
| `PORT` | no Render, sim | Render injeta automaticamente em muitos casos |

### Formato correto do `SPRING_DATASOURCE_URL`
✅ Correto:
```text
jdbc:postgresql://HOST:5432/NOME_DO_BANCO
```

❌ Evite:
```text
jdbc:postgresql://usuario:senha@HOST/NOME_DO_BANCO
```

Use usuário e senha em variáveis separadas.

## Rodar local usando o PostgreSQL do Render (opcional)

Se você quiser rodar no seu computador apontando para o Postgres do Render, use o **host externo** (o host interno do Render normalmente não resolve fora da rede do Render).

Em muitos casos, você também precisa de SSL:

```text
jdbc:postgresql://HOST_EXTERNO:5432/DB?sslmode=require
```

Exemplo (PowerShell), rodando com `prod` localmente:

```powershell
$env:SPRING_PROFILES_ACTIVE="prod"
$env:SPRING_DATASOURCE_URL="jdbc:postgresql://HOST_EXTERNO:5432/DB?sslmode=require"
$env:SPRING_DATASOURCE_USERNAME="USUARIO"
$env:SPRING_DATASOURCE_PASSWORD="SENHA"
$env:JWT_SECRET="SEGREDO"
$env:JWT_EXPIRATION="3600000"

mvn spring-boot:run
```

Atenção: para não poluir dados de produção, prefira um banco separado no Render para testes e desenvolvimento.

## Troubleshooting (erros comuns)

### `'url' must start with "jdbc"`
Causa comum:
- `SPRING_DATASOURCE_URL` está vazio, não foi lido, ou está no formato `postgresql://...` (sem `jdbc:`)

### `invalid port number` ou erro estranho na URL
Causa comum:
- credenciais embutidas na URL (`usuario:senha@...`) e o driver interpreta parte como “porta”

### `28P01 password authentication failed`
Causa:
- `SPRING_DATASOURCE_PASSWORD` não confere com o Postgres atual

## Segurança

- Não commite senha nem segredos no repositório.
- Se uma credencial vazou, a ação correta é rotacionar a senha no provedor e atualizar as env vars.

## Licença

Defina aqui a licença do projeto (ex.: MIT) ou remova esta seção se não for aplicar.