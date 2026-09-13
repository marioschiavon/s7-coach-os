# 07 — UX System — S7 Coach OS

> Documento de especificação de experiência e interface.
> Versão: 0.1.0
> Escopo: **mobile-first estrito** — nenhuma tela deste documento assume viewport de desktop.
> Referência de arquitetura: `01-prd.md` v0.2

---

## 0. Princípios de UX (Os 10 Mandamentos)

Todas as decisões de tela abaixo devem obedecer a estes princípios. Qualquer proposta de tela nova precisa ser validada contra esta lista antes de entrar no repositório.

| # | Princípio | Aplicação prática |
|---|---|---|
| 1 | Uma missão por vez | Nunca listar mais de 5 itens acionáveis na Home |
| 2 | Zero culpa | Mensagens de IA sempre explicam consequência, nunca repreendem |
| 3 | Zero digitação sempre que possível | Prioridade: toque > seleção > busca > texto livre |
| 4 | IA invisível | Sem avatar de robô, sem "chat bubble" como interface principal |
| 5 | Menos dashboard, mais ação | CTA acima de gráfico em qualquer tela |
| 6 | Tudo explica o motivo | Toda decisão automática (carga, dieta) tem uma linha de justificativa |
| 7 | O plano é adaptativo, nunca do zero | Toda tela de treino/dieta referencia o Plano Mestre |
| 8 | Contexto acima de dados | Rotina (aeroporto, sono, horário) pesa mais que números isolados |
| 9 | Consistência é a métrica principal | Peso nunca é o elemento mais destacado da Home |
| 10 | O usuário sempre sabe o próximo passo | Toda tela termina com uma ação clara, nunca um beco sem saída |

### Regras mobile-first (não-negociáveis)

* Área de toque mínima: **48×48px**.
* Zona de polegar: ações primárias sempre no **terço inferior da tela**.
* Navegação principal: **bottom tab bar fixa**, nunca menu hambúrguer para as seções centrais.
* Uma coluna. Sem grids multi-coluna, sem sidebar.
* Fonte mínima de corpo: 15px. Títulos de missão: 18–20px.
* Cantos arredondados 24px (cards) / 28px (botões primários).
* Nenhuma tela deve exigir scroll horizontal.
* Todo formulário é substituído por seleção quando possível (ver Design System, seção 9).

---

## 1. Mapa de Navegação

```
┌─────────────────────────────────────────┐
│              SPLASH / LOGIN              │
└───────────────────┬───────────────────────┘
                     │ primeiro acesso
                     ▼
┌─────────────────────────────────────────┐
│         ONBOARDING (90 segundos)         │
└───────────────────┬───────────────────────┘
                     ▼
┌─────────────────────────────────────────┐
│                                           │
│     BOTTOM TAB BAR (navegação fixa)      │
│                                           │
│  [Home]  [Treino]  [Diário]  [Evolução]  │
│                     [•IA•]               │
└─────────────────────────────────────────┘
```

**Tabs (5 posições fixas):**

| Ícone | Tab | Conteúdo |
|---|---|---|
| 🏠 | Home | Missões do Dia |
| 🏋️ | Treino | Treino de hoje + histórico |
| 📓 | Diário | Timeline do dia |
| 📈 | Evolução | Peso, medidas, fotos, Coach Score |
| 💬 | Coach | Chat assistido (secundário, não é a interface principal) |

Configurações não fica na tab bar — vive dentro de um ícone de perfil no canto superior direito da Home, acessível em qualquer tela.

---

## 2. Splash / Login

Tela mínima. Objetivo: sair dela o mais rápido possível.

```
┌───────────────────────┐
│                         │
│                         │
│        S7 COACH        │
│           OS           │
│                         │
│    [Entrar com e-mail]  │
│    [Continuar com       │
│        Google]          │
│                         │
└───────────────────────┘
```

Decisão: sem tela de "features do app" antes do login (comum em apps fitness). O usuário já veio decidido — atrito aqui só gera abandono.

---

## 3. Onboarding Inteligente (90 segundos)

Fluxo de **8 telas**, uma pergunta por tela, botões grandes, sem campos de texto livre exceto nome e peso/altura (numéricos, via scroll picker, não teclado).

