# Relatório de Revisão de PRD — Controle de Qualidade

## 1. Veredito e Score de Prontidão

<auditoria_interna>
**Processo de Auditoria — Contexto e Raciocínio:**

**1. Critérios de Aceitação (BDD):**
A base do documento é forte: todas as US seguem o formato Dado/Quando/Então e cobrem, na maioria dos casos, o caminho feliz e ao menos um caminho triste. Casos exemplares:
- US 2.7 (desconto) — cobre 3 cenários (dentro do limite, acima do limite com senha, remoção do desconto).
- US 3.3 (DRE) — cobre lucro, prejuízo com destaque visual e alternância de períodos com recálculo.
- US 5.4 (troca de usuário) — cobre login, troca com venda pendente (duas ramificações) e lockout por tentativas.

Porém, há lacunas significativas:
- US 2.4 (finalização c/ 2 formas de pagamento) — não aborda troco quando o pagamento em dinheiro excede o total. Lacuna grave para um PDV.
- US 3.4 (sangria) — não aborda sangria maior que o saldo disponível em caixa.
- US 2.6 (preço 2) — não aborda o que acontece se preço 2 não foi configurado (zero ou vazio).
- US 1.1 (cadastro empresa) — não aborda formato/tamanho máximo da imagem de logotipo.
- Várias US com mensagens de erro descritas genericamente ("exibe mensagem clara" — US 5.1) sem especificar o texto exato.

**2. Coesão de Escopo (US vs RF):**
Mapeamento quase completo. Uma exceção importante:
- RF17 (Impressão de Nota de Venda A4) NÃO possui uma História de Usuário correspondente com critérios de aceitação BDD. É mencionada de passagem na US 2.4 como "a nota de venda A4 é gerada", mas não há cenários dedicados ao leiaute, controle de fila de impressão, reimpressão, etc.

**3. Blindagem Tecnológica:**
Pontuação máxima neste quesito. O documento não menciona bancos de dados, linguagens, frameworks ou bibliotecas específicas. As referências a "banco de dados local" e "off-line" são restrições de domínio, não decisões de implementação. O PM manteve a neutralidade tecnológica corretamente.

**4. Ambiguidades e Perguntas Não Respondidas:**
- US 5.3 — Persona "Fornecedor do sistema (Natural Tecnologia)" quebra o modelo estabelecido na seção 2 (Proprietário e Vendedor). O requisito de licenciamento é legítimo, mas a perspectiva está errada.
- Acréscimo em vendas a prazo (US 2.5 / RF14) — não está claro como a taxa é configurada: é por forma de pagamento? Por número de parcelas? Uma tabela de juros? Valor fixo ou percentual? O PRD diz "taxa fixa ou percentual conforme número de parcelas" sem especificar a mecânica.
- US 4.1 (importação XML) — "localizados no cadastro (ou criados se novos)" — qual o critério de matching? Código de barras? GTIN? Descrição? Se houver divergência de preço entre XML e cadastro, o que prevalece?
- US 3.1 / RF22 — "baixa (total) de cada parcela" — não há previsão de baixa parcial. E se o cliente pagar menos que o valor da parcela?
- US 5.4 — "bloqueia temporariamente novas tentativas por 5 minutos" — é rolling lockout (a cada tentativa errada reinicia a contagem)? O desbloqueio é automático após 5 minutos ou requer intervenção manual?
- US 3.4 — "exige senha de gerente se a configuração assim determinar" — que configuração? Onde é definida?
- RNF07 — "PDV deve ser operável exclusivamente por teclado" — isso tem implicações profundas de UX e deveria ser uma História de Usuário própria com cenários BDD, não apenas um RNF.

**5. Pontos Fortes do Documento:**
- Escopo muito bem delimitado (dentro/fora do escopo).
- Personas claras e distintas com dores bem definidas.
- BDD bem aplicado na maioria das US.
- DRE com cálculo detalhado de CMV (US 3.3).
- Cenário transversal de resiliência (RNF09) bem pensado.
- RF numerados e alinhados com as US.
- Métricas de sucesso específicas e mensuráveis.

**6. Cálculo do Score:**

Deduções:
- (-1.0) Ausência de US para impressão de nota A4 (RF17)
- (-0.8) Troco não tratado na US 2.4 — lacuna grave em PDV
- (-0.5) Persona "Fornecedor" em US 5.3 — quebra o modelo estabelecido
- (-0.3) Mecânica de acréscimo em vendas a prazo ambígua
- (-0.2) Baixa parcial não prevista em contas a receber
- (-0.2) Critério de matching na importação XML subespecificado

