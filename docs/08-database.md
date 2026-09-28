# 08 — Database — S7 Coach OS

> Schema do banco de dados. Roda na infraestrutura PostgreSQL própria da S7 (Dokploy), não em Supabase.
> Versão: 0.1.0

---

## 0. Contexto de infraestrutura

O S7 Coach OS **não usa Supabase**. Usa a infra PostgreSQL centralizada da S7, hospedada via Dokploy, compartilhada com outros serviços (Hook7, n8n, Cal.com etc.) — um Postgres único com múltiplos bancos separados por serviço.

| Item | Decisão |
|---|---|
| Banco | PostgreSQL (infra Dokploy da S7) |
| Nome do database | `s7_coach_os` (schema isolado, não compartilha tabelas com outros serviços) |
| Auth | **JWT próprio** — não usa Supabase Auth. API do Coach OS emite/valida tokens (login/senha com hash bcrypt, refresh token) |
| Storage de arquivos (fotos de evolução/refeições) | Abstraído via interface própria (`storage_provider`), assumindo S3-compatível (ex: MinIO) na mesma VPS — ver seção 8. Trocável sem alterar schema |
| Migrations | SQL puro versionado em `database/migrations/` |
| Admin do banco | Adminer (já em uso na infra da S7) |

> Nota: como o auth não é mais gerenciado por um serviço externo (Supabase Auth), a tabela `users` abaixo carrega os campos de autenticação diretamente. Nenhuma tabela usa Row Level Security (RLS) do Postgres — o isolamento de dados por usuário é feito na camada de aplicação (API), via `user_id` obrigatório em toda query.

---

