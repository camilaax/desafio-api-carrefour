# Desafio de Automação de Testes de API

Requisito:O candidato deve desenvolver um conjunto de testes automatizados que garanta 100% de
cobertura para essa API, utilizando uma ferramenta de testes de API de sua escolha
(Postman, RestAssured, etc.). Também deve integrar esses testes a uma pipeline de CI
(como Jenkins, GitLab, GitHub, etc.) e gerar relatórios dos resultados dos testes,
disponibilizando-os como artefato na pipeline.

## Configuração do ambiente

- **Node.js** 18 ou superior (recomendado 20)
- **npm** 9 ou superior

Clone o repositório e instale as dependências:

```bash
git clone <url-do-repositorio>
cd desafio-api-carrefour
npm install
```

A única dependência de execução é o Newman; o relatório HTML usa `newman-reporter-htmlextra`.

## Como rodar os testes

```bash
npm test
```

Os testes chamam a API em `https://serverest.dev`. Não é necessário configurar variáveis de ambiente para rodar localmente; a collection usa o environment em `postman/Carrefour-API.postman_environment.json`, que já define a URL base.

Para rodar direto com o Newman:

```bash
npx newman run postman/Carrefour-API.postman_collection.json -e postman/Carrefour-API.postman_environment.json
```

## Relatórios

Após `npm test` são gerados na pasta `reports/`:

- **newman-report.html** – relatório visual (abrir no navegador)
- **newman-results.xml** – resultado em formato JUnit para integração com ferramentas de CI

## Testes implementados

A collection cobre os endpoints de usuários e login da API (ServeRest), com autenticação via token JWT obtido no login. Primeiro é criado um usuário administrador e feito o login para obter o token; em seguida são executados os cenários de listagem, criação, consulta, atualização e exclusão.
**Casos cobertos:**

| # | Request | Descrição |
|---|--------|---------------|
| 01 | POST /usuarios | Criação de usuário admin (para uso no login). Status 201 ou 400 (se já existir). |
| 02 | POST /login | Login com email/senha do admin. Captura do token JWT. |
| 03 | GET /usuarios | Listagem com Authorization. Status 200 e retorno com `quantidade` e `usuarios[]`. |
| 04 | POST /usuarios | Criação de usuário comum com nome, email, password, administrador. Status 201 e retorno de `_id`. |
| 05 | POST /usuarios | Mesmo email do passo anterior. Esperado 400 e mensagem de email já em uso. |
| 06 | POST /usuarios | Body vazio. Esperado 400 e retorno dos campos obrigatórios inválidos. |
| 07 | GET /usuarios/:id | Busca do usuário criado no passo 04. Status 200 e presença dos campos principais. |
| 08 | GET /usuarios/:id | Id inexistente (16 caracteres válidos). Esperado 400 e mensagem de não encontrado. |
| 09 | PUT /usuarios/:id | Atualização do usuário criado. Payload válido. Esperado 200 e mensagem de sucesso. |
| 10 | PUT /usuarios/:id | Body vazio. Esperado 400. |
| 11 | DELETE /usuarios/:id | Sem token. Esperado 401 ou 403. |
| 12 | GET /usuarios | Sem token. Esperado 401 ou 403. |
| 13 | GET /usuarios/:id | Sem token. Esperado 401 ou 403. |
| 14 | POST /usuarios | Sem token (criação de usuário comum). Esperado 401 ou 403. |
| 15 | PUT /usuarios/:id | Sem token. Esperado 401 ou 403. |
| 16 | DELETE /usuarios/:id | Exclusão do usuário criado, com token. Esperado 200 ou 204. |
| 17 | DELETE /usuarios/:id | Id inexistente com token. Comportamento aceito conforme documentação da API. |
| 18 | GET /usuarios/:id | Busca do id já excluído no passo 16. Esperado 400/404 e mensagem de não encontrado. |

Cada requisição da collection verifica se a resposta da API chegou em até **3 segundos**; se passar disso, o teste falha. Entre uma requisição e outra há uma pausa de **150 ms** para não estourar o limite da API (100 requisições por minuto).

**Sobre os testes sem token (11 a 15):** o requisito do desafio é que a autenticação seja via JWT. Os testes 11 a 15 validam que DELETE, GET (lista e por id), POST (criação de usuário comum) e PUT, quando chamados sem o header Authorization, retornem 401 ou 403. Se a API retornar 200 ou outro sucesso nesses cenários, o teste falha, indicando que o endpoint não está exigindo autenticação como esperado.

## CI

O workflow em `.github/workflows/ci.yml` roda em todo push e em pull requests: instala dependências com `npm ci`, executa `npm test` e publica a pasta `reports/` como artefato do job. Em caso de falha em qualquer teste, o pipeline falha.

## Estrutura do projeto

```
desafio-api-carrefour/
  postman/
    Carrefour-API.postman_collection.json   # collection com os 18 requests
    Carrefour-API.postman_environment.json  # baseUrl da API
  .github/workflows/
    ci.yml                                  # pipeline de testes
  reports/                                  # gerado ao rodar npm test
  package.json
  README.md
```
