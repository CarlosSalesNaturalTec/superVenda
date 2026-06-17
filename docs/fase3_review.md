# Relatório de Revisão de PRD — Controle de Qualidade

<auditoria_interna>

## Análise Crítica do PRD do SuperVendas

### 1. Critérios de Aceitação (BDD — Dado/Quando/Então)
O formato BDD está consistentemente aplicado na maioria das US. Cenários de caminho feliz e triste são contemplados em 90% dos casos. Porém, encontrei as seguintes fragilidades:

- **US 2.6 (Preço 2):** Apenas 1 cenário positivo. Não cobre: (a) o que acontece se o produto não tiver preço 2 cadastrado e o vendedor tentar selecioná-lo; (b) alternância entre preço normal e preço 2 no mesmo item durante a edição do carrinho. A US fica subespecificada.
- **US 3.3 (DRE):** Apenas 1 cenário (mensal com lucro). Faltam: (a) cenário diário e anual mencionados na descrição da US; (b) cenário de prejuízo (compras > vendas); (c) cenário de período sem vendas.
- **US 3.4 (Sangria):** Apenas 1 cenário. A descrição da US menciona "Vendedor ou Proprietário", e o cenário termina com "exige senha de gerente se a configuração assim determinar" — mas essa configuração não é definida em nenhum RF ou US do sistema. É uma referência circular a algo que não existe.
- **US 5.4 (Troca de Usuário):** O cenário 2 diz "as operações pendentes de Carlos são preservadas (ex.: venda em andamento é mantida)". O que o novo usuário vê ao fazer login? Uma venda em andamento de outra pessoa? Isso é uma decisão de produto não resolvida — precisa de clarificação.

### 2. Coesão de Escopo (RF vs US)

Realizei o cruzamento completo dos 32 Requisitos Funcionais contra as 21 Histórias de Usuário:

**Requisitos Funcionais SEM História de Usuário correspondente:**
- **RF04 — Cadastro de Fornecedores:** Fornecedores são mencionados nas personas e no escopo, mas NÃO EXISTE uma US para seu cadastro. É uma entidade central referenciada em US 4.1 (importação de NF-e vinculada a fornecedor) e em todo o módulo de compras. Lacuna grave.
- **RF12 — Desconto no PDV:** O sistema prevê aplicação de desconto (percentual ou valor) com limite configurável, mas NÃO EXISTE uma US para esta funcionalidade. É mencionado tangencialmente em US 1.4 (limite de desconto no perfil) e US 5.4 (senha de gerente para desconto acima do limite), mas o ato de aplicar desconto durante a venda não está coberto por nenhuma história. Lacuna crítica no fluxo principal do PDV.

**Requisitos Funcionais PARCIALMENTE cobertos:**
- **RF26 — Consultas Rápidas:** Apenas "consulta de produtos" (US 2.2) e "consulta de situação do cliente" (US 2.3) têm US dedicadas. Faltam US para: consulta de aniversariantes, consulta de estoque financeiro, e consulta de informativo de contas a pagar/receber. Três funcionalidades do escopo MVP sem especificação comportamental.

### 3. Blindagem Tecnológica (Vazamento de Decisões de Arquitetura)

O PRD é razoavelmente contido, mas há escapes:

- **RNF05:** Menciona "checksum" como mecanismo de verificação de integridade — decisão de implementação.
- **RNF06:** Menciona "hash" para armazenamento de senhas — decisão de implementação. O requisito deveria ser "senhas armazenadas de forma irreversível" e ponto.
- **US 5.1, Cenário 2:** "Destino sem espaço" — presume que o sistema consegue detectar espaço livre em disco. Essa capacidade precisa ser validada como viável, mas o PRD não deveria especificar *como*.
- **RF18:** "validar a estrutura do XML antes do processamento" — beira o detalhe técnico. Melhor seria "rejeitar arquivos que não sejam NF-e válidos e exibir mensagem descritiva".

Nenhum banco de dados, linguagem ou framework específico foi mencionado. O vazamento é leve.

### 4. Problemas de Domínio e Negócio

