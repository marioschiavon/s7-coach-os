# PRD v0.1 — S7 Coach OS

> Documento Fundador do Produto
>
> Versão: 0.1.0 (Foundation)
>
> Status: Em definição
>
> Owner: Mario Schiavon (S7)
>
> Produto: S7 Coach OS (nome provisório)

---

## 1. Visão do Produto

### Problema

A maioria dos aplicativos de treino e alimentação falha porque exige que o usuário tome decisões o tempo inteiro.

O usuário precisa decidir:

- Qual treino fazer.
- Qual aparelho usar.
- Qual peso colocar.
- O que comer.
- Quanto comer.
- Quando beber água.
- Como reorganizar a semana quando perde um treino.

Isso gera abandono.

### Nossa proposta

O **S7 Coach OS** é um assistente pessoal que pensa pela pessoa.

O usuário apenas executa as **Missões do Dia**.

A IA utiliza contexto contínuo (treinos, alimentação, sono, rotina e trabalho) para adaptar o plano sem perder o objetivo principal.

---

## Objetivo principal

Levar qualquer usuário do ponto A ao ponto B através de consistência.

Exemplo do MVP:

- Mario.
- 124 kg.
- Objetivo: 100 kg.
- Hipertrofia + perda de gordura.
- Prazo estimado: 120 dias.

---

## Filosofia do Produto

### Princípio nº 1

> O usuário nunca deve precisar decidir o próximo passo.

### Princípio nº 2

A IA adapta o plano.

Ela **não cria um plano novo todos os dias**.

### Princípio nº 3

Consistência vale mais do que perfeição.

---

# 2. Escopo do MVP (v1.0)

O MVP resolve apenas quatro pilares.

## Pilar 1 — Missões do Dia

Tela principal do aplicativo.

Responsável por mostrar apenas o que importa hoje.

Missões possíveis:

- Treino.
- Alimentação.
- Água.
- Creatina.
- Sono.
- Peso (quando necessário).

---

## Pilar 2 — Coach de Treino

Treinos adaptativos.

A IA gera versões do treino conforme:

- Tempo disponível.
- Energia.
- Sono.
- Trabalho físico.
- Recuperação.

Nunca muda o objetivo da semana.

---

## Pilar 3 — Coach de Alimentação

Plano alimentar adaptativo.

O usuário pode:

- Seguir exatamente.
- Registrar outra refeição.

A IA reorganiza o restante do dia.

---

## Pilar 4 — Diário Inteligente

Timeline automática.

Tudo que acontece durante o dia gera contexto.

---

# Fora do Escopo do MVP

- Contagem automática por smartwatch.
- Integração Apple Health.
- Integração Google Fit.
- Vídeos próprios de exercícios.
- Plano para corrida.
- Receitas.
- Comunidade/social.

---

# 3. Persona Inicial

## Persona Primária

Homens e mulheres entre 25 e 50 anos.

Características:

- Trabalham.
- Pouco tempo.
- Querem emagrecer.
- Querem ganhar massa.
- Desistem facilmente da academia.

Mario é nossa primeira persona.

---

## Dados da Persona Mario

- Desenvolvedor.
- Trabalho físico pela manhã.
- Academia às 13:30.
- Dorme cedo.
- Não gosta de peixe.
- Quer hipertrofia.
- Tendência a abandonar academia.

---

# 4. Jornada do Usuário

## Primeiro acesso (90 segundos)

Perguntas obrigatórias.

1. Nome.
2. Idade.
3. Sexo.
4. Altura.
5. Peso.
6. Objetivo.
7. Dias disponíveis para treino.
8. Academia ou casa.

Fim.

O usuário entra imediatamente.

---

## Descoberta Progressiva

Durante os dias seguintes.

Perguntas aparecem apenas quando úteis.

Exemplos:

- Dormiu bem?
- Treino foi difícil?
- Gostou dessa refeição?
- Trabalhou pesado hoje?

Sem questionário gigante.

---

# 5. Arquitetura de Contexto

O Coach possui três memórias.

## Memória Permanente

Informações quase imutáveis.

- Altura.
- Objetivo.
- Restrições alimentares.
- Academia.
- Horários.

---

## Memória da Fase

Informações do ciclo atual.

- Semana.
- Peso.
- Medidas.
- Carga dos exercícios.
- Calorias atuais.

---

## Memória do Dia

Informações temporárias.

- Sono.
- Dor.
- Energia.
- Tempo disponível.
- Trabalho pesado.

Essa memória expira diariamente.

---

# 6. Sistema de Missões

As missões são a interface principal.

## Estrutura

Cada missão possui:

- objetivo.
- progresso.
- CTA.
- recompensa.

Exemplos:

Treino.

Status:

- pendente.
- em andamento.
- concluído.
- reagendado.

---

Alimentação.

Status:

- sugerida.
- registrada.
- adaptada.

---

Água.

Incrementos rápidos.

+300 ml.

---

Sono.

Check-in diário.

---

Peso.

Somente em dias programados.

---

# 7. Coach de Treino

## Plano Mestre

O Coach cria um plano semanal.

Exemplo:

Segunda: Treino A.

Quarta: Treino B.

Sexta: Treino C.

---

## IA Tática

Adapta apenas o treino do dia.

Entradas:

- Tempo.
- Dor.
- Sono.
- Trabalho.
- Energia.

Saída:

Versão do treino.

30 min.

40 min.

60 min.

Casa.

Academia.

---

## Biblioteca Fechada de Exercícios

Cada músculo possui exercícios classificados.

Categorias:

- Principal.
- Auxiliar.
- Isolador.
- Substituto.

A IA apenas escolhe dentro dessa biblioteca.

---

## Tela do Treino

Sequência pronta.

Cada exercício contém:

- aparelho.
- imagem.
- músculo.
- peso sugerido.
- séries.
- repetições.
- tempo de descanso.

---

## Feedback Pós Exercício

Cinco estados.

😀 Muito fácil.

🙂 Fácil.

😐 Ideal.

😮 Difícil.

🥵 Não consegui terminar.

Somente respostas difíceis abrem perguntas extras.

---

## Progressão de Carga

Determinística.

Regras.

Muito fácil duas sessões seguidas.

Aumenta carga.

Ideal.

Mantém.

Não conseguiu terminar.

Reduz.

A IA explica a decisão.

---

# 8. Coach de Alimentação

## Missão da Refeição

Cada refeição possui:

Plano sugerido.

Exemplo.

3 ovos.

Banana.

Aveia.

Café.

---

## Registrar Refeição

Botões.

Foi exatamente isso.

Comi outra coisa.

---

## Registro Flexível

Busca por alimento.

Quantidade simples.

Sem exigir gramas.

---

## Compensação Inteligente

A IA ajusta o restante do dia.

Exemplo.

Pouca proteína.

Mais proteína no almoço.

Muito carboidrato.

Reduz carboidrato no jantar.

---

## Sem Culpa

O usuário nunca recebe bronca.

Recebe consequências.

---

# 9. Diário Inteligente

Timeline do dia.

Eventos.

- refeições.
- treino.
- peso.
- água.
- sono.
- humor.
- trabalho.
- observações.

Tudo gera contexto.

---

# 10. Consistência

Principal métrica do produto.

Não mede acesso ao aplicativo.

Mede aderência.

## Componentes

Treino.

Alimentação.

Proteína.

Água.

Sono.

Peso.

Cada item possui peso.

Resultado final.

0–100%.

---

## Semáforo

Verde.

Amarelo.

Laranja.

Vermelho.

Cada faixa possui mensagem diferente.

---

## Chance de Atingir a Meta

Calculada pela consistência.

Não pelo peso apenas.

---

# 11. Sistema de Notificações

## Hook7

WhatsApp.

Lembretes inteligentes.

Exemplos.

13:05.

Hora do treino.

20:15.

Hora de dormir.

08:00.

Peso hoje.

---

## Push Notification

Mesmo conteúdo.

Usuário escolhe canal.

---

# 12. Arquitetura Técnica

Frontend.

Next.js.

PWA.

Backend.

Supabase.

Banco.

PostgreSQL.

Storage.

Supabase Storage.

IA.

GPT-5.6.

Mensageria.

Hook7.

---

# 13. Estrutura do Repositório

```
s7-coach-os/

docs/
  00-visao.md
  01-prd.md
  02-regras-da-ia.md
  03-ux-system.md
  04-database.md
  05-api-hook7.md

apps/
  coach-web/

packages/
  coach-engine/
  nutrition-engine/
  workout-engine/
  ui/

supabase/

patch-logs/
```

---

# 14. Roadmap Oficial

## v0.1 Foundation

PRD.

Arquitetura.

Regras da IA.

---

## v0.2 UX System

Wireframes.

Fluxos.

Componentes.

---

## v0.3 Coach Engine

Treino.

Progressão.

Consistência.

---

## v0.4 Nutrition Engine

Missões.

Compensação.

Timeline.

---

## v0.5 Hook7 Integration

WhatsApp.

Check-ins.

Lembretes.

---

## v1.0 MVP

Aplicativo funcional para uso diário.
