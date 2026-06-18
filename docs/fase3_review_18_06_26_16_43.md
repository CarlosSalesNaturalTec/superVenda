# Relatório de Revisão de PRD — Controle de Qualidade

<auditoria_interna>

### 1. Critérios de Aceitação (BDD)

**Pontos fortes:**
- A maioria das US cobre ao menos 1 cenário feliz e 1 caminho triste
- Cenários de controle de acesso (Gerente vs Vendedor) bem distribuídos entre as US
- US 1.5 (Cancelamento) cobre bem o edge case de carnê
- US 3.1 (Venda Carnê) tem 3 cenários bem distintos (feliz, limite, bloqueio)
- US 5.1 e 5.2 (Backup/Restauração) cobrem falhas de forma adequada

**Vulnerabilidades encontradas:**

a) US 2.3 (Cadastro de Fornecedores) — apenas 1 cenário, nenhum caminho triste. Não cobre CNPJ duplicado, campos obrigatórios não preenchidos ou formato inválido de CNPJ.

b) US 2.4 (Cadastro da Empresa) — apenas 1 cenário. Não cobre formatos de imagem inválidos, imagem muito grande, ou campos obrigatórios ausentes.

c) US 3.4 (Contas a Pagar) — apenas 2 cenários felizes. Zero caminhos tristes. Não cobre valor negativo, fornecedor inexistente, lançamento duplicado, nem cancelamento de baixa.

d) US 3.5 (Despesas Fixas) — apenas 1 cenário feliz. Nenhum caminho triste.

e) US 3.2 (Juros/Multa) — inconsistência matemática: o cenário 2 afirma que R$50,00 + multa R$5,00 + juros 0,033% a.d. × 10 dias = R$55,00. O cálculo correto seria R$50,00 + R$5,00 + (R$50,00 × 0,00033 × 10) = R$55,165, não R$55,00 exato. A imprecisão pode gerar bugs de arredondamento.

f) US 3.3 C2 (Baixa em atraso) — diz que o "limite de crédito é restaurado proporcionalmente". A palavra "proporcionalmente" é ambígua: restaura R$50,00 (valor original) ou R$55,00 (valor pago com encargos)? A engenharia não conseguirá implementar isso sem esclarecimento.

g) US 4.2 (Fluxo de Caixa) — apenas 1 cenário. Não cobre período sem movimentações.

h) US 4.4 (Consulta) — não cobre cenários de "produto não encontrado" e "cliente não encontrado", que são os caminhos tristes mais frequentes.

i) Várias US não cobrem validação de entrada: valor zero ou negativo (abertura de caixa, sangria, preço de produto), senha em branco, login duplicado, exclusão de registros em uso.

### 2. Coesão de Escopo (RF × US)

**RFs que NÃO possuem User Story correspondente (GAP CRÍTICO):**

- **RF18 — Reposição de Estoque:** "Permitir que o Gerente adicione quantidade ao estoque de um produto, registrando fornecedor, quantidade e custo de aquisição." Este RF está dentro do escopo do MVP (Seção 3, item "reposição manual de estoque") mas **não possui NENHUMA história de usuário**. É uma funcionalidade inteira sem cenário BDD. **CRÍTICO.**

- **RF36 — Proteção contra Execuções Simultâneas:** "Impedir que múltiplas instâncias do sistema sejam executadas simultaneamente no mesmo computador." Está nos RFs mas não tem US correspondente.

**RFs com cobertura BDD insuficiente:**

- **RF19 — Estoque Mínimo:** A definição da quantidade mínima por produto não aparece em nenhum cenário BDD. A US 2.1 (Cadastro de Produto) não inclui o campo "estoque mínimo" em seu cenário feliz. A US 4.3 cobre apenas a consulta/relatório de produtos abaixo do mínimo. O ato de CONFIGURAR o mínimo não está coberto.

- **RF25 — Relatório por Forma de Pagamento:** Mencionado nos RFs, mas a US 4.1 não tem cenário específico para agrupamento por forma de pagamento. O agrupamento é uma funcionalidade distinta que mereceria ao menos 1 cenário.

- **RF31 — Histórico de Vendas por Cliente:** Listado nos RFs. A US 4.4 C2 (consulta de cliente) menciona "histórico de compras" nos dados exibidos, mas não há um cenário BDD específico detalhando o que esse histórico contém.

