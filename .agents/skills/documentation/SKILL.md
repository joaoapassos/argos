---
name: documentation
description: Escrever e revisar documentação do Argos em português, separando GoDoc e detalhes de implementação de arquitetura, contratos públicos, regras de negócio e decisões em docs/.
license: Apache-2.0
---

# Documentação no Argos

`AGENTS.md` é a autoridade principal. Mantenha a documentação específica do Argos em português; APIs, comandos, caminhos e termos idiomáticos podem permanecer em inglês. Documente o comportamento confirmado e diferencie propostas de funcionalidades implementadas.

## Divisão de responsabilidades

`docs/` explica arquitetura, decisões, regras de negócio, comportamento geral e fluxos importantes. Detalhes da implementação permanecem próximos ao código. Evite duplicar algoritmos, listas de funções ou SQL completo nos dois lugares; faça referências quando isso ajudar a manutenção.

## Comentários no código Go

- Tipos, funções, métodos e constantes exportados devem usar comentários GoDoc com `//`, iniciados pelo nome da declaração. Explique propósito, contrato e efeitos relevantes.
- Documente funções internas importantes quando o comportamento não for evidente. Comentários devem explicar intenção, invariantes, restrições ou escolhas incomuns; não repetir o que a instrução já mostra.
- Tipos importantes de domínio devem esclarecer seu propósito e significado dos campos quando necessário. Regras complexas devem incluir contexto suficiente para manutenção, com referência a uma decisão ou regra conceitual quando útil.
- Handlers HTTP devem indicar método, rota e responsabilidade. O comentário não substitui a descrição do contrato público na documentação da API.

Exemplos ilustrativos, a adaptar aos nomes e contratos reais:

```go
// Offer representa uma oferta recebida de uma fonte e associada a um produto.
// listOffers atende GET /api/offers e retorna as ofertas consultadas pelo serviço.
```

Veja [golang](../golang/SKILL.md) para implementação e [chi](../chi/SKILL.md) para responsabilidades HTTP.

## Organização de docs/

Use os documentos conforme o assunto e a necessidade real, sem criar arquivos vazios apenas para completar a estrutura:

| Caminho | Responsabilidade |
| --- | --- |
| `docs/README.md` | Índice, orientação de leitura e links para documentos existentes. |
| `docs/architecture.md` | Módulos, limites, dependências e fluxos entre componentes. |
| `docs/business-rules.md` | Regras de ofertas, relevância, preços e histórico conforme decisões do domínio. |
| `docs/database.md` | Entidades, relações, precisão monetária e estratégia de evolução do schema. |
| `docs/api.md` | Rotas implementadas, entradas, respostas, status e erros públicos. |
| `docs/sources.md` | Integrações de entrada, normalização e limites entre formatos externos e conceitos internos. |
| `docs/decisions/` | Decisões arquiteturais importantes, com contexto, decisão, motivo e consequências. |

Mantenha decisões identificáveis e referências ao documento que substitui uma decisão anterior, quando houver. Não exija um registro separado para cada escolha trivial.

## Referências ao código e manutenção

Quando explicar uma implementação, confira o arquivo real e use um link relativo ao documento. Por exemplo, em `docs/architecture.md`, a referência `[modelo de ofertas](../internal/offers/model.go)` só é válida se esse arquivo existir e realmente definir a entidade `Offer`. Não invente caminhos nem descreva exemplos de estrutura como implementação existente.

Ao alterar arquitetura, comportamento público, integração ou regra de negócio, verifique quais documentos precisam acompanhar a mudança. Atualize o índice quando adicionar ou reorganizar documentos. Preserve contexto conceitual e evite copiar o funcionamento interno de cada função para `docs/`.

Na revisão, confira consistência com `AGENTS.md`, precisão dos exemplos, links locais, status de funcionalidades e ausência de duplicação. Referências oficiais devem sustentar orientações técnicas; decisões do Argos devem ser apresentadas como decisões do projeto.

## Skills complementares

- [architecture](../architecture/SKILL.md): limites entre módulos.
- [database](../database/SKILL.md): persistência em runtime.
- [migrations](../migrations/SKILL.md): evolução do schema e documentação de riscos.

Conteúdo original do Argos, sob a Apache License 2.0 do projeto, disponível em [LICENSE](../../../LICENSE).

## Referências oficiais

- [Documentação Go](https://go.dev/doc/)
- [Comentários de documentação Go](https://go.dev/doc/comment)
- [Packages Go](https://pkg.go.dev/)
