# Product Requirements Document (PRD) — SG-Varejo

## 1. Visão Geral e Problema

Pequenos varejistas brasileiros multi-segmento (mercados, vestuário, calçados, cosméticos, autopeças, depósitos, informática, materiais de construção, confecções e bijuterias) operam hoje sem um sistema único que centralize todo o seu ciclo operacional. Utilizam cadernos, planilhas soltas ou sistemas fragmentados, o que gera retrabalho, perda de informações e falta de visibilidade sobre a saúde financeira do negócio.

O SG-Varejo resolve essa dor com um sistema de gestão completo e off-line para o pequeno varejista: cadastros, ponto de venda (PDV) com leitor de código de barras, vendas a prazo com controle de crédito, compras via importação de XML de nota fiscal, controle de estoque com alertas, fluxo de caixa, contas a pagar/receber e relatórios gerenciais (incluindo DRE simplificado). Tudo isso operando em um único computador, sem dependência de internet, com banco de dados local e backup manual.

O objetivo do MVP é entregar um sistema funcional e confiável que permita ao lojista operar o dia a dia do seu negócio com autonomia total — da abertura do caixa pela manhã até o backup dos dados ao final do expediente.

## 2. Personas

* **Proprietário/Gerente Geral:** Dono do negócio. Tem acesso irrestrito ao sistema. Opera relatórios gerenciais (DRE, fluxo de caixa, lucro), gerencia cadastros mestres (empresa, fornecedores, formas de pagamento, taxas), define limites de crédito dos clientes, aprova exclusões e alterações críticas via senha de gerente, e é o único responsável por realizar backup e restauração dos dados. Sua principal dor é não saber exatamente quanto está ganhando ou perdendo, e não ter controle sobre quem deve e quando pagar.

* **Vendedor/Atendente de Balcão:** Opera o dia a dia das vendas. Abre e fecha o caixa, consulta produtos (por código, descrição ou referência), verifica a situação do cliente (limite de crédito disponível, atrasos), registra vendas à vista e a prazo com até duas formas de pagamento, imprime notas de venda A4. Tem acesso restrito: pode ter limite máximo de desconto, não acessa relatórios gerenciais e não pode excluir ou alterar vendas já finalizadas sem senha de gerente. Sua dor é a lentidão no atendimento quando precisa consultar múltiplas fontes para fechar uma venda.

* **Financeiro/Contas:** Responsável pela retaguarda financeira. Lança contas a pagar e a receber, dá baixas, registra sangrias de caixa e acompanha o fluxo de caixa. Não necessariamente opera vendas, mas precisa de visibilidade sobre o que entrou e saiu. Sua dor é a falta de um registro centralizado do que está pendente de pagamento ou recebimento, o que gera atrasos e perda de prazos.

* **Estoquista/Comprador:** Responsável pelo cadastro e manutenção de produtos (foto, código de barras, grupo/subgrupo, preço normal e preço alternativo), importação de compras via XML de nota fiscal, importação de produtos via planilha Excel, ajuste de estoque e emissão de etiquetas (pós-MVP). Sua dor é o tempo gasto para cadastrar produtos um a um e a falta de visibilidade sobre o que está acabando no estoque.

## 3. Escopo do MVP

* **Dentro do Escopo:**
  * Cadastro completo de Empresa, Produtos (com foto, código de barras, grupo/subgrupo, preço normal e preço 2, validade), Clientes (com foto, limite de crédito), Fornecedores, Usuários (com perfis de acesso) e Formas de Pagamento (com taxa configurável)
  * PDV completo: abertura e fechamento de caixa com valor inicial, registro de vendas (consulta de produtos por código/descrição/referência, leitura de código de barras, seleção de cliente, aplicação de preço 2, desconto controlado), finalização com até 2 formas de pagamento por venda, impressão de nota de venda em A4 com logotipo da empresa
  * Vendas a prazo (carnê) com controle de limite de crédito por cliente e acréscimo configurável (taxa fixa ou percentual) conforme número de parcelas
  * Controle de estoque com alerta de estoque mínimo e bloqueio opcional de vendas de produtos sem estoque
  * Importação de compras via arquivo XML de nota fiscal, com atualização automática de estoque e preços
  * Backup e restauração manuais dos dados, com validação de integridade e prevenção de sobrescrita acidental
  * Senha de gerente obrigatória para exclusão e alteração de vendas, preços e produtos
  * Fluxo de caixa detalhado e resumido (entradas, saídas, saldo)
  * Lançamento e baixa de contas a pagar e contas a receber
  * Sangria de caixa
  * DRE simplificado: total de vendas menos total de compras, com visão diária, mensal e anual
  * Relatórios: vendas (por vendedor, período, cliente, produto, categoria, forma de pagamento, margem de lucro, produtos mais vendidos), compras (por período, fornecedor, produto), estoque (posição atual, estoque mínimo, validade), financeiro (contas a pagar/receber/pagas/recebidas, fluxo de caixa)
  * Consultas: produtos (código, descrição, referência), situação do cliente (limite, atrasos, histórico de vendas, bloqueios), aniversariantes, estoque financeiro, informativos de contas a pagar e a receber
  * Importação de produtos e clientes via planilha Excel
  * Licenciamento por arquivo-chave (anti-pirataria), vinculado à instalação
  * Controle de sessão: login/logout com múltiplos perfis, operação mono-terminal (1 pessoa por vez)

