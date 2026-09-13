# PRD v0.2 — S7 Coach OS

> Documento Fundador do Produto
>
> Versão: 0.2.0 (Foundation — atualizado)
>
> Status: Em definição
>
> Owner: Mario Schiavon (S7)
>
> Produto: S7 Coach OS (nome provisório)

---

## Changelog desta versão (v0.1 → v0.2)

* **Motor de IA:** substituído de GPT (OpenAI) para **Claude (API Anthropic)** em todas as camadas — Coach de Treino, Coach de Alimentação, Diário Inteligente e chat assistido.
* **Plataforma:** mantido **PWA** (não migra para app nativo no MVP), porém o design deixa de ser "responsivo" e passa a ser **mobile-first estrito** — o produto é desenhado assumindo uso majoritário em smartphone, sem otimização dedicada para desktop.
* **Execução técnica:** a implementação passa a ser conduzida via **Claude Code**, não mais Codex. Toda a documentação do repositório deve ser escrita para servir de contexto de engenharia ao Claude Code.

---

## 1. Visão do Produto

### Problema

A maioria dos aplicativos de treino e alimentação falha porque exige que o usuário tome decisões o tempo inteiro.

O usuário precisa decidir:

* Qual treino fazer.
* Qual aparelho usar.
* Qual peso colocar.
* O que comer.
* Quanto comer.
* Quando beber água.
* Como reorganizar a semana quando perde um treino.

Isso gera abandono.

### Nossa proposta

O **S7 Coach OS** é um assistente pessoal que pensa pela pessoa.

O usuário apenas executa as **Missões do Dia**.

A IA (Claude, via API Anthropic) utiliza contexto contínuo (treinos, alimentação, sono, rotina e trabalho) para adaptar o plano sem perder o objetivo principal.

### Objetivo principal

Levar qualquer usuário do ponto A ao ponto B através de consistência.

Exemplo do MVP:

* Mario.
* 124 kg.
* Objetivo: 100 kg.
* Hipertrofia + perda de gordura.
* Prazo estimado: 120 dias.

### Filosofia do Produto

**Princípio nº 1**
> O usuário nunca deve precisar decidir o próximo passo.

**Princípio nº 2**
A IA adapta o plano. Ela **não cria um plano novo todos os dias**.

**Princípio nº 3**
Consistência vale mais do que perfeição.

**Princípio nº 4 (novo nesta versão)**
> O produto é mobile-first, não mobile-friendly.

Todo componente, fluxo e decisão de UX é desenhado assumindo tela de smartphone como único contexto de uso real. Desktop não é um caso de uso relevante para o MVP — não haverá layout dedicado para telas largas.

---

## 2. Escopo do MVP (v1.0)

O MVP resolve apenas quatro pilares.

### Pilar 1 — Missões do Dia
Tela principal do aplicativo. Responsável por mostrar apenas o que importa hoje.

Missões possíveis: Treino, Alimentação, Água, Creatina, Sono, Peso (quando necessário).

### Pilar 2 — Coach de Treino
Treinos adaptativos. A IA gera versões do treino conforme: Tempo disponível, Energia, Sono, Trabalho físico, Recuperação. Nunca muda o objetivo da semana.

### Pilar 3 — Coach de Alimentação
Plano alimentar adaptativo. O usuário pode seguir exatamente ou registrar outra refeição. A IA reorganiza o restante do dia.

### Pilar 4 — Diário Inteligente
Timeline automática. Tudo que acontece durante o dia gera contexto.

### Fora do Escopo do MVP
* Contagem automática por smartwatch.
* Integração Apple Health / Google Fit.
* Vídeos próprios de exercícios.
* Plano para corrida.
* Receitas.
* Comunidade/social.
* Layout ou otimização para desktop/web.

---

## 3. Persona Inicial

### Persona Primária
Homens e mulheres entre 25 e 50 anos. Trabalham, têm pouco tempo, querem emagrecer, querem ganhar massa, desistem facilmente da academia.

