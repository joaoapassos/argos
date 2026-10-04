# Argos

**Argos** é um sistema de monitoramento e análise de preços desenvolvido para observar continuamente diferentes fontes de promoções, identificar oportunidades relevantes e manter um histórico das ofertas encontradas.

O nome é inspirado em **Argos Panoptes**, personagem da mitologia grega conhecido por seus muitos olhos e por sua capacidade de vigilância constante. A ideia representa o objetivo do projeto: ser um sistema capaz de observar múltiplas fontes simultaneamente e destacar aquilo que realmente importa.

## Objetivo

O Argos busca centralizar o monitoramento de promoções e preços.

Inicialmente, o sistema utilizará o **Telegram** como fonte de dados, acompanhando grupos e canais de promoções.

A arquitetura, entretanto, é projetada para permitir novas fontes futuramente, como:

- sites;
- APIs;
- feeds;
- Discord;
- e-mail;
- outras plataformas de mensagens.

As informações coletadas poderão ser filtradas, analisadas e armazenadas para construção de histórico de preços.

Quando uma oferta relevante for identificada, o Argos poderá encaminhá-la por diferentes canais de notificação, como Telegram, Windows e e-mail.

## Funcionamento geral

O fluxo básico do sistema é:

```text
Fonte
  │
  ▼
Coleta
  │
  ▼
Normalização
  │
  ▼
Análise
  │
  ├── Ignorar
  │
  └── Oferta relevante
          │
          ├── Armazenar
          ├── Atualizar histórico
          └── Notificar
```

Uma fonte é responsável apenas por obter informações externas.

A lógica de análise não deve depender diretamente de Telegram ou de qualquer outra fonte específica.

Isso permite que diferentes fontes sejam adicionadas sem alterar o núcleo da aplicação.

## Arquitetura

O Argos é desenvolvido inicialmente com:

- **Go** como linguagem principal;
- **Chi** para roteamento HTTP;
- **MySQL** como banco de dados;
- **database/sql** para acesso ao banco;
- **golang-migrate** para versionamento do schema;
- **Telegram** como primeira fonte de promoções.

A estrutura básica do projeto é:

```text
argos/
├── .agents/
│   └── skills/
├── .codex/
├── cmd/
├── docs/
├── internal/
│   ├── config/
│   ├── database/
│   ├── sources/
│   │   └── telegram/
│   ├── offers/
│   └── web/
├── migrations/
├── AGENTS.md
├── go.mod
└── README.md
```

### `cmd/`

Contém os pontos de entrada dos executáveis do projeto.

O executável principal deverá permanecer pequeno e ser responsável principalmente por inicializar e conectar os componentes da aplicação.

### `internal/`

Contém a implementação interna do Argos.

Entre seus principais módulos estão:

- `sources/` — integrações responsáveis pela coleta de informações;
- `offers/` — domínio e regras relacionadas às ofertas;
- `database/` — infraestrutura de acesso ao banco;
- `web/` — API HTTP e interface com o sistema web;
- `config/` — configuração da aplicação.

### `migrations/`

Contém as alterações versionadas da estrutura do banco de dados.

As migrations são gerenciadas utilizando **golang-migrate**.

### `.agents/`

Contém skills e instruções especializadas utilizadas por agentes de desenvolvimento.

As regras gerais do projeto ficam definidas em [`AGENTS.md`](./AGENTS.md).

## Documentação

A documentação técnica e conceitual do projeto está em [`docs/`](./docs/).

O objetivo dessa pasta é documentar aspectos de alto nível, como:

- arquitetura;
- regras de negócio;
- banco de dados;
- fontes externas;
- API;
- decisões técnicas.

Detalhes específicos de implementação devem permanecer próximos ao código correspondente.

Como regra geral:

```text
docs/
→ explica o que existe e por que foi projetado dessa forma

código
→ explica como cada componente funciona
```

O índice da documentação está disponível em [`docs/README.md`](./docs/README.md).

## Princípios do projeto

O desenvolvimento do Argos busca manter alguns princípios:

- componentes com responsabilidades bem definidas;
- baixo acoplamento entre fontes externas e regras de negócio;
- possibilidade de adicionar novas fontes sem modificar o núcleo;
- código Go simples e idiomático;
- SQL explícito e controlado;
- banco de dados versionado através de migrations;
- documentação mantida junto com o projeto;
- evitar abstrações e dependências sem necessidade concreta.

## Estado do projeto

O Argos está em fase inicial de desenvolvimento.

A primeira versão será focada em:

```text
Telegram
   ↓
Coleta de promoções
   ↓
Filtro e análise
   ↓
Histórico de preços
   ↓
MySQL
   ↓
Notificações
```

Funcionalidades e integrações adicionais serão incorporadas conforme a evolução do projeto.

## Licença

Este projeto é distribuído sob a **Apache License 2.0**.

Ela permite uso, modificação e distribuição do software, inclusive para fins comerciais, desde que seus termos e avisos de licença sejam preservados.

Consulte o arquivo [`LICENSE`](./LICENSE) para os termos completos.

Copyright 2026 João Alves Passos