* **Fora de Escopo:**
  * Emissão de documentos fiscais eletrônicos (NFC-e, NF-e, NFS-e)
  * Operação em rede / múltiplos terminais (PDVs simultâneos)
  * Emissão de recibos avulsos
  * Emissão de cartas de cobrança para clientes
  * Impressão de etiquetas de produtos para código de barras
  * Módulo de conferência e inventário físico de estoque
  * Gráfico visual de vendas (substituído por relatório tabular no MVP)
  * Gestão de comissão sobre vendas
  * Alteração de preço em massa
  * Integração com sistemas externos (e-commerce, marketplaces, ERPs)
  * Funcionalidades que dependam de conexão com internet
  * Relatório de margem de lucro consolidado com rateio de despesas fixas (além do DRE simplificado)

## 4. Histórias de Usuário e Critérios de Aceitação

### Épico 1: Fundação — Cadastros e Configurações

* **US 1.1:** Como Proprietário, eu quero cadastrar os dados da minha empresa (incluindo logotipo) para que as notas de venda e relatórios sejam personalizados com a identidade do meu negócio.
  * **Critérios de Aceitação:**
    * *Cenário 1: Primeiro acesso ao sistema*
      * **Dado** que o sistema foi instalado e ativado com sucesso
      * **Quando** o proprietário acessa o sistema pela primeira vez e preenche os dados da empresa (nome, endereço, telefone) e carrega uma imagem de logotipo
      * **Então** os dados são persistidos e o logotipo aparece em todas as impressões de notas de venda e relatórios
    * *Cenário 2: Alteração de dados da empresa*
      * **Dado** que a empresa já está cadastrada e o proprietário está autenticado
      * **Quando** o proprietário altera qualquer campo dos dados da empresa e salva
      * **Então** as alterações são aplicadas imediatamente a todas as novas impressões, sem afetar documentos já emitidos

* **US 1.2:** Como Estoquista, eu quero cadastrar produtos com foto, código de barras, grupo/subgrupo, preço normal, preço 2 e validade para que o catálogo de produtos esteja completo e pronto para venda.
  * **Critérios de Aceitação:**
    * *Cenário 1: Cadastro completo de um produto*
      * **Dado** que o estoquista está autenticado e na tela de cadastro de produtos
      * **Quando** ele preenche descrição, código de barras, seleciona grupo e subgrupo, define preço normal, preço 2, estoque inicial, estoque mínimo, data de validade e carrega uma foto, então salva
      * **Então** o produto fica disponível para consulta e venda no PDV, e aparece nos relatórios de estoque
    * *Cenário 2: Código de barras duplicado*
      * **Dado** que já existe um produto cadastrado com o código de barras "7891234567890"
      * **Quando** o estoquista tenta cadastrar outro produto com o mesmo código de barras
      * **Então** o sistema rejeita o cadastro e exibe mensagem "Código de barras já cadastrado para o produto [nome do produto]"

* **US 1.3:** Como Proprietário, eu quero cadastrar clientes com foto e limite de crédito para que o sistema controle automaticamente as vendas a prazo.
  * **Critérios de Aceitação:**
    * *Cenário 1: Cadastro de cliente com limite de crédito*
      * **Dado** que o usuário está na tela de cadastro de clientes
      * **Quando** ele preenche nome, CPF/CNPJ, endereço, telefone, define um limite de crédito de R$ 500,00, carrega uma foto e salva
      * **Então** o cliente fica disponível no PDV e o limite de R$ 500,00 é aplicado nas vendas a prazo
    * *Cenário 2: Cliente sem limite de crédito*
      * **Dado** que um cliente está cadastrado com limite de crédito igual a zero
      * **Quando** o vendedor tenta finalizar uma venda a prazo para este cliente
      * **Então** o sistema bloqueia a venda e exibe "Cliente sem limite de crédito disponível"

* **US 1.4:** Como Proprietário, eu quero cadastrar usuários com login, senha e restrições de acesso para que cada funcionário acesse apenas as funções adequadas ao seu cargo.
  * **Critérios de Aceitação:**
    * *Cenário 1: Criação de usuário vendedor*
      * **Dado** que o proprietário está autenticado e na tela de cadastro de usuários
      * **Quando** ele cria um novo usuário com perfil "Vendedor", define login e senha, e restringe desconto máximo a 5%
      * **Então** esse usuário consegue fazer login, operar o PDV, mas não acessa relatórios gerenciais e não pode conceder descontos acima de 5%
    * *Cenário 2: Senha de gerente para operações críticas*
      * **Dado** que um vendedor está logado e tenta excluir uma venda já finalizada
      * **Quando** o sistema solicita a senha de gerente
      * **Então** a exclusão só é concluída se a senha de um usuário com perfil de gerente/proprietário for fornecida; caso contrário, a operação é cancelada

