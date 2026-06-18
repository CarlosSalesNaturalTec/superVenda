# Relatório de Revisão de PRD — Controle de Qualidade

<auditoria_interna>

## Análise Crítica por Dimensão

### 1. Critérios de Aceitação (BDD)

**Pontos positivos:**
- A grande maioria das US adota o formato Dado/Quando/Então de forma consistente.
- A US 1.5 (Cancelamento de Venda) é exemplar: cobre 5 cenários incluindo cancelamento de carnê com parcelas parcialmente pagas e cancelamento de venda de dia anterior — demonstra maturidade de pensamento de borda.
- A US 3.1 (Venda no Carnê) cobre bem o caminho feliz, limite de crédito estourado e cliente bloqueado.
- As US 2.3, 2.6 e 3.4 incluem cenários de validação de entrada (valores negativos, zero, campos obrigatórios).

**Vulnerabilidades encontradas:**

a) **Cenários de borda ausentes em várias US:**
  - US 1.1: Não cobre "o que acontece se o Gerente tentar abrir o caixa com valor inicial R$ 0,00 ou negativo?"
  - US 1.2: Não cobre "tentativa de fechar um caixa que já está fechado" (simétrico ao Cenário 2 da US 1.1).
  - US 1.3: Não especifica o que ocorre se, durante a venda, um produto tiver seu preço alterado por outro fluxo ou for excluído da base.
  - US 1.4: Não define desconto máximo permitido (ex: 100% faria a venda sair de graça?). Não esclarece se o desconto é sobre valor total ou por item.
  - US 1.6: Não cobre "tentativa de sangria com caixa fechado".
  - US 2.2: Não define campos obrigatórios no cadastro de cliente (nome e CPF apenas? Endereço e telefone são opcionais?).
  - US 2.4: Apenas 1 cenário. Não cobre: formatos de imagem aceitos, tamanho máximo, comportamento se logotipo não for carregado, falha no upload.
  - US 2.5: Não cobre requisitos mínimos de senha (tamanho, complexidade), nem troca de senha, nem exclusão do último Gerente (risco de lockout).
  - US 3.5: Apenas 1 cenário. Não cobre edição/exclusão de despesa fixa, nem validação de dia de vencimento (ex: dia 31 para meses com 30 dias).
  - US 4.2: Apenas 1 cenário. Não menciona filtro por período customizado (apenas "do dia").
  - US 5.2: Não trata compatibilidade de versão — o que acontece ao restaurar backup de uma versão antiga em uma versão mais nova do sistema?

b) **Cenários felizes sem contrapartida triste:**
  - US 3.2 (Cenário 2): O exemplo de cálculo R$ 55,17 usa premissas de arredondamento que não estão explicitadas. O PRD diz "0,033% ao dia (1% ao mês)" mas 0,033% × 30 = 0,99%, não 1% — inconsistência que afeta cálculo de juros composto em atrasos longos.
  - US 1.3 (Cenário 1): "emite a nota de venda A4" — não fica claro se a emissão é automática (impressão imediata) ou sob demanda (visualização + opção de imprimir).

c) **Rastreabilidade BDD inconsistente:**
  - Alguns critérios usam linguagem imperativa em vez de observável: "o Vendedor digita ou escaneia" (US 4.4) — "escaneia" implica hardware (leitor de código de barras) que não está listado em requisitos não funcionais ou premissas.
  - US 4.4 Cenário 2: "o sistema exige os dados cadastrais" — provável typo de "exibe" em vez de "exige".

### 2. Coesão de Escopo

**Mapeamento RF → US:**
Todos os 37 RFs foram rastreados contra as US. 35 dos 37 possuem User Story correspondente com critérios de aceitação. Exceções:

a) **RF25 (Relatório de Vendas por Forma de Pagamento):** Mencionado na seção de Escopo mas SEM User Story dedicada. A US 4.1 cobre vendas por período, vendedor e cliente — mas o agrupamento por forma de pagamento (dinheiro, cartão, PIX) fica órfão de BDD.

b) **RF31 (Histórico de Vendas por Cliente):** Citado nos RFs e tangenciado na US 4.4 (consulta de situação do cliente), mas não tem US própria com critérios de aceitação que validem a exibição de: data, valor, itens, situação (ativa/cancelada) para TODAS as compras do cliente.