- **RF37 — Alerta de Estoque Mínimo no PDV:** A Seção 3 menciona "Alerta visual de estoque mínimo durante a venda (sem bloqueio)" como dentro do escopo, mas não há um cenário BDD na US 1.3 que demonstre esse comportamento durante a venda.

### 3. Blindagem Tecnológica

**Vazamentos identificados (leves):**

a) **RNF04 — "Banco de Dados Local":** Usa o termo "banco de dados", que é uma decisão de arquitetura. O requisito deveria descrever O QUE (armazenamento local) sem COMO (banco de dados relacional, arquivo, etc.).

b) **RNF06 — "hash criptográfico":** Especifica o mecanismo técnico de proteção de senha. Bastaria dizer "Senhas devem ser armazenadas de forma segura e irreversível".

c) **RNF13 — "pasta do sistema":** Menciona explicitamente "pasta do sistema" ao descrever o mecanismo de proteção de licença, assumindo um modelo de deploy baseado em diretório de arquivos.

**Avaliação geral:** Vazamentos leves — não há menção a linguagens de programação, frameworks, engines de banco de dados ou protocolos específicos. Os deslizes são pontuais e não comprometem a independência técnica do documento, mas devem ser ajustados.

### 4. Outros Problemas

- **Ambiguidade de Escopo — RF16 vs DRE:** O RF16 (Despesas Fixas) diz que as despesas ficam armazenadas "para compor a base de cálculo do DRE em versão futura". Porém a Seção 3 (Fora de Escopo) afirma que o "cálculo consolidado DRE virá depois". O status é claro, mas a fronteira exata do que será implementado no MVP (só lançamento ou também alguma visualização agregada?) merece uma linha de esclarecimento.

- **US 4.1 — "Vendas por vendedor" e "Vendas por produto/categoria":** A Seção 3 menciona esses relatórios no escopo do MVP, mas a US 4.1 só cobre vendas por período. Os relatórios por vendedor, cliente, produto e categoria não têm cenários BDD.

- **Seção 7 (Métricas) — Métrica 3:** "30 segundos para venda de 3 itens" — inclui ou não o tempo de pagamento? A definição diz "da leitura do primeiro código de barras até a emissão da nota A4", o que é razoavelmente claro, mas o momento exato de "início" e "fim" poderia ser mais preciso para instrumentação.

### 5. Cálculo do Score

| Dimensão | Peso | Nota | Justificativa |
|---|---|---|---|
| Cobertura BDD (caminhos tristes) | 30% | 5 | 4 US sem caminhos tristes; inconsistência matemática em US 3.2; ambiguidade em US 3.3 |
| Coesão RF × US | 25% | 4 | RF18 sem US (crítico); RF36 sem US; RF19/RF25/RF31/RF37 com cobertura parcial |
| Blindagem Tecnológica | 20% | 8 | Vazamentos leves e pontuais; sem menção a frameworks/linguagens |
| Clareza e Não-Ambiguidade | 15% | 7 | "Proporcionalmente" ambíguo; RF16 vs DRE com leve tensão |
| Cobertura de Edge Cases | 10% | 6 | Validação de entrada (zero/negativo) ausente; produto/cliente não encontrado em consultas |

**Score ponderado:** (5×0.30) + (4×0.25) + (8×0.20) + (7×0.15) + (6×0.10) = 1.5 + 1.0 + 1.6 + 1.05 + 0.6 = **5.75**

**Arredondamento:** 6/10

</auditoria_interna>

---

## 1. Veredito e Score de Prontidão

* **Score de Prontidão:** 6 / 10
* **Status:** REQUER REFATORAÇÃO (Score < 8)
* **Resumo:** O PRD apresenta uma visão de produto sólida e bem estruturada, com 37 requisitos funcionais e 15 não funcionais cobrindo adequadamente o ciclo operacional do varejista. Contudo, o documento não está pronto para a engenharia por duas razões principais: **(1) gap de coesão de escopo** — o RF18 (Reposição de Estoque) e o RF36 (Execuções Simultâneas) não possuem histórias de usuário correspondentes, e outros 4 RFs têm cobertura BDD insuficiente; **(2) qualidade irregular dos cenários BDD** — 4 US não possuem nenhum caminho triste, há inconsistência matemática nos juros do carnê (US 3.2) e ambiguidade na restauração de limite de crédito (US 3.3). Os vazamentos tecnológicos são leves e pontuais — o documento acerta ao não mencionar frameworks, linguagens ou engines de banco de dados.

---

## 2. Pontos Críticos Identificados