* **US 1.5:** Como Proprietário, eu quero cadastrar formas de pagamento com taxas configuráveis e definir se aparecem no fluxo de caixa para que o cálculo financeiro reflita a realidade das operadoras de cartão e outros meios.
  * **Critérios de Aceitação:**
    * *Cenário 1: Cadastro de forma de pagamento com taxa*
      * **Dado** que o proprietário está na tela de formas de pagamento
      * **Quando** ele cadastra "Cartão de Crédito" com taxa de 3% e marca "mostrar no fluxo de caixa"
      * **Então** toda venda finalizada com esta forma de pagamento tem o valor líquido calculado automaticamente (valor bruto - 3%) no fluxo de caixa
    * *Cenário 2: Forma de pagamento sem impacto no fluxo de caixa*
      * **Dado** que "Vale-Presente" está cadastrado com a opção "mostrar no fluxo de caixa" desmarcada
      * **Quando** uma venda é finalizada com Vale-Presente
      * **Então** o valor não aparece nas entradas do fluxo de caixa

### Épico 2: Operação de Vendas — PDV

* **US 2.1:** Como Vendedor, eu quero abrir o caixa com um valor inicial e fechar o caixa ao final do expediente para que haja controle exato do dinheiro que entrou e saiu durante o dia.
  * **Critérios de Aceitação:**
    * *Cenário 1: Abertura de caixa*
      * **Dado** que o vendedor está autenticado e o caixa está fechado
      * **Quando** ele informa o valor de troco inicial (ex.: R$ 100,00) e confirma a abertura
      * **Então** o sistema registra a abertura com data/hora, valor inicial, e o PDV fica liberado para vendas. O caixa aparece como "Aberto" no sistema
    * *Cenário 2: Tentativa de venda com caixa fechado*
      * **Dado** que o caixa não foi aberto no dia
      * **Quando** o vendedor tenta iniciar uma venda
      * **Então** o sistema bloqueia a operação e exibe "Caixa fechado. Abra o caixa antes de iniciar vendas"
    * *Cenário 3: Fechamento de caixa*
      * **Dado** que o caixa está aberto e houve vendas, sangrias e movimentações no dia
      * **Quando** o vendedor aciona o fechamento de caixa
      * **Então** o sistema exibe um resumo do dia (total de vendas por forma de pagamento, sangrias, saldo final esperado) e, após confirmação, fecha o caixa, impedindo novas vendas até a próxima abertura

* **US 2.2:** Como Vendedor, eu quero consultar produtos por código, descrição ou referência e adicioná-los a uma venda para que o atendimento no balcão seja rápido e preciso.
  * **Critérios de Aceitação:**
    * *Cenário 1: Busca de produto por código de barras*
      * **Dado** que o vendedor está na tela de venda (PDV) com o caixa aberto
      * **Quando** ele utiliza o leitor de código de barras para ler o código "7891234567890"
      * **Então** o produto correspondente é localizado e adicionado automaticamente ao carrinho com quantidade 1, exibindo descrição, preço unitário e subtotal
    * *Cenário 2: Busca por descrição parcial*
      * **Dado** que o vendedor está na tela de venda e digita "arroz" no campo de busca
      * **Quando** o sistema retorna todos os produtos cuja descrição contém "arroz"
      * **Então** o vendedor seleciona o produto desejado da lista e ele é adicionado ao carrinho
    * *Cenário 3: Produto sem estoque e bloqueio ativado*
      * **Dado** que o controle de estoque com bloqueio está ativado e o produto "Sabão em Pó" tem estoque zero
      * **Quando** o vendedor tenta adicionar este produto à venda
      * **Então** o sistema exibe "Produto sem estoque disponível" e não permite a adição

* **US 2.3:** Como Vendedor, eu quero consultar a situação do cliente durante a venda para saber se ele tem limite de crédito disponível ou se está com pagamentos em atraso.
  * **Critérios de Aceitação:**
    * *Cenário 1: Cliente com limite disponível e sem atrasos*
      * **Dado** que o vendedor seleciona um cliente na venda
      * **Quando** o sistema carrega a situação do cliente
      * **Então** são exibidos: nome, saldo devedor atual, limite de crédito total, limite disponível e status "Regular"
    * *Cenário 2: Cliente bloqueado por atraso*
      * **Dado** que o cliente "João" possui parcelas vencidas há mais de 30 dias
      * **Quando** o vendedor seleciona "João" na venda a prazo
      * **Então** o sistema exibe "Cliente com pagamentos em atraso" e bloqueia a finalização da venda a prazo, mas ainda permite venda à vista

* **US 2.4:** Como Vendedor, eu quero finalizar uma venda com até 2 formas de pagamento para que o cliente possa, por exemplo, pagar parte em dinheiro e parte no cartão.
  * **Critérios de Aceitação:**
    * *Cenário 1: Venda à vista com duas formas de pagamento*
      * **Dado** que o carrinho possui R$ 150,00 em produtos
      * **Quando** o vendedor seleciona R$ 100,00 em "Dinheiro" e R$ 50,00 em "Cartão de Débito", e finaliza a venda
      * **Então** a venda é registrada, o estoque é baixado, o fluxo de caixa registra R$ 100,00 (dinheiro) e R$ 50,00 (cartão de débito, com taxa aplicada se configurada), e a nota de venda A4 é gerada para impressão
    * *Cenário 2: Soma das formas de pagamento não cobre o total*
      * **Dado** que o total da venda é R$ 200,00
      * **Quando** o vendedor informa R$ 150,00 em dinheiro e R$ 30,00 em cartão (total R$ 180,00)
      * **Então** o sistema não permite finalizar e exibe "Valor insuficiente. Restam R$ 20,00 a pagar"

