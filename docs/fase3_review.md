# Relatório de Revisão de PRD — Controle de Qualidade

<auditoria_interna>

## Chain of Thought — Análise Crítica

### 1. Critérios de Aceitação (BDD)
O formato BDD (Dado/Quando/Então) está consistentemente aplicado em todo o documento — ponto positivo. A maioria das US possui de 2 a 4 cenários, cobrindo happy path e alguns caminhos tristes. No entanto, identifico fragilidades:

- **Cobertura superficial em ações de manutenção:** Nenhuma US cobre edição ou exclusão de registros mestres (produtos, clientes, fornecedores, usuários). O PRD só descreve criação (CRUD incompleto). Isso é grave: a engenharia vai precisar decidir sozinha se um produto com histórico de vendas pode ser excluído, por exemplo.

- **Cenários de borda omitidos:** Diversos cenários importantes não foram modelados:
  - Cancelamento de venda com carnê onde ALGUMAS parcelas já foram pagas (US 1.5 Cenário 3 só cobre "nenhuma parcela paga")
  - Cancelamento de venda de um dia anterior (caixa já fechado)
  - Venda com desconto de 100% ou desconto negativo
  - Bloqueio de conta após múltiplas tentativas de login (US 2.5)
  - Restauração de backup com caixa aberto no sistema atual (US 5.2)
  - Alteração de hardware que modifica o fingerprint da licença (ex: upgrade de RAM, HD) (US 5.3)

- **Cálculo de juros com precisão:** US 3.2 Cenário 2 apresenta R$ 0,17 de juros (0,033% × 10 dias × R$ 50,00). O valor exato seria R$ 0,165 — o arredondamento para R$ 0,17 está correto para 2 casas decimais, mas o total de R$ 55,17 está inconsistente (50,00 + 5,00 + 0,17 = 55,17 ✅). A fórmula está correta, mas a redação poderia ser mais precisa sobre o momento do arredondamento.

### 2. Coesão de Escopo (Requisitos Funcionais × Histórias de Usuário)

Fiz o cruzamento exaustivo de cada RF com as US. Os seguintes RFs NÃO possuem cobertura de História de Usuário:

| RF | Descrição | Situação |
|---|---|---|
| **RF13** | "O Gerente pode reverter o bloqueio manualmente após regularização" | ❌ Nenhuma US cobre o desbloqueio manual de cliente |
| **RF25** | "Relatório de Vendas por Forma de Pagamento" | ❌ Nenhum cenário BDD cobre este relatório específico, apenas citado no escopo |
| **RF36** | "Impedir execuções simultâneas no mesmo computador" | ❌ Nenhuma US ou cenário cobre esta funcionalidade |
| **RF11** | "número de parcelas é escolhido pelo cliente até um máximo parametrizável pelo Gerente" | ⚠️ Nenhuma US cobre a configuração do número máximo de parcelas |
| **RF37** | "Alerta visual de estoque mínimo durante a venda" | ⚠️ Apenas mencionado como nota lateral na US 2.1 Cenário 1, sem cenário BDD próprio |

Além disso, identifiquei ações de manutenção de dados que aparecem implicitamente em vários RFs (editar/excluir produtos, clientes, fornecedores) mas não possuem US dedicada. A engenharia ficará sem especificação para:
- Excluir produto com histórico de vendas (soft delete? impedir?)
- Excluir cliente com carnê ativo
- Editar limite de crédito de cliente após cadastro
- Alterar preço de produto (impacto em vendas já realizadas?)
- Excluir usuário (o que acontece com o log de auditoria que referencia esse usuário?)

### 3. Blindagem Tecnológica
O PRD está bem blindado. Não encontrei vazamento de stack tecnológica (nenhuma menção a banco de dados específico, linguagem de programação ou framework). As menções a "Windows 11", "formato A4" e "arquivo-chave" são restrições de plataforma/negócio, não decisões de arquitetura. ✅

### 4. Outras Vulnerabilidades