Mario é nossa primeira persona.

### Dados da Persona Mario
* Desenvolvedor.
* Trabalho físico pela manhã (aeroporto).
* Academia às 13:30.
* Dorme cedo.
* Não gosta de peixe.
* Quer hipertrofia.
* Tendência a abandonar academia.
* Usa o app quase exclusivamente pelo celular, durante o dia de trabalho e na academia.

---

## 4. Jornada do Usuário

### Primeiro acesso (90 segundos)
1. Nome
2. Idade
3. Sexo
4. Altura
5. Peso
6. Objetivo
7. Dias disponíveis para treino
8. Academia ou casa

Fim. O usuário entra imediatamente.

### Descoberta Progressiva
Perguntas aparecem apenas quando úteis (dormiu bem? treino foi difícil? gostou dessa refeição? trabalhou pesado hoje?). Sem questionário gigante.

---

## 5. Arquitetura de Contexto

O Coach possui três memórias:

**Memória Permanente** — Altura, objetivo, restrições alimentares, academia, horários.

**Memória da Fase** — Semana, peso, medidas, carga dos exercícios, calorias atuais.

**Memória do Dia** — Sono, dor, energia, tempo disponível, trabalho pesado. Expira diariamente.

---

## 6. Sistema de Missões

Estrutura de cada missão: objetivo, progresso, CTA, recompensa.

* **Treino** — pendente / em andamento / concluído / reagendado.
* **Alimentação** — sugerida / registrada / adaptada.
* **Água** — incrementos rápidos (+300 ml).
* **Sono** — check-in diário.
* **Peso** — somente em dias programados.

---

## 7. Coach de Treino

**Plano Mestre** — o Coach cria um plano semanal fixo (ex: Segunda Treino A, Quarta Treino B, Sexta Treino C).

**IA Tática** — adapta apenas o treino do dia, com base em tempo, dor, sono, trabalho e energia. Saída: versão do treino (30/40/60 min, casa/academia).

**Biblioteca Fechada de Exercícios** — cada músculo possui exercícios classificados como Principal, Auxiliar, Isolador ou Substituto. A IA apenas escolhe dentro dessa biblioteca — nunca inventa exercício novo.

**Tela do Treino** — sequência pronta de aparelhos, na ordem correta, cada um com peso sugerido, séries, repetições e tempo de descanso.

**Feedback Pós-Exercício** — cinco estados por toque único: 😀 Muito fácil / 🙂 Fácil / 😐 Ideal / 😮 Difícil / 🥵 Não consegui terminar. Somente respostas difíceis abrem perguntas extras.

**Modo Coach Ativo vs. Modo Rápido** — o usuário escolhe se quer dar feedback após cada aparelho (mais dado para a IA calibrar carga) ou só ao final do treino (menos atrito).

**Progressão de Carga** — determinística, não decidida livremente pela IA:

| Resultado | Próximo treino |
|---|---|
| Muito fácil 2 sessões seguidas | Aumenta carga |
| Ideal | Mantém |
| Difícil | Mantém ou reduz repetições |
| Não conseguiu terminar | Reduz peso |

A IA sempre explica a decisão ("Próximo treino: 37,5 kg. Motivo: você completou todas as séries com facilidade duas vezes seguidas.").

---

## 8. Coach de Alimentação

Cada refeição tem um **plano sugerido**. O usuário escolhe "Foi exatamente isso" ou "Comi outra coisa" (busca simples de alimento + quantidade, sem exigir gramas).

**Compensação Inteligente** — a IA ajusta o restante do dia (pouca proteína → mais proteína no almoço; muito carboidrato → reduz carboidrato no jantar).

**Sem Culpa** — o usuário nunca recebe bronca, recebe consequência explicada.

---

## 9. Diário Inteligente

Timeline do dia com eventos: refeições, treino, peso, água, sono, humor, trabalho, observações. Tudo gera contexto para as decisões seguintes da IA.