Score final: 10 - 3.0 = 7.0

Justificativa: O documento é bom, com estrutura sólida e BDD bem aplicado. Porém, as lacunas identificadas — especialmente a ausência de troco no PDV e a falta de US para impressão de nota — gerariam retrabalho na engenharia. O documento não está maduro o suficiente para seguir para implementação sem revisão.
</auditoria_interna>

- **Score de Prontidão:** 7.0 / 10
- **Status:** REQUER REFATORAÇÃO (Score < 8)
- **Resumo:** O PRD apresenta uma base sólida com BDD bem estruturado na maioria das histórias, escopo bem delimitado e neutralidade tecnológica exemplar. No entanto, contém lacunas críticas para um sistema PDV: a ausência de tratamento de troco na finalização de vendas, a falta de uma História de Usuário dedicada à impressão de nota A4, e ambiguidades na mecânica de acréscimo de vendas a prazo e baixa parcial de contas. A persona "Fornecedor" na US 5.3 foge do modelo estabelecido. O documento precisa de correções antes de seguir para a engenharia.

## 2. Pontos Críticos Identificados

- **Troco não especificado na US 2.4:** A finalização de venda com dinheiro não define como o troco é calculado e exibido. Para um sistema de PDV, esta é uma omissão grave.
- **US de Impressão de Nota A4 ausente:** O RF17 define a impressão, mas não há uma História de Usuário com BDD cobrindo leiaute, reimpressão, fila de impressão ou falha de impressora.
- **Persona "Fornecedor" na US 5.3:** A história de licenciamento está escrita da perspectiva do fornecedor de software, não do usuário do sistema, quebrando o modelo de personas definido na Seção 2.
- **Mecânica de acréscimo em vendas a prazo vaga:** A US 2.5 e RF14 dizem "taxa fixa ou percentual conforme número de parcelas" sem definir como e onde essa taxa é configurada (por forma de pagamento? Tabela de juros? Por produto?).
- **Baixa parcial de contas a receber não prevista:** RF22 especifica apenas "baixa (total)", sem tratar o cenário comum de pagamento parcial de uma parcela.
- **Critério de matching na importação XML subespecificado:** US 4.1 diz "localizados no cadastro (ou criados se novos)" sem definir a regra de correspondência entre produto do XML e produto do cadastro.
- **Mensagem de erro genérica na US 5.1:** "exibe uma mensagem clara informando o problema" é subjetivo e não testável.
- **Sangria sem validação de saldo:** US 3.4 não especifica o comportamento se o valor da sangria exceder o saldo disponível em caixa.
- **Mecanismo de lockout vago na US 5.4:** "bloqueia temporariamente novas tentativas por 5 minutos" não especifica se é rolling lockout, se o desbloqueio é automático, ou se há contagem regressiva visível.
- **PDV exclusivamente por teclado (RNF07) sem US dedicada:** Um requisito de UX tão restritivo merece cenários BDD próprios, não apenas uma linha em não-funcionais.

## 3. Propostas de Correção Direta

### Correção 1: Nova US — Impressão de Nota de Venda A4 (RF17)
- **Como está no PRD original:**
  > *Ausente. RF17 existe, mas não há US correspondente.*
- **Como deve ficar (Sugestão de nova US):**
  > **US 2.8:** Como Vendedor, eu quero imprimir a nota de venda em formato A4 com o logotipo da empresa e detalhamento dos itens para entregar ao cliente ao final da compra.
  > **Critérios de Aceitação:**
  > *Cenário 1: Impressão bem-sucedida*
  > **Dado** que a venda foi finalizada com sucesso e há uma impressora configurada no sistema
  > **Quando** o sistema gera a nota de venda
  > **Então** a nota A4 é exibida para impressão contendo: logotipo da empresa, dados da empresa, data/hora, número da venda, vendedor, cliente (se informado), itens (descrição, qtd, preço unitário, subtotal), total, formas de pagamento, troco (se aplicável), e mensagem de rodapé
  > *Cenário 2: Reimpressão de nota*
  > **Dado** que uma venda foi finalizada anteriormente
  > **Quando** o vendedor acessa o histórico de vendas e seleciona "Reimprimir"
  > **Então** o sistema exibe a nota exatamente como foi gerada no momento da venda
  > *Cenário 3: Falha na impressão*
  > **Dado** que a impressora está desligada ou sem papel
  > **Quando** o sistema tenta enviar o documento para impressão
  > **Então** o sistema exibe "Falha na impressão. Verifique a impressora e tente novamente" e mantém a venda registrada, permitindo reimpressão posterior