- **US 3.3 — DRE Simplificado (Erro Contábil):** A US define DRE como "total de vendas menos total de compras", e o cenário usa CMV (Custo de Mercadoria Vendida) = total de compras do período. Isso está **contabilmente incorreto**. O CMV correto é: `Estoque Inicial + Compras − Estoque Final`. Usar apenas o total de compras como custo gera um lucro bruto artificialmente menor em períodos de formação de estoque e artificialmente maior em períodos de queima de estoque. Para o pequeno varejista, essa distorção pode levar a decisões erradas de precificação e compras. **O PM anterior precisa decidir conscientemente se mantém essa simplificação (com ressalva documentada) ou corrige o cálculo.**

- **US 4.6 — Conferência de Estoque (Escalabilidade):** O cenário descreve contagem produto a produto ("localiza o produto e informa a quantidade real de 47"). Para um varejista com 5.000 produtos, isso é inviável. Falta especificação de como a conferência funciona em lote (por grupo, por fornecedor, por localização).

- **Métrica de Sucesso #6 (Infalsificável):** "A diferença entre o saldo de caixa calculado pelo sistema e o saldo real [...] deve ser zero em condições normais de operação (sem erros de troco ou esquecimento de registro pelo operador)." A cláusula de exceção anula a métrica — qualquer divergência pode ser atribuída a "erro do operador", tornando o sistema não responsabilizável. Métrica inválida para avaliação objetiva de sucesso.

### 5. Cenários de Borda Ausentes (Resiliência)

- **RNF09 vs. comportamento funcional:** O PRD diz que o sistema "não deve perder dados ou corromper o banco em caso de queda de energia" mas **nenhuma US especifica o comportamento funcional durante uma queda**. Se a energia cair no meio de uma finalização de venda: a venda é concluída? Desfeita? O usuário vê uma mensagem ao reiniciar? A engenharia precisa dessa especificação.
- **Concorrência mono-terminal:** US 5.4 diz que a troca de usuário preserva "venda em andamento". Mas o que o segundo usuário vê? Herda o carrinho? Precisa decidir se mantém ou descarta? Isso é ambíguo e precisa de decisão de produto.

### 6. Cálculo do Score

| Categoria | Penalidade |
|-----------|-----------|
| RF04 (Fornecedores) sem US | -0.5 |
| RF12 (Desconto) sem US | -0.4 |
| DRE com erro contábil (US 3.3) | -0.4 |
| Cenários de borda ausentes (resiliência) | -0.3 |
| RF26 parcialmente coberto | -0.3 |
| US 2.6 (Preço 2) sem bordas | -0.2 |
| US 3.4 (Sangria) referência circular | -0.2 |
| Métrica #6 infalsificável | -0.2 |
| Vazamento técnico (hash, checksum) | -0.1 |
| US 5.4 ambiguidade de sessão | -0.1 |
| **Score final** | **6.5/10** |

O documento tem méritos: estrutura clara, BDD majoritariamente bem aplicado, escopo bem delimitado (Fora de Escopo), e personas bem definidas. No entanto, as lacunas de cobertura (Fornecedores e Desconto sem US), o erro de domínio contábil no DRE e a ausência de especificação de resiliência funcional impedem que este PRD siga para engenharia sem retrabalho.

</auditoria_interna>

---

## 1. Veredito e Score de Prontidão

- **Score de Prontidão:** 6.5 / 10
- **Status:** ⛔ REQUER REFATORAÇÃO (Score < 8)
- **Resumo:** O PRD apresenta estrutura sólida, personas bem definidas, escopo MVP bem delimitado e BDD aplicado de forma consistente na maioria das histórias. No entanto, o documento possui **duas lacunas críticas de cobertura** — os Requisitos Funcionais RF04 (Cadastro de Fornecedores) e RF12 (Desconto no PDV) não possuem Histórias de Usuário correspondentes. Além disso, a US 3.3 (DRE Simplificado) contém um **erro de domínio contábil** que distorce o cálculo do lucro bruto, e faltam especificações comportamentais para cenários de queda de energia e concorrência de sessão. O PRD não deve ser encaminhado à engenharia neste estado.

---

## 2. Pontos Críticos Identificados

- **🔴 Crítico — RF04 (Cadastro de Fornecedores) sem História de Usuário:** A entidade Fornecedor é central para o módulo de compras e é referenciada na US 4.1 (importação de NF-e) e nos relatórios de compras, mas não existe nenhuma US que especifique seu cadastro, edição ou exclusão. A engenharia não tem especificação comportamental para implementar esta tela/fluxo.

