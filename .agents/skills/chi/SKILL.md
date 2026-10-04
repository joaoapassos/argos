---
name: chi
description: Criar e revisar rotas, handlers, DTOs e testes HTTP do Argos em internal/web usando Chi v5 e net/http, separados dos casos de uso.
license: Apache-2.0
---

# HTTP no Argos

`AGENTS.md` prevalece sobre esta skill e as referências externas. Use [golang](../golang/SKILL.md) para padrões Go e [architecture](../architecture/SKILL.md) para limites entre módulos.

## Contratos e responsabilidades

Use `github.com/go-chi/chi/v5` com implementação em `internal/web/`. Prefira `net/http` e `encoding/json`; não introduza `go-chi/render` ou outra dependência sem necessidade concreta e verificação de licença.

Mantenha os handlers pequenos, limitados ao fluxo:

1. Receber e interpretar parâmetros, query string e corpo da requisição.
2. Validar a entrada do contrato HTTP, como formato e campos obrigatórios.
3. Chamar o service/use case apropriado, propagando `r.Context()`.
4. Converter o resultado para DTO HTTP quando o contrato diferir do domínio.
5. Produzir status, headers e corpo da resposta.

Validações de negócio pertencem ao caso de uso. Não consulte o banco diretamente nem implemente regras de ofertas nos handlers. Entidades de domínio não devem importar HTTP ou conhecer contratos de transporte. DTOs com nomes JSON, campos ou representações diferentes permanecem na camada HTTP; não duplique tipos sem diferença concreta de contrato.

## Rotas e middleware

Organize rotas por domínio/recurso. Extraia funções de registro ou subrouters quando o volume justificar, sem exigir uma árvore complexa para poucas rotas.

Trecho ilustrativo de registro, com handlers injetados pela composição; os contratos finais dependem dos casos de uso implementados:

```go
// NewRouter organiza os endpoints HTTP do Argos.
func NewRouter(listOffers, listSources, health http.HandlerFunc) http.Handler {
    r := chi.NewRouter()
    r.Get("/health", health)
    r.Route("/api", func(r chi.Router) {
        r.Get("/offers", listOffers)
        r.Get("/sources", listSources)
    })
    return r
}
```

Use middleware apenas para uma responsabilidade transversal real, como identificação da requisição ou recuperação de panic. Escolha escopo e ordem conscientemente; registre-o antes das rotas afetadas. Não copie listas de middleware por padrão. Se usar headers de proxy para identificar IP, considere a confiança nos proxies configurados.

## Respostas e erros

- Defina `Content-Type` antes de escrever o corpo e o status antes da resposta. Para JSON, serialize antes de enviar headers quando precisar tratar uma falha de serialização com outro status.
- Trate erros de leitura, parsing, serialização e escrita. Depois de iniciar a resposta, não tente enviar um segundo status; registre a falha de escrita pelo mecanismo de observabilidade existente.
- Traduza erros conhecidos do caso de uso para o contrato HTTP, por exemplo ausência para `404` ou entrada inválida para `400`, conforme o endpoint. Falhas internas recebem mensagem pública estável, nunca `err.Error()`.
- Ao aceitar JSON, valide o formato e os limites de entrada de acordo com o contrato. Rejeição de campos desconhecidos e limites de tamanho devem ser escolhas explícitas do endpoint.
- Defina se `/health` indica processo ativo ou prontidão das dependências. Se precisar verificar dependências, use um componente apropriado; não coloque SQL no handler.

Todo handler deve documentar método, rota e responsabilidade, inclusive se não exportado:

```go
// listOffers atende GET /api/offers e retorna ofertas consultadas pelo serviço.
// listSources atende GET /api/sources e retorna as fontes disponíveis.
// health atende GET /health e informa se o processo está ativo.
```

Esses comentários exemplificam a documentação das respectivas funções; consulte [documentation](../documentation/SKILL.md).

## Testes e revisão

Use `httptest.NewRequest`, `httptest.NewRecorder` e o router real para verificar método, rota, parâmetros, status, headers e payload. Injete um fake pequeno do caso de uso para testar sucesso, entrada inválida, ausência e falha interna quando aplicáveis, incluindo que detalhes internos não vazem.

Exemplo de exercício do registro de rotas, usando `NewRouter` acima:

```go
func TestHealth(t *testing.T) {
    // O stub isola o registro da rota; o handler real exige seus próprios cenários.
    health := func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusNoContent)
    }
    router := NewRouter(health, health, health)
    recorder := httptest.NewRecorder()
    router.ServeHTTP(recorder, httptest.NewRequest(http.MethodGet, "/health", nil))
    if recorder.Code != http.StatusNoContent {
        t.Fatalf("status = %d, esperado %d", recorder.Code, http.StatusNoContent)
    }
}
```

Os trechos usam `net/http`, `github.com/go-chi/chi/v5`, `net/http/httptest` e `testing`. São exemplos de organização, não uma definição de endpoints já existentes.

Conteúdo original do Argos, sob a Apache License 2.0 do projeto, disponível em [LICENSE](../../../LICENSE).

## Referências oficiais

- [Chi](https://github.com/go-chi/chi)
- [net/http](https://pkg.go.dev/net/http)
- [httptest](https://pkg.go.dev/net/http/httptest)
- [encoding/json](https://pkg.go.dev/encoding/json)
- [Documentação Go](https://go.dev/doc/)
