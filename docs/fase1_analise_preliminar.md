# Análise Preliminar de Discovery — superVenda

## 1. Desconstrução do Problema

* **Dor Real do Negócio:** O cliente é um pequeno/médio varejista multi-segmento (ou uma software house que atende esse público) que opera em ambiente completamente offline, com apenas 1 computador Windows 11 e 1 operador por vez. A dor central é: inexistência de um sistema unificado simples que cubra PDV, controle de estoque e gestão financeira com creditário próprio (carnê) em um único ambiente local, sem depender de internet, com proteção contra cópia ilegal e resiliência contra perda de dados via backup manual. O diferencial esperado é o cálculo de lucratividade real (DRE) integrando despesas fixas (aluguel, luz, salários) — algo raro em sistemas de entrada.

* **Solução Assumida pelo Cliente:** O cliente forneceu uma lista exaustiva de funcionalidades organizadas em 6 módulos (Cadastro, Movimentação, Financeiro, Relatórios, Consultas, Gerencial). Ele demonstra conhecer bem o domínio e assumiu um escopo amplo e detalhado. Isso é positivo, mas traz o risco clássico de "lista de desejos" — tudo parece essencial, sem priorização clara para um MVP. Não há indicação do que é crítico para a operação abrir a loja amanhã versus o que pode ser entregue depois. Além disso, o cliente descreveu "o que" quer, mas raramente detalhou "como" as regras de negócio devem se comportar (exceções, validações, fluxos alternativos).

## 2. Mapeamento de Personas Ocultas

* **Gerente / Proprietário (Acesso Total):** Responsável por cadastros, finanças, relatórios, backup/restauração e configurações do sistema. É o tomador de decisão que analisa o DRE e o fluxo de caixa. Precisa de visão gerencial consolidada e controle sobre todas as operações.

* **Vendedor / Operador de Caixa (Acesso Restrito):** Opera o PDV, realiza vendas e consultas. Não pode alterar preços, excluir vendas ou acessar dados financeiros sem senha do gerente. Precisa de um fluxo de venda rápido, intuitivo e com poucas possibilidades de erro.

* **Cliente Final / Consumidor (Entidade Cadastrada):** Pessoa física que compra na loja. Pode ter cadastro com foto, limite de crédito para carnê, histórico de compras e situação de crédito (bloqueado/liberado). Pode receber cartas de cobrança em caso de atraso. Seu CPF ou identificação pode ser usado para consulta rápida de situação.

* **Suporte Técnico / Fornecedor do Sistema (Agente Externo):** Responsável pelo processo de liberação de nova chave de ativação quando o lojista troca de computador ou formata a máquina. Precisa de um fluxo definido para validar a solicitação e liberar a reativação sem comprometer a proteção de licença.

## 3. Avaliação de Maturidade e Lacunas Críticas

O material do cliente é notavelmente maduro para uma demanda inicial — ele já definiu módulos, restrições técnicas (offline, local, 2 níveis de acesso, Windows 11) e até o modelo de licenciamento. No entanto, as seguintes lacunas críticas foram identificadas:

**Lacunas de Regras de Negócio:**
- **Carnê / Creditário:** Listado como funcionalidade central (limite de crédito, preço carnê, contas a receber, cobrança), mas não há regras sobre prazos, número de parcelas, incidência de juros/multa por atraso, cálculo do limite, se o limite é por cliente ou compartilhado.
- **Formas de Pagamento:** Não foram especificadas (dinheiro, cartão de crédito/débito, PIX, cheque, vale?). Isso impacta diretamente o PDV, o fechamento de caixa e os relatórios de vendas por forma de pagamento.
- **Descontos em Venda:** Pode haver desconto? Quem pode conceder (apenas gerente ou vendedor também)? Há limite percentual? Isso impacta o cálculo de margem de lucro.
- **Sangria:** O fluxo de sangria (retirada de dinheiro do caixa) não foi detalhado. Quem pode fazer? Precisa de justificativa? Há limite?
- **Devolução / Troca / Cancelamento de Venda:** Não foi mencionado nenhum fluxo. É uma operação comum no varejo. Como tratar estorno de carnê, devolução ao estoque, cancelamento de conta a receber?
- **Estoque Mínimo:** Apenas alerta visual ou o sistema deve bloquear vendas quando o estoque atinge o mínimo?
- **Múltiplos Segmentos:** O sistema promete atender desde mercados até auto peças. Há necessidades diferentes por segmento? (ex: grade de tamanho/cor para roupas, número de série para eletrônicos?)

**Lacunas de Integração e Ambiente:**
- **Impressão:** A nota de venda A4 é o único formato. Precisa de impressora térmica para cupom também? A nota A4 carrega logotipo — o layout é fixo ou customizável?
- **Base Offline:** Confirmado que é offline, mas não está claro se eventualmente precisará de funcionalidades de internet (ex: PIX offline, atualização automática).

**Lacunas de Segurança e Operação:**
- **Concorrência:** Embora seja 1 pessoa por vez, um sistema desktop pode ter múltiplas instâncias abertas acidentalmente. Como tratar?
- **Auditoria:** Não há menção a logs de auditoria para ações críticas (exclusão, alteração de preços, desconto, sangria, uso de senha de gerente para override).

**Lacunas de Implantação:**
- **Carga Inicial:** Como será a migração de dados de um sistema anterior ou planilhas? O cliente precisará de importação de produtos, clientes e saldo de carnê?

## 4. Estratégia de Mitigação de Riscos (MVP)

O principal risco é a amplitude do escopo sem priorização, o que pode levar a um produto que "faz tudo mas nada funciona bem". Recomenda-se validar as premissas na seguinte ordem:

1. **Núcleo da Operação Diária (Validação Imediata):**
   - Cadastro de produtos (com código de barras, grupo/subgrupo, preço normal)
   - Abertura e fechamento de caixa com valor inicial
   - Venda simples (PDV) com 1 forma de pagamento (dinheiro)
   - Nota de venda A4 com logotipo
   - Baixa automática de estoque na venda
   - Reposição de estoque manual
   - Login com 2 níveis (Gerente e Vendedor)

2. **Financeiro Básico (Semana 2-3):**
   - Fluxo de caixa básico (entradas e saídas)
   - Sangria
   - Contas a pagar (lançamento e baixa)
   - Relatórios essenciais (vendas do dia, vendas por período)

3. **Carnê e Creditário (Após validação do núcleo):**
   - Carnê com regras de parcelamento definidas
   - Contas a receber com baixa
   - Limite de crédito por cliente
   - Cartas de cobrança
   - Bloqueio de cliente por atraso

4. **Funcionalidades Complementares (Pós-MVP):**
   - Dashboard com gráficos
   - DRE com despesas fixas
   - Relatórios avançados (margem de lucro, produtos mais vendidos)
   - Aniversariantes, fotos de clientes
   - Mecanismo de licenciamento com chave (pode ser desenvolvido em paralelo)

**Premissa Crítica a Validar com o Cliente:** O sistema será usado por UM segmento de cada vez (cada loja é de um tipo), ou a mesma instalação atenderá múltiplos segmentos simultaneamente? Esta resposta define se precisamos de parametrização por tipo de negócio.