- **🔴 Crítico — RF12 (Desconto no PDV) sem História de Usuário:** O sistema prevê aplicação de desconto (percentual ou valor absoluto) com limite máximo por perfil de usuário, mas não há US cobrindo o ato de aplicar, editar ou remover um desconto durante a venda. É um fluxo central do PDV completamente omitido das histórias.

- **🔴 Grave — Erro Contábil na US 3.3 (DRE Simplificado):** O DRE está definido como "total de vendas − total de compras do período". Isso está contabilmente incorreto. O CMV (Custo da Mercadoria Vendida) real é `Estoque Inicial + Compras − Estoque Final`. Usar apenas as compras do período como custo distorce o lucro bruto sempre que houver variação de estoque (formação ou queima), induzindo o lojista a decisões erradas. O PM precisa decidir se mantém a simplificação com ressalva explícita ou adota o cálculo correto.

- **🟠 Grave — Ausência de Especificação de Resiliência Funcional:** O RNF09 estabelece que o sistema não deve perder dados em quedas de energia, mas nenhuma US especifica o comportamento funcional esperado. Se a energia cair durante a finalização de uma venda: a venda é atomicamente concluída ou desfeita? O usuário vê uma notificação de recuperação ao reiniciar? A engenharia não tem como implementar resiliência sem essa definição.

- **🟠 Grave — RF26 (Consultas Rápidas) Parcialmente Coberto:** As consultas de aniversariantes, estoque financeiro e informativo de contas a pagar/receber fazem parte do escopo MVP mas não possuem US. O PM especificou o "o quê" sem o "para quem" e "por quê", e sem critérios de aceitação.

- **🟡 Moderado — US 2.6 (Preço 2) sem Cenários de Borda:** Apenas 1 cenário (positivo). Não especifica: (a) o que ocorre quando o vendedor tenta selecionar preço 2 para um produto que não o tem cadastrado; (b) se é possível alternar entre preço normal e preço 2 durante a edição do carrinho.

- **🟡 Moderado — US 3.4 (Sangria) com Referência Circular:** O cenário menciona "exige senha de gerente se a configuração assim determinar", mas essa configuração não está definida em nenhum RF, US ou seção de requisitos do documento. É uma referência a algo inexistente.

- **🟡 Moderado — US 5.4 (Troca de Usuário) com Ambiguidade:** O cenário 2 afirma que "as operações pendentes de Carlos são preservadas (ex.: venda em andamento é mantida)". O que o novo usuário (proprietário) vê ao assumir a sessão? Ele herda o carrinho de Carlos? Precisa decidir se descarta ou assume? Isso é uma decisão de produto não resolvida.

- **🟡 Moderado — Métrica de Sucesso #6 Infalsificável:** A métrica estabelece que o saldo do sistema deve bater com o saldo real "em condições normais de operação (sem erros (...) pelo operador)". A cláusula de exceção torna a métrica impossível de falsificar: qualquer divergência pode ser atribuída a erro humano, eximindo o sistema de responsabilidade. Uma métrica de qualidade não pode depender de premissas externas não controláveis.

- **🟢 Leve — Vazamento Técnico nos RNFs:** RNF05 menciona "checksum" e RNF06 menciona "hash" como mecanismos específicos de implementação. O PRD deve limitar-se ao requisito (ex.: "verificação de integridade", "armazenamento irreversível") e deixar a escolha do mecanismo para a engenharia.

- **🟢 Leve — US 4.6 (Conferência de Estoque) sem Mecanismo de Lote:** O cenário descreve contagem produto a produto. Para lojistas com milhares de SKUs, falta especificação de conferência por grupo, fornecedor ou lote.

---

## 3. Propostas de Correção Direta

### Correção 1: Adicionar US faltante — Cadastro de Fornecedores (RF04)

- **Como está no PRD original:**
  > [Não existe História de Usuário para RF04. O requisito funcional apenas lista: "O sistema deve permitir cadastro de fornecedores com: razão social, CNPJ, endereço, telefone e contato."]