- **Recuperação de senha do Gerente:** Se o único Gerente perder a senha, como o sistema se recupera? O PRD é omisso. Em um sistema offline sem suporte remoto, isso pode significar perda total dos dados.
- **Métrica 5 não mensurável offline:** "validado pela execução bem-sucedida de ao menos 2 backups em dias diferentes" — com sistema 100% offline, quem mede isso e como?
- **Métrica 6 subjetiva:** "vendedor deve relatar que o sistema é 'fácil de usar'" é uma métrica puramente qualitativa sem instrumento de medição definido.
- **US 5.3 é narrada na perspectiva do Fornecedor,** mas o fluxo de trabalho real do Suporte Técnico (validação de solicitação, emissão de nova chave) não está especificado — não é uma US do sistema superVenda em si, é um processo externo.

### 5. Cálculo do Score

Contagem de falhas:
- **Graves (bloqueantes para engenharia):** 5 (RF13 sem US, RF36 sem US, max parcelas sem US, ausência de CRUD completo, cancelamento parcial de carnê)
- **Médias (causariam retrabalho):** 5 (RF25 sem cenário, bordas de login, bordas de backup com caixa aberto, bordas de licença com upgrade HW, senha perdida)
- **Leves (aprimoramentos):** 3 (precisão de arredondamento, métricas não mensuráveis offline, processo externo do suporte)

**Nota: 7/10**

O documento é bem estruturado e a maioria das US está bem escrita, mas as lacunas de cobertura (RFs órfãos, CRUD incompleto, cenários de borda críticos) impedem que a engenharia receba este PRD sem assumir decisões de produto por conta própria.

</auditoria_interna>

---

## 1. Veredito e Score de Prontidão
* **Score de Prontidão:** 7 / 10
* **Status:** REQUER REFATORAÇÃO (Score < 8)
* **Resumo:** O PRD está bem estruturado, com BDD consistentemente aplicado e boa separação de épicos. No entanto, foram identificadas 5 falhas graves de cobertura — Requisitos Funcionais (RFs) sem História de Usuário correspondente e operações de manutenção de dados (edição/exclusão) completamente ausentes. Essas lacunas forçariam a engenharia a tomar decisões de produto por conta própria (ex: o que acontece ao excluir um cliente com carnê ativo?), gerando retrabalho e inconsistências. O documento **não está pronto para ser entregue à engenharia** e requer uma rodada de correções focadas antes de seguir para OpenSpec.

---

## 2. Pontos Críticos Identificados

* **RF13 (Bloqueio de Cliente) sem cobertura de US:** O RF13 estabelece que "O Gerente pode reverter o bloqueio manualmente após regularização", mas não existe nenhuma História de Usuário ou cenário BDD que especifique o fluxo de desbloqueio manual de cliente. A engenharia não saberá em que tela isso ocorre, se exige senha, se gera log de auditoria.

* **RF36 (Proteção contra Execuções Simultâneas) sem cobertura de US:** O RF36 define que o sistema deve impedir múltiplas instâncias simultâneas no mesmo computador, mas não há nenhuma US que descreva o comportamento esperado quando isso ocorre (mensagem? bloqueio silencioso?).

* **Parâmetro "máximo de parcelas do carnê" sem US de configuração:** O RF11 menciona "número de parcelas é escolhido pelo cliente até um máximo parametrizável pelo Gerente", mas não existe US para o Gerente configurar esse máximo. Onde esse parâmetro é definido? Qual o valor default?

* **CRUD de dados mestres incompleto — apenas Create:** Nenhuma US cobre edição ou exclusão de produtos, clientes, fornecedores ou usuários. Isso é uma omissão grave. Exemplos de perguntas não respondidas:
  * Um produto que já teve vendas pode ser excluído? (soft delete ou bloqueio?)
  * Um cliente com parcelas de carnê em aberto pode ser excluído?
  * Ao alterar o preço de um produto, vendas já realizadas são impactadas?
  * Ao excluir um usuário, o que acontece com os registros de log que o referenciam?

* **Cancelamento de carnê com parcelas parcialmente pagas não especificado:** A US 1.5 Cenário 3 cobre apenas o cancelamento de carnê "onde nenhuma parcela foi paga ainda". O que acontece quando 1 ou 2 parcelas das 3 já foram pagas? O valor pago é estornado? O limite de crédito é restaurado parcialmente?

* **Recuperação de acesso do Gerente omitida:** Em um sistema 100% offline com apenas um Gerente cadastrado, a perda da senha do Gerente representa perda total de acesso administrativo. O PRD não prevê nenhum mecanismo de recuperação (pergunta de segurança, chave mestra de recuperação, arquivo de reset).

