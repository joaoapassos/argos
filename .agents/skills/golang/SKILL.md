---
name: golang
description: Criar, revisar e refatorar código Go no Argos com packages pequenos, dependências explícitas, contexto e tratamento de erros idiomáticos.
license: Apache-2.0
---

# Go no Argos

`AGENTS.md` é a autoridade do projeto. Esta skill contém orientações gerais de implementação; consulte [architecture](../architecture/SKILL.md) para limites entre módulos.

## Organização e dependências

- Mantenha `cmd/` restrito à composição, inicialização e encerramento da aplicação. Código da aplicação pertence a `internal/`.
- Respeite a organização atual: `internal/config/` para configuração, `internal/database/` para inicialização do banco, `internal/sources/` para fontes externas, `internal/offers/` para ofertas e `internal/web/` para HTTP. Alterações de schema pertencem a `migrations/`.
- Prefira packages pequenos com responsabilidade clara e nomes que expressem seu propósito. Não crie camadas, `pkg/`, helpers genéricos ou uma nova árvore de domínio apenas para reproduzir um template.
- Defina interfaces pequenas, normalmente no consumidor, quando houver uma necessidade concreta de separar responsabilidades ou substituir uma dependência. Não crie uma interface para cada struct.
- Injete dependências explicitamente por construtores; retorne tipos concretos quando adequado. Evite dependências da aplicação em variáveis globais e service locators.
- Prefira parâmetros diretos ou uma struct de configuração simples. Functional options, frameworks de injeção, worker pools e outros padrões exigem uma necessidade concreta; não são requisitos padrão.
- A lógica de negócio não deve depender de HTTP ou Telegram. Adaptadores de fontes externas permanecem em `internal/sources/`.

## Contexto, erros e ciclo de vida

- Receba `context.Context` como primeiro argumento em operações que possam envolver I/O, banco, rede ou cancelamento. Propague o contexto recebido às dependências; não o substitua por `context.Background()` durante uma operação em andamento.
- Funções puras não precisam de contexto. Não use contexto como recipiente de dependências da aplicação.
- Quando criar um contexto com timeout ou cancelamento, libere-o com `defer cancel()`. Ao iniciar goroutines, defina quem as encerra e aguarda sua conclusão.
- Trate os erros retornados. Adicione contexto útil com `fmt.Errorf("buscar oferta %d: %w", id, err)` quando a informação ajudar o chamador; use `errors.Is`/`errors.As` para classificação, sem comparar mensagens.
- Não use `panic` para falhas operacionais esperadas. Preserve a distinção entre ausência, entrada inválida e falha de infraestrutura conforme o contrato do caso de uso.

## Exemplo de dependência no consumidor

Trecho ilustrativo em `internal/offers/`; `Offer` representa o tipo de domínio do caso de uso, não um DTO HTTP. Adapte nomes ao código existente.

```go
type offerReader interface {
    FindByID(ctx context.Context, id uint64) (Offer, error)
}

// Service coordena os casos de uso de ofertas.
type Service struct {
    reader offerReader
}

// NewService cria o serviço com sua dependência de leitura de ofertas.
func NewService(reader offerReader) *Service {
    return &Service{reader: reader}
}

// FindByID consulta uma oferta pelo identificador.
func (s *Service) FindByID(ctx context.Context, id uint64) (Offer, error) {
    offer, err := s.reader.FindByID(ctx, id)
    if err != nil {
        return Offer{}, fmt.Errorf("buscar oferta %d: %w", id, err)
    }
    return offer, nil
}
```

O exemplo usa `context` e `fmt`. A implementação da persistência é injetada na composição da aplicação, sem expor SQL ao consumidor.

## Criação e revisão

Antes de implementar, leia o package afetado e seus consumidores. Reutilize convenções existentes sem perpetuar conflitos com `AGENTS.md`. Na revisão, verifique responsabilidade do package, dependências explícitas, propagação do contexto, tratamento de erros e liberação de recursos.

Use `gofmt` e execute os testes pertinentes à mudança. Prefira `testing` e doubles simples; testes por tabela ajudam quando existem vários cenários comparáveis. Adicione testes que verifiquem comportamento relevante, sem exigir bibliotecas adicionais.

Tipos, funções, métodos e constantes exportados precisam de comentários GoDoc com `//`, iniciados pelo nome da declaração. Documente comportamento interno não óbvio, sem narrar o código. Consulte [documentation](../documentation/SKILL.md) para as demais convenções.

## Skills complementares

- [chi](../chi/SKILL.md): contratos e adaptação HTTP.
- [database](../database/SKILL.md): persistência com MySQL e `database/sql`.
- [migrations](../migrations/SKILL.md): alterações de schema.

Conteúdo original do Argos, sob a Apache License 2.0 do projeto, disponível em [LICENSE](../../../LICENSE).

## Referências oficiais

- [Documentação Go](https://go.dev/doc/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Packages Go](https://pkg.go.dev/)
- [context](https://pkg.go.dev/context)
- [errors](https://pkg.go.dev/errors)
