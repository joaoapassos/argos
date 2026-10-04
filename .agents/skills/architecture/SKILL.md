---
name: architecture
description: Projetar e revisar limites, responsabilidades e dependências entre módulos do Argos, preservando domínio independente de HTTP e fontes externas e evitando abstrações sem necessidade.
license: Apache-2.0
---

# Arquitetura do Argos

`AGENTS.md` é a autoridade principal. Esta skill orienta limites arquiteturais na stack Go, Chi v5, MySQL, database/sql e golang-migrate. Decisões de implementação detalhadas pertencem às skills complementares.

## Estrutura e responsabilidades

```text
cmd/
internal/
    config/
    database/
    sources/
    offers/
    web/
migrations/
docs/
```

- `cmd/`: ponto de entrada, composição de dependências, inicialização e encerramento. Deve permanecer pequeno e sem regras de negócio.
- `internal/config/`: leitura e validação das configurações necessárias à aplicação.
- `internal/database/`: inicialização de `database/sql`, conexão MySQL, configuração do pool e infraestrutura compartilhada de banco. Não contém regras de domínio.
- `internal/sources/`: integrações de entrada. Adaptadores como `telegram/` e, futuramente, `discord/` traduzem dados externos para conceitos internos. Esses caminhos exemplificam fontes possíveis, não comprovam integrações já implementadas.
- `internal/offers/`: entidades, services, regras de ofertas e interfaces necessárias aos seus consumidores. Não depende diretamente de HTTP, Chi ou Telegram.
- `internal/web/`: router Chi v5, handlers, DTOs, middleware e helpers HTTP com responsabilidade clara. Adapta requisições aos casos de uso sem conter regras de negócio ou acesso direto ao banco.
- `migrations/`: exclusivamente evolução versionada do schema, sem lógica de negócio.
- `docs/`: documentação conceitual, contratos públicos, regras e decisões.

## Direção das dependências

Fontes e HTTP chamam casos de uso do domínio. O domínio conhece conceitos internos e contratos pequenos de que precisa; implementações de infraestrutura satisfazem esses contratos e são injetadas na composição.

```text
sources ──→ domínio ←── web
               │
               ▼
       interfaces consumidoras
               ▲
               │ implementa
         infraestrutura
```

O fluxo de execução pode chegar à infraestrutura por uma interface; isso não exige que o domínio importe o adaptador concreto. Evite `offers → telegram`, `offers → chi` ou entidades com tipos de transporte externos. Cada fonte deve normalizar sua entrada antes de invocar o domínio.

Quando um repository separar domínio de persistência com benefício real, mantenha o contrato no consumidor e injete a implementação. Escolha sua localização conforme coesão e convenções existentes; não concentre automaticamente toda persistência em `internal/database/` nem crie uma camada genérica obrigatória.

## Princípios e evolução

- Prefira simplicidade, packages pequenos e coesos e dependências explícitas. Não crie camadas somente para seguir um padrão arquitetural.
- Crie interfaces quando houver uma razão concreta, como isolar persistência ou transporte. Não exija uma interface para cada struct.
- Separe domínio de infraestrutura quando isso trouxer benefício real, sem introduzir frameworks ou abstrações prematuras.
- Não use packages `utils`, `helpers` ou `common` como depósitos de código sem responsabilidade definida; mantenha funções próximas de seu propósito.
- Novas fontes devem reutilizar conceitos e casos de uso internos sem exigir mudanças substanciais no domínio apenas por diferenças de formato externo.
- Futuras formas de notificação devem seguir a mesma separação por contratos e adaptadores, quando essa necessidade existir. Não implemente uma arquitetura de notificações antecipadamente.

## Criação e revisão

Leia os módulos afetados antes de escolher onde colocar uma mudança. Verifique quem compõe as dependências, quem traduz dados externos, onde vive a regra e se o domínio passa a conhecer detalhes de transporte. Avalie importações e contratos reais, sem impor uma árvore adicional de packages.

Se uma mudança alterar limites ou fluxo importante, registre o contexto conforme [documentation](../documentation/SKILL.md). Exemplos de estrutura não autorizam adicionar bibliotecas nem mudar a stack; essas decisões seguem `AGENTS.md` e a tarefa em andamento.

## Skills complementares

Os nomes abaixo correspondem às pastas atuais das skills específicas do Argos:

- [golang](../golang/SKILL.md): padrões gerais da implementação Go.
- [chi](../chi/SKILL.md): camada HTTP.
- [database](../database/SKILL.md): acesso ao banco em runtime.
- [migrations](../migrations/SKILL.md): evolução do schema.
- [documentation](../documentation/SKILL.md): documentação no código e em `docs/`.

Conteúdo original do Argos, sob a Apache License 2.0 do projeto, disponível em [LICENSE](../../../LICENSE).

## Referências oficiais

- [Documentação Go](https://go.dev/doc/)
- [Organização de módulos Go](https://go.dev/doc/modules/layout)
- [Chi](https://github.com/go-chi/chi)
- [database/sql](https://pkg.go.dev/database/sql)