c) **RF19/RF37 — Duplicidade:** Ambos tratam de alerta de estoque mínimo no PDV. RF37 é redundante com RF19. Nenhum dos dois tem US dedicada — o alerta aparece como efeito colateral no Cenário 1 da US 2.1, mas sem cenário próprio de "Vendedor vê alerta de estoque mínimo durante a venda e decide se prossegue ou não".

d) **Exclusão de entidades:** O log de auditoria (RF35) menciona "exclusão de registros", mas NENHUMA US cobre o fluxo de exclusão de produto, cliente ou fornecedor. Excluir um produto com histórico de vendas é destrutivo? Soft-delete? Não está especificado.

e) **Alteração de entidades:** Nenhuma US cobre edição de produto (alterar preço, descrição), edição de cliente (mudar limite de crédito), ou edição de fornecedor. O RF35 menciona "alteração de preço" como ação auditável, mas o fluxo de alteração em si não tem US.

f) **Reabertura de caixa:** Se o sistema travar ou houver queda de energia com o caixa aberto, como o Gerente retoma? Não há US para "reabertura de caixa após interrupção".

### 3. Blindagem Tecnológica

**Avaliação geral: EXCELENTE.** O PRD é notavelmente limpo de vazamento tecnológico.

- Nenhum banco de dados específico é mencionado (SQLite, PostgreSQL, etc.).
- Nenhuma linguagem de programação ou framework é sugerido.
- Nenhuma biblioteca de terceiros é referenciada.
- O mecanismo de licenciamento é descrito em termos de negócio ("arquivo-chave vinculado ao hardware"), sem especificar implementação (hash de hardware, criptografia, etc.).

**Únicos pontos de atenção (menores):**
- RNF06 menciona "armazenadas de forma segura e irreversível" — isso tangencia decisão de hashing, mas está no nível correto de abstração para um PRD (requisito não funcional de segurança, não especificação de algoritmo).
- RNF07 menciona "operações críticas sejam atômicas" — conceito de atomicidade de transações. Aceitável como requisito não funcional de integridade.
- RNF11 menciona "totalmente operável por teclado (atalhos, navegação por Tab)" — isso é requisito de usabilidade, não de tecnologia.

**Veredito: Sem contaminação tecnológica relevante. O PRD está maduro nesse quesito.**

### 4. Perguntas de Negócio Não Respondidas

a) **PIX e Cartões OFFLINE (CRÍTICO):** O sistema é definido como 100% offline (RNF01), mas suporta PIX, cartão de crédito e cartão de débito como formas de pagamento (RF05). No mundo real, PIX exige conectividade com o Banco Central para liquidação; cartões exigem comunicação com adquirentes. O PRD NÃO esclarece se:
  - O sistema apenas REGISTRA a forma de pagamento (confiando que o lojista processou a transação em outro dispositivo, como maquininha de cartão ou app de banco)?
  - Ou se há expectativa de integração com adquirentes/PSPs?
  Esta ambiguidade é perigosa e pode levar a retrabalho significativo na engenharia.

b) **Precificação dinâmica:** Se o preço de um produto for alterado hoje, as parcelas de carnê já geradas (vendas passadas) são recalculadas ou mantêm o preço da data da venda? O PRD não responde.

c) **Comportamento do caixa na virada do dia:** Se o Gerente esquecer de fechar o caixa, ele permanece aberto indefinidamente? Existe fechamento automático? Vendas podem cruzar a meia-noite? Não especificado.

d) **Valor mínimo de parcela (US 3.7):** O Cenário 2 menciona R$ 5,00 como piso, mas não diz se este valor é fixo ou configurável pelo Gerente.

e) **Grupos e Subgrupos de produtos (US 2.1):** São entidades separadas gerenciáveis (cadastro de grupos) ou texto livre? Se forem texto livre, como garantir consistência nos relatórios?

f) **Backup e LGPD:** O backup contém dados pessoais de clientes (CPF, endereço). O arquivo de backup deve ter algum mecanismo de proteção (senha, criptografia)? Isso não é abordado.

