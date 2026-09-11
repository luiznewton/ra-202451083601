# HANDOUT — AULA 03

## Consultoria de Design: a API da EscolaTech

*Identifique os anti-padrões e proponha o redesenho — Arquitetura de Aplicações Web*

## 🎯 MISSÃO

A EscolaTech contratou a consultoria de vocês para auditar a API do sistema escolar. Todos os endpoints abaixo FUNCIONAM e estão em produção — mas o time novo se recusa a mexer neles. Para CADA endpoint:

- Identifiquem o(s) problema(s) de design (pode haver mais de um!)
- Proponham o redesenho: método HTTP + rota + status codes corretos

*⏱️ Tempo: 25 minutos  |  👥 Formato: em duplas  |  Dica: se a rota conta o que faz em português, algo está errado.*

> **Nomes:** Luiz Felipe Vieira de Paula   **Turma:** ADS   **Data:** 20 / 08 / 2026

## ENDPOINT 01 — POST /api/getAlunos

**Documentação atual (extraída da wiki da EscolaTech):**

```text
POST /api/getAlunos
Retorna TODOS os alunos cadastrados (hoje: 12.482 registros).
Resposta: 200 OK + array JSON completo (~9 MB).
Obs. da wiki: "usar POST porque GET não estava funcionando".
```

1. Qual(is) problema(s) de design vocês identificam?
    - Está usando POST para fazer uma consulta.
    - O nome da rota getAlunos descreve uma ação; em uma API REST, a rota deve representar um recurso.
    - Está retornando uma quantidade muito grande de dados de uma vez.
    - A observação "GET não estava funcionando" indica um problema de implementação, não um motivo para trocar GET por POST.

2. Seu redesenho (método + rota + status codes):
    - GET /api/alunos
      200 OK


## ENDPOINT 02 — GET /deletarAluno?id=7

**Documentação atual (extraída da wiki da EscolaTech):**

```text
GET /deletarAluno?id=7
Remove o aluno do banco de dados.
Resposta: 200 OK + "OK" (mesmo se o aluno não existir).
Obs. da wiki: "dá pra deletar pelo navegador, bem prático".
```

1. Qual(is) problema(s) de design vocês identificam?
    - Está usando GET para apagar um recurso.
    GET deve ser seguro, ou seja, não deve alterar o servidor.
    A rota descreve uma ação (deletarAluno) em vez de representar o recurso.
    O ID deveria fazer parte da rota.
    Retornar 200 OK mesmo quando o aluno não existe esconde o erro.

2. Seu redesenho (método + rota + status codes):
    - DELETE /api/alunos/7
      204 No Content


## ENDPOINT 03 — POST /api/alunos (criação)

**Documentação atual (extraída da wiki da EscolaTech):**

```text
POST /api/alunos
Body: { "nome": "...", "curso": "..." }
Cria o aluno e responde: 200 OK + body "OK".
O app precisa buscar a lista inteira de novo para descobrir o ID gerado.
```

1. Qual(is) problema(s) de design vocês identificam?
    - O principal problema é o 200 OK.
    Como um novo aluno foi criado, o status mais adequado é:
    201 Created
    Além disso, o servidor deveria informar ao cliente qual foi o ID criado, em vez de obrigá-lo a buscar a lista inteira novamente.

2. Seu redesenho (método + rota + status codes):
    - POST /api/alunos

    Body:
    {
    "nome": "...",
    "curso": "..."
    }

    201 Created
    Location: /api/alunos/13


## ENDPOINT 04 — GET /escolas/1/turmas/3/alunos/25/matriculas/88/disciplinas/12

**Documentação atual (extraída da wiki da EscolaTech):**

```text
GET /escolas/1/turmas/3/alunos/25/matriculas/88/disciplinas/12
Retorna os dados da disciplina 12 da matrícula 88.
Para montar a URL o app precisa conhecer 5 IDs diferentes.
Resposta: 200 OK + JSON da disciplina.
```

1. Qual(is) problema(s) de design vocês identificam?
    - A URL está excessivamente complexa.
    - Existem muitos IDs para o cliente conhecer.
    - Há um excesso de recursos aninhados.
    - Para buscar uma disciplina, o cliente não deveria precisar conhecer toda essa hierarquia se o ID da disciplina já for suficiente.

2. Seu redesenho (método + rota + status codes):
    - Uma opção simples:
    GET /api/disciplinas/12
    200 OK

    - Se a disciplina realmente precisar estar relacionada à matrícula:
    GET /api/matriculas/88/disciplinas/12
    200 OK

    - Se a disciplina não existir:
    404 Not Found

## ENDPOINT 05 — GET /api/alunos/7/matriculas (erro)

**Documentação atual (extraída da wiki da EscolaTech):**

```text
GET /api/alunos/7/matriculas
Se o aluno 7 não existe, responde:
200 OK + "<html><b>Erro: aluno nao existe!</b></html>"
O app mobile quebra tentando fazer parse do JSON.
```