* **Restauração de backup com caixa aberto não tratada:** A US 5.2 não especifica o que acontece se o sistema atual estiver com um caixa aberto no momento da restauração. A restauração sobrescreve tudo? O sistema impede e exige fechar o caixa primeiro?

* **US 1.5 não cobre cancelamento de venda de dia anterior:** Se uma venda foi realizada ontem (caixa já fechado) e o cliente retorna hoje para devolução, o fluxo de cancelamento é o mesmo? Como fica o saldo do caixa do dia corrente?

---

## 3. Propostas de Correção Direta

### Correção 1: Adicionar US para Desbloqueio Manual de Cliente (cobre lacuna do RF13)
* **Como está no PRD original:** 
  > RF13 — Bloqueio de Cliente: "O Gerente pode reverter o bloqueio manualmente após regularização." (sem US correspondente)
* **Como deve ficar (Sugestão de Reescrita):**
  > **US 3.6:** Como Gerente, eu quero desbloquear manualmente um cliente que regularizou seus débitos para que ele volte a comprar no carnê.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Desbloqueio de cliente regularizado*
  >     * **Dado** que o cliente "João" está com situação "Bloqueado" por atraso e todas as suas parcelas foram pagas
  >     * **Quando** o Gerente acessa a ficha do cliente e seleciona "Desbloquear crédito"
  >     * **Então** a situação do cliente é alterada para "Liberado", o limite de crédito original é restaurado, e a ação é registrada em log com data/hora e identificação do Gerente
  >   * *Cenário 2: Tentativa de desbloqueio com parcelas ainda em aberto*
  >     * **Dado** que o cliente "João" está "Bloqueado" e ainda possui 2 parcelas vencidas não pagas
  >     * **Quando** o Gerente tenta desbloquear o cliente
  >     * **Então** o sistema exibe "Cliente possui parcelas em aberto — regularize antes de desbloquear" e impede a ação

### Correção 2: Adicionar US para Proteção contra Execuções Simultâneas (cobre lacuna do RF36)
* **Como está no PRD original:**
  > RF36 — Proteção contra Execuções Simultâneas: "Impedir que múltiplas instâncias do sistema sejam executadas simultaneamente no mesmo computador." (sem US correspondente)
* **Como deve ficar (Sugestão de Reescrita):**
  > **US 5.5:** Como Gerente, eu quero que o sistema impeça abrir duas vezes ao mesmo tempo para que os dados não sejam corrompidos por acesso duplicado.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Tentativa de abrir segunda instância*
  >     * **Dado** que o superVenda já está em execução no computador
  >     * **Quando** alguém tenta abrir o superVenda novamente (atalho na área de trabalho ou executável)
  >     * **Então** o sistema exibe a mensagem "O superVenda já está em execução" e a nova instância é encerrada sem alterar os dados da instância ativa

### Correção 3: Adicionar US para Configuração do Número Máximo de Parcelas do Carnê (cobre lacuna do RF11)
* **Como está no PRD original:**
  > RF11 — Carnê (Creditário): "O número de parcelas é escolhido pelo cliente até um máximo parametrizável pelo Gerente." (sem US de configuração)
* **Como deve ficar (Sugestão de Reescrita):**
  > **US 3.7:** Como Gerente, eu quero definir o número máximo de parcelas permitido no carnê para que eu controle o prazo máximo de parcelamento da loja.
  > * **Critérios de Aceitação:**
  >   * *Cenário 1: Definição do máximo de parcelas*
  >     * **Dado** que o Gerente acessa as configurações do carnê
  >     * **Quando** ele define o máximo de 6 parcelas
  >     * **Então** durante uma venda no carnê, o sistema limita a escolha do cliente a no máximo 6 parcelas
  >   * *Cenário 2: Valor mínimo de parcela*
  >     * **Dado** que o máximo de parcelas está definido como 12 e o valor total da venda é R$ 30,00
  >     * **Quando** o Vendedor tenta parcelar em 12 vezes (R$ 2,50 por parcela)
  >     * **Então** o sistema exibe "Valor mínimo da parcela: R$ 5,00" e limita automaticamente o número de parcelas ao máximo que respeite o piso (neste caso, 6 parcelas de R$ 5,00)
  >   * *Cenário 3: Venda com parcela única*
  >     * **Dado** que o máximo de parcelas é 12
  >     * **Quando** o cliente escolhe pagar o carnê em 1 parcela (à vista no carnê)
  >     * **Então** o sistema aceita e gera 1 parcela com vencimento em 30 dias, aplicando o preço carnê