g) **Troca de senha do Gerente:** Não há US para "esqueci minha senha" ou troca de senha. Em ambiente offline, sem e-mail de recuperação, como o Gerente recupera o acesso?

h) **Múltiplos Gerentes:** O sistema suporta mais de um usuário com perfil Gerente? Se sim, como diferenciar logs de auditoria?

### 5. Análise das Métricas de Sucesso

- Métrica #3 (Tempo de Atendimento): "30 segundos para venda de 3 itens" — como será medido? Não há baseline do processo manual atual para comparação.
- Métrica #6 (Satisfação): "relatar que é fácil de usar" e "não precisa chamar o gerente mais do que 2 vezes por dia" — altamente subjetiva, sem instrumento de medição definido.
- Métrica #5 (Autonomia Operacional): "execução bem-sucedida de ao menos 2 backups em dias diferentes nos primeiros 30 dias" — métrica boa, mas assume que o lojista documentará isso. Sem instrumento.

### 6. Cálculo do Score

Partindo de 10:

- **(-1.0) — Questão PIX/Cartões offline não resolvida:** Impacto arquitetural severo. A engenharia pode implementar algo que não atende à realidade operacional do lojista.
- **(-0.5) — RF25 e RF31 sem User Stories:** Dois relatórios do escopo MVP ficaram sem BDD, o que significa que engenharia não tem critério objetivo para considerar "pronto".
- **(-0.5) — Fluxos de manutenção de entidades ausentes:** Exclusão e edição de produtos, clientes e fornecedores não têm US. São operações de risco (auditáveis conforme RF35) que chegarão à engenharia sem especificação.
- **(-0.3) — Cenários de borda relevantes ausentes:** Fechamento de caixa já fechado, desconto máximo, recuperação de senha, formatos de logotipo, compatibilidade de versão de backup.
- **(-0.2) — Inconsistência matemática na US 3.2:** 0,033% × 30 = 0,99% ≠ 1%. Parece menor, mas em cálculos financeiros de carnê com muitas parcelas, o erro composto importa.
- **(-0.3) — Métricas subjetivas ou sem instrumento de medição:** Métricas #3 e #6 não são mensuráveis objetivamente como escritas.
- **(-0.2) — Duplicidade e dispersão:** RF19/RF37 duplicados; alerta de estoque mínimo sem US própria; "escaneia" na US 4.4 assume hardware não declarado.

**Score final: 7.0 / 10**

</auditoria_interna>

---

## 1. Veredito e Score de Prontidão

* **Score de Prontidão:** 7.0 / 10
* **Status:** REQUER REFATORAÇÃO (Score < 8)
* **Resumo:** O PRD demonstra maturidade em estrutura, cobertura BDD e blindagem tecnológica — está entre os melhores 30% dos PRDs que este revisor já auditou. No entanto, **duas vulnerabilidades impedem o envio imediato para engenharia**: (1) a contradição não resolvida entre "100% offline" e formas de pagamento que exigem conectividade no mundo real (PIX, cartões) — sem clarificação, a arquitetura pode ser construída sobre premissa errada; e (2) duas funcionalidades do escopo MVP (RF25 e RF31) não têm User Stories com critérios de aceitação, deixando a engenharia sem definição objetiva de "pronto". Adicionalmente, fluxos inteiros de manutenção de dados mestres (exclusão e edição) estão órfãos de especificação, apesar de serem citados como ações auditáveis. Com as 7 correções propostas abaixo, o documento atingirá Score 8.5+ e estará apto para Spec-Driven Development.

---

## 2. Pontos Críticos Identificados