- **Como deve ficar (Sugestão de Reescrita):**
  > **US 1.6:** Como Proprietário, eu quero cadastrar fornecedores com razão social, CNPJ, endereço, telefone e contato para que as compras importadas sejam corretamente atribuídas e os relatórios de compras por fornecedor façam sentido.
  > - **Critérios de Aceitação:**
  >   - *Cenário 1: Cadastro de fornecedor*
  >     - **Dado** que o proprietário está na tela de cadastro de fornecedores
  >     - **Quando** ele preenche razão social, CNPJ, endereço, telefone e nome do contato, então salva
  >     - **Então** o fornecedor fica disponível para seleção na importação de NF-e e nos filtros de relatórios de compras
  >   - *Cenário 2: CNPJ duplicado*
  >     - **Dado** que já existe um fornecedor cadastrado com o CNPJ "12.345.678/0001-90"
  >     - **Quando** o proprietário tenta cadastrar outro fornecedor com o mesmo CNPJ
  >     - **Então** o sistema rejeita o cadastro e exibe "CNPJ já cadastrado para o fornecedor [nome]"

### Correção 2: Adicionar US faltante — Desconto no PDV (RF12)

- **Como está no PRD original:**
  > [Não existe História de Usuário para RF12. O requisito funcional lista: "O sistema deve permitir aplicação de desconto (percentual ou valor) respeitando o limite máximo configurado no perfil do usuário. Descontos acima do limite exigem senha de gerente."]

- **Como deve ficar (Sugestão de Reescrita):**
  > **US 2.7:** Como Vendedor, eu quero aplicar desconto (percentual ou valor) a itens ou ao total da venda para negociar com o cliente dentro dos limites autorizados pelo meu perfil.
  > - **Critérios de Aceitação:**
  >   - *Cenário 1: Desconto percentual dentro do limite*
  >     - **Dado** que o vendedor "Carlos" tem limite de desconto de 5% e o total da venda é R$ 200,00
  >     - **Quando** Carlos aplica um desconto de 3% sobre o total
  >     - **Então** o sistema calcula o novo total de R$ 194,00 e identifica visualmente que houve desconto
  >   - *Cenário 2: Desconto acima do limite do vendedor*
  >     - **Dado** que Carlos tem limite de desconto de 5% e tenta aplicar 10%
  >     - **Quando** o sistema detecta que o desconto excede o limite
  >     - **Então** exibe "Desconto acima do limite autorizado (5%). Solicite autorização de gerente" e só aplica se for fornecida senha de gerente válida
  >   - *Cenário 3: Remoção de desconto aplicado*
  >     - **Dado** que um desconto de 3% foi aplicado à venda
  >     - **Quando** o vendedor remove o desconto antes de finalizar
  >     - **Então** o total da venda retorna ao valor original sem desconto

### Correção 3: Corrigir Cálculo do DRE (US 3.3)

- **Como está no PRD original:**
  > "DRE simplificado: total de vendas menos total de compras, com visão diária, mensal e anual"
  > *Cenário:* "[...] Receita Bruta (vendas) = R$ 50.000,00, CMV (compras) = R$ 30.000,00, Lucro Bruto = R$ 20.000,00, e Margem = 40%"

- **Como deve ficar (Sugestão de Reescrita):**
  > **US 3.3:** Como Proprietário, eu quero visualizar um DRE simplificado (Receita Bruta − CMV) para entender o lucro bruto diário, mensal e anual do negócio.
  > - **Critérios de Aceitação:**
  >   - *Cenário 1: DRE mensal com CMV calculado corretamente*
  >     - **Dado** que no mês corrente o estoque inicial era R$ 10.000,00, o total de vendas foi R$ 50.000,00, o total de compras foi R$ 30.000,00 e o estoque final apurado é R$ 12.000,00
  >     - **Quando** o proprietário acessa o DRE com filtro do mês corrente
  >     - **Então** o sistema exibe: Receita Bruta = R$ 50.000,00, CMV (Estoque Inicial + Compras − Estoque Final) = R$ 10.000 + R$ 30.000 − R$ 12.000 = R$ 28.000,00, Lucro Bruto = R$ 22.000,00, Margem Bruta = 44%
  >   - *Cenário 2: Período com prejuízo (CMV > Receita)*
  >     - **Dado** que no mês corrente as vendas foram R$ 10.000,00 e o CMV foi R$ 15.000,00
  >     - **Quando** o proprietário acessa o DRE do mês
  >     - **Então** o sistema exibe Lucro Bruto = −R$ 5.000,00 com destaque visual de valor negativo
  >   - *Cenário 3: Alternância entre visão diária, mensal e anual*
  >     - **Dado** que existem dados de vendas e compras em múltiplos períodos
  >     - **Quando** o proprietário alterna entre as visões diária, mensal e anual
  >     - **Então** o sistema recalcula Receita Bruta, CMV e Lucro Bruto para o período selecionado, utilizando o estoque inicial e final correspondentes ao intervalo