* **🔴 CRÍTICO — RF18 sem User Story:** A funcionalidade "Reposição de Estoque" está listada no escopo do MVP (Seção 3: "reposição manual de estoque") e no RF18, mas **não possui nenhuma história de usuário ou cenário BDD**. A engenharia não tem especificação para implementar esta feature.

* **🔴 CRÍTICO — RF36 sem User Story:** "Proteção contra Execuções Simultâneas" consta nos RFs mas não tem US correspondente. Comportamento esperado (mensagem? bloqueio silencioso?) não foi especificado.

* **🟠 ALTO — US 2.3, 2.4, 3.4, 3.5 sem caminhos tristes:** Cadastro de Fornecedores, Cadastro da Empresa, Contas a Pagar e Despesas Fixas possuem apenas cenários felizes (1-2 cada), sem cobertura de: duplicidade, campos obrigatórios, formatos inválidos, valores negativos, cancelamento de baixa ou dependências inexistentes.

* **🟠 ALTO — Inconsistência matemática na US 3.2:** O Cenário 2 afirma que R$50,00 + multa R$5,00 + juros 0,033% a.d. × 10 dias = R$55,00. O cálculo correto é R$55,165. Esta imprecisão induzirá bugs de arredondamento na implementação.

* **🟠 ALTO — Ambiguidade na US 3.3 (restauração de limite):** O Cenário 2 diz que o limite de crédito é restaurado "proporcionalmente". Não fica claro se restaura o valor original da parcela (R$50,00) ou o valor total pago com encargos (R$55,00). A engenharia não consegue implementar sem esta definição.

* **🟡 MÉDIO — RF19 (Estoque Mínimo) com cobertura parcial:** A definição da quantidade mínima por produto não aparece em nenhum cenário BDD. A US 2.1 (Cadastro de Produto) não inclui o campo "estoque mínimo". Apenas o relatório (US 4.3) está coberto.

* **🟡 MÉDIO — RF25, RF31, RF37 com cenários BDD insuficientes:** Relatório agrupado por forma de pagamento, histórico de vendas por cliente e alerta de estoque mínimo no PDV não possuem cenários BDD específicos, embora estejam no escopo.

* **🟡 MÉDIO — US 4.4 sem caminhos tristes:** A consulta de produtos e clientes não cobre "produto não encontrado" e "cliente não encontrado", que são os resultados mais frequentes em buscas no balcão.

* **🟢 BAIXO — Vazamento tecnológico leve:** RNF04 menciona "Banco de Dados Local", RNF06 especifica "hash criptográfico" e RNF13 cita "pasta do sistema". Nenhum é grave, mas devem ser neutralizados.

* **🟢 BAIXO — RF16 com leve tensão de escopo:** O lançamento de despesas fixas está no MVP, mas o cálculo DRE está fora de escopo. A fronteira exata do que será entregue vs. postergado poderia ser mais nítida.

---

## 3. Propostas de Correção Direta

### Correção 1: Adicionar US para Reposição de Estoque (RF18) — GAP CRÍTICO

* **Como está no PRD original:**
  > [Não existe. O RF18 está listado nos Requisitos Funcionais, mas não há história de usuário correspondente.]

* **Como deve ficar (Sugestão de Reescrita):**
  > **US 2.6:** Como Gerente, eu quero repor o estoque de um produto registrando fornecedor, quantidade e custo para que o inventário reflita as novas aquisições.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Reposição de estoque com sucesso*
  >     * **Dado** que o produto "Refrigerante Cola 2L" está cadastrado com estoque atual de 5 unidades e o fornecedor "Distribuidora ABC" está cadastrado
  >     * **Quando** o Gerente acessa a função de reposição, seleciona o produto, informa o fornecedor "Distribuidora ABC", quantidade 20 e custo unitário de R$ 4,50
  >     * **Então** o estoque do produto é atualizado para 25 unidades, o custo de aquisição fica registrado, e a movimentação aparece no histórico de reposições
  >   * *Cenário 2: Reposição com quantidade zero ou negativa*
  >     * **Dado** que o Gerente acessa a função de reposição de estoque
  >     * **Quando** ele informa quantidade 0 ou negativa
  >     * **Então** o sistema exibe "Quantidade inválida" e impede o registro
  >   * *Cenário 3: Vendedor não acessa reposição de estoque*
  >     * **Dado** que o Vendedor está logado
  >     * **Quando** ele tenta acessar a função de reposição de estoque
  >     * **Então** o sistema não exibe a opção no menu