* **🔴 [CRÍTICO #1] — Contradição offline vs. PIX/Cartões:** O RNF01 exige funcionamento 100% offline, mas o RF05 lista PIX, cartão de crédito e cartão de débito como formas de pagamento. No mundo real, PIX exige conectividade com o SPI/Banco Central e cartões exigem comunicação com adquirentes. O PRD não esclarece se o sistema apenas **registra** a forma de pagamento (confiando em processamento externo via maquininha/app) ou se há expectativa de **integração**. Esta ambiguidade pode levar a retrabalho arquitetural severo.
* **🔴 [CRÍTICO #2] — RF25 e RF31 sem User Stories:** O Relatório de Vendas por Forma de Pagamento (RF25) e o Histórico de Vendas por Cliente (RF31) são listados no escopo do MVP e nos Requisitos Funcionais, mas NENHUM dos dois possui User Story com critérios de aceitação BDD. A engenharia receberá esses itens sem definição objetiva de "concluído".
* **🟠 [ALTA #3] — Fluxos de exclusão e edição de entidades ausentes:** O RF35 (Log de Auditoria) referencia "exclusão de registros" e "alteração de preços" como ações auditáveis, mas **nenhuma US** especifica como funciona a exclusão ou edição de produtos, clientes e fornecedores. Excluir um produto com histórico de vendas é destrutivo? Soft-delete? Alterar preço de produto afeta parcelas de carnê já geradas?
* **🟠 [ALTA #4] — Cenários de borda críticos não cobertos:** US 1.2 não cobre "fechar caixa que já está fechado" (simétrico ao cenário de duplicidade da abertura). US 1.4 não define percentual máximo de desconto (100% = venda de graça?). US 1.6 não cobre "sangria com caixa fechado". US 2.5 não trata exclusão do último Gerente (risco de lockout total do sistema).
* **🟡 [MÉDIA #5] — Inconsistência matemática nos juros do carnê:** A US 3.2 define juros de "0,033% ao dia (1% ao mês)", mas 0,033% × 30 = 0,99%, não 1%. Em carnês com dezenas de parcelas e atrasos longos, o erro composto pode gerar divergências financeiras reais nos relatórios e expectativas do lojista.
* **🟡 [MÉDIA #6] — Grupos e Subgrupos sem definição de gestão:** A US 2.1 menciona grupo "Bebidas" e subgrupo "Refrigerantes", mas não define se esses valores são entidades gerenciáveis (com cadastro próprio) ou texto livre. Se forem texto livre, a consistência dos relatórios por categoria fica comprometida.
* **🟡 [MÉDIA #7] — Premissas de hardware não declaradas:** A US 4.4 menciona "escaneia o código" e a seção de Escopo lista "código de barras", mas o PRD não declara em nenhum lugar a premissa de que o computador terá um leitor de código de barras conectado — ou se a leitura será apenas por digitação manual.

---

## 3. Propostas de Correção Direta

### Correção 1: Esclarecer processamento offline de PIX e Cartões

* **Como está no PRD original (RNF01 + RF05):**
  > "RNF01 — Funcionamento Offline: O sistema deve operar 100% offline, sem nenhuma dependência de conexão com internet para suas funcionalidades de negócio."
  > "RF05 — Formas de Pagamento: Suportar dinheiro, cartão de crédito, cartão de débito e PIX como formas de pagamento."

* **Como deve ficar (Sugestão de Reescrita):**
  > **RF05 — Formas de Pagamento:** Suportar dinheiro, cartão de crédito, cartão de débito e PIX como formas de pagamento **para fins de registro e conciliação de caixa**. O sistema NÃO processa transações eletrônicas: o lojista utiliza dispositivo externo (maquininha de cartão, app de banco no celular) para liquidação financeira. O superVenda atua como **registro contábil** da forma de pagamento informada, permitindo que o fechamento de caixa confira o dinheiro físico e os comprovantes das transações eletrônicas. Uma venda pode ter mais de uma forma de pagamento (pagamento misto).
  
  Adicionar em **RNF01:**
  > "O sistema não depende de internet para nenhuma funcionalidade de negócio. O registro de formas de pagamento eletrônicas (PIX, cartões) é puramente contábil — a liquidação financeira ocorre em dispositivos externos independentes do sistema."

---

### Correção 2: Criar User Stories para RF25 e RF31

* **Como está no PRD original:** RF25 e RF31 são listados apenas nos Requisitos Funcionais, sem User Story correspondente.

* **Como deve ficar (Sugestão de Reescrita):** Adicionar ao Épico 4:

  > **US 4.6:** Como Gerente, eu quero visualizar o relatório de vendas agrupado por forma de pagamento para que eu saiba a participação de cada meio de pagamento no faturamento.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Relatório por forma de pagamento no período*
  >     * **Dado** que houve vendas no mês com diferentes formas de pagamento: R$ 5.000 em dinheiro, R$ 3.000 em PIX, R$ 2.000 em cartão de crédito e R$ 1.000 em cartão de débito
  >     * **Quando** o Gerente acessa o relatório de vendas por forma de pagamento e seleciona o período do mês
  >     * **Então** o sistema exibe o totalizador por forma de pagamento com valor absoluto e percentual do total, e o total geral de R$ 11.000,00 no rodapé
  >   * *Cenário 2: Período sem vendas*
  >     * **Dado** que não houve nenhuma venda no período selecionado
  >     * **Quando** o Gerente acessa o relatório
  >     * **Então** o sistema exibe "Nenhuma venda no período" e totalizadores zerados

  > **US 4.7:** Como Gerente, eu quero visualizar todo o histórico de compras de um cliente para que eu possa analisar seu comportamento de consumo.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Histórico com vendas à vista e carnê*
  >     * **Dado** que o cliente "Maria" realizou 3 compras: uma à vista, uma no carnê (em andamento) e uma no carnê (já quitada)
  >     * **Quando** o Gerente acessa o histórico de compras de "Maria"
  >     * **Então** o sistema exibe a lista das 3 compras em ordem cronológica inversa, com data, valor total, forma de pagamento, situação (Ativa/Cancelada/Quitada) e, para carnê, quantidade de parcelas e status de cada parcela
  >   * *Cenário 2: Cliente sem histórico de compras*
  >     * **Dado** que o cliente "João" está cadastrado mas nunca realizou nenhuma compra
  >     * **Quando** o Gerente acessa o histórico de compras de "João"
  >     * **Então** o sistema exibe "Nenhuma compra registrada para este cliente"

---

### Correção 3: Adicionar User Stories para exclusão e edição de entidades

* **Como está no PRD original:** O RF35 menciona "exclusão de registros" e "alteração de preço" como ações auditáveis, mas nenhuma US cobre esses fluxos.

* **Como deve ficar (Sugestão de Reescrita):** Adicionar ao Épico 2:

  > **US 2.7:** Como Gerente, eu quero editar os dados de um produto cadastrado para que eu possa corrigir informações ou ajustar preços.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Alteração de preço de produto*
  >     * **Dado** que o produto "Refrigerante Cola 2L" está cadastrado com preço normal R$ 8,00
  >     * **Quando** o Gerente altera o preço normal para R$ 9,00
  >     * **Então** o novo preço é aplicado apenas para vendas futuras; parcelas de carnê já geradas mantêm o preço da data da venda original. A alteração é registrada no log de auditoria com data/hora, usuário e valores anterior/novo.
  >   * *Cenário 2: Alteração de limite de crédito do cliente*
  >     * **Dado** que o cliente "Maria" tem limite de crédito de R$ 500,00 e R$ 200,00 em parcelas em aberto (saldo disponível: R$ 300,00)
  >     * **Quando** o Gerente reduz o limite de crédito para R$ 300,00
  >     * **Então** o novo limite é salvo, e o saldo disponível passa a ser R$ 100,00 (R$ 300,00 − R$ 200,00). A alteração é registrada em log. Se o novo limite for menor que o saldo em aberto, o sistema exibe "Limite inferior ao saldo em aberto — reduza após a quitação" e impede a alteração.
  >   * *Cenário 3: Vendedor não edita produtos*
  >     * **Dado** que o Vendedor está logado
  >     * **Quando** ele tenta acessar a edição de um produto
  >     * **Então** o sistema não exibe a opção de edição ou exibe mensagem de acesso restrito

  > **US 2.8:** Como Gerente, eu quero excluir um produto ou cliente que foi cadastrado por engano para que a base de dados não acumule registros inúteis.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Exclusão de produto sem vendas*
  >     * **Dado** que o produto "Teste" está cadastrado e nunca foi utilizado em nenhuma venda
  >     * **Quando** o Gerente seleciona excluir o produto e confirma com sua senha
  >     * **Então** o produto é removido definitivamente da base, e a ação é registrada no log de auditoria
  >   * *Cenário 2: Tentativa de exclusão de produto com histórico de vendas*
  >     * **Dado** que o produto "Refrigerante Cola 2L" possui vendas registradas
  >     * **Quando** o Gerente tenta excluí-lo
  >     * **Então** o sistema exibe "Produto possui histórico de vendas e não pode ser excluído. Você pode inativá-lo para impedir novas vendas." e oferece a opção de inativação. Se inativado, o produto não aparece nas buscas do PDV mas permanece nos relatórios e histórico.
  >   * *Cenário 3: Exclusão de cliente sem compras*
  >     * **Dado** que o cliente "José" está cadastrado e nunca realizou nenhuma compra
  >     * **Quando** o Gerente seleciona excluir o cliente e confirma com sua senha
  >     * **Então** o cliente é removido definitivamente, e a ação é registrada no log de auditoria
  >   * *Cenário 4: Tentativa de exclusão de cliente com compras ou parcelas em aberto*
  >     * **Dado** que o cliente "Maria" possui vendas registradas ou parcelas de carnê em aberto
  >     * **Quando** o Gerente tenta excluí-la
  >     * **Então** o sistema exibe "Cliente possui histórico — não é possível excluir" e impede a exclusão

---

### Correção 4: Preencher cenários de borda ausentes

* **Como está no PRD original (US 1.2):**
  > A US 1.2 não possui cenário para "fechamento de caixa que já está fechado".

* **Como deve ficar (Sugestão de Reescrita):** Adicionar após o Cenário 3 da US 1.2:
  > * *Cenário 4: Tentativa de fechar caixa já fechado*
  >   * **Dado** que o caixa do dia atual já foi fechado
  >   * **Quando** o Gerente tenta acessar a função de fechamento de caixa
  >   * **Então** o sistema exibe "Caixa já está fechado" e exibe os dados do fechamento anterior (data/hora, saldo final, divergência se houver)

* **Como está no PRD original (US 1.4):**
  > A US 1.4 não define limite máximo de desconto.

* **Como deve ficar (Sugestão de Reescrita):** Adicionar após o Cenário 2 da US 1.4:
  > * *Cenário 3: Desconto acima do máximo permitido*
  >   * **Dado** que o Gerente configurou o desconto máximo permitido como 30% nas configurações do sistema
  >   * **Quando** o Gerente tenta aplicar 50% de desconto em uma venda
  >   * **Então** o sistema exibe "Desconto máximo permitido: 30%" e impede o desconto acima do teto. O Gerente pode alterar o teto nas configurações do sistema com sua senha.
  > * *Cenário 4: Desconto que zera o valor da venda*
  >   * **Dado** que o Gerente aplica um desconto que torna o valor da venda R$ 0,00
  >   * **Quando** o Gerente confirma
  >   * **Então** o sistema exibe "Valor da venda não pode ser zero. Reduza o desconto." e impede a finalização

* **Como está no PRD original (US 1.6):**
  > A US 1.6 não cobre sangria com caixa fechado.

* **Como deve ficar (Sugestão de Reescrita):** Adicionar após o Cenário 3 da US 1.6:
  > * *Cenário 4: Tentativa de sangria com caixa fechado*
  >   * **Dado** que o caixa está fechado
  >   * **Quando** o Gerente tenta registrar uma sangria
  >   * **Então** o sistema exibe "Caixa fechado — abra o caixa para realizar sangria" e impede a operação

---

### Correção 5: Corrigir inconsistência matemática dos juros

* **Como está no PRD original (US 3.2):**
  > "juros de 0,033% ao dia (1% ao mês)"

* **Como deve ficar (Sugestão de Reescrita):**
  > "juros de 1% ao mês (equivalente a aproximadamente 0,0333% ao dia, calculado como taxa diária equivalente com capitalização composta: (1 + 0,01)^(1/30) − 1 ≈ 0,0332% ao dia). O sistema deve utilizar a fórmula de juros compostos com a taxa mensal como parâmetro base para garantir precisão em atrasos longos."

  E ajustar o Cenário 2:
  > **Quando** o Gerente consulta o contas a receber
  > **Então** o sistema exibe o valor atualizado conforme a fórmula: `valor_original × (1 + taxa_mensal)^(dias_atraso/30) + multa_fixa`. Para o exemplo: R$ 50,00 × (1,01)^(10/30) + R$ 5,00 = aproximadamente R$ 55,17 (valor final arredondado para 2 casas decimais).

---

### Correção 6: Definir gestão de Grupos e Subgrupos

* **Como está no PRD original (US 2.1):**
  > "grupo 'Bebidas', subgrupo 'Refrigerantes'" — sem definição se é texto livre ou entidade.

* **Como deve ficar (Sugestão de Reescrita):** Adicionar ao Escopo do MVP:
  > * Cadastro de grupos e subgrupos de produtos como entidades separadas, permitindo que o Gerente crie e gerencie a hierarquia de categorias. O cadastro de produto deve selecionar grupo e subgrupo a partir de listas pré-definidas, garantindo consistência nos relatórios por categoria.

  E adicionar ao Épico 2:
  > **US 2.9:** Como Gerente, eu quero cadastrar grupos e subgrupos de produtos para que eu possa categorizar meu estoque e gerar relatórios por categoria.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Cadastro de grupo com subgrupos*
  >     * **Dado** que o Gerente acessa o cadastro de grupos
  >     * **Quando** ele cria o grupo "Bebidas" e adiciona os subgrupos "Refrigerantes", "Cervejas" e "Águas"
  >     * **Então** os grupos e subgrupos ficam disponíveis para seleção no cadastro de produtos e como filtro nos relatórios
  >   * *Cenário 2: Tentativa de excluir grupo com produtos vinculados*
  >     * **Dado** que o grupo "Bebidas" possui produtos cadastrados
  >     * **Quando** o Gerente tenta excluí-lo
  >     * **Então** o sistema exibe "Grupo possui X produtos vinculados — remova os vínculos primeiro" e impede a exclusão
  >   * *Cenário 3: Subgrupo duplicado no mesmo grupo*
  >     * **Dado** que o grupo "Bebidas" já possui o subgrupo "Refrigerantes"
  >     * **Quando** o Gerente tenta cadastrar "Refrigerantes" novamente no mesmo grupo
  >     * **Então** o sistema exibe "Subgrupo já existe neste grupo" e impede a duplicidade

---

### Correção 7: Declarar premissas de hardware e esclarecer nomenclatura

* **Como está no PRD original:** Não há seção de Premissas. A US 4.4 usa "escaneia" sem declarar necessidade de leitor de código de barras.

* **Como deve ficar (Sugestão de Reescrita):** Adicionar antes da seção de Escopo do MVP:

  > **Premissas de Hardware e Ambiente:**
  > * Computador com Windows 11, teclado, mouse e monitor
  > * Impressora A4 (jato de tinta ou laser) para emissão de notas de venda e relatórios
  > * Leitor de código de barras USB (recomendado, mas não obrigatório — o sistema aceita digitação manual do código)
  > * Dispositivo externo para processamento de transações eletrônicas — maquininha de cartão (POS) e/ou smartphone com app de banco para PIX — operado de forma independente do superVenda
  > * Mídia removível (pen drive ou HD externo) para backup dos dados

---

## 4. Próximos Passos

1. **Aplicar as 7 correções acima no PRD original** (`docs/fase2_prd.md`), com atenção especial às Correções 1 (PIX/Cartões offline), 2 (US para RF25 e RF31) e 3 (exclusão e edição de entidades), que são bloqueadoras para a engenharia.

2. **Revisar os 37 RFs para eliminar duplicidade** entre RF19 e RF37 (unificar em um único requisito de "Alerta de Estoque Mínimo no PDV").

3. **Rodar novamente a validação** (Fase 3) para confirmar que o Score de Prontidão atingiu ≥ 8.0.

4. **Após Score ≥ 8.0 confirmado**, o PRD estará apto para:
   - Envio para a Fase 4 (Patcher) para refinamento incremental, ou
   - Envio direto para Spec-Driven Development via OpenSpec (`/openspec-new-change`).

5. **Sobre as métricas:** Recomenda-se que a equipe de produto defina instrumentos de medição para as Métricas #3 (cronometragem durante o onboarding) e #6 (questionário simples de 3 perguntas após 30 dias) antes do lançamento, mas isso não bloqueia o desenvolvimento.