### Correção 2: Troco na Finalização de Venda (US 2.4)
- **Como está no PRD original:**
  > *Não há menção a troco/change na US 2.4 ou cenários relacionados.*
- **Como deve ficar (Sugestão de novo cenário na US 2.4):**
  > *Cenário 3: Pagamento em dinheiro com troco*
  > **Dado** que o total da venda é R$ 47,00
  > **Quando** o vendedor informa "Dinheiro: R$ 50,00"
  > **Então** o sistema calcula o troco de R$ 3,00, exibe "Troco: R$ 3,00" em destaque na tela, registra a venda como paga, e a nota de venda exibe o valor recebido e o troco

### Correção 3: Persona da US 5.3 (Licenciamento)
- **Como está no PRD original:**
  > "Como **Fornecedor do sistema (Natural Tecnologia)**, eu quero que o sistema utilize licenciamento por arquivo-chave de uso único..."
- **Como deve ficar (Sugestão de Reescrita):**
  > **US 5.3:** Como Proprietário, eu quero que o sistema exija um arquivo-chave de licença para ser ativado, de forma que apenas clientes autorizados pela Natural Tecnologia possam utilizar o sistema.
  > *Nota: Os cenários de validação do arquivo-chave (consumo único, rejeição de chave já utilizada, reativação via suporte) permanecem idênticos.*

### Correção 4: Mecânica de Acréscimo em Vendas a Prazo (RF14 / US 2.5)
- **Como está no PRD original:**
  > "Deve aplicar acréscimo configurável (taxa fixa ou percentual) conforme número de parcelas"
- **Como deve ficar (Sugestão de Reescrita):**
  > **RF14 — Venda a Prazo (Carnê):** O sistema deve permitir venda parcelada com número de parcelas definido pelo vendedor, entre 2 e 12 parcelas (limite configurável). O acréscimo deve ser configurado por faixa de parcelas, permitindo definir uma taxa percentual mensal ou um valor fixo total para cada faixa (ex.: 1 a 3x = 0%, 4 a 6x = 5%, 7 a 12x = 10%). O sistema deve aplicar a taxa correspondente ao número de parcelas selecionado, calcular o valor total com acréscimo, dividir pelo número de parcelas (valor da parcela = total / parcelas, arredondado para centavos), e ajustar a última parcela para compensar diferenças de arredondamento. Deve validar o limite de crédito do cliente considerando o valor total com acréscimo. As parcelas devem ser geradas automaticamente no contas a receber com datas de vencimento baseadas na data da venda.

### Correção 5: Baixa Parcial de Parcelas (US 3.1 / RF22)
- **Como está no PRD original:**
  > "baixa (total) de cada parcela"
- **Como deve ficar (Sugestão de Reescrita):**
  > *Adicionar cenário na US 3.1:*
  > *Cenário 3: Baixa parcial de parcela*
  > **Dado** que o cliente "Maria" possui uma parcela de R$ 70,00 com vencimento em 15/06/2026
  > **Quando** o proprietário registra um pagamento parcial de R$ 50,00 para esta parcela
  > **Então** o sistema registra o pagamento parcial, altera o status da parcela para "Paga parcialmente — Saldo: R$ 20,00", restaura R$ 50,00 do limite de crédito de Maria, e o valor de R$ 50,00 entra no fluxo de caixa

### Correção 6: Critério de Matching na Importação XML (US 4.1)
- **Como está no PRD original:**
  > "Os 10 produtos são localizados no cadastro (ou criados se novos)"
- **Como deve ficar (Sugestão de Reescrita):**
  > *Adicionar Especificação:*
  > O matching entre produtos do XML e produtos cadastrados deve seguir a ordem de precedência:
  > 1. **GTIN (código de barras)** do produto no XML × código de barras do cadastro
  > 2. Se não houver GTIN ou correspondência, tentar **código interno do fornecedor** (campo opcional no cadastro)
  > 3. Se nenhuma correspondência for encontrada, o produto é criado como novo
  > Em caso de divergência de preço entre XML e cadastro, o preço de custo é atualizado para o valor do XML, e o preço de venda **NÃO é alterado automaticamente** (o proprietário deve revisar manualmente)

