---
name: migrations
description: Criar, revisar, aplicar ou reverter migrations do Argos com MySQL e golang-migrate, preservando integridade, precisão monetária e histórico de schema.
license: Apache-2.0
---

# Migrations no Argos

`AGENTS.md` é a autoridade principal. Esta skill trata a evolução do schema em `migrations/`, na raiz do projeto. A skill [database](../database/SKILL.md) trata acesso ao banco em runtime; não use inicialização da aplicação ou repositories para criar ou alterar tabelas.

## Versionamento e escopo

- Toda alteração de schema deve usar `golang-migrate`, com SQL compatível com o MySQL adotado pelo projeto.
- Cada versão possui arquivos `.up.sql` e `.down.sql`. Nunca altere uma migration já aplicada; crie uma nova versão para alterações posteriores.
- Verifique as versões existentes antes de escolher o próximo número sequencial. Use nomes descritivos em `snake_case`, por exemplo `000003_create_price_history.up.sql` e `000003_create_price_history.down.sql`; o número é ilustrativo.
- Prefira SQL simples, explícito e reproduzível sobre o estado esperado da versão anterior. Não acrescente lógica de negócio, triggers, procedures ou funcionalidades ocultas sem decisão explícita.
- Use nomes do domínio, como `products`, `offers`, `sources` e `price_history`. Crie tabelas referenciadas antes das dependentes e reverta na ordem adequada.

## Tipos, relações e índices

Use `DECIMAL` para preços; nunca `FLOAT` ou `DOUBLE`. Escolha precisão e escala conforme limites e regras reais do domínio, mantendo compatibilidade com a representação monetária usada em Go. `DECIMAL(12, 2)` é um exemplo, não uma escala obrigatória para todo valor.

Defina foreign keys com tipos compatíveis entre as colunas relacionadas. Escolha `ON DELETE` conscientemente e `ON UPDATE` quando relevante; não presuma cascatas. Explique consequências como preservar o histórico de preços ou impedir a exclusão de um produto referenciado.

Crie índices conforme consultas, filtros, ordenação e relacionamentos reais. Considere índices já criados por constraints para evitar duplicação. Justifique mudanças com padrões de acesso; não indexe todos os campos por padrão.

## Exemplo de schema

Par ilustrativo para uma instalação nova, em que `products` e `sources` já existem com `id BIGINT UNSIGNED`. Ajuste colunas e precisão às migrations reais; este exemplo não define o schema atual do Argos.

```sql
-- up: cria ofertas vinculadas ao produto e à fonte.
CREATE TABLE offers (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    product_id BIGINT UNSIGNED NOT NULL,
    source_id BIGINT UNSIGNED NOT NULL,
    price DECIMAL(12, 2) NOT NULL,
    PRIMARY KEY (id),
    CONSTRAINT fk_offers_product FOREIGN KEY (product_id)
        REFERENCES products (id) ON DELETE RESTRICT,
    CONSTRAINT fk_offers_source FOREIGN KEY (source_id)
        REFERENCES sources (id) ON DELETE RESTRICT
) ENGINE=InnoDB;
```

```sql
-- down: destrutivo; remove todas as ofertas armazenadas nesta tabela.
DROP TABLE offers;
```

`RESTRICT` exemplifica a decisão de impedir exclusões de registros referenciados; escolha outra política somente quando o domínio justificar.

## Reversão e execução

- Faça o `down` reverter o `up` quando razoavelmente possível. Reverter a estrutura não recupera dados apagados: documente perda de dados e limites da reversão. Se não houver reversão segura, explicite a limitação e a estratégia de recuperação no arquivo `down` e na descrição da mudança; não apresente um arquivo vazio como reversão bem-sucedida.
- Identifique operações destrutivas, como remover tabelas ou colunas, reduzir precisão e mudar tipos incompatíveis. Prefira estratégias que preservem dados e avalie locks, duração e compatibilidade com a aplicação para mudanças em tabelas existentes.
- MySQL pode realizar commits implícitos em DDL. Não presuma rollback atômico de uma migration inteira; considere recuperação de aplicação parcial e estado `dirty` do golang-migrate. Não use `force` para ocultar uma falha sem conferir o estado real do schema.
- Antes de aplicar ou reverter, confira ambiente, versão atual e efeitos esperados. Não exponha credenciais em logs ou comandos exibidos. Gerar arquivos não implica executar migrations em um ambiente compartilhado.
- Valide SQL e o ciclo `up`/`down` em MySQL de teste quando pertinente, incluindo relações, precisão e dados existentes. Não substitua essa validação por SQLite.

## Revisão e skills complementares

Confira sequência de versões, imutabilidade das migrations aplicadas, pares `up`/`down`, compatibilidade MySQL, relações, índices e riscos de perda. Siga [architecture](../architecture/SKILL.md) para limites e [documentation](../documentation/SKILL.md) para registrar decisões em `docs/database.md` ou implicações em `docs/business-rules.md`, quando necessário. Detalhes SQL permanecem nas migrations.

Conteúdo original do Argos, sob a Apache License 2.0 do projeto, disponível em [LICENSE](../../../LICENSE).

## Referências oficiais

- [golang-migrate](https://github.com/golang-migrate/migrate)
- [MySQL](https://dev.mysql.com/doc/)
- [Commits implícitos no MySQL](https://dev.mysql.com/doc/refman/8.4/en/implicit-commit.html)
