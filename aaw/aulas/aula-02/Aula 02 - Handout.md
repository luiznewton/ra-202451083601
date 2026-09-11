# HANDOUT — AULA 02

## Dissecando o HTTP

*6 requisições sob o microscópio — Arquitetura de Aplicações Web*

## 🎯 MISSÃO

Vocês interceptaram 6 conversas entre um app e a API de uma biblioteca. Para CADA card:

- Descrevam o que o cliente pediu (verbo + recurso na URI)
- Expliquem o que o status code da resposta informa
- Respondam: repetindo a MESMA requisição 3 vezes seguidas, o estado do servidor muda?

Ao final, preencham juntos a TABELA-SÍNTESE dos verbos na última página.

*⏱️ Tempo: 30 minutos  |  👥 Formato: em duplas  |  Dica: o card 6 esconde uma pegadinha de quem é a culpa.*

> **Nomes:** Luiz Felipe Vieira   **Turma:** ADS   **Data:** 20 / 08 / 2026

## REQUISIÇÃO 01 — A prateleira inteira

```text
→ REQUISIÇÃO
GET /api/livros HTTP/1.1
Host: biblioteca.newton.br
Accept: application/json
```

```text
← RESPOSTA
HTTP/1.1 200 OK
Content-Type: application/json

[ { "id": 1, "titulo": "Clean Code", "autor": "Robert C. Martin" },
  { "id": 7, "titulo": "O Programador Pragmático", "autor": "Hunt & Thomas" } ]
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
    - GET /api/livros: pede a lista de livros.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
    - 200 OK informa que a requisição foi realizada com sucesso.

3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?
    - Não muda o estado do servidor. As respostas tendem a ser iguais, se o estado da biblioteca não mudar por outro motivo.

## REQUISIÇÃO 02 — O livro fantasma

```text
→ REQUISIÇÃO
GET /api/livros/99 HTTP/1.1
Host: biblioteca.newton.br
Accept: application/json
```

```text
← RESPOSTA
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{ "title": "Not Found", "status": 404 }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
    - GET /api/livros/99: pede o livro de ID 99.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
    - 404 Not Found informa que o recurso não foi encontrado.

3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?
    - O estado não muda. A resposta esperada continua sendo 404 Not Found.

## REQUISIÇÃO 03 — Livro novo na estante

```text
→ REQUISIÇÃO
POST /api/livros HTTP/1.1
Host: biblioteca.newton.br
Content-Type: application/json

{ "titulo": "Domain-Driven Design", "autor": "Eric Evans" }
```

```text
← RESPOSTA
HTTP/1.1 201 Created
Location: /api/livros/8
Content-Type: application/json

{ "id": 8, "titulo": "Domain-Driven Design", "autor": "Eric Evans" }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
    - POST /api/livros: solicita a criação de um novo livro.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
    - 201 Created informa que o livro foi criado com sucesso.

3. Enviando este POST 3 vezes seguidas, o que acontece na estante? Para que serve o header Location?
    - Enviando três vezes, podem ser criados três livros, portanto o estado do servidor muda a cada requisição. O Location indica a URI do recurso recém-criado.

## REQUISIÇÃO 04 — Corrigindo a ficha completa

```text
→ REQUISIÇÃO
PUT /api/livros/7 HTTP/1.1
Host: biblioteca.newton.br
Content-Type: application/json

{ "id": 7, "titulo": "O Programador Pragmático", "autor": "D. Hunt; D. Thomas" }
```

```text
← RESPOSTA
HTTP/1.1 200 OK
Content-Type: application/json

{ "id": 7, "titulo": "O Programador Pragmático", "autor": "D. Hunt; D. Thomas" }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
    - PUT /api/livros/7: solicita a atualização/substituição dos dados do livro 7.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
    - 200 OK informa que a atualização foi realizada com sucesso.

3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?
    - O estado final do servidor permanece igual após a primeira aplicação. Repetir o PUT não causa novas mudanças no estado.

## REQUISIÇÃO 05 — Fora do catálogo

```text
→ REQUISIÇÃO
DELETE /api/livros/7 HTTP/1.1
Host: biblioteca.newton.br
```

```text
← RESPOSTA
HTTP/1.1 204 No Content
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
    - DELETE /api/livros/7: solicita a exclusão do livro 7.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
    - 204 No Content informa que a exclusão ocorreu com sucesso e não há conteúdo na resposta.

3. Repetindo o DELETE, o estado do servidor muda? Que resposta você ESPERA na segunda vez?
    - Na primeira vez, o livro é excluído. Nas seguintes, o estado não muda mais. Na segunda tentativa, espera-se normalmente 404 Not Found, pois o livro já não existe.

## REQUISIÇÃO 06 — O cadastro capenga

```text
→ REQUISIÇÃO
POST /api/livros HTTP/1.1
Host: biblioteca.newton.br
Content-Type: application/json

{ "autor": "Anônimo" }
```

```text
← RESPOSTA
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{ "title": "Bad Request", "status": 400,
  "errors": { "Titulo": [ "O campo Titulo é obrigatório" ] } }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
    - POST /api/livros: solicita a criação de um novo livro.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
    - 400 Bad Request informa que a requisição enviada é inválida. Nesse caso, faltou o campo obrigatório Titulo; a responsabilidade está nos dados enviados pelo cliente.

3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?
    - O estado do servidor não muda, pois o livro não é criado. A resposta esperada continua sendo 400 Bad Request.

## TABELA-SÍNTESE — Os verbos do HTTP

*Preencham com base nos 6 cards. “Seguro” = não altera nada no servidor. “Idempotente” = repetir N vezes deixa o servidor no mesmo estado que 1 vez.*

| ***Verbo**** | **********Para que serve******* | **Seguro?** | *Idempotente?* | ******Status típicos****** |
|--------------|---------------------------------|-------------|----------------|----------------------------|
| **`GET`***** | Consultar/obter um recurso ---- | ****Sim**** | *****Sim****** | `200`, `404` ------------- |
| **`POST`**** | Criar um novo recurso --------- | ****Não**** | *****Não****** | `201`, `400`, `409` ------ |
| **`PUT`***** | Substituir/atualizar um recurso | ****Não**** | *****Sim****** | `200`, `201`, `204`, `404` |
| **`PATCH`*** | Alterar parcialmente um recurso | ****Não**** | *****Não****** | `200`, `204`, `400`, `404` |
| **`DELETE`** | Excluir um recurso ------------ | ****Não**** | *****Sim****** | `204`, `404` ------------- |


## DESAFIO

1. O verbo PATCH não apareceu em nenhum card. Qual a diferença entre PATCH e PUT? Um app de banco quer alterar SÓ o apelido do usuário, entre dezenas de campos do perfil — qual dos dois você usaria e por quê?
    - O PUT é usado para substituir ou atualizar um recurso completo. Já o PATCH é usado para fazer uma alteração parcial em um recurso, modificando apenas os campos necessários. Eu usaria o PATCH, porque quero alterar somente o apelido do usuário, sem precisar enviar ou alterar os outros campos do perfil.