### Correção 4: Adicionar cenário de cancelamento de carnê com parcelas parcialmente pagas (expande US 1.5)
* **Como está no PRD original:**
  > *Cenário 3: Cancelamento de venda com carnê*
  >   * **Dado** que uma venda foi realizada no carnê em 3 parcelas e nenhuma parcela foi paga ainda
  >   * **Quando** o Gerente cancela a venda
  >   * **Então** o carnê é cancelado, as parcelas são removidas do contas a receber, o limite de crédito do cliente é restaurado, e o estoque é devolvido
* **Como deve ficar (Sugestão de Reescrita):**
  > *Cenário 3: Cancelamento de venda com carnê sem parcelas pagas*
  >   * **Dado** que uma venda foi realizada no carnê em 3 parcelas de R$ 50,00 e nenhuma parcela foi paga ainda
  >   * **Quando** o Gerente cancela a venda
  >   * **Então** o carnê é cancelado, as 3 parcelas são removidas do contas a receber, o limite de crédito do cliente é restaurado em R$ 150,00, e o estoque é devolvido. A ação é registrada em log.
  > *Cenário 4: Cancelamento de venda com carnê e parcelas parcialmente pagas*
  >   * **Dado** que uma venda foi realizada no carnê em 3 parcelas de R$ 50,00 e a primeira parcela já foi paga
  >   * **Quando** o Gerente cancela a venda
  >   * **Então** as 2 parcelas não pagas são removidas do contas a receber, o limite de crédito do cliente é restaurado em R$ 100,00 (valor das parcelas não pagas), o estoque é devolvido integralmente, o valor já pago (R$ 50,00) permanece no caixa e é registrado como crédito para o cliente ou estornado conforme decisão do Gerente no ato do cancelamento. A ação é registrada em log com o destino do valor pago.

### Correção 5: Adicionar cenário de cancelamento de venda de dia anterior (expande US 1.5)
* **Como está no PRD original:**
  > (sem cenário para cancelamento de venda de dia anterior)
* **Como deve ficar (Sugestão de Reescrita):**
  > *Cenário 5: Cancelamento de venda de dia anterior (caixa já fechado)*
  >   * **Dado** que uma venda à vista de R$ 80,00 foi realizada ontem (caixa daquele dia já está fechado) e o cliente retorna hoje para devolver o produto
  >   * **Quando** o Gerente cancela a venda com sua senha
  >   * **Então** o estoque é devolvido, o valor de R$ 80,00 é deduzido do caixa do dia corrente (se houver saldo suficiente) ou registrado como saldo negativo, a venda é marcada como cancelada, e a ação é registrada em log com data/hora e identificação do Gerente. Se o saldo do caixa atual for insuficiente para o estorno, o sistema exibe "Saldo insuficiente para estorno — valor necessário: R$ 80,00" e pergunta se deseja prosseguir com saldo negativo.

---

## 4. Próximos Passos

* **Ação requerida:** O PRD atual (docs/fase2_prd.md) **não está aprovado** para entrega à engenharia. O Product Manager deve aplicar as 5 correções propostas acima e, adicionalmente:
  1. Adicionar US para edição e exclusão de registros mestres (produtos, clientes, fornecedores, usuários) com tratamento de dependências (ex: produto com vendas, cliente com carnê ativo)
  2. Adicionar cenário de recuperação de senha do Gerente (mecanismo offline)
  3. Adicionar cenário de restauração de backup com caixa aberto
  4. Especificar cenário BDD para RF25 (relatório de vendas por forma de pagamento)
  5. Tratar borda de alteração de hardware no licenciamento (upgrade de RAM/HD)

* **Ferramenta recomendada:** Após aplicar as correções, execute a **Fase 4 (PRD Patcher)** para gerar um diff consolidado das alterações e revalide o score. O PRD corrigido deve atingir score ≥ 8 para seguir para OpenSpec / SDD.

* **Fase 3.5 (PRD Patcher):** Caso deseje aplicar as correções automaticamente, utilize o comando `/prd_fase4_patcher` que lerá este relatório de auditoria e aplicará as alterações sugeridas diretamente no PRD original, gerando um novo documento `docs/fase2_prd_patched.md`.
