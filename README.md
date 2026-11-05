# Análise de Cesta de Compras (Market Basket Analysis): Supermercado

Quais produtos os clientes compram juntos e **quanto dessa relação é real** e quanto é só popularidade dos itens?
Este projeto implementa **do zero, em pandas**, as métricas de regras de associação usadas em varejo para responder a isso.

> 🇺🇸 *Association-rule mining (support, confidence, lift, conviction) implemented from scratch in pandas on 4,627 supermarket baskets, with a critical reading of lift.*

## Por que isso importa

Regras de associação embasam decisões comuns de varejo: **layout da loja** (o que colocar perto do quê), **combos e promoções**, **recomendação de "quem comprou X também comprou Y"** e reposição de estoque.
O risco é agir com base em uma regra que parece forte, mas que só reflete itens muito populares. A análise do lift trata exatamente desse risco.

## Dados

- `supermarket.csv`: **4.627 compras (cestas)** × **216 departamentos/itens** (+ coluna `total`).
- Cada célula indica se o item estava na compra (`t`) ou não (`?`). A mediana é de 18 itens por cesta.
- Os itens mais frequentes são pão e bolo (*bread and cake*, 72% das compras), frutas (64%), vegetais (64%), creme de leite (*milk-cream*, 64%) e itens de panificação (*baking needs*, 60%).

## Método

1. **Pré-processamento:** `t` → 1 e `?` → 0.
2. **Itens frequentes:** filtro de **suporte ≥ 60%** (a mesma ideia de poda do Apriori), restando 5 itens.
3. **Métricas implementadas manualmente** para regras de 2, 3 e 4 itens (`A ⇒ B`, `{A,B} ⇒ C`, `{A,B,C} ⇒ D`, `{A,B} ⇒ {C,D}`):

| Métrica | Fórmula | O que responde |
|---|---|---|
| Suporte | P(A ∩ B) | Com que frequência a combinação aparece? |
| Confiança | P(B \| A) | De quem levou A, quantos levaram B? |
| Lift | P(B \| A) / P(B) | A presença de A **aumenta** a chance de B além do esperado? (1 = independência) |
| Convicção | (1 − P(B)) / (1 − conf) | Quanto a regra "erra menos" do que se A e B fossem independentes? |

Limiares usados: confiança ≥ 0,56, lift ≥ 1 e convicção ≥ 1,2.

## Principais resultados

- **Confiança alta:** quem leva creme de leite leva também pão e bolo em **79,5%** das compras. Na regra `{frutas, creme de leite, panificação} ⇒ pão e bolo`, isso sobe para **85%** (suporte de 25% das cestas, convicção **1,87**).
- **Mas o lift é modesto:** nos pares, o lift fica entre **1,06 e 1,16**. Como pão e bolo já aparecem em 72% de todas as compras, uma confiança de 80% está pouco acima do que se esperaria ao acaso. A regra é **frequente, mas não é uma afinidade forte**.
- O par mais "dependente" é **vegetais ⇒ frutas** (lift 1,16 e convicção 1,41), coerente com uma compra de hortifrúti.

### O que dá para fazer com isso

- **Layout:** aproximar hortifrúti (frutas e vegetais) e manter pão e bolo como produto âncora no caminho do cliente.
- **Promoções:** como o lift é baixo, um combo "creme de leite + pão e bolo" provavelmente **daria desconto para quem já compraria os dois**. Antes de lançar, vale testar (A/B por loja ou período).
- **Próximo passo:** baixar o suporte mínimo para encontrar pares menos óbvios com **lift alto**, que costumam ser mais úteis para venda cruzada.

## Limitações

- Com suporte mínimo de 60%, só entram os 5 itens mais populares. As regras encontradas são, por construção, entre itens comuns.
- O notebook calcula o "lift" das regras com 3 ou 4 itens como `P(todos) / Π P(item)` (valores entre ~1,3 e 1,5). Essa não é a definição padrão de lift de uma regra, que é `conf / P(consequente)`. Pela definição padrão, a regra de 85% de confiança acima tem lift **≈ 1,18**, e o maior lift entre as regras de até 4 itens é **≈ 1,26**.
- Os dados não têm data, preço nem cliente, então não dá para medir receita, sazonalidade ou recorrência.

## Como executar

```bash
pip install pandas numpy jupyter
jupyter notebook Regras_de_Associação_Supermarket.ipynb
```

O notebook foi feito no Google Colab e lê `/content/supermarket.csv`. Para rodar localmente, troque o caminho por `supermarket.csv`.

## Tecnologias

Python · pandas · NumPy · Jupyter/Colab