---

## 10. Consistência

Métrica principal do produto — mede aderência, não acesso ao app.

**Componentes:** Treino, Alimentação, Proteína, Água, Sono, Peso. Cada item tem peso próprio no cálculo. Resultado final: 0–100%.

**Semáforo:** 🟢 Verde / 🟡 Amarelo / 🟠 Laranja / 🔴 Vermelho — cada faixa com mensagem diferente, para que o usuário entenda a causa antes de "sentir rejeição" pelo app (ex: "uso há um mês e só engordei" quando a aderência real foi baixa).

**Chance de Atingir a Meta** — calculada pela consistência dos últimos dias, não apenas pelo peso.

---

## 11. Sistema de Notificações

**Hook7 (WhatsApp)** — lembretes inteligentes (hora do treino, hora de dormir, peso hoje).

**Push Notification** — mesmo conteúdo, usuário escolhe o canal.

---

## 12. Arquitetura Técnica

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js (PWA) |
| UI | Tailwind + shadcn/ui |
| Design | **Mobile-first estrito** — sem breakpoint dedicado para desktop no MVP |
| Backend | Supabase |
| Banco | PostgreSQL |
| Storage | Supabase Storage |
| **IA** | **Claude — API Anthropic** (substitui GPT-5.6) |
| Mensageria | Hook7 (WhatsApp) |
| Execução/engenharia | Claude Code |

### Nota sobre a troca de IA

Todos os pontos do produto que antes assumiam "GPT" (Coach de Treino, Coach de Alimentação, geração de missões, explicações de decisão, chat assistido) passam a ser implementados com a API da Anthropic. Isso deve ser refletido em qualquer documento futuro de prompts/regras da IA (`03-ai-constitution.md`, `06-ai-prompts.md` etc.): exemplos de chamada, formatos de resposta e nomenclatura de modelo precisam citar Claude, não GPT.

### Nota sobre mobile-first

"Mobile-first" aqui não significa apenas responsivo — significa que todo o desenho de tela (`07-ux-system.md`), o Design System e os componentes do MVP serão especificados apenas para viewport de smartphone. Não haverá versão desktop no roadmap do MVP.

---

## 13. Estrutura do Repositório

```
s7-coach-os/

docs/
  00-visao.md
  01-prd.md
  02-roadmap.md
  03-ai-constitution.md
  04-context-engine.md
  05-workout-engine.md
  06-nutrition-engine.md
  07-ux-system.md
  08-database.md
  09-hook7-integration.md
  10-metrics-achievements.md
  patch-logs/

apps/
  coach-web/

packages/
  coach-engine/
  nutrition-engine/
  workout-engine/
  ui/

supabase/
```

---

## 14. Roadmap Oficial

**v0.1 Foundation** — PRD, arquitetura, regras da IA.

**v0.2 UX System** — Wireframes mobile-first, fluxos, componentes.

**v0.3 Coach Engine** — Treino, progressão, consistência (motor com Claude).

**v0.4 Nutrition Engine** — Missões, compensação, timeline (motor com Claude).

**v0.5 Hook7 Integration** — WhatsApp, check-ins, lembretes.

**v1.0 MVP** — Aplicativo funcional para uso diário, mobile-first, PWA, movido por Claude.

---

## 15. Identidade Visual (fechada)

**Tema:** Dark mode (obrigatório no MVP).

| Uso | Cor |
|---|---|
| Primária (missões concluídas, progresso, CTA) | Emerald `#22C55E` |
| Hover | `#16A34A` |
| Escura | `#15803D` |
| Background | `#0F172A` / `#1E293B` |
| Apoio (água, recuperação) | Cyan `#06B6D4` |
| Alertas / adaptação | Amber `#F59E0B` |
| Erro / dor / risco | Red `#EF4444` |

Sem roxo — evita associação visual com "assistente de IA genérico".
