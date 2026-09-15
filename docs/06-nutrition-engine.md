# 06 — Nutrition Engine — S7 Coach OS

> Motor de alimentação: como o plano do dia é sugerido, registrado e compensado.
> Versão: 0.1.0
> Sujeito às regras de `03-ai-constitution.md` (seção 2).

---

## 1. Duas camadas (mesmo padrão do Workout Engine)

| Camada | Quem decide | Frequência | Pode mudar? |
|---|---|---|---|
| **Plano da Fase** | Gerado no onboarding + revisado semanalmente | 1x/semana | Só por revisão semanal ou ação explícita do usuário |
| **IA Tática** | Claude, a cada refeição | Por refeição | Compõe as refeições do dia dentro das metas da fase |

O Plano da Fase define **as metas** (calorias, macros, restrições). A IA Tática define **a composição de cada refeição** e **as compensações** quando o usuário se desvia do sugerido.

A IA Tática nunca altera as metas de calorias/macros — isso é regra 2.5 da Constituição.

---

## 2. Plano da Fase

Gerado a partir de:

* Peso, altura, idade, sexo (TDEE estimado)
* Objetivo (perder gordura / hipertrofia / ambos)
* Nível de atividade física do trabalho (ex: trabalho físico no aeroporto conta como gasto extra)
* Restrições alimentares cadastradas (ex: "não gosto de peixe")

### Saída do Plano da Fase

```
{
  calorias_meta: number
  proteina_g: number
  carboidratos_g: number
  gorduras_g: number
  agua_litros: number
  refeicoes_por_dia: number
  restricoes: [string]
}
```

Essas metas são revisadas semanalmente considerando: peso registrado na semana, consistência de aderência, feedback subjetivo (fome, energia).

---

## 3. Estrutura de uma Missão de Refeição

```
{
  refeicao_id: string
  nome: string            // "Café da manhã"
  horario_sugerido: string
  proteina_alvo_g: number
  itens_sugeridos: [
    { alimento: string, quantidade_coloquial: string, icone: string }
  ]
  status: "sugerida" | "confirmada" | "trocada" | "registrada_livre"
}
```

Quantidade **sempre coloquial** ("3 ovos", "1 xícara de aveia") — nunca em gramas para alimentos do dia a dia (regra 2.1).

---

## 4. Fluxo de Registro

### 4.1 Confirmação direta

Usuário toca **"Foi exatamente isso"** → refeição marcada como `confirmada`, macros computados a partir do plano sugerido, nenhuma ação adicional da IA.

### 4.2 Troca Inteligente

Usuário toca em um item específico do plano e escolhe um substituto de uma lista curta (alimentos equivalentes/comuns). Fluxo:

1. Sistema calcula a diferença de macros entre o item original e o substituto.
2. Se a diferença for relevante (ver limiares na seção 5), a IA gera uma frase de compensação.
3. Refeição marcada como `trocada`.

### 4.3 Registro livre ("Comi outra coisa")

Usuário busca o alimento e informa quantidade em unidades comuns (não gramas). Sistema:

1. Resolve o alimento numa base de dados nutricional (tabela de composição de alimentos).
2. Calcula macros aproximados.
3. Aplica a mesma lógica de compensação da seção 5.
4. Refeição marcada como `registrada_livre`.

---

## 5. Compensação Inteligente

### Limiares de disparo (determinístico — não é "julgamento" da IA)

| Desvio detectado | Ação |
|---|---|
| Proteína da refeição < 70% do alvo da refeição | Adiciona sugestão de +proteína na próxima refeição do dia |
| Carboidrato da refeição > 150% do alvo da refeição | Reduz sugestão de carboidrato na próxima refeição |
| Gordura da refeição > 150% do alvo da refeição | Reduz sugestão de gordura na próxima refeição, mantém proteína |
| Desvio dentro de ±30% do alvo em todos os macros | Nenhuma compensação — não gera ruído por diferenças pequenas |

### Regra de linguagem (regra 2.2 e 2.3 da Constituição)

Toda mensagem de compensação segue o formato **fato + ação**, nunca **julgamento**:

✅ *"Essa troca ficou com pouca proteína. Vou aumentar 20g de proteína no almoço."*
❌ ~~"Você deveria ter seguido o plano."~~

### Limite de recompensação em cascata

A IA nunca compensa mais de 1 refeição à frente por vez. Se o desvio persistir ao longo do dia, a compensação seguinte é recalculada com base no estado real (não empilha ajustes projetados sobre ajustes projetados).

---

## 6. Restrições Alimentares (regra 2.4 da Constituição)

* Restrições vêm exclusivamente da Memória Permanente (cadastradas no onboarding ou em Configurações).
* Nenhum item sugerido, nem em plano nem em troca, pode conter um alimento restrito.
* A IA nunca sugere um substituto que viole uma restrição, mesmo que nutricionalmente equivalente.

---

## 7. Cálculo de Macros do Dia (referência técnica)

```
TDEE = Mifflin-St Jeor (peso, altura, idade, sexo)
        + fator de atividade (trabalho físico do usuário)

Meta calórica = TDEE - déficit definido no Plano da Fase
                (déficit nunca excede o limite de segurança da regra 2.6)

Proteína = 1.6–2.2 g/kg de peso corporal (definido na criação do Plano da Fase)
Gordura  = 25–30% das calorias totais
Carboidrato = calorias restantes
```

Esse cálculo roda **fora do modelo**, em código determinístico — a IA nunca calcula TDEE "de cabeça"; ela recebe os números já calculados e só decide como distribuir/compensar entre refeições.

---

## 8. O que este motor nunca faz

* Não exige gramas para registro comum (regra 2.1).
* Não repreende escolha alimentar (regra 2.2).
* Não compensa em silêncio — toda compensação é anunciada (regra 2.3).
* Não sugere alimento restrito (regra 2.4).
* Não altera a meta calórica/macro do dia sem confirmação do usuário (regra 2.5).
* Não aplica déficit calórico além do limite de segurança do Plano da Fase (regra 2.6).
* Não empilha compensações projetadas sobre compensações projetadas (seção 5).