### Correção 2: Adicionar cenários tristes para US com cobertura zero

* **Como está no PRD original — US 2.3 (Cadastro de Fornecedores):**
  > [Apenas Cenário 1: Cadastro de fornecedor — caminho feliz]

* **Como deve ficar (Sugestão de Reescrita — adicionar cenários):**
  > * *Cenário 2: CNPJ duplicado*
  >   * **Dado** que já existe um fornecedor cadastrado com CNPJ "12.345.678/0001-90"
  >   * **Quando** o Gerente tenta cadastrar outro fornecedor com o mesmo CNPJ
  >   * **Então** o sistema exibe "CNPJ já cadastrado para o fornecedor [nome]" e impede a duplicidade
  > * *Cenário 3: Campos obrigatórios não preenchidos*
  >   * **Dado** que o Gerente acessa o cadastro de fornecedores
  >   * **Quando** ele tenta salvar sem preencher razão social ou CNPJ
  >   * **Então** o sistema exibe mensagem indicando os campos obrigatórios e impede o salvamento

* **Como está no PRD original — US 3.4 (Contas a Pagar):**
  > [Apenas Cenário 1 (lançamento) e Cenário 2 (baixa) — ambos felizes]

* **Como deve ficar (Sugestão de Reescrita — adicionar cenários):**
  > * *Cenário 3: Lançamento com valor negativo ou zero*
  >   * **Dado** que o Gerente acessa o lançamento de contas a pagar
  >   * **Quando** ele informa valor R$ 0,00 ou valor negativo
  >   * **Então** o sistema exibe "Valor inválido" e impede o lançamento
  > * *Cenário 4: Fornecedor não cadastrado*
  >   * **Dado** que o Gerente tenta lançar uma conta a pagar
  >   * **Quando** ele informa um fornecedor que não existe na base
  >   * **Então** o sistema exibe "Fornecedor não encontrado — cadastre o fornecedor primeiro" e impede o lançamento

### Correção 3: Corrigir inconsistência matemática na US 3.2

* **Como está no PRD original:**
  > *Cenário 2: Cálculo de parcela em atraso*
  >   * **Dado** que uma parcela de R$ 50,00 venceu há 10 dias, multa fixa de R$ 5,00 e juros de 0,033% ao dia
  >   * **Quando** o Gerente consulta o contas a receber
  >   * **Então** o sistema exibe o valor atualizado de R$ 55,00 (R$ 50,00 original + R$ 5,00 multa + juros sobre o valor original) como valor devido

* **Como deve ficar (Sugestão de Reescrita):**
  > * *Cenário 2: Cálculo de parcela em atraso*
  >   * **Dado** que uma parcela de R$ 50,00 venceu há 10 dias, com multa fixa de R$ 5,00 e juros de 0,033% ao dia
  >   * **Quando** o Gerente consulta o contas a receber
  >   * **Então** o sistema exibe o valor atualizado de R$ 55,17 (R$ 50,00 original + R$ 5,00 de multa + R$ 0,17 de juros, correspondente a 0,033% × 10 dias sobre R$ 50,00, arredondado para 2 casas decimais)

### Correção 4: Eliminar ambiguidade "proporcionalmente" na US 3.3

* **Como está no PRD original:**
  > *Cenário 2: Baixa de parcela em atraso*
  >   * **Dado** que o cliente "João" possui uma parcela de R$ 50,00 vencida há 10 dias, totalizando R$ 55,00 com juros e multa
  >   * **Quando** o Gerente dá baixa na parcela recebendo R$ 55,00 em dinheiro
  >   * **Então** a parcela é marcada como "Paga" com o valor pago de R$ 55,00, o saldo do caixa é acrescido de R$ 55,00 e o limite de crédito é restaurado proporcionalmente

* **Como deve ficar (Sugestão de Reescrita):**
  > * *Cenário 2: Baixa de parcela em atraso*
  >   * **Dado** que o cliente "João" possui uma parcela de R$ 50,00 vencida há 10 dias, totalizando R$ 55,00 com juros e multa
  >   * **Quando** o Gerente dá baixa na parcela recebendo R$ 55,00 em dinheiro
  >   * **Então** a parcela é marcada como "Paga" com o valor pago de R$ 55,00 (sendo R$ 50,00 de principal + R$ 5,00 de multa + R$ 0,17 de juros), o saldo do caixa é acrescido de R$ 55,00, e o limite de crédito do cliente é restaurado no valor original da parcela (R$ 50,00)