* **US 2.5:** Como Vendedor, eu quero realizar vendas a prazo (carnê) com número de parcelas e acréscimo configurável para que o cliente possa comprar parcelado dentro do seu limite de crédito.
  * **Critérios de Aceitação:**
    * *Cenário 1: Venda a prazo com acréscimo percentual*
      * **Dado** que o cliente "Maria" tem limite disponível de R$ 600,00, e o sistema está configurado com acréscimo de 5% para 3 parcelas
      * **Quando** o vendedor seleciona venda a prazo em 3 parcelas para um carrinho de R$ 200,00
      * **Então** o sistema calcula o valor final de R$ 210,00 (R$ 200,00 + 5%), divide em 3 parcelas de R$ 70,00, gera as parcelas no contas a receber e reduz o limite disponível de Maria para R$ 390,00
    * *Cenário 2: Venda a prazo acima do limite de crédito*
      * **Dado** que o cliente "José" tem limite disponível de R$ 100,00
      * **Quando** o vendedor tenta finalizar uma venda a prazo de R$ 150,00
      * **Então** o sistema bloqueia a venda e exibe "Limite de crédito insuficiente. Disponível: R$ 100,00"

* **US 2.6:** Como Vendedor, eu quero poder selecionar entre o preço normal e o preço 2 (atacado/prazo) durante a venda para aplicar o preço adequado a cada situação.
  * **Critérios de Aceitação:**
    * *Cenário 1: Aplicação do preço 2 na venda*
      * **Dado** que o produto "Camisa" tem preço normal de R$ 50,00 e preço 2 de R$ 40,00
      * **Quando** o vendedor adiciona o produto ao carrinho e seleciona a opção "Preço 2"
      * **Então** o preço aplicado é R$ 40,00 e o sistema identifica visualmente que o preço alternativo está em uso

### Épico 3: Gestão Financeira

* **US 3.1:** Como Financeiro, eu quero lançar contas a pagar e a receber e registrar suas baixas para que o sistema reflita com precisão as obrigações e direitos financeiros do negócio.
  * **Critérios de Aceitação:**
    * *Cenário 1: Lançamento de conta a pagar*
      * **Dado** que o financeiro está autenticado e na tela de contas a pagar
      * **Quando** ele lança uma conta de R$ 500,00 com vencimento em 15 dias, informando fornecedor, número do documento e data de vencimento
      * **Então** a conta aparece nos relatórios de contas a pagar e no informativo de vencimentos
    * *Cenário 2: Baixa de conta a receber*
      * **Dado** que o cliente "Maria" possui uma parcela de R$ 70,00 vencendo hoje
      * **Quando** o financeiro registra o pagamento dessa parcela
      * **Então** a parcela é marcada como paga, o limite de crédito de Maria é restaurado em R$ 70,00, e o valor entra no fluxo de caixa

* **US 3.2:** Como Proprietário, eu quero visualizar o fluxo de caixa detalhado e resumido para saber exatamente quanto dinheiro entrou, saiu e qual o saldo em qualquer período.
  * **Critérios de Aceitação:**
    * *Cenário 1: Fluxo de caixa resumido do dia*
      * **Dado** que houve vendas, sangrias e baixas no dia
      * **Quando** o proprietário acessa o fluxo de caixa resumido com filtro do dia atual
      * **Então** o sistema exibe: saldo inicial (abertura de caixa), total de entradas (vendas + recebimentos), total de saídas (sangrias + pagamentos), e saldo final, com abertura por forma de pagamento
    * *Cenário 2: Fluxo de caixa detalhado mensal*
      * **Dado** que o mês corrente possui múltiplas movimentações
      * **Quando** o proprietário acessa o fluxo de caixa detalhado com filtro do mês
      * **Então** o sistema exibe cada movimentação em ordem cronológica com data, descrição, valor, tipo (entrada/saída) e saldo acumulado linha a linha

* **US 3.3:** Como Proprietário, eu quero visualizar um DRE simplificado (total de vendas menos total de compras) para entender o lucro diário, mensal e anual do negócio.
  * **Critérios de Aceitação:**
    * *Cenário 1: DRE mensal*
      * **Dado** que no mês corrente o total de vendas foi R$ 50.000,00 e o total de compras foi R$ 30.000,00
      * **Quando** o proprietário acessa o DRE com filtro do mês corrente
      * **Então** o sistema exibe: Receita Bruta (vendas) = R$ 50.000,00, CMV (compras) = R$ 30.000,00, Lucro Bruto = R$ 20.000,00, e Margem = 40%

