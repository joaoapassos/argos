# Documentação do Argos

Esta pasta contém a documentação de alto nível do projeto.

A documentação aqui deve explicar principalmente:

- arquitetura;
- decisões técnicas;
- regras de negócio;
- funcionamento dos módulos;
- integrações externas;
- fluxos importantes do sistema.

Detalhes específicos de implementação devem permanecer próximos ao código-fonte.

## Estrutura

### `architecture.md`

Visão geral da arquitetura do sistema.

Inclui:

- organização dos módulos;
- responsabilidades;
- fluxo de dados;
- dependências entre componentes;
- limites entre domínio, infraestrutura e fontes externas.

### `business-rules.md`

Regras de negócio do Argos.

Exemplos:

- quando uma promoção é considerada relevante;
- classificação de preços;
- regras de histórico;
- comportamento das notificações.

### `database.md`

Documentação geral do banco de dados.

Inclui:

- principais entidades;
- relacionamentos;
- estratégia de migrations;
- decisões relacionadas ao MySQL.

As migrations reais ficam em:

`/migrations`

### `api.md`

Visão geral da API HTTP.

Inclui:

- organização das rotas;
- convenções de respostas;
- tratamento de erros;
- autenticação, caso seja adicionada futuramente.

A implementação das rotas fica em:

`/internal/web`

### `sources.md`

Documentação das fontes externas de promoções.

Exemplos:

- Telegram;
- futuras integrações;
- normalização dos dados recebidos;
- responsabilidades comuns entre fontes.

As implementações ficam em:

`/internal/sources`

### `decisions/`

Contém decisões arquiteturais importantes do projeto.

Exemplo:

```text
decisions/
├── 001-use-go.md
├── 002-use-chi.md
├── 003-use-mysql.md
└── 004-use-database-sql.md
```

Cada documento deve explicar:

- contexto;
- decisão;
- motivo;
- consequências.

## Relação entre documentação e código

A documentação desta pasta deve permanecer em nível conceitual.

Quando necessário, deve apontar para arquivos do código-fonte.

Exemplo:

```text
A entidade Offer está definida em:

internal/offers/model.go

A lógica relacionada às ofertas está em:

internal/offers/service.go
```

O código-fonte deve documentar os detalhes específicos da implementação usando comentários idiomáticos de Go.

## Regra geral

Use esta divisão:

```text
docs/
→ por que o sistema funciona dessa forma

código
→ como cada parte funciona
```

Evite duplicar documentação detalhada entre `docs/` e o código.