# 05 — Workout Engine — S7 Coach OS

> Motor de treino: como o Plano Mestre vira o treino do dia, e como a carga evolui.
> Versão: 0.1.0
> Sujeito às regras de `03-ai-constitution.md` (seção 1).

---

## 1. Duas camadas

O Workout Engine opera em duas camadas que nunca se misturam:

| Camada | Quem decide | Frequência | Pode mudar? |
|---|---|---|---|
| **Plano Mestre** | Gerado no onboarding + revisado semanalmente | 1x/semana | Só por revisão semanal ou ação explícita do usuário |
| **IA Tática** | Claude, a cada sessão | 1x/treino | Sempre, dentro dos limites do Plano Mestre |

O Plano Mestre define **o quê** (quais grupos musculares, quantas vezes por semana, frequência mínima por grupo). A IA Tática define **como hoje** (qual versão do treino, quais exercícios da biblioteca, qual volume, considerando tempo/dor/sono/trabalho/energia).

A IA Tática nunca pode contradizer o Plano Mestre — ela só reduz, adapta ou substitui dentro do que já foi definido.

---

## 2. Plano Mestre

Gerado a partir de:

* Objetivo (perder gordura / hipertrofia / ambos)
* Dias de treino disponíveis por semana
* Local (academia / casa / ambos)
* Restrições relatadas no onboarding (dor prévia, limitação física)

Exemplo de estrutura (3 dias/semana):

```
Segunda → Treino A (Peito + Tríceps + Core)
Quarta  → Treino B (Costas + Bíceps + Ombro)
Sexta   → Treino C (Pernas + Core)
```

### Regras fixas do Plano Mestre (não mudam por sessão)

* Cada grupo muscular grande (peito, costas, pernas) aparece pelo menos 1x/semana.
* Nenhum grupo fica mais de 7 dias sem estímulo.
* Descanso mínimo de 48h entre sessões que trabalham o mesmo grupo.
* Revisão do Plano Mestre acontece toda semana, considerando: consistência da semana anterior, progressão registrada, peso atual.

### Reagendamento automático

Se o usuário perde um dia de treino, o Coach reagenda o treino perdido para o próximo slot livre da semana, sem duplicar volume no mesmo dia e sem quebrar o descanso mínimo de 48h. O Plano Mestre da semana seguinte não é afetado.

---

## 3. Biblioteca Fechada de Exercícios

A IA Tática só escolhe dentro desta biblioteca — nunca gera exercício novo (regra 1.1 da Constituição).

### Estrutura de um exercício

```
{
  id: string
  nome: string
  grupo_muscular: string        // ex: "costas"
  categoria: "principal" | "auxiliar" | "isolador" | "substituto"
  equipamento: string           // ex: "máquina", "halteres", "peso corporal"
  local: "academia" | "casa" | "ambos"
  substitutos: [id, id, ...]    // exercícios equivalentes de menor/maior demanda de equipamento
  articulacoes_envolvidas: [string]  // para regra 1.8 (dor articular)
}
```

### Regra de seleção por treino

Cada sessão de um grupo muscular deve conter, no mínimo:

* 1 exercício **principal**
* 1 exercício **auxiliar** ou **isolador**

E pode incluir um **substituto** quando o local, tempo ou dor exigirem.

### Exemplo — grupo "Costas"

| Exercício | Categoria | Local |
|---|---|---|
| Puxada Alta | Principal | Academia |
| Remada Baixa | Auxiliar | Academia |
| Remada com mochila | Substituto | Casa |
| Face Pull | Isolador | Academia |

---

## 4. IA Tática — geração do treino do dia

### Entradas permitidas (única fonte de contexto)

| Variável | Origem |
|---|---|
| Tempo disponível hoje | Usuário informa ou padrão do Plano Mestre |
| Local | Academia / casa / aeroporto (sem treino formal) |
| Dor muscular (0–5) | Check-in do dia |
| Energia (1–5) | Check-in do dia |
| Horas de sono | Check-in do dia |
| Trabalho físico pesado hoje | Check-in do dia (Sim/Não) |
| Equipamento disponível | Perfil da academia cadastrada |

Nenhuma outra variável entra na decisão do treino do dia.

### Saída: versão do treino

A IA Tática gera uma entre versões padronizadas de duração, nunca uma duração arbitrária:

| Versão | Duração | Nº de exercícios | Séries totais aprox. |
|---|---|---|---|
| Expressa | 30 min | 4–5 | 12–14 |
| Padrão | 40–45 min | 5–6 | 16–18 |
| Completa | 60 min | 7–8 | 20–22 |
| Estendida | 70–90 min | 8–9 + cardio | 22 (limite da regra 1.7) |

### Lógica de adaptação (exemplos determinísticos)

| Condição do dia | Ajuste aplicado |
|---|---|
| Tempo < padrão do Plano Mestre | Reduz para versão Expressa: mantém 1 principal por grupo, remove isoladores |
| Trabalho físico pesado hoje = Sim | Reduz volume de pernas/lombar; prioriza máquina guiada em vez de peso livre |
| Dor muscular ≥ 4 no grupo do dia | Substitui exercício principal por substituto de menor impacto; reduz 1 série por exercício |
| Dor articular relatada | Aciona regra 1.8 da Constituição — substitui/remove o exercício, nunca apenas avisa |
| Sono < 5h | Remove cardio final, mantém força |
| Energia = 1 ou 2 | Oferece a versão Expressa como sugestão, mas não impõe |

O Coach sempre explica a adaptação em uma frase (regra 4.2 da Constituição), ex: *"Você trabalhou pesado hoje de manhã. Reduzi o volume de pernas e troquei agachamento livre por leg press."*

---

## 5. Progressão de Carga (determinística — regra 1.3 da Constituição)

A IA nunca decide a carga "por julgamento". A tabela abaixo é a única fonte de verdade.

### Pré-requisito

Só se aplica progressão automática após **no mínimo 2 sessões registradas** do mesmo exercício (regra 1.2).

### Tabela de decisão

| Resultado da última sessão | Feedback do usuário | Ação na próxima sessão |
|---|---|---|
| Completou todas as séries no topo da faixa de reps (ex: 12/12/12) | 😀 Muito fácil (2x seguidas) | +2,5 kg (ou +5% se peso corporal) |
| Completou todas as séries no topo da faixa | 🙂 Fácil | Mantém peso, aumenta 1 rep por série no alvo |
| Completou dentro da faixa esperada | 😐 Ideal | Mantém peso e reps |
| Completou abaixo do topo da faixa | 😮 Difícil | Mantém peso, mantém reps |
| Não completou todas as séries | 🥵 Não consegui terminar | -10% de peso ou reduz 1 série |

### Regras adicionais

* Nunca aumenta carga em duas sessões consecutivas sem um "Ideal" ou "Fácil" intermediário confirmando estabilidade.
* Redução por "Não consegui terminar" nunca é revertida na sessão seguinte automaticamente — precisa de 1 sessão "Ideal" para retomar progressão.
* Toda mudança de carga é acompanhada do motivo (regra 4.2), no formato: *"Próximo treino: 30 kg. Motivo: você completou todas as séries com facilidade duas vezes seguidas."*

---

## 6. Modo Coach Ativo vs. Modo Rápido

| Modo | Frequência de feedback | Dado disponível pra progressão |
|---|---|---|
| Coach Ativo | A cada exercício | Direto (emoji por exercício) |
| Modo Rápido | 1x ao final do treino | Proxy: todas as séries concluídas no tempo esperado = equivalente a "Ideal"; treino interrompido antes do fim = equivalente a "Difícil" no(s) exercício(s) não concluído(s) |

No Modo Rápido, a tabela da seção 5 ainda se aplica — apenas o dado de entrada muda de "emoji direto" para "proxy calculado". Nunca se inventa um resultado não observável.

---

## 7. Nível de Domínio do Exercício (opcional, pós-MVP)

Registrado aqui apenas como backlog — **não faz parte do MVP v1.0**. Se implementado futuramente, deve seguir a mesma lógica determinística da seção 5 (baseado em nº de execuções + estabilidade de progressão), nunca em avaliação subjetiva da IA.

---

## 8. O que este motor nunca faz

* Não gera treino fora da Biblioteca Fechada (regra 1.1).
* Não decide carga por "achismo" do modelo — sempre a tabela da seção 5.
* Não ultrapassa 22 séries por treino sem confirmação explícita (regra 1.7).
* Não ignora dor articular ajustando volume "na esperança de melhorar" (regra 1.8) — substitui ou remove.
* Não altera a frequência semanal por grupo muscular definida no Plano Mestre — isso só muda na revisão semanal.
