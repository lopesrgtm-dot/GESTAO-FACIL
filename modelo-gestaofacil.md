
# -------------------------  TELA DASHBOARD -------------------------


# 📊 INDICADOR DE DESEMPENHO COMPARATIVO (****BADGE DE VARIAÇÃO DE FATURAMENTO***)

> **Contexto no Modelo de Negócio (GestãoFácil):** Ferramenta de inteligência operacional rápida (*Business Intelligence* simplificado) projetada para MEIs, food trucks e pequenas lanchonetes tomarem decisões durante o turno sem precisar analisar planilhas complexas.

## 1. O que significa a Badge (`▲ +14% vs ontem`)?

A **badge comparativa** é um termômetro visual de desempenho que coloca o faturamento atual em perspectiva. Em vez de exibir apenas um número isolado (ex.: **R$ 339,00**), ela responde imediatamente à pergunta principal do empreendedor:

> *"O movimento de hoje está melhor, igual ou pior do que o padrão recente do meu negócio?"*

* **Tendência Positiva (`▲ +14% vs ontem`):** Indica crescimento nas vendas em relação ao período anterior de referência.
* **Tendência Negativa (`▼ -8% vs ontem`):** Sinaliza queda de faturamento e serve como alerta preventivo enquanto o turno ainda está aberto.
* **Estabilidade (`0% vs ontem`):** Mantém o mesmo patamar financeiro do período anterior.

----------------------------------------

## 2. Como funciona na regra de negócio?

O indicador opera em **duas janelas temporais** selecionáveis no Dashboard Mobile (`dashboard-mobile.html`), adaptando a base de comparação conforme o filtro ativo:

| Filtro Ativo no Dashboard | Texto Exibido na Badge | Base de Comparação Utilizada | Objetivo para o Negócio |
| :--- | :--- | :--- | :--- |
| **Turno Hoje** | `▲ +14% vs ontem` | Faturamento do caixa aberto hoje **vs.** caixa do turno anterior (ou mesmo horário do dia anterior). | Permitir ações imediatas no turno (promoções relâmpago, reforço de preparo na chapa ou reposição rápida). |
| **Últimos 7 dias** | `▲ +21% vs sem. ant.` | Consolidado dos últimos 7 dias **vs.** os 7 dias imediatamente anteriores. | Avaliar crescimento semanal, impacto de ações de marketing e sazonalidade do ponto de venda. |

----------------------------------------

### Impacto Operacional Direto:
1. **Ação Comercial em Tempo Real:** Se às 20h a badge apontar queda acentuada, o operador pode lançar um combo ou divulgar no WhatsApp/Instagram antes de encerrar o expediente.
2. **Previsão de Ruptura de Estoque:** Um pico positivo (`▲ +14%` ou mais) avisa antecipadamente que insumos críticos (pão, blend, embalagens, bebidas) serão consumidos antes do previsto.

----------------------------------------

## 3. Cálculo Utilizado

A conta pega a diferença entre o que vendeu hoje e o que vendeu ontem, divide pelo valor de ontem e multiplica por 100 para virar porcentagem:

**Fórmula:**
`((Faturamento Hoje - Faturamento Ontem) / Faturamento Ontem) * 100`

### Exemplo Prático (Baseado no Dashboard do Turno):
* **Faturamento Hoje:** R$ 339,00
* **Faturamento Ontem:** R$ 297,37