```
Tela 1        Tela 2        Tela 3        Tela 4
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│ Nome?  │ → │ Idade? │ → │ Sexo?  │ → │ Altura?│
│[input] │   │[picker]│   │ M / F  │   │[picker]│
└────────┘   └────────┘   └────────┘   └────────┘

Tela 5        Tela 6              Tela 7           Tela 8
┌────────┐   ┌──────────────┐   ┌────────────┐   ┌──────────┐
│ Peso?  │ → │  Objetivo?    │ → │ Dias de    │ → │ Academia │
│[picker]│   │ Perder peso   │   │ treino/sem │   │ ou casa? │
│        │   │ Hipertrofia   │   │ [1-2-3-4-5]│   │          │
│        │   │ Ambos         │   └────────────┘   └──────────┘
└────────┘   └──────────────┘
```

Ao final da Tela 8: usuário cai direto na Home, já com a primeira missão do dia gerada. Nenhuma tela de "carregando seu plano" com spinner longo — geração acontece em background enquanto ele ainda está na Tela 8.

**Descoberta progressiva (pós-onboarding):** perguntas contextuais aparecem como cards dispensáveis dentro da Home, no máximo 1 por dia. Nunca em modal bloqueante.

---

## 4. Home — Missões do Dia

A tela mais importante do produto. Decide retenção.

```
┌───────────────────────────┐
│ Terça • 12 set        [⚙] │
│                             │
│ Bom dia, Mario 👋           │
│ 🟢 87% consistente          │
│                             │
│ ┌─────────────────────────┐│
│ │ ☐ Treino B — 40 min      ││
│ │   [Iniciar treino]       ││
│ └─────────────────────────┘│
│ ┌─────────────────────────┐│
│ │ ☐ Café da manhã          ││
│ │   78g / 180g proteína    ││
│ └─────────────────────────┘│
│ ┌─────────────────────────┐│
│ │ ☐ Água — 1,2L / 3L       ││
│ │   [+300ml]               ││
│ └─────────────────────────┘│
│ ┌─────────────────────────┐│
│ │ ☐ Creatina               ││
│ │   [Tomei]                ││
│ └─────────────────────────┘│
│                             │
│  Meta: 124kg → 100kg        │
│  ▓▓▓▓░░░░░░  4,2kg          │
│                             │
│ [Home] [Treino] [Diário]    │
│      [Evolução] [Coach]     │
└───────────────────────────┘
```

**Decisões de UX desta tela:**

| Decisão | Motivo |
|---|---|
| Coach/IA aparece só na saudação, uma linha | Não vira chatbot visual |
| Missões em checklist vertical, máx. 5 | Escaneável em 2 segundos |
| Barra de consistência no topo, sempre visível | Reforça o comportamento certo |
| Meta de peso no rodapé, discreta | Motiva sem gerar ansiedade diária |
| Toque em qualquer missão abre a tela dedicada dela | Nunca resolve a missão inline na Home |

---

## 5. Treino do Dia

Ao tocar em "Iniciar treino" na Home, abre a lista de aparelhos na sequência gerada pela IA para o contexto de hoje.

```
┌───────────────────────────┐
│ ← Treino B • Hoje           │
│ Costas + Peito + Ombro      │
│ 40 min • Gerado às 11:15    │
│                             │
│ ①  Puxada Alta               │
│    Máquina 17 • 3×10-12      │
│    35 kg sugerido             │
│                             │
│ ②  Supino Máquina             │
│    Máquina 08 • 3×10-12      │
│    28 kg sugerido             │
│                             │
│ ③  Remada Baixa               │
│    Máquina 11 • 3×12          │
│    30 kg sugerido             │
│                             │
│ ④  Elevação Lateral           │
│    Halteres • 3×15            │
│                             │
│ ⑤  Prancha + Abdominal        │
│    Core • 3×30-45s            │
│                             │
│      [Começar Treino]        │
└───────────────────────────┘
```

Cada linha já mostra aparelho + séries + repetições + carga — não existe "ficha" separada para consultar. Tocar em qualquer item abre a Tela de Exercício (seção 6).

### Modo Academia

Ativado automaticamente quando o usuário toca em "Começar Treino". Remove toda navegação exceto o necessário para o exercício atual.

```
┌───────────────────────────┐
│      Exercício 2 de 5       │
│                             │
│      Supino Máquina          │
│                             │
│         28 kg                │
│      3 séries × 10            │
│                             │
│    Descanso: 90s              │
│    [Concluir Série]           │
│                             │
└───────────────────────────┘
```

Nada além de: nome do aparelho, carga, séries/reps, cronômetro de descanso, botão de concluir. Sem menu, sem tab bar visível.

---

## 6. Tela de Exercício (Feedback Pós-Exercício)

Ao concluir as séries de um aparelho, aparece o card de feedback — **1 toque, sem digitação**.