* **US 3.4:** Como Vendedor ou Financeiro, eu quero registrar uma sangria de caixa para que retiradas de dinheiro do caixa durante o expediente sejam registradas e o saldo permaneça correto.
  * **Critérios de Aceitação:**
    * *Cenário 1: Registro de sangria*
      * **Dado** que o caixa está aberto com saldo atual de R$ 500,00 em dinheiro
      * **Quando** o usuário registra uma sangria de R$ 200,00 com o motivo "Depósito bancário"
      * **Então** o saldo de caixa em dinheiro é reduzido para R$ 300,00, a sangria aparece no fluxo de caixa como saída, e exige senha de gerente se a configuração assim determinar

### Épico 4: Compras, Estoque e Relatórios

* **US 4.1:** Como Estoquista, eu quero importar produtos de um arquivo XML de nota fiscal de compra para que o estoque e os preços sejam atualizados automaticamente sem redigitação.
  * **Critérios de Aceitação:**
    * *Cenário 1: Importação de XML válido*
      * **Dado** que o estoquista está na tela de compras
      * **Quando** ele seleciona um arquivo XML de nota fiscal de compra contendo 10 produtos, e confirma a importação
      * **Então** os 10 produtos são localizados no cadastro (ou criados se novos), o estoque é incrementado com as quantidades do XML, os preços de custo são atualizados, e a compra fica registrada nos relatórios de compras por fornecedor e período
    * *Cenário 2: XML inválido ou corrompido*
      * **Dado** que o estoquista seleciona um arquivo que não é um XML de NF-e válido
      * **Quando** o sistema tenta processar o arquivo
      * **Então** o sistema rejeita o arquivo e exibe "Arquivo inválido. Selecione um XML de Nota Fiscal eletrônica (NF-e)"

* **US 4.2:** Como Estoquista, eu quero importar produtos e clientes via planilha Excel para migrar rapidamente dados de outros sistemas ou fazer cadastros em lote.
  * **Critérios de Aceitação:**
    * *Cenário 1: Importação de produtos via Excel*
      * **Dado** que o estoquista possui uma planilha Excel com 50 produtos (colunas: descrição, código de barras, grupo, preço, estoque)
      * **Quando** ele seleciona o arquivo e confirma a importação
      * **Então** os 50 produtos são cadastrados, e o sistema exibe um resumo: "50 produtos importados com sucesso, 0 com erro"
    * *Cenário 2: Planilha com dados inconsistentes*
      * **Dado** que a planilha Excel contém linhas sem descrição ou com código de barras duplicado
      * **Quando** o estoquista tenta importar
      * **Então** o sistema importa apenas as linhas válidas e exibe um relatório: "45 produtos importados, 5 linhas com erro (linha 3: descrição vazia, linha 7: código de barras duplicado, ...)"

* **US 4.3:** Como Proprietário, eu quero que o sistema me alerte sobre produtos com estoque abaixo do mínimo para que eu possa fazer novos pedidos antes de ficar sem mercadoria.
  * **Critérios de Aceitação:**
    * *Cenário 1: Alerta de estoque mínimo*
      * **Dado** que o produto "Arroz 5kg" tem estoque mínimo configurado como 10 unidades e estoque atual de 8
      * **Quando** qualquer usuário acessa a tela inicial do sistema
      * **Então** o sistema exibe um alerta visível listando "Arroz 5kg — Estoque: 8 (Mínimo: 10)" entre os produtos em situação crítica

* **US 4.4:** Como Proprietário, eu quero gerar relatórios de vendas, compras, estoque e financeiro com filtros por período, pessoa e categoria para ter visibilidade completa sobre o desempenho do negócio.
  * **Critérios de Aceitação:**
    * *Cenário 1: Relatório de vendas por período e vendedor*
      * **Dado** que o proprietário acessa a seção de relatórios de vendas
      * **Quando** ele seleciona o filtro "últimos 30 dias" e "Vendedor: Carlos", e gera o relatório
      * **Então** o sistema exibe todas as vendas de Carlos nos últimos 30 dias com data, cliente, valor total, forma de pagamento e produtos vendidos, com totais ao final
    * *Cenário 2: Relatório de estoque mínimo*
      * **Dado** que existem 3 produtos com estoque abaixo do mínimo
      * **Quando** o proprietário gera o relatório de estoque mínimo
      * **Então** o relatório lista apenas esses 3 produtos com estoque atual, estoque mínimo, grupo e sugestão de quantidade a comprar (mínimo - atual)
    * *Cenário 3: Relatório de contas a receber vencidas*
      * **Dado** que existem 5 parcelas vencidas de 3 clientes diferentes
      * **Quando** o financeiro gera o relatório de contas a receber com status "Vencidas"
      * **Então** o relatório lista as 5 parcelas com cliente, valor, data de vencimento, dias de atraso e valor acumulado por cliente

### Épico 5: Segurança, Administração e Resiliência

