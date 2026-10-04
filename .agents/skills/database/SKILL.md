---
name: database
description: Implementar, revisar e depurar persistência do Argos com MySQL e database/sql, queries parametrizadas, contexto, transações e precisão monetária. Alterações de schema usam a skill migrations.
license: Apache-2.0
---

# Persistência no Argos

`AGENTS.md` é a autoridade. Esta skill adapta os conceitos da referência externa `golang-database` à stack do Argos, sem incorporar suas recomendações incompatíveis.

## Stack e limites

- Use MySQL e `database/sql`, com o driver MySQL definido pelo projeto. Confira `go.mod` e a inicialização existente antes de escolher imports ou opções do driver. Se ainda não houver driver definido, explicite essa lacuna; não presuma uma dependência existente.
- A recomendação externa de usar `sqlx` ou `pgx` não se aplica. Não use `sqlx`, `pgx`, GORM, ent ou outro ORM sem uma decisão explícita futura que altere a arquitetura.
- Inicialização do banco pertence a `internal/database/`. Injete `*sql.DB` nas implementações de persistência; não abra uma conexão por requisição. Veja [golang](../golang/SKILL.md) para construtores, contexto e erros.
- Repositories podem encapsular persistência quando isso separar domínio de infraestrutura. Prefira operações específicas do domínio, sem um repository genérico obrigatório. Respeite [architecture](../architecture/SKILL.md); handlers HTTP nunca acessam o banco diretamente.
- Mantenha SQL explícito. Não introduza triggers, procedures ou regras de negócio escondidas no banco sem decisão explícita.
- Esta skill não trata alterações de schema. Use [migrations](../migrations/SKILL.md), com MySQL e `golang-migrate`, em vez das alternativas da referência externa. A exclusão de migrations deste escopo não proíbe criá-las quando a tarefa as solicitar, seguindo aquela skill.

## Queries, resultados e recursos

- Parametrize todos os valores externos usando `?` e argumentos separados; nunca concatene input em SQL. Placeholders não representam nomes de tabela, coluna ou direção de ordenação: selecione trechos SQL constantes de uma lista permitida para esses casos.
- Prefira `QueryContext` para várias linhas, `QueryRowContext` para uma linha, `ExecContext` para comandos sem linhas e `PrepareContext` quando a reutilização justificar preparar uma instrução.
- Liste as colunas e mantenha a ordem de `Scan` compatível. Trate seus erros e represente `NULL` com tipos como `sql.NullString` ou ponteiros adequados ao contrato; não apague diferenças entre ausência e valor zero por conveniência.
- `QueryRowContext` reporta erros em `Scan`. Trate `sql.ErrNoRows` explicitamente com `errors.Is`, traduzindo para ausência ou erro de domínio conforme o contrato existente.
- Após um `QueryContext` bem-sucedido, use `defer rows.Close()`. Verifique `rows.Err()` após a iteração e feche resultados antes de executar operações que precisem liberar a conexão.
- Verifique os erros de `ExecContext` e dos métodos de `sql.Result` usados. Quando uma atualização exigir saber se houve correspondência, considere a semântica de linhas afetadas do driver e sua configuração.

## Exemplo de leitura

Trecho ilustrativo de persistência; as colunas devem ser conferidas nas migrations reais. `sourceID` filtra ofertas de uma fonte, sem transformar o contrato em um repository genérico.

```go
func listOfferIDs(ctx context.Context, db *sql.DB, sourceID uint64) ([]uint64, error) {
    rows, err := db.QueryContext(ctx,
        "SELECT id FROM offers WHERE source_id = ? ORDER BY id", sourceID)
    if err != nil {
        return nil, fmt.Errorf("consultar ofertas da fonte %d: %w", sourceID, err)
    }
    defer rows.Close()

    ids := make([]uint64, 0)
    for rows.Next() {
        var id uint64
        if err := rows.Scan(&id); err != nil {
            return nil, fmt.Errorf("ler identificador da oferta: %w", err)
        }
        ids = append(ids, id)
    }
    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("iterar ofertas: %w", err)
    }
    return ids, nil
}
```

Para uma consulta individual, a decisão de ausência deve ficar visível:

```go
var id uint64
err := db.QueryRowContext(ctx, "SELECT id FROM offers WHERE id = ?", offerID).Scan(&id)
if errors.Is(err, sql.ErrNoRows) {
    return 0, false, nil // Contrato ilustrativo: (id, existe, erro).
}
if err != nil {
    return 0, false, fmt.Errorf("consultar oferta %d: %w", offerID, err)
}
return id, true, nil
```

Os trechos usam `context`, `database/sql`, `errors` e `fmt`; não representam um schema já implementado.

## Transações e concorrência

Use `BeginTx(ctx, ...)` quando várias operações precisarem ser atômicas, por exemplo atualizar uma oferta e registrar seu preço em `price_history`. Execute todas as operações desse conjunto pelo mesmo `*sql.Tx`, sem alternar com `db.ExecContext` ou outras chamadas fora da transação.

Garanta rollback nos caminhos de falha, normalmente com um `defer` logo após abrir a transação. Trate erros de rollback relevantes sem ocultar a falha original; `sql.ErrTxDone` é esperado após uma transação já encerrada. Verifique `Commit()` antes de informar sucesso. Mantenha a transação curta e evite chamadas de rede enquanto retém locks.

Não copie a suposição externa de que `READ COMMITTED` é o isolamento padrão. `BeginTx(ctx, nil)` usa o padrão configurado no banco/driver. Escolha isolamento e `SELECT ... FOR UPDATE` quando a consistência do caso de uso justificar; não imponha locks a toda leitura nem isolamento serializável a toda operação monetária.

## Pool e precisão

Configure conscientemente `SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime` e `SetConnMaxIdleTime` conforme capacidade do MySQL, número de instâncias, carga e timeouts da infraestrutura, sem copiar valores universais. `sql.Open` não garante conectividade: use `PingContext` quando precisar validar a conexão na inicialização. Defina quem fecha o `*sql.DB` no encerramento; `db.Stats()` pode apoiar ajustes com evidência.

As migrations determinam precisão e escala de valores monetários, usando `DECIMAL`. Não converta preços para `float32`/`float64` ao ler, calcular ou gravar. Preserve a representação decimal exata, por exemplo lendo como string e convertendo para o tipo monetário adotado pelo domínio sem passar por ponto flutuante. Uma string preserva transporte, mas não resolve cálculos: escala, limites e arredondamento devem ser explícitos. Não adicione biblioteca decimal sem decisão de necessidade e verificação de licença.

## Revisão e verificação

Confira argumentos parametrizados, propagação de contexto, `Scan`, ausência, `NULL`, fechamento de resultados, erros de iteração, atomicidade e precisão monetária. Testes de integração relevantes devem usar MySQL com schema preparado pelas migrations; SQLite não substitui a verificação de semântica MySQL. Doubles de repositories podem isolar testes do caso de uso, sem exigir frameworks adicionais.

Siga [documentation](../documentation/SKILL.md) para documentar contratos e decisões importantes. Conteúdo original do Argos, sob a Apache License 2.0 do projeto, disponível em [LICENSE](../../../LICENSE).

## Referências oficiais

- [database/sql](https://pkg.go.dev/database/sql)
- [Acesso a bancos em Go](https://go.dev/doc/database/)
- [Queries e resultados](https://go.dev/doc/database/querying)
- [Transações em Go](https://go.dev/doc/database/execute-transactions)
- [MySQL](https://dev.mysql.com/doc/)