```
┌───────────────────────────┐
│  Exercício concluído         │
│  Puxada Alta                 │
│                             │
│  Como foi esse exercício?    │
│                             │
│  😀      🙂      😐      😮      🥵 │
│ Muito   Fácil  Ideal  Difícil Não  │
│ fácil                    consegui  │
│                                     │
└───────────────────────────┘
```

Se o usuário tocar em 😮 ou 🥵, abre um segundo nível **opcional** (não obrigatório):

```
┌───────────────────────────┐
│  O que aconteceu?            │
│                             │
│  [ Sem força ]               │
│  [ Má postura ]              │
│  [ Dor muscular ]            │
│  [ Dor na articulação ]      │
│  [ Fadiga do trabalho ]      │
│                             │
│         [Pular]              │
└───────────────────────────┘
```

**Configuração de modo (em Configurações do Coach, seção 9):**

| Modo | Comportamento |
|---|---|
| Coach Ativo | Pergunta a cada aparelho |
| Modo Rápido | Pergunta uma vez, só ao final do treino inteiro |

Isso resolve o trade-off entre "dado suficiente pra IA calibrar carga" e "cansar o usuário" — quem prefere atenção constante liga o modo ativo; quem só quer terminar logo usa o rápido, e a IA usa proxies (tempo total, se todas as séries foram concluídas) para estimar progressão nesse modo.

Ao final do treino (qualquer modo), aparece o resultado da progressão:

```
┌───────────────────────────┐
│  Supino Máquina               │
│  Hoje: 12 / 12 / 12            │
│  Feedback: 😀 Muito fácil       │
│                             │
│  Coach decidiu:                │
│  Próximo treino: 30 kg         │
│  Motivo: você completou         │
│  todas as séries com            │
│  facilidade 2x seguidas.        │
└───────────────────────────┘
```

---

## 7. Alimentação

### 7.1 Missão da Refeição

```
┌───────────────────────────┐
│ Café da manhã                 │
│ 38g proteína                  │
│                             │
│ Plano sugerido:                │
│ 🍌 Banana                      │
│ 🥣 Aveia                       │
│ ☕ Café                        │
│ 🥚 3 ovos                      │
│                             │
│ [Foi exatamente isso]          │
│ [Comi outra coisa]             │
└───────────────────────────┘
```

### 7.2 Registro Flexível ("Comi outra coisa")

```
┌───────────────────────────┐
│ O que você comeu?             │
│                             │
│ [ 🔍 Buscar alimento... ]      │
│                             │
│ Cream Cracker                  │
│ Quantidade: (1) (3) (5) (10)   │
│                             │
│ + Adicionar item               │
│                             │
│      [Salvar refeição]         │
└───────────────────────────┘
```

Sem exigir gramas. Quantidade por unidades comuns ("1 fatia", "1 xícara", "3 unidades").

### 7.3 Trocas Inteligentes (RFC-0001)

Alternativa ao registro livre — o usuário toca direto no item do plano sugerido e escolhe "Trocar":

```
┌───────────────────────────┐
│ Banana → trocar por:          │
│                             │
│ [ Maçã ]                      │
│ [ Cream Cracker (3un) ]        │
│ [ Outro... ]                   │
└───────────────────────────┘

↓ ao confirmar

┌───────────────────────────┐
│ Refeição registrada            │
│ 3 Cream Crackers + Café         │
│                             │
│ Coach:                         │
│ Essa troca ficou com pouca      │
│ proteína. Vou adicionar +30g    │
│ de sugestão no almoço.          │
│                             │
│ [Adicionar sugestão no almoço]  │
└───────────────────────────┘
```

Sem culpa — mensagem sempre no formato consequência + solução, nunca repreensão.

---

## 8. Diário / Timeline

Sem botão "Registrar" genérico. A tela é só uma linha do tempo somativa, alimentada pelas outras telas.

```
┌───────────────────────────┐
│ Terça-feira                   │
│                             │
│ 03:45 — Café da manhã          │
│         registrado             │
│                             │
│ 07:50 — Lanche no aeroporto     │
│                             │
│ 11:20 — Almoço registrado       │
│                             │
│ 13:30 — Treino concluído        │
│         (5/5 exercícios)        │
│                             │
│ 20:30 — Hora sugerida            │
│         para dormir             │
└───────────────────────────┘
```

Cada evento é somente leitura — tocar nele leva de volta à missão de origem (ex: tocar em "Almoço" abre o detalhe da refeição registrada).

---

## 9. Evolução