## 1. Tabela `users`

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email           TEXT UNIQUE NOT NULL,
  password_hash   TEXT NOT NULL,          -- bcrypt
  nome            TEXT NOT NULL,
  criado_em       TIMESTAMPTZ NOT NULL DEFAULT now(),
  atualizado_em   TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE refresh_tokens (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token_hash      TEXT NOT NULL,
  expira_em       TIMESTAMPTZ NOT NULL,
  criado_em       TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Autenticação: login gera JWT de curta duração (access token, ~15min) + refresh token de longa duração armazenado com hash. API valida o JWT em todo request; nenhuma tabela de negócio é acessível sem `user_id` extraído de um token válido.

---

## 2. Memória Permanente → `user_profile`

```sql
CREATE TABLE user_profile (
  user_id                 UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  idade                   INT,
  sexo                    TEXT,
  altura_cm               NUMERIC,
  objetivo                TEXT CHECK (objetivo IN ('perder_gordura','hipertrofia','ambos')),
  local_treino_padrao     TEXT CHECK (local_treino_padrao IN ('academia','casa','ambos')),
  academia_cadastrada     TEXT,
  horario_treino_habitual TIME,
  rotina_trabalho         JSONB,          -- { tipo, turno }
  modo_coach              TEXT CHECK (modo_coach IN ('ativo','rapido')) DEFAULT 'rapido',
  atualizado_em           TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE user_restricoes_alimentares (
  id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id   UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  alimento  TEXT NOT NULL
);
```

Regra 5.1 da Constituição: só a API de Configurações grava aqui. Nenhum job/engine de IA escreve nesta tabela.

---

## 3. Memória da Fase → `phase_state`

```sql
CREATE TABLE phase_state (
  id                        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id                   UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  semana_atual              INT NOT NULL DEFAULT 1,
  fase_inicio               DATE NOT NULL,
  peso_inicio_fase          NUMERIC,
  peso_atual                NUMERIC,
  medidas                   JSONB,        -- { cintura, peito, braco, ... }
  calorias_meta             INT,
  proteina_g                INT,
  carboidratos_g            INT,
  gorduras_g                INT,
  agua_litros                NUMERIC,
  consistencia_semana_anterior NUMERIC,
  atualizado_em             TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Revisada semanalmente por job agendado — não a cada request (regra 5.2).

---

## 4. Memória do Dia → `daily_context`

```sql
CREATE TABLE daily_context (
  id                         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id                    UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  data                       DATE NOT NULL,
  sono_horas                 NUMERIC,
  dor_muscular               INT CHECK (dor_muscular BETWEEN 0 AND 5),
  energia                    INT CHECK (energia BETWEEN 1 AND 5),
  trabalho_pesado_hoje       BOOLEAN,
  tempo_disponivel_treino_min INT,
  UNIQUE (user_id, data)
);
```

Nunca é lida para decisões fora da própria `data` (regra 3.3). Não há job de expiração explícito — a query sempre filtra por `data = hoje`.

---

## 5. Plano Mestre de Treino

```sql
CREATE TABLE training_plan (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  dias_semana  INT NOT NULL,
  estrutura    JSONB NOT NULL,   -- { segunda: "treino_a", quarta: "treino_b", ... }
  vigente_desde DATE NOT NULL,
  ativo        BOOLEAN NOT NULL DEFAULT true
);

CREATE TABLE exercise_library (
  id                     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  nome                   TEXT NOT NULL,
  grupo_muscular         TEXT NOT NULL,
  categoria              TEXT CHECK (categoria IN ('principal','auxiliar','isolador','substituto')),
  equipamento            TEXT,
  local                  TEXT CHECK (local IN ('academia','casa','ambos')),
  substitutos            UUID[],
  articulacoes_envolvidas TEXT[]
);

CREATE TABLE daily_workout (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  data          DATE NOT NULL,
  nome_treino   TEXT,             -- "Treino B"
  versao        TEXT CHECK (versao IN ('expressa','padrao','completa','estendida')),
  exercicios    JSONB NOT NULL,   -- lista ordenada com carga/séries/reps sugeridos
  gerado_em     TIMESTAMPTZ NOT NULL DEFAULT now(),
  concluido     BOOLEAN NOT NULL DEFAULT false
);

CREATE TABLE workout_log (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  daily_workout_id UUID NOT NULL REFERENCES daily_workout(id) ON DELETE CASCADE,
  exercicio_id    UUID NOT NULL REFERENCES exercise_library(id),
  peso_kg         NUMERIC,
  series_reps     JSONB,          -- [{serie: 1, reps: 12}, ...]
  feedback        TEXT CHECK (feedback IN ('muito_facil','facil','ideal','dificil','nao_terminou')),
  motivo_extra    TEXT,           -- resposta do nível 2 (sem força, dor articular, etc.)
  registrado_em   TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 6. Alimentação

```sql
CREATE TABLE daily_meal_plan (
  id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id        UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  data           DATE NOT NULL,
  refeicao_nome  TEXT NOT NULL,
  horario_sugerido TIME,
  proteina_alvo_g NUMERIC,
  itens_sugeridos JSONB NOT NULL   -- [{ alimento, quantidade_coloquial, icone }]
);

CREATE TABLE meal_log (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id           UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  daily_meal_plan_id UUID REFERENCES daily_meal_plan(id) ON DELETE SET NULL,
  status            TEXT CHECK (status IN ('confirmada','trocada','registrada_livre')),
  itens_reais       JSONB NOT NULL,  -- [{ alimento, quantidade_coloquial, macros_estimados }]
  macros_totais     JSONB,           -- { proteina_g, carboidratos_g, gorduras_g, calorias }
  registrado_em     TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 7. Diário / Timeline

```sql
CREATE TABLE diary_event (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  timestamp   TIMESTAMPTZ NOT NULL DEFAULT now(),
  tipo        TEXT CHECK (tipo IN ('refeicao','treino','peso','agua','sono','humor','trabalho','observacao')),
  origem_ref  UUID,        -- id da missão/registro de origem, polimórfico por tipo
  payload     JSONB NOT NULL
);

CREATE INDEX idx_diary_event_user_data ON diary_event (user_id, timestamp);
```

Imutável — nenhum `UPDATE`/`DELETE` a partir da API de usuário (ver `04-context-engine.md` §2).

---

## 8. Fotos e arquivos → `user_files`

```sql
CREATE TABLE user_files (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tipo         TEXT CHECK (tipo IN ('foto_evolucao','foto_refeicao')),
  storage_key  TEXT NOT NULL,     -- chave/path no provider de storage, não a URL final
  storage_provider TEXT NOT NULL DEFAULT 's3_compat',
  criado_em    TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

`storage_provider` é assumido S3-compatível (MinIO na mesma VPS) até decisão final — a API deve acessar arquivos só através de uma interface `StorageProvider` (upload/get_url/delete), nunca hardcoded, para trocar de provider sem migrar dados.

---

## 9. Água e Creatina (missões rápidas, sem tabela própria de "plano")

```sql
CREATE TABLE water_log (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  data        DATE NOT NULL,
  ml_total    INT NOT NULL DEFAULT 0,
  UNIQUE (user_id, data)
);

CREATE TABLE creatine_log (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  data        DATE NOT NULL,
  tomada      BOOLEAN NOT NULL DEFAULT false,
  UNIQUE (user_id, data)
);
```

---

## 10. Consistência (calculada, com cache diário)

```sql
CREATE TABLE consistency_daily (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id           UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  data              DATE NOT NULL,
  treino_ok         BOOLEAN,
  proteina_ok       BOOLEAN,
  agua_ok           BOOLEAN,
  sono_ok           BOOLEAN,
  checkin_ok        BOOLEAN,
  score_dia         NUMERIC,    -- 0-100, calculado pela fórmula do Context Engine
  UNIQUE (user_id, data)
);
```

`score_dia` é recalculado (não recebido do cliente) por job/rotina no backend, seguindo a fórmula de `04-context-engine.md` §3. `consistencia_semana` e a média móvel de 21 dias são derivadas via query, não armazenadas redundantemente.

---

## 11. Índices e observações gerais

* Toda tabela de domínio (treino, alimentação, diário, água, creatina, consistência) tem `user_id` indexado — nenhuma tabela é global entre usuários.
* Sem RLS do Postgres: **toda query da API precisa filtrar por `user_id` explicitamente**. Isso é responsabilidade da camada de aplicação, já que não há Supabase para fazer isso automaticamente.
* `exercise_library` é a única tabela verdadeiramente global (compartilhada entre usuários) — biblioteca fechada mantida pela S7, não editável pelo usuário final.
* Migrations ficam em `database/migrations/`, versionadas e aplicadas via ferramenta simples (ex: `node-pg-migrate`, `golang-migrate` ou script SQL sequencial — a decidir conforme stack final do backend).
* O banco `s7_coach_os` roda na mesma instância Postgres compartilhada com Hook7, n8n e Cal.com (infra Dokploy) — mas como database isolado, não como schema dentro de um banco compartilhado.

---

## 12. O que mudou em relação à versão anterior (Supabase)

| Antes (Supabase) | Agora (Postgres próprio / Dokploy) |
|---|---|
| Supabase Auth | JWT próprio + tabela `users`/`refresh_tokens` |
| Supabase Storage | `user_files` + `StorageProvider` abstrato (assume S3-compatível) |
| RLS automático | Filtro por `user_id` obrigatório na camada de API |
| Painel Supabase Studio | Adminer (já em uso na infra S7) |
| `supabase/migrations/` | `database/migrations/` |