### Correção 5: Adicionar campo "estoque mínimo" ao cenário de cadastro de produto (US 2.1)

* **Como está no PRD original — US 2.1 Cenário 1:**
  > **Quando** ele preenche descrição, código de barras "7891234567890", grupo "Bebidas", subgrupo "Refrigerantes", preço normal R$ 8,00 e preço carnê R$ 10,00
  > **Então** o produto é salvo, fica disponível para busca por código de barras e descrição no PDV, e o preço aplicado na venda à vista é R$ 8,00 e no carnê é R$ 10,00

* **Como deve ficar (Sugestão de Reescrita):**
  > **Quando** ele preenche descrição, código de barras "7891234567890", grupo "Bebidas", subgrupo "Refrigerantes", preço normal R$ 8,00, preço carnê R$ 10,00 e estoque mínimo de 10 unidades
  > **Então** o produto é salvo, fica disponível para busca por código de barras e descrição no PDV, o preço aplicado na venda à vista é R$ 8,00 e no carnê é R$ 10,00, e o alerta de estoque mínimo será disparado quando o estoque ficar abaixo de 10 unidades

### Correção 6: Neutralizar vazamentos tecnológicos nos RNFs

* **Como está no PRD original — RNF04:**
  > **RNF04 — Banco de Dados Local:** Todo o armazenamento de dados deve ser feito localmente na máquina onde o sistema está instalado, sem acesso remoto ou via rede.

* **Como deve ficar (Sugestão de Reescrita):**
  > **RNF04 — Armazenamento Local:** Todos os dados do sistema devem ser armazenados exclusivamente na máquina onde o sistema está instalado, sem dependência de acesso remoto, rede ou infraestrutura externa.

* **Como está no PRD original — RNF06:**
  > **RNF06 — Segurança de Acesso:** Senhas de usuários devem ser armazenadas de forma segura (hash criptográfico, não em texto plano).

* **Como deve ficar (Sugestão de Reescrita):**
  > **RNF06 — Segurança de Acesso:** Senhas de usuários devem ser armazenadas de forma segura e irreversível, impossibilitando a recuperação do valor original por qualquer meio.

### Correção 7: Adicionar cenários de "não encontrado" na US 4.4

* **Como está no PRD original — US 4.4:**
  > [Apenas Cenário 1 (produto encontrado) e Cenário 2 (cliente encontrado)]

* **Como deve ficar (Sugestão de Reescrita — adicionar cenários):**
  > * *Cenário 3: Produto não encontrado*
  >   * **Dado** que o Vendedor está na tela de consulta
  >   * **Quando** ele digita um código de barras ou descrição que não corresponde a nenhum produto cadastrado
  >   * **Então** o sistema exibe "Nenhum produto encontrado" e sugere verificar o código ou realizar cadastro
  > * *Cenário 4: Cliente não encontrado*
  >   * **Dado** que o Vendedor está na tela de consulta de clientes
  >   * **Quando** ele busca por um nome ou CPF que não corresponde a nenhum cliente cadastrado
  >   * **Então** o sistema exibe "Nenhum cliente encontrado" e oferece atalho para novo cadastro (se o usuário logado for Gerente)

---

## 4. Próximos Passos

1. **Aplicar as correções acima no `docs/fase2_prd.md`**, com atenção prioritária para:
   - Criar a US de Reposição de Estoque (Correção 1 — GAP crítico)
   - Completar os caminhos tristes das US 2.3, 2.4, 3.4 e 3.5 (Correção 2)
   - Corrigir a inconsistência matemática da US 3.2 (Correção 3)
   - Eliminar a ambiguidade "proporcionalmente" da US 3.3 (Correção 4)

2. **Após as correções**, executar novamente a auditoria de qualidade (esta mesma fase) para reavaliar o score. O documento **não deve ser encaminhado para a engenharia ou para ferramentas de SDD (OpenSpec)** enquanto o score estiver abaixo de 8.

3. **Tarefas opcionais (não bloqueiam engenharia, mas melhoram qualidade):**
   - Adicionar cenário BDD para RF36 (Execuções Simultâneas)
   - Adicionar cenário BDD para RF37 (Alerta de Estoque Mínimo no PDV) dentro da US 1.3
   - Expandir US 4.1 com cenários para os relatórios por vendedor, cliente, produto e categoria
   - Expandir US 4.4 com cenário de histórico de compras (RF31)