```
┌───────────────────────────┐
│ Evolução                      │
│                             │
│ Coach Score hoje                │
│         84                     │
│  Treino 20 · Proteína 18        │
│  Água 16 · Sono 14 · Diário 16  │
│                             │
│ Peso                           │
│  📉 124kg → 119,8kg (4,2kg)     │
│  [gráfico de linha simples]     │
│                             │
│ Semana 5 — Consistência 61%     │
│  Esperado: -3,1kg               │
│  Real: -0,8kg                   │
│                             │
│ Coach:                         │
│ Você treinou 2 de 5 dias e       │
│ bateu a proteína em apenas       │
│ 3 dias. Seu resultado está       │
│ compatível com sua consistência. │
│                             │
│ Chance de atingir 100kg           │
│ até dezembro: 87%                 │
│ (baseada nos últimos 21 dias)     │
│                             │
│ [+ Registrar peso]                │
│ [+ Adicionar foto]                │
└───────────────────────────┘
```

Decisão-chave: sempre que a consistência estiver abaixo do esperado, a tela **antecipa a explicação** ("seu resultado está compatível com sua consistência") antes que o usuário sinta frustração e culpe o app.

---

## 10. Coach (Chat Assistido) — interface secundária

Não é a tela principal do produto (Princípio nº 4 — IA invisível). Existe para casos que não cabem nas missões estruturadas:

```
┌───────────────────────────┐
│ Coach                          │
│                             │
│ "Estou sem tempo hoje"          │
│ "Estou com dor na lombar"        │
│ "Pular treino de hoje"           │
│                             │
│ [ Digite sua mensagem... ]       │
└───────────────────────────┘
```

Sugestões de atalho ficam acima do campo de texto para reduzir digitação. O chat nunca substitui as telas estruturadas — se o usuário disser "dor na lombar", a resposta do Coach deve reduzir automaticamente o treino do dia e navegar de volta para a Tela de Treino atualizada, não resolver tudo dentro do próprio chat.

---

## 11. Configurações

```
┌───────────────────────────┐
│ ← Configurações                │
│                             │
│ Perfil                         │
│  Nome, idade, altura, peso      │
│                             │
│ Coach                          │
│  ( ) Coach Ativo                │
│  (•) Modo Rápido                │
│                             │
│ Notificações                    │
│  Hook7 (WhatsApp)  [Ativo]      │
│  Push               [Ativo]     │
│                             │
│ Restrições alimentares          │
│  Não gosto de: peixe            │
│                             │
│ Sair                           │
└───────────────────────────┘
```

---

## 12. Design System

### 12.1 Cores (fechado — ver `01-prd.md` §15)

| Token | Hex | Uso |
|---|---|---|
| `primary` | `#22C55E` | CTA principal, missão concluída |
| `primary-hover` | `#16A34A` | Estado pressionado |
| `primary-dark` | `#15803D` | Texto sobre fundo claro |
| `bg-base` | `#0F172A` | Fundo principal |
| `bg-elevated` | `#1E293B` | Cards |
| `accent` | `#06B6D4` | Água, recuperação |
| `warning` | `#F59E0B` | Alertas, adaptação |
| `error` | `#EF4444` | Dor, risco, falha |

### 12.2 Tipografia

| Elemento | Tamanho | Peso |
|---|---|---|
| Título de tela | 22px | Bold |
| Título de card/missão | 18px | Semibold |
| Corpo | 15px | Regular |
| Legenda/metadado | 13px | Regular, opacidade 70% |

### 12.3 Componentes base

* **Card de missão** — 24px de raio, padding 16px, checkbox à esquerda, CTA à direita.
* **Botão primário** — altura mínima 52px, raio 28px, fundo `primary`.
* **Botão secundário** — mesmo tamanho, borda 1px, fundo transparente.
* **Botão de emoji (feedback)** — círculo 56px, um toque, sem estado "selecionado" persistente (ação é imediata).
* **Barra de progresso/consistência** — altura 8px, raio total, cor dinâmica pelo semáforo.
* **Bottom tab bar** — altura 64px + safe area, 5 ícones, item ativo em `primary`.

### 12.4 Estados de missão (cores)

| Status | Cor |
|---|---|
| Pendente | Cinza neutro |
| Em andamento | `accent` |
| Concluído | `primary` |
| Reagendado / adaptado | `warning` |
| Falhou / não concluído | `error` (uso raro, nunca como "erro do usuário") |

---

## 13. Telas fora do MVP (mencionar, não desenhar agora)

* Comunidade / social
* Integração com smartwatch
* Modo desktop/web
* Vídeo dos exercícios (fase 2)

Essas ficam registradas aqui apenas para não serem esquecidas quando o roadmap avançar além do v1.0.