1. Qual(is) problema(s) de design vocês identificam?
    - O servidor retorna 200 OK quando o aluno não existe.
    - Está retornando HTML para uma API que deveria trabalhar com JSON.
    - O app mobile espera JSON e recebe HTML, causando erro no parse.
    - O status HTTP deveria representar corretamente o problema.

2. Seu redesenho (método + rota + status codes):
    - Se o aluno não existir:
    GET /api/alunos/7/matriculas

    404 Not Found
    Content-Type: application/problem+json

    Por exemplo:
    {
    "title": "Aluno não encontrado",
    "status": 404
    }


## ENDPOINT 06 — PUT /api/atualizarNotaParcial?aluno=7&disc=12&nota=8.5

**Documentação atual (extraída da wiki da EscolaTech):**

```text
PUT /api/atualizarNotaParcial?aluno=7&disc=12&nota=8.5
Atualiza SÓ a nota parcial da disciplina, sem body.
Todos os dados vão na query string.
Resposta: 200 OK + "OK".
```

1. Qual(is) problema(s) de design vocês identificam?
    - A rota descreve uma ação: atualizarNotaParcial.
    - Os dados da alteração estão todos na query string.
    - O PUT está sendo usado para alterar apenas uma parte do recurso.
    - Para uma atualização parcial, PATCH é mais adequado.
    - O corpo da requisição seria mais apropriado para transportar os dados que serão alterados.

2. Seu redesenho (método + rota + status codes):
    - PATCH /api/alunos/7/disciplinas/12

    Body:
    {
    "notaParcial": 8.5
    }

    200 OK


## DESAFIO

1. A EscolaTech quer lançar mudanças na API sem quebrar o app mobile antigo, que não recebe atualização há 2 anos. Que decisão de design — que falta na API INTEIRA — resolve esse problema? Como ficariam as rotas?

ENDPOINT 01

    - Problemas de design:
    O endpoint usa POST para realizar uma consulta, quando o correto seria GET. Além disso, a rota descreve uma ação (getAlunos) em vez de representar um recurso. Também seria melhor utilizar paginação, pois são muitos registros.

    - Redesenho:
    GET /api/alunos
    200 OK

    Com paginação:
    GET /api/alunos?page=1&limit=100
    200 OK

ENDPOINT 02

    - Problemas de design:
    Está usando GET para excluir um aluno, mas GET não deve alterar o estado do servidor. A rota também descreve uma ação e o ID está na query string. Além disso, retornar 200 mesmo quando o aluno não existe esconde o erro.

    - Redesenho:
    DELETE /api/alunos/7
    204 No Content

    Se o aluno não existir:
    404 Not Found

ENDPOINT 03

    - Problemas de design:
    O método POST e a rota estão adequados. O problema é retornar 200 OK após criar o aluno e não informar diretamente o ID gerado.

    - Redesenho:
    POST /api/alunos

    201 Created
    Location: /api/alunos/13

    O servidor pode também retornar os dados do aluno criado em JSON.

ENDPOINT 04

    - Problemas de design:
    A rota é muito longa e possui muitos recursos aninhados. O cliente precisa conhecer vários IDs para conseguir acessar uma disciplina.

    - Redesenho:
    GET /api/disciplinas/12
    200 OK

    Se a disciplina depender da matrícula:
    GET /api/matriculas/88/disciplinas/12
    200 OK

    Se o recurso não existir:
    404 Not Found

ENDPOINT 05

    - Problemas de design:
    O endpoint retorna 200 OK mesmo quando o aluno não existe. Além disso, retorna HTML em vez de JSON, fazendo o aplicativo mobile quebrar ao tentar interpretar a resposta.

    - Redesenho:
    GET /api/alunos/7/matriculas
    404 Not Found
    Content-Type: application/problem+json

    Exemplo de resposta:
    {
    "title": "Aluno não encontrado",
    "status": 404
    }

ENDPOINT 06

    - Problemas de design:
    A rota descreve uma ação (atualizarNotaParcial) e todos os dados estão na query string. Como a alteração é parcial, PATCH é mais adequado que PUT. Os dados que serão alterados devem ser enviados no body.

    - Redesenho:
    PATCH /api/alunos/7/disciplinas/12

    {
    "notaParcial": 8.5
    }

    200 OK

DESAFIO

A decisão de design que falta na API é o versionamento.

A EscolaTech poderia criar versões da API, como:

/api/v1/alunos
/api/v1/disciplinas
/api/v1/matriculas

O aplicativo antigo continuaria utilizando a versão 1, enquanto os novos aplicativos poderiam utilizar a versão 2:

/api/v2/alunos
/api/v2/disciplinas
/api/v2/matriculas

Assim, a EscolaTech consegue fazer mudanças na API sem quebrar o aplicativo mobile antigo.