### Correção 4: Especificar Comportamento de Recuperação Pós-Falha (RNF09)

- **Como está no PRD original:**
  > "RNF09 — Resiliência: O sistema não deve perder dados ou corromper o banco em caso de queda de energia ou fechamento inesperado. Ao reiniciar, deve recuperar o último estado consistente."

- **Como deve ficar (Sugestão de Reescrita — adicionar Cenário de Aceitação Transversal):**
  > **Cenário Transversal de Resiliência — Queda de energia durante finalização de venda:**
  > - **Dado** que o vendedor estava finalizando uma venda de R$ 150,00 e a energia caiu após a confirmação, mas antes da impressão da nota
  > - **Quando** o sistema é reiniciado e o usuário faz login
  > - **Então** o sistema detecta a transação interrompida e exibe a notificação: "Uma venda estava em andamento e pode ter sido concluída. Verifique o histórico de vendas." Se a venda foi atomicamente concluída (estoque baixado, valores registrados), ela aparece no histórico. Se não foi concluída, o estoque e os valores permanecem inalterados e o carrinho NÃO é recuperado (o vendedor deve iniciar nova venda).

### Correção 5: Resolver Ambiguidade da Troca de Usuário (US 5.4)

- **Como está no PRD original:**
  > "[...] as operações pendentes de Carlos são preservadas (ex.: venda em andamento é mantida)"

- **Como deve ficar (Sugestão de Reescrita do Cenário 2):**
  > *Cenário 2: Troca de usuário com venda em andamento*
  > - **Dado** que o vendedor Carlos está logado com uma venda em andamento (carrinho não finalizado)
  > - **Quando** Carlos aciona "Trocar Usuário" e o proprietário faz login com suas credenciais
  > - **Então** o sistema exibe a mensagem: "Existe uma venda em andamento do usuário Carlos. Deseja mantê-la ou descartá-la?" Se o proprietário escolher "Manter", a venda permanece no carrinho para continuação. Se escolher "Descartar", o carrinho é limpo e o estoque reservado é liberado. Em ambos os casos, uma nova sessão é iniciada para o proprietário.

---

## 4. Próximos Passos

1. ⛔ **Este PRD NÃO está pronto para engenharia.** O score de 6.5/10 está abaixo do mínimo de 8/10 exigido para prosseguir para OpenSpec ou ferramentas de SDD.

2. As correções obrigatórias antes do reenvio são:
   - Criar US para Cadastro de Fornecedores (RF04)
   - Criar US para Desconto no PDV (RF12)
   - Corrigir o cálculo do DRE (US 3.3) ou documentar explicitamente a simplificação contábil e seus riscos
   - Especificar o comportamento funcional de recuperação pós-falha (RNF09)
   - Resolver a ambiguidade de sessão na troca de usuário (US 5.4)

3. Ações recomendadas (não bloqueantes, mas fortemente sugeridas):
   - Criar US para as consultas de aniversariantes, estoque financeiro e informativos (RF26)
   - Adicionar cenários de borda à US 2.6 (Preço 2) e US 3.4 (Sangria)
   - Remover menções a mecanismos técnicos específicos (checksum, hash) dos RNFs
   - Revisar a Métrica de Sucesso #6 para ser objetivamente mensurável

4. Após aplicar as correções, execute a **Fase 3 novamente** (`/prd_fase3_review`) para reavaliação do score. Somente após atingir score ≥ 8, prossiga para a engenharia com `/prd_fase4_patcher` ou encaminhamento direto ao time técnico.