**Passo a passo:**
1. `339,00 - 297,37 = 41,63` *[faturamento anterior - faturamento atual] = (diferença em reais)*
2. `41,63 / 297,37 = 0,1399` *[diferença em reais / faturamento atual = (proporção de crescimento)*
3. `0,1399 * 100 = +14%` *(resultado exibido na badge: ▲ +14% vs ontem)*

### Regras de Tratamento no Sistema:
* **Protótipo Front-end (`localStorage`):** Exibe valores simulados para validar a interface e a leitura rápida da tela.
* **Integração com Banco de Dados (`CAIXA` e `VENDA`):**
  1. Soma todas as vendas concluídas do caixa atual (`Pix + Cartão + Dinheiro`).
  2. Compara com o total do último caixa encerrado (ou com o mesmo horário do dia anterior).
  3. **Se ontem não teve venda (`Faturamento Ontem = 0`):** O sistema não faz a divisão (para não dar erro) e mostra `"Novo turno"` ou `"Sem base anterior"`.

### Regras de Tratamento no Sistema (Implementação Técnica):
* **Protótipo Front-end (`localStorage`):** Exibe valores simulados para validação de interface e hierarquia visual.
* **Integração com Banco de Dados (`CAIXA` e `VENDA`):**
  1. Soma todas as vendas concluídas vinculadas ao `id_caixa` atual (`Pix + Cartão + Dinheiro`).
  2. Busca o total faturado no último `CAIXA` encerrado (ou no mesmo intervalo de horas do dia anterior).
  3. **Regra de Divisão por Zero:** Caso o período anterior não possua vendas registradas (`Faturamento Anterior = 0`), a badge exibe `"Novo turno"` ou `"Sem base anterior"` para evitar erro matemático (`NaN` / divisão por zero).

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 💰 Regra de Negócio: Caixa, Fundo de Troco e Sangria

O controle financeiro de cada turno no **GestãoFácil** separa o dinheiro que já estava na gaveta do que realmente entrou como venda, garantindo que o fechamento do operador bata sem erros.

### 1. Fundo de Troco (Abertura)
* **O que é:** Dinheiro em espécie (notas menores e moedas) colocado na gaveta ao abrir o turno para facilitar a devolução de troco aos clientes.
* **Regra no sistema:** Não é um valor fixo — o operador informa manualmente quanto está colocando na abertura de cada turno (ex.: R$ 50,00 hoje, R$ 20,00 amanhã).
* **Impacto nos cálculos:** Como já pertencia ao negócio, **não soma no Faturamento**, mas entra no **Saldo em Caixa** porque está sob responsabilidade do operador.

### 2. Sangria (Retirada no Turno)
* **O que é:** Toda retirada de dinheiro em espécie feita da gaveta com o caixa ainda aberto (ex.: pagar diária de motoboy, comprar gelo/gás de emergência ou aliviar notas altas para o cofre).
* **Regra no sistema:** Deve ser registrada na hora com **valor e motivo**. Sem esse lançamento, o sistema acusará falta de dinheiro (*quebra de caixa*) no fechamento.

### 3. Cálculo do Saldo em Caixa
A conta que define quanto deve existir sob responsabilidade do operador ao longo e no final do turno é:

`Saldo em Caixa = Fundo de Troco Inicial + Vendas do Turno - Sangrias`

**Exemplo prático (Dashboard atual):**
* **Fundo de Troco:** R$ 100,00
* **Vendas do Turno:** R$ 339,00 *(Faturamento real)*
* **Sangrias:** R$ 0,00
* **Saldo em Caixa:** `100,00 + 339,00 - 0,00 = R$ 439,00`

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

##  Fluxo de Integração com o Dashboard (Ordem de Apresentação: De Cima para Baixo)

**tela de origem:**
Caixa (`caixa-mobile.html`)
**no sistema:**
Grava o fundo de troco inicial, os recebimentos separados por meio (`Pix`, `Cartão`, `Dinheiro`) e as sangrias registradas no turno.
**no dashboard:**
Alimenta os blocos superiores e centrais: **Faturamento do Turno**, **Badge comparativa** (`▲ +14% vs ontem`), **Saldo em Caixa** e **Recebimentos por Meio** (barra colorida).

---

**tela de origem:**
Pedidos (`pedidos-mobile.html`)
**no sistema:**
Grava os novos pedidos lançados, o canal de venda (`Balcão`, `Mesa`, `Delivery`) e o status de preparo (`Em preparo`, `Pronto`, `Entregue`).
**no dashboard:**
Alimenta a **Quantidade de pedidos**, o **Ticket médio**, o card **Fila na Cozinha** (`4 ativos • 2 prontos`) e o gráfico de **Vendas por Canal**.

---

**tela de origem:**
Controle (`controle-estabelecimento-mobile.html`)
**no sistema:**
Grava os canais de venda habilitados, o nome do estabelecimento, o segmento e os módulos ativos (como o Fiado).
**no dashboard:**
Define quais canais aparecem em **Vendas por Canal** e quais alertas de módulos são exibidos ou ocultados nos **Pontos de Atenção**.

---

**tela de origem:**
Estoque (`estoque-mobile.html`)
**no sistema:**
Grava o saldo atual de cada insumo e bebida após conferência física ou baixa automática por venda.
**no dashboard:**
Alimenta o bloco **Pontos de Atenção** (na base da tela), alertando automaticamente quais itens ficaram abaixo do estoque mínimo.

---

**tela de origem:**
Fiado (`fiado-mobile.html`)
**no sistema:**
Grava as vendas penduradas na conta de clientes e os pagamentos parciais ou totais recebidos.
**no dashboard:**
Alimenta o bloco **Pontos de Atenção** (na base da tela), somando o total em aberto e exibindo quantos clientes têm saldo pendente.