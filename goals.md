# Tabelas

## goals

**Schema:** `live_cdp`

**Base das metas (período e loja)**

| id | year | month | store_id | created_at | created_at | updated_at | deleted_at |
| -- | ---- | ----- | -------- | ---------- | ---------- | ---------- | ---------- |
| 1  | 2025 | 10    | 1        |            | 2022-01-04 | 2022-01-04 |            |

**Unique Index:**

	year + month + store_id

<sup>Vincula com a tabela `stores`</sup>

---

## goal_dailies

**Schema:** `live_cdp`

**Registra os pesos (%) de cada dia nas metas**

| id | date       | goal_id | share | created_at | updated_at | deleted_at |
| -- | ---------- | ------- | ----- | ---------- | ---------- | ---------- |
| 1  | 2022-01-02 | 1       | 5     | 2022-01-01 | 2022-01-04 |            |

**Unique Index:**

	date + goal_id

---

## goal_tiers

**Schema:** `live_cdp`

**Registro dos tipos de metas**

| id | name                 | type     | priority | franchise_id | created_at | updated_at | deleted_at |
| -- | -------------------- | -------- | -------- | ------------ | ---------- | ---------- | ---------- |
| 1  | Meta                 | Contábil | 1        |              | 2022-01-04 | 2022-01-04 |            |
| 2  | Super Meta           | Contábil | 2        |              | 2022-01-04 | 2022-01-04 |            |
| 3  | Hiper Meta           | Contábil | 3        |              | 2022-01-04 | 2022-01-04 |            |
| 4  | Ultra Meta           | Contábil | 4        |              | 2022-01-04 | 2022-01-04 |            |
| 5  | Meta Outlets 2024-05 | Contábil | 5        | 2            | 2022-01-04 | 2022-01-04 |            |

**Unique Index:**

	Name

<sup>`franchise_id` é opcional e indica quando a meta é específica para um tipo de loja</sup>

---

## goal_values

**Schema:** `live_cdp`

**Registro das metas totais da marca**

| id | goal_id | goal_tier_id | value    | created_at | updated_at | deleted_at |
| -- | ------- | ------------ | -------- | ---------- | ---------- | ---------- |
| 1  | 1       | 2            | 16000000 | 2022-01-04 | 2022-01-04 |            |

**Unique Index:**

	goal_id + goal_tier_id

---

## goal_sellers

**Schema:** `live_cdp`

**Registro das metas por vendedor**

| id | goal_id | goal_tier_id | employee_id | value    | created_at | updated_at | deleted_at |
| -- | ------- | ------------ | ----------- | -------- | ---------- | ---------- | ---------- |
| 1  | 1       | 2            | 1           | 16000000 | 2022-01-04 | 2022-01-04 |            |

**Unique Index:**

	goal_id + goal_tier_id + employee_id


# Painéis

## Lojas – Metas

### Visão Geral

* A tabela principal exibe um **overview das metas da marca**, ordenado do período mais recente para o mais antigo.
* Colunas exibidas:

  * `year`
  * `month`
  * `store.name`
  * `store.franchise.name`
  * `store.supervisor`
  * `value`
  * `created_at`
  * `updated_at`

### Ações

#### Adicionar Meta

* Ao clicar em **Adicionar**, o usuário deve informar:

  * **Período**

    * Ano (ex.: 2025)
    * Mês (ex.: 11)
  * **Escopo da Meta** (o usuário pode escolher apenas um tipo por cadastro):

    * **Rede**

      * Abre um seletor de *tipos de loja / franquias*
      * Permite múltiplas seleções
    * **E-commerce**
    * **Loja individual**

      * Abre um seletor contendo todas as lojas disponíveis

> Caso o usuário escolha **Rede**, ao finalizar o cadastro o sistema deve:
>
> * Identificar todas as lojas pertencentes às redes selecionadas
> * Replicar automaticamente as metas cadastradas para cada loja
>
> Deve existir uma ação separada **“Atualizar redes”** para permitir a reaplicação das metas em todas as lojas da rede, caso seja necessário.

---

## Painel de Criação / Edição de Metas

### Estrutura

* O painel utiliza um **wizard em etapas**, garantindo um fluxo guiado e consistente.

### Step 1 – Metas Gerais

* Cadastro das metas por meio de um **Repeater**.
* Regras:

  * Os valores devem respeitar a distribuição definida na tabela `goal_dailies`.
  * É permitido **apenas um registro por tipo de meta** (`goal_tier`) para cada loja.

### Step 2 – Metas Diárias

* Exibe uma **tabela contendo todos os dias do mês selecionado**.

#### Estrutura da tabela

* **Dia**
* **Share (%)**
* **Valor**

#### Comportamento

* A coluna **Share (%)** representa o peso percentual daquele dia em relação à meta mensal da loja.

* A coluna **Valor** permite que o usuário informe um **valor bruto diário**.

* Quando o **Valor** é preenchido:

  * O sistema converte automaticamente o valor bruto em **percentual (Share)**.
  * A conversão utiliza como base a **Meta de priority = 1** (meta principal) vinculada à loja.

* **Regra de ajuste automático (limite de 100%)**:

  * A soma total dos shares do mês **não pode ultrapassar 100%**.
  * Caso a inserção de um valor bruto faça o total exceder 100%, o sistema deve:

    1. Calcular o excedente:

       * Exemplo: total = 105% → excedente = `105 - 100 = 5%`
    2. Distribuir o excedente negativamente entre os demais dias:

       * Quantidade de dias considerados = total de dias do mês **menos o dia ajustado manualmente**
       * Exemplo: `5% / 29 dias ≈ 0,16%`
    3. Subtrair o valor proporcional de cada um dos outros dias.
    4. Caso algum dia fique com valor inferior a `0%`:

       * Recalcular o excedente apenas entre os dias restantes válidos
       * Exemplo: `4 dias × 0,16% = 0,64%`
       * Abater esse valor do dia com **maior participação percentual**.

> Esse processo garante que o total mensal permaneça sempre em 100%, sem perda de consistência na distribuição.

---

### Step 3 – Metas por Vendedor

* Permite o **cadastro e a distribuição das metas no nível dos vendedores da loja**.

#### Comportamento

* O sistema deve oferecer as seguintes opções de configuração:

1. **Distribuir igualmente entre vendedores**

   * O total da meta da loja é dividido igualmente pela quantidade de vendedores ativos.

2. **Atribuir peso percentual por vendedor**

   * O usuário define o peso (%) individual de cada vendedor, não podendo ultrapassar 100%.
   * Exemplo:

     * Vendedor A: 50%
     * Vendedor B: 20%
     * Vendedor C: 30%

3. **Criar meta por vendedor (manual)**

   * Permite que o usuário informe livremente **valores consolidados de meta por vendedor**.
   * Utiliza o mesmo modelo de **Repeater do Step 1**, porém aplicado ao nível do vendedor.

## Alterações em Painéis Existentes

### Painel de Lojas

* Replicar os **Steps 1 e 2** descritos anteriormente dentro do módulo de Lojas.
* Se possível, **unificar os dois steps em uma única experiência**, simplificando o fluxo para o usuário.

### Painel de Funcionários

* Replicar os **Steps 1 e 2 do Painel de Lojas**, convertendo-os para a **visão de funcionários**.
* Se possível, **unificar os steps em um único painel**, mantendo consistência de regras e comportamento.