* **US 5.1:** Como Proprietário, eu quero realizar backup manual dos dados para um local de minha escolha (pendrive, HD externo, outra pasta) para proteger meu negócio contra falhas de hardware.
  * **Critérios de Aceitação:**
    * *Cenário 1: Backup bem-sucedido*
      * **Dado** que o proprietário está autenticado e na tela de backup
      * **Quando** ele seleciona uma pasta de destino (ex.: "E:\backup\") e aciona "Gerar Backup"
      * **Então** o sistema gera um arquivo de backup íntegro com data/hora no nome, exibe "Backup gerado com sucesso em E:\backup\sg-varejo_2026-06-17_15-30.bak" e registra a operação no log do sistema
    * *Cenário 2: Destino sem espaço ou inacessível*
      * **Dado** que o destino selecionado não tem espaço suficiente ou está inacessível
      * **Quando** o proprietário tenta gerar o backup
      * **Então** o sistema exibe uma mensagem clara informando o problema ("Espaço insuficiente no destino" ou "Destino inacessível") e não gera um arquivo de backup parcial ou corrompido

* **US 5.2:** Como Proprietário, eu quero restaurar um backup anterior para recuperar meus dados em caso de troca de computador, formatação ou falha de hardware.
  * **Critérios de Aceitação:**
    * *Cenário 1: Restauração bem-sucedida*
      * **Dado** que o sistema está instalado e o proprietário possui um arquivo de backup válido
      * **Quando** ele acessa a função de restauração, seleciona o arquivo de backup e confirma a operação
      * **Então** o sistema restaura todos os dados (cadastros, vendas, financeiro, configurações) exatamente como estavam no momento do backup, e exige nova autenticação após a restauração
    * *Cenário 2: Arquivo de backup inválido ou corrompido*
      * **Dado** que o proprietário seleciona um arquivo que não é um backup válido
      * **Quando** o sistema tenta restaurar
      * **Então** o sistema rejeita o arquivo, exibe "Arquivo de backup inválido ou corrompido" e NÃO altera os dados atuais

* **US 5.3:** Como Fornecedor do sistema (Natural Tecnologia), eu quero que o sistema utilize licenciamento por arquivo-chave de uso único para que cada instalação seja devidamente licenciada e o uso não autorizado seja prevenido.
  * **Critérios de Aceitação:**
    * *Cenário 1: Ativação com arquivo-chave válido*
      * **Dado** que o sistema foi instalado e está no primeiro acesso
      * **Quando** o usuário carrega um arquivo-chave válido fornecido pela Natural Tecnologia
      * **Então** o sistema é ativado, o arquivo-chave é consumido (não pode ser reutilizado em outra instalação), e o sistema fica operacional
    * *Cenário 2: Arquivo-chave já utilizado*
      * **Dado** que um arquivo-chave já foi consumido em outra instalação
      * **Quando** alguém tenta usar o mesmo arquivo-chave em uma segunda instalação
      * **Então** o sistema rejeita a ativação e exibe "Arquivo-chave inválido ou já utilizado. Entre em contato com o suporte"
    * *Cenário 3: Reinstalação por falha de hardware*
      * **Dado** que um cliente teve falha de hardware e precisa reinstalar o sistema no mesmo computador (após reparo) ou em um novo computador
      * **Quando** o cliente entra em contato com o suporte da Natural Tecnologia para solicitar reativação
      * **Então** o processo de reativação deve ser possível mediante verificação e fornecimento de novo arquivo-chave, sem perda dos dados restaurados via backup

* **US 5.4:** Como Proprietário, eu quero que o sistema gerencie sessões de usuário (login/logout) para que múltiplos funcionários possam se alternar no mesmo terminal, cada um com seu próprio perfil de acesso.
  * **Critérios de Aceitação:**
    * *Cenário 1: Login bem-sucedido*
      * **Dado** que o sistema está na tela de login e o caixa está fechado
      * **Quando** o vendedor "Carlos" informa seu login e senha corretos
      * **Então** o sistema autentica Carlos, carrega seu perfil de acesso, e exibe as funcionalidades permitidas para o perfil "Vendedor"
    * *Cenário 2: Troca de usuário*
      * **Dado** que o vendedor Carlos está logado
      * **Quando** Carlos aciona "Trocar Usuário" e o proprietário faz login com suas credenciais
      * **Então** a sessão de Carlos é encerrada, a sessão do proprietário é iniciada com acesso irrestrito, e as operações pendentes de Carlos são preservadas (ex.: venda em andamento é mantida)
    * *Cenário 3: Credenciais inválidas*
      * **Dado** que o sistema está na tela de login
      * **Quando** um usuário informa login ou senha incorretos 3 vezes consecutivas
      * **Então** o sistema bloqueia temporariamente novas tentativas por 5 minutos e registra a ocorrência no log de segurança

## 5. Requisitos Funcionais

1. **RF01 — Cadastro de Empresa:** O sistema deve permitir o cadastro e edição dos dados da empresa (razão social, nome fantasia, CNPJ, endereço, telefone) e upload de logotipo para uso em impressões.
2. **RF02 — Cadastro de Produtos:** O sistema deve permitir cadastro de produtos com: descrição, código de barras (único), grupo e subgrupo, preço normal, preço 2, estoque inicial e mínimo, data de validade e foto. O código de barras deve ser validado contra duplicidade.
3. **RF03 — Cadastro de Clientes:** O sistema deve permitir cadastro de clientes com: nome, CPF/CNPJ, endereço, telefone, foto e limite de crédito para vendas a prazo.
4. **RF04 — Cadastro de Fornecedores:** O sistema deve permitir cadastro de fornecedores com: razão social, CNPJ, endereço, telefone e contato.
5. **RF05 — Cadastro de Usuários:** O sistema deve permitir cadastro de usuários com login, senha e perfil de acesso (com restrições configuráveis por funcionalidade). Deve suportar no mínimo os perfis: Proprietário/Gerente, Vendedor, Financeiro e Estoquista.
6. **RF06 — Cadastro de Formas de Pagamento:** O sistema deve permitir cadastro de formas de pagamento com taxa configurável (percentual) e flag para inclusão/exclusão do fluxo de caixa.
7. **RF07 — Controle de Sessão:** O sistema deve exigir autenticação por login e senha. Deve permitir troca de usuário sem fechar o sistema, preservando operações em andamento. Deve bloquear temporariamente após 3 tentativas consecutivas de login com credenciais inválidas.
8. **RF08 — Abertura e Fechamento de Caixa:** O sistema deve exigir abertura de caixa (com valor de troco inicial) antes de permitir vendas. O fechamento deve exibir resumo de movimentações do dia e, após confirmação, impedir novas vendas.
9. **RF09 — PDV — Busca e Adição de Produtos:** O sistema deve permitir busca de produtos por código de barras (leitor), código interno, descrição parcial e referência. A adição ao carrinho deve respeitar o controle de estoque (bloqueio opcional).
10. **RF10 — PDV — Consulta de Cliente:** O sistema deve permitir associar um cliente à venda e exibir sua situação: saldo devedor, limite de crédito, limite disponível, status (regular/atrasado/bloqueado).
11. **RF11 — PDV — Aplicação de Preço 2:** O sistema deve permitir que o vendedor selecione manualmente entre preço normal e preço 2 para cada item da venda, com indicação visual clara de qual preço está sendo aplicado.
12. **RF12 — PDV — Desconto:** O sistema deve permitir aplicação de desconto (percentual ou valor) respeitando o limite máximo configurado no perfil do usuário. Descontos acima do limite exigem senha de gerente.
13. **RF13 — PDV — Finalização com Até 2 Formas de Pagamento:** O sistema deve permitir finalizar a venda com até 2 formas de pagamento, validando que a soma cobre o valor total. Deve calcular automaticamente as taxas associadas a cada forma de pagamento.
14. **RF14 — Venda a Prazo (Carnê):** O sistema deve permitir venda parcelada com número de parcelas definido pelo vendedor. Deve aplicar acréscimo configurável (taxa fixa ou percentual) conforme número de parcelas. Deve validar o limite de crédito do cliente antes de finalizar. As parcelas devem ser geradas automaticamente no contas a receber.
15. **RF15 — Bloqueio de Venda a Prazo por Inadimplência:** O sistema deve bloquear vendas a prazo para clientes com parcelas em atraso (configurável: dias de atraso para bloqueio). Vendas à vista permanecem permitidas.
16. **RF16 — Senha de Gerente:** O sistema deve exigir senha de usuário com perfil de gerente/proprietário para: exclusão de vendas, alteração de vendas já finalizadas, alteração de preços de produtos, exclusão de produtos e concessão de descontos acima do limite do vendedor.
17. **RF17 — Impressão de Nota de Venda A4:** O sistema deve gerar uma nota de venda não fiscal em formato A4 contendo: logotipo da empresa, dados da empresa, data/hora, número da venda, vendedor, cliente (se informado), itens (descrição, quantidade, preço unitário, subtotal), total, formas de pagamento, e mensagem de rodapé.
18. **RF18 — Importação de XML de NF-e:** O sistema deve permitir importação de arquivo XML de nota fiscal eletrônica de compra, atualizando estoque e preço de custo dos produtos. Deve validar a estrutura do XML antes do processamento.
19. **RF19 — Importação via Planilha Excel:** O sistema deve permitir importação de produtos e clientes via arquivo Excel (.xlsx, .xls), com validação linha a linha e relatório de erros ao final.
20. **RF20 — Controle de Estoque:** O sistema deve decrementar estoque automaticamente a cada venda finalizada e incrementar a cada compra importada. Deve exibir alerta visível na tela inicial quando produtos estiverem abaixo do estoque mínimo. Deve suportar bloqueio opcional de vendas de produtos com estoque zero.
21. **RF21 — Fluxo de Caixa:** O sistema deve registrar automaticamente todas as entradas (vendas, recebimentos) e saídas (sangrias, pagamentos), consolidando saldo por forma de pagamento. Deve oferecer visões detalhada (linha a linha) e resumida (consolidada por período).
22. **RF22 — Contas a Pagar e Receber:** O sistema deve permitir lançamento manual de contas a pagar e a receber, e gerar automaticamente parcelas no contas a receber oriundas de vendas a prazo. Deve permitir baixa (total) de cada parcela.
23. **RF23 — Sangria:** O sistema deve permitir registro de sangria de caixa com valor, motivo e data/hora, deduzindo o valor do saldo de caixa em dinheiro.
24. **RF24 — DRE Simplificado:** O sistema deve calcular e exibir o lucro bruto do período (total de vendas - total de compras) com visão diária, mensal e anual.
25. **RF25 — Relatórios:** O sistema deve gerar relatórios para todos os módulos (vendas, compras, estoque, financeiro) com filtros configuráveis (período, pessoa, categoria, forma de pagamento). Deve exibir totais e subtotais em cada relatório.
26. **RF26 — Consultas Rápidas:** O sistema deve oferecer consultas de: produtos (por código, descrição, referência), clientes (situação, histórico de vendas, atrasos, bloqueios), aniversariantes do mês, estoque financeiro, informativo de contas a pagar e a receber.
27. **RF27 — Backup Manual:** O sistema deve permitir que o usuário gere uma cópia de segurança completa dos dados em local de sua escolha, validando a integridade do arquivo gerado e impedindo sobrescrita acidental de backups anteriores.
28. **RF28 — Restauração de Backup:** O sistema deve permitir restaurar completamente os dados a partir de um arquivo de backup válido, com verificação de integridade antes da restauração e proteção contra restauração de arquivos inválidos ou corrompidos.
29. **RF29 — Licenciamento por Arquivo-Chave:** O sistema deve exigir ativação via arquivo-chave de uso único, vinculado à instalação. O arquivo-chave não pode ser reutilizado. Deve haver processo de reativação para casos de reinstalação por falha de hardware.
30. **RF30 — Operação Off-line:** Todas as funcionalidades do sistema devem operar completamente sem conexão com internet, com exceção do processo de ativação de licença (que pode exigir contato com o fornecedor por meios externos).

## 6. Requisitos Não Funcionais

1. **RNF01 — Desempenho (PDV):** A busca de produtos por código de barras deve retornar resultado em menos de 1 segundo. A finalização de uma venda (baixa de estoque, registro financeiro, geração de nota) deve ser concluída em menos de 2 segundos.
2. **RNF02 — Desempenho (Relatórios):** Relatórios com filtro de período mensal devem ser gerados em menos de 5 segundos para uma base de até 10.000 vendas.
3. **RNF03 — Disponibilidade Off-line:** O sistema deve ser 100% funcional sem conexão com internet. Nenhuma funcionalidade do dia a dia pode depender de conectividade externa.
4. **RNF04 — Persistência Local:** Todos os dados devem ser armazenados localmente no computador onde o sistema está instalado. Nenhum dado operacional pode ser armazenado em servidores externos.
5. **RNF05 — Integridade de Dados:** O backup deve incluir verificação de integridade (checksum). A restauração não deve alterar os dados atuais se o arquivo de backup for inválido. Operações financeiras e de estoque devem ser atômicas (ou completam totalmente, ou não alteram nada).
6. **RNF06 — Segurança de Acesso:** Senhas devem ser armazenadas de forma irreversível (hash). A sessão do usuário deve expirar após período de inatividade configurável. Operações críticas devem exigir senha de gerente mesmo que o usuário logado já seja gerente (dupla confirmação para exclusões).
7. **RNF07 — Usabilidade:** O PDV deve ser operável exclusivamente por teclado (atalhos para todas as funções de venda). A interface deve usar linguagem em português brasileiro (PT-BR). Campos monetários devem respeitar o formato brasileiro (R$ 1.234,56).
8. **RNF08 — Instalação e Manutenção:** O processo de instalação deve ser simples, sem necessidade de conhecimento técnico avançado. O backup deve ser executável por usuário não técnico em no máximo 3 passos.
9. **RNF09 — Resiliência:** O sistema não deve perder dados ou corromper o banco em caso de queda de energia ou fechamento inesperado. Ao reiniciar, deve recuperar o último estado consistente.
10. **RNF10 — Licenciamento:** O sistema deve impedir execução sem ativação válida. O mecanismo de licença não pode bloquear o acesso aos dados já cadastrados em caso de expiração (apenas impede novas operações).
11. **RNF11 — Mono-Usuário Simultâneo:** O sistema deve garantir que apenas um usuário esteja ativo por vez no terminal. A troca de usuário deve encerrar corretamente a sessão anterior.

## 7. Métricas de Sucesso

1. **Tempo de Atendimento no Balcão:** O tempo médio para registrar e finalizar uma venda simples (1 produto, 1 forma de pagamento, sem cliente) deve ser inferior a 30 segundos, medido do primeiro clique/leitura até a impressão da nota.
2. **Adoção do Sistema:** O lojista deve conseguir operar o ciclo completo (abertura de caixa → venda → fechamento de caixa → backup) sem consultar documentação externa após 2 dias de uso.
3. **Zero Perda de Dados:** Nenhum incidente de perda ou corrupção de dados reportado nos primeiros 90 dias de uso, desde que o usuário realize backups conforme orientação.
4. **Cobertura do Ciclo Operacional:** O lojista não deve precisar recorrer a nenhum sistema externo (planilha, caderno) para gerenciar seu negócio após a adoção completa do SG-Varejo.
5. **Autonomia do Usuário:** O lojista deve conseguir instalar, ativar e configurar o sistema sem assistência técnica remota ou presencial em pelo menos 80% dos casos.
6. **Precisão Financeira:** A diferença entre o saldo de caixa calculado pelo sistema e o saldo real (conferência física) deve ser zero em condições normais de operação (sem erros de troco ou esquecimento de registro pelo operador).
7. **Conformidade com Licenciamento:** Zero instâncias de uso não licenciado viabilizadas por falha no mecanismo de arquivo-chave nos primeiros 12 meses.