### Correção 7: Mensagens de Erro Específicas (US 5.1)
- **Como está no PRD original:**
  > "o sistema exibe uma mensagem clara informando o problema"
- **Como deve ficar (Sugestão de Reescrita):**
  > "o sistema exibe exatamente: 'Espaço insuficiente no destino. Libere espaço ou selecione outro local.' ou 'Destino inacessível. Verifique se a unidade está conectada.' conforme o caso, e não gera um arquivo de backup parcial ou corrompido."

### Correção 8: Validação de Saldo na Sangria (US 3.4)
- **Como está no PRD original:**
  > *Não há cenário de sangria maior que o saldo disponível.*
- **Como deve ficar (Sugestão de novo cenário na US 3.4):**
  > *Cenário 2: Sangria maior que o saldo disponível*
  > **Dado** que o caixa está aberto com saldo atual de R$ 200,00 em dinheiro
  > **Quando** o usuário tenta registrar uma sangria de R$ 300,00
  > **Então** o sistema bloqueia a operação e exibe "Saldo insuficiente para sangria. Saldo atual em dinheiro: R$ 200,00"

### Correção 9: Mecanismo de Lockout Especificado (US 5.4)
- **Como está no PRD original:**
  > "o sistema bloqueia temporariamente novas tentativas por 5 minutos"
- **Como deve ficar (Sugestão de Reescrita):**
  > "o sistema bloqueia o login por 5 minutos (contagem regressiva). O contador de tentativas é rolling: cada nova tentativa durante a janela de 5 minutos reinicia o período de bloqueio. O desbloqueio é automático após os 5 minutos sem novas tentativas. O sistema exibe: 'Conta temporariamente bloqueada por [X] tentativas consecutivas inválidas. Tente novamente em [N] minutos.' O bloqueio é por usuário, não global."

### Correção 10: Nova US — Operação por Teclado (RNF07)
- **Como está no PRD original:**
  > *RNF07: "O PDV deve ser operável exclusivamente por teclado (atalhos para todas as funções de venda)"*
- **Como deve ficar (Sugestão de nova US no Épico 2):**
  > **US 2.9:** Como Vendedor, eu quero operar o PDV inteiramente pelo teclado para agilizar o atendimento sem precisar alternar entre teclado e mouse.
  > **Critérios de Aceitação:**
  > *Cenário 1: Atalho para busca de produtos*
  > **Dado** que o vendedor está na tela de PDV com o caixa aberto
  > **Quando** ele pressiona F2 ou Ctrl+B
  > **Então** o foco é direcionado para o campo de busca de produtos sem necessidade de clique do mouse
  > *Cenário 2: Atalho para finalizar venda*
  > **Dado** que o carrinho possui itens
  > **Quando** o vendedor pressiona F8
  > **Então** o sistema avança para a tela de finalização/formas de pagamento
  > *Cenário 3: Atalho para desconto*
  > **Dado** que um item está selecionado no carrinho
  > **Quando** o vendedor pressiona Ctrl+D
  > **Então** o sistema abre o campo para digitar o valor do desconto
  > *Nota: A lista completa de atalhos deve ser definida e documentada em anexo técnico. Este requisito não impede que o PDV também aceite interação por mouse — apenas garante que todas as funções essenciais sejam acessíveis por teclado.*

## 4. Próximos Passos

1. **Aplicar as correções propostas acima** no arquivo `docs/prd_revisado.md` antes de enviar para a Engenharia.
2. **Atenção especial:** Inserir a US 2.8 (Impressão de Nota A4) e o cenário de troco na US 2.4 são **mandatórios** — sem essas correções, a equipe de desenvolvimento iniciará o PDV com especificação incompleta.
3. **Revisar a US 5.3** para alinhar a persona ao modelo Proprietário/Vendedor já estabelecido.
4. **Corrigir as especificações de acréscimo, baixa parcial e matching XML** para eliminar ambiguidades que gerariam perguntas durante o desenvolvimento.
5. Após aplicar as correções, rodar novamente a etapa de validação (Fase 3 — PRD Reviewer) para reavaliar o Score de Prontidão.
6. Com Score ≥ 8, o documento estará apto para seguir para o OpenSpec/Gerador de SDD.
