# 04 — Context Engine — S7 Coach OS

> Como o Coach guarda, organiza e usa contexto sobre o usuário.
> Versão: 0.1.0
> Sujeito às regras de `03-ai-constitution.md` (seção 5).

---

## 1. As Três Memórias

O Context Engine é o que permite ao Workout Engine e ao Nutrition Engine tomarem decisões sem perguntar tudo de novo a cada interação. Existem três camadas, com regras de escrita e expiração diferentes.

| Memória | Muda quando | Expira | Quem escreve |
|---|---|---|---|
| Permanente | Ação explícita do usuário em Configurações/Onboarding | Nunca | Usuário (via app) |
| Da Fase | Revisão semanal automática | 7 dias (ciclo) | Coach (revisão semanal) |
| Do Dia | Check-in diário / eventos do dia | 24h | Coach (a partir de check-ins e eventos) |

### 1.1 Memória Permanente

```
{
  nome: string
  idade: number
  sexo: string
  altura_cm: number
  objetivo: "perder_gordura" | "hipertrofia" | "ambos"
  restricoes_alimentares: [string]
  local_treino_padrao: "academia" | "casa" | "ambos"
  academia_cadastrada: string | null
  horario_treino_habitual: string
  rotina_trabalho: {
    tipo: string,               // ex: "trabalho físico + sedentário"
    turno: string
  }
  modo_coach: "ativo" | "rapido"
}
```

Regra 5.1 da Constituição: **nunca** é sobrescrita por inferência da IA. Só muda quando o usuário edita em Configurações.

### 1.2 Memória da Fase

```
{
  semana_atual: number
  fase_inicio: date
  peso_inicio_fase: number
  peso_atual: number
  medidas: { cintura, peito, braco, ... } | null
  plano_mestre_treino: ref(training_plan)
  plano_fase_nutricao: {
    calorias_meta, proteina_g, carboidratos_g, gorduras_g, agua_litros
  }
  cargas_atuais: [{ exercicio_id, peso_kg, data_ultima_sessao }]
  consistencia_semana_anterior: number  // 0-100
}
```

Revisada em cadência semanal (regra 5.2). A revisão considera: peso registrado, consistência real vs. esperada, progressão de carga acumulada.

### 1.3 Memória do Dia

```
{
  data: date
  sono_horas: number | null
  dor_muscular: 0-5 | null
  energia: 1-5 | null
  trabalho_pesado_hoje: boolean | null
  tempo_disponivel_treino_min: number | null
  eventos_do_dia: [ref(diary_event)]
}
```

Expira às 00:00 (regra 3.3 e 5.3 da Constituição) — nunca influencia decisões do dia seguinte. Um novo registro de Memória do Dia é criado a cada dia, vazio até o primeiro check-in ou evento.

---

## 2. Timeline / Diário

Cada evento do dia gera um registro imutável na timeline. A timeline **não é editável** pelo usuário (é log, não formulário) — para corrigir algo, o usuário registra um novo evento ou edita a missão de origem, nunca a linha do tempo diretamente.

### 2.1 Estrutura de um evento

```
{
  evento_id: string
  timestamp: datetime
  tipo: "refeicao" | "treino" | "peso" | "agua" | "sono" | "humor" | "trabalho" | "observacao"
  origem: ref(missao | check-in | registro manual)
  payload: { ... específico do tipo }
}
```

### 2.2 O que cada evento atualiza

| Evento | Atualiza |
|---|---|
| Treino concluído | Memória da Fase (cargas), Consistência do dia |
| Refeição registrada | Memória do Dia (proteína/macros acumulados), Nutrition Engine (compensação) |
| Dormiu pouco | Memória do Dia (sono), reduz intensidade sugerida do treino de hoje |
| Peso registrado | Memória da Fase (peso_atual), Evolução |
| Não treinou | Reagendamento automático (Workout Engine §2) |

---

## 3. Cálculo de Consistência

Métrica composta, não um contador de acesso ao app.

### 3.1 Componentes e pesos (padrão do MVP)

| Componente | Peso |
|---|---|
| Treino concluído | 30% |
| Proteína batida (≥90% da meta) | 25% |
| Água (≥90% da meta) | 15% |
| Sono (≥6h) | 15% |
| Check-in do dia preenchido | 15% |

Pesos ajustáveis por fase futura — no MVP, fixos conforme acima.

### 3.2 Fórmula

```
consistencia_dia = Σ (peso_componente × (1 se cumprido, 0 se não))
consistencia_semana = média dos 7 dias
consistencia_geral = média móvel dos últimos 21 dias
```

A média móvel de 21 dias é a usada para "Chance de atingir a meta" (seção 4).

### 3.3 Semáforo

| Faixa | Cor | Mensagem-tipo |
|---|---|---|
| 90–100% | 🟢 Verde | "Você está no caminho da meta." |
| 75–89% | 🟡 Amarelo | "Bom, mas dá para melhorar." |
| 50–74% | 🟠 Laranja | "Seu progresso ficará mais lento." |
| 0–49% | 🔴 Vermelho | "Com essa consistência você provavelmente não atingirá a meta." |

Mensagem factual, nunca de repreensão (regra 4.4 da Constituição) — comunica consequência, não julgamento.

---

## 4. Chance de Atingir a Meta

Calculada a partir do comportamento, não apenas do peso.

```
chance_atingir_meta = f(consistencia_media_21_dias, dias_restantes_ate_prazo, taxa_perda_atual_vs_esperada)
```

Regra de exibição: sempre acompanhada da consistência que a sustenta (ex: *"87% — baseada na consistência dos últimos 21 dias"*), nunca um número isolado sem contexto (regra 4.2 da Constituição — explicabilidade).

---

## 5. Como as três memórias alimentam o Coach

A cada geração de treino ou refeição do dia, o Coach recebe as três camadas combinadas — nunca uma isolada:

```
contexto_completo = {
  permanente: memoria_permanente,
  fase: memoria_da_fase,
  dia: memoria_do_dia (do dia atual, ou vazia se ainda sem check-in)
}
```

Ordem de precedência quando há conflito aparente: **Memória do Dia > Memória da Fase > Memória Permanente** para decisões táticas (ex: dor relatada hoje pesa mais que a carga registrada na fase). Para decisões estratégicas (objetivo, restrições), a Memória Permanente sempre prevalece — a Memória do Dia nunca a sobrescreve.

---

## 6. O que este motor nunca faz

* Não deixa a IA sobrescrever a Memória Permanente por inferência (regra 5.1).
* Não usa dado da Memória do Dia depois de expirado (regra 3.3, 5.3).
* Não revisa a Memória da Fase fora da cadência semanal, exceto ação explícita do usuário.
* Não expõe o plano de 120 dias inteiro em nenhuma tela — só a fase/semana atual quando solicitado (regra 5.3, Princípio nº 1 do UX).
* Não calcula "Chance de atingir a meta" sem vincular à consistência que a gerou.
