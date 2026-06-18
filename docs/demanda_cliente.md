# Sistema P/ Loja, Controle De Estoque, Pdv, Caixa e Vendas

# Público alvo:
Mercados, Vendas de Roupas, sapatos, Cosméticos, Auto Peças, Depósitos, Lojas de Informática, Materiais de construção, Lojas de Confecções, Lojas de Bijuterias!

Funcionalidades:

## CADASTRO
- Empresa (com seu logotipo)
- Clientes (com foto e limite para compras no carnê)
- Fornecedores
- Produtos (com foto, código de barras, Grupo e Subgrupo)
- Vendedor/Usuário (acesso por login e senha com restrições de acesso ao sistema)
- Formas de Pagamento com taxa e configure se vai mostrar no fluxo de caixa

## MOVIMENTAÇÃO
- Abertura e fechamento de caixa
- Compras (Importação de produtos de arquivo XML de nota fiscal para o sistema)
- Vendas (nota de venda A4 com logotipo)
- Vendas á prazo (coloque um limite para compras para cada cliente! Deve permitir opcionalmente configurar um acréscimo (taxa fixa ou percentual) conforme o número de parcelas.
- 2 Formas de Pgto por Venda (você pode cadastrar diversas formas de pgto inclusive incluir taxas)
- 2º Opção de preço no produto (venda a prazo ou atacado por exemplo)
- Opção de Controle de estoque (aviso de estoque baixo e bloqueio de vendas de produtos sem estoque)
- Ativação de Senha de gerente para exclusão e alteração de vendas, preços, produtos e etc.

## FINANCEIRO
- Caixa Inicial
- Sangria
- Fluxo de caixa (Detalhado e Resumido)
- Lucro das vendas (Relatório DRE - total de vendas menos total de compras no período, mostrando o lucro diário, mensal e anual)
- Lançar contas a pagar
- Lançar contas a receber
- Baixa de contas a pagar
- Baixa de contas a receber
- Gráfico das vendas do dia/período

## RELATÓRIOS
- Vendas: Vendas por vendedor, período, clientes, produtos, categorias de produtos, formas de pagamento, margem de lucro e produtos mais vendidos
- Vendas: Resumo de vendas mensais, diário.
- Compras: Compras por período, fornecedor e produtos
- Estoque: Estoque, Estoque mínimo, Inventário 
- Financeiro: Contas a pagar
- Financeiro: Contas pagas
- Financeiro: Contas a receber
- Financeiro: Contas recebidas
- Financeiro: Fluxo de caixa 

## CONSULTAS
- Produtos (por código, descrição, referência)
- Aniversariantes
- Estoque financeiro dos produtos
- Clientes em atraso
- Informativo contas a pagar
- Informativo contas a receber
- Histórico de vendas por cliente
- Situação do cliente (Pagamentos, bloqueios de compra e etc.)

## GERENCIAL
- Backup (Cópia de segurança dos dados) - O próprio lojista faz o backup manualmente quando quiser, salvando onde preferir (pen drive, HD externo, etc.)
- Restauração de backup

## UTILITÁRIOS
- Emissão de recibos
- Impressão de etiquetas de produtos (para uso com leitor de código de barras)
- Conferência e ajuste de estoque
- Emissão de cartas de cobrança para cliente
- Importação de produtos e clientes via planilha Excel
- Importação de produtos de arquivo XML de nota fiscal para o sistema
- Alteração de preço/comissão em massa


## Observações gerais
- Apenas 1 computador com windows 11, 1 pessoa por vez no sistema. sem acesso via rede
- Não emite documentos fiscais
- Criar proteção contra multiplas instalações. licença com arquivo-chave. Ao instalar utiliza arquivo chave. Arquivo chave só pode ser utilizado uma vez
- Banco de dados local
- Não conectado à internet
- Apenas 2 níveis: Gerente (acesso total) e Vendedor (apenas vendas e consultas)
- O segundo preço do produto (preço a prazo ou atacado) : O vendedor escolhe manualmente qual preço usar em cada venda. 
- Não preciso de controle de validade, apenas do controle de estoque mínimo
- Nas vendas a prazo (carnê), o acréscimo por parcelamento deve ser calculado da seguinte forma: Uma tabela onde eu defino manualmente o acréscimo para cada número de parcelas (ex: 2x = 3%, 3x = 5%, 4x = 8%)
- No cálculo do lucro (DRE), além de vendas menos compras, incluir despesas fixas (aluguel, luz, salários)
- Se o computador quebrar, for formatado ou trocado: usuário entra em contato com o suporte para liberar uma nova chave de ativação, comprovando a situação.
- Sobre a comissão de vendedores : Um percentual definido por produto, podendo ser diferente para cada item