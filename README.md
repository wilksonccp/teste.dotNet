# 📚 LivrariaTheos — API de Gerenciamento de Livros

---

## 1️⃣ Visão Geral

Aplicação desenvolvida como parte do desafio técnico da Théos, para gerenciar exemplares de uma livraria.  
O projeto implementa um backend em **.NET 8 + ASP.NET Core Web API**, com persistência em **SQL Server** e autenticação via **JWT**.

- **Status atual:** Todos os requisitos obrigatórios do desafio atendidos.
- **Diferenciais:** (Angular + Testes) planejados como próximos passos.

---

## 2️⃣ Requisitos do Desafio

- ✅ **Listagem de livros ordenada por título (ASC)**
- ✅ **CRUD completo de livros**
- ✅ **Validação de duplicidade (Título + Autor + Ano)**
- ✅ **Persistência em SQL Server 2014+**
- ✅ **Logging de registros e erros (Serilog)**
- ✅ **Swagger configurado**
- ✅ **Autenticação com dois níveis:**
  - **Público (anônimo):** leitura
  - **Admin (JWT):** CRUD

> **Critério de desempate:** aplicação de boas práticas (DDD, TDD, Design Patterns, SOLID, Clean Code)  
> **Diferencial:** Frontend em Angular (planejado, não implementado ainda)

---

## 3️⃣ Como Rodar Localmente

### 3.1 Pré-requisitos

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- SQL Server 2014+ (pode ser Express)
- (Opcional) Docker Desktop

### 3.2 Configurações

No arquivo `appsettings.json`:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost\\SQLEXPRESS;Database=LivrariaTheos;User Id=sa;Password=SUASENHA;TrustServerCertificate=True;MultipleActiveResultSets=True"
},
"Jwt": {
  "Key": "LivrariaTheos_Desafio2025@SuperSecretKey"
}
```

### 3.3 Banco de Dados

**Opção A — Migrations**

```sh
dotnet ef database update --project src/LivrariaTheos.Infrastructure --startup-project src/LivrariaTheos.Api
```

**Opção B — Scripts SQL**

```sh
sqlcmd -S .\SQLEXPRESS -d LivrariaTheos -i db/scripts/initial.sql
```

### 3.4 Executar a API

```sh
dotnet run --project src/LivrariaTheos.Api
```

➡️ Acesse [http://localhost:5122/swagger/index.html](http://localhost:5122/swagger/index.html)

---

## 4️⃣ Autenticação & Autorização

**Usuário seedado (Admin):**

- Email: `admin@theos`
- Senha: `Admin#2024!`

**Login:**

- Endpoint: `POST /api/auth/login`
- Body:

```json
{ "email": "admin@theos", "senha": "Admin#2024!" }
```

- Retorna:

```json
{ "token": "<jwt>" }
```

No Swagger, clique em **Authorize** e informe:  
`Bearer <token>`

---

## 5️⃣ Endpoints Principais

**Público (sem autenticação):**

- `GET /api/livros?busca=&page=&pageSize=` → lista ordenada por Título ASC, com paginação/filtro
- `GET /api/livros/{id}`

**Admin (JWT + Role=Admin):**

- `POST /api/livros`
- `PUT /api/livros/{id}`
- `DELETE /api/livros/{id}`

**Regras de negócio:**

- Duplicidade (Titulo, Autor, AnoPublicacao) → retorna **409 Conflict**

---

## 6️⃣ Modelo de Domínio

**Entidade Livro**

| Campo           | Tipo      | Regras                                 |
|-----------------|-----------|----------------------------------------|
| Id              | Guid      |                                        |
| Titulo          | string    | obrigatório, máx 200                   |
| Autor           | string    | obrigatório, máx 150                   |
| AnoPublicacao   | int?      | 1450..ano atual                        |
| ISBN            | string?   | único se informado                     |
| Editora         | string?   | opcional                               |
| DataPublicacao  | DateOnly  | obrigatório                            |
| Preco           | decimal   | ≥0                                     |
| Estoque         | int       | ≥0                                     |

**Índices**

- Único: (Titulo, Autor, AnoPublicacao)
- Único condicional: ISBN (quando não nulo)

---

## 7️⃣ Arquitetura & Estrutura de Pastas

```
/src
  LivrariaTheos.Api            -> Controllers, Auth, Swagger, DI
  LivrariaTheos.Application    -> DTOs, Use Cases/Handlers, Contracts
  LivrariaTheos.Domain         -> Entidades, Regras, Interfaces
  LivrariaTheos.Infrastructure -> EF Core, Repos, Migrations, Logging

/tests
  LivrariaTheos.Tests          -> (planejado) Unit/Integration

/db
  scripts/initial.sql          -> Script inicial do schema
```

---

## 8️⃣ Logging & Erros

- **Serilog** → console + arquivo (`logs/api-*.log`)
- Middleware global → padroniza resposta de erro JSON com `traceId`, `message`, `details` (em Dev)

---

## 9️⃣ Swagger

- Documentação completa em `/swagger`
- Esquema Bearer para autenticação
- Códigos de resposta documentados: `200`, `201`, `400`, `401`, `403`, `404`, `409`, `500`

---

## 🔟 Decisões Arquiteturais

- EF Core escolhido pela agilidade
- DDD enxuto com separação em camadas
- Validações no domínio + índices no banco reforçando unicidade
- SOLID aplicado (SRP, DIP, OCP)

---

## 1️⃣1️⃣ Scripts & Migrations

**Gerar migration inicial:**

```sh
dotnet ef migrations add Initial --project src/LivrariaTheos.Infrastructure --startup-project src/LivrariaTheos.Api
```

**Aplicar migration:**

```sh
dotnet ef database update --project src/LivrariaTheos.Infrastructure --startup-project src/LivrariaTheos.Api
```

**Exportar script SQL:**

```sh
dotnet ef migrations script --project src/LivrariaTheos.Infrastructure --startup-project src/LivrariaTheos.Api -o db/scripts/initial.sql
```

---

## 1️⃣2️⃣ Testes (Planejado)

- Domínio: título obrigatório, ano fora da faixa
- Integração: unicidade (409), ordenação ASC
- API: 401 sem token; 201 com token válido

---

## 1️⃣3️⃣ Roadmap (Próximos Passos)

- Frontend Angular (público + admin, JWT)
- FluentValidation nos DTOs
- Health Checks em `/health`
- Docker Compose (API + SQL Server)
- Testes automatizados (unit + integração)

---

## 1️⃣4️⃣ Troubleshooting

- Não conecta no SQL Server? Verifique `TrustServerCertificate=True`, instância `SQLEXPRESS` e credenciais
- Token inválido no Swagger? Use `Bearer <token>`
- 409 Conflict ao criar livro? Já existe um com mesmo (Titulo, Autor, AnoPublicacao)

---

## 1️⃣5️⃣ Entrega

- Fork do repositório base → implementado no repo `teste.dotNet`
- Currículo adicionado na raiz
- PR enviado para avaliação

---

## 1️⃣6️⃣ Créditos

Desenvolvido por **Wilkson Colares**  
[GitHub](#) | [LinkedIn](#)