# Análise Preliminar de Discovery — Sistema de Gestão para Varejo (SG-Varejo)

## 1. Desconstrução do Problema

* **Dor Real do Negócio:** O cliente representa o perfil clássico do pequeno varejista brasileiro multi-segmento (mercados, vestuário, calçados, cosméticos, autopeças, depósitos, informática, materiais de construção, confecções e bijuterias). A dor central é a necessidade de centralizar em um único sistema, de operação simples e local, todo o ciclo operacional do negócio: cadastros, vendas no balcão (PDV), vendas a prazo com controle de crédito, compras via XML de nota fiscal, controle de estoque com alertas, fluxo de caixa, contas a pagar/receber e relatórios gerenciais (incluindo DRE simplificado). O cenário é de operação mono-terminal (1 computador, 1 pessoa por vez), sem necessidade de internet e sem emissão de documentos fiscais eletrônicos. O cliente quer autonomia total: backup manual, licenciamento por arquivo-chave e banco de dados local.

* **Solução Assumida pelo Cliente:** O cliente entregou uma lista de funcionalidades notavelmente detalhada e estruturada em módulos (Cadastro, Movimentação, Financeiro, Relatórios, Consultas, Gerencial, Utilitários). Assume-se um sistema desktop tradicional, com banco local, operação off-line, acesso mono-usuário por vez, mas com múltiplos perfis de login. A riqueza de detalhes sugere que o cliente já operou sistemas similares e sabe exatamente o que precisa no nível funcional. A principal premissa não validada é a plataforma tecnológica (S.O. alvo) e o mecanismo de persistência, que estão implícitos mas não declarados.

## 2. Mapeamento de Personas Ocultas

* **Proprietário/Gerente Geral:** Acesso irrestrito. Opera relatórios gerenciais (DRE, fluxo de caixa, lucro), gráficos de vendas, cadastros mestres (empresa, fornecedores), backup/restauração e configurações globais (taxas, formas de pagamento). É quem define limites de crédito e aprova exclusões/alterações via senha de gerente.
* **Vendedor/Atendente de Balcão:** Opera o PDV diário: abertura de venda, consulta de produtos (código, descrição, referência), Consulta de situação do cliente (limite disponível, atrasos), finalização com até 2 formas de pagamento, impressão de nota de venda A4. Pode ter restrições de desconto e acesso a relatórios.
* **Financeiro/Contas:** Responsável por lançar contas a pagar e a receber, dar baixas, registrar sangrias e acompanhar fluxo de caixa. Não necessariamente opera vendas.
* **Estoquista/Comprador:** Responsável por importar XML de compras, ajustar estoque, conferência de inventário, cadastro de produtos (foto, código de barras, grupo/subgrupo, preço normal e preço 2), emissão de etiquetas e importação de produtos via planilha Excel.

## 3. Avaliação de Maturidade e Lacunas Críticas

O texto do cliente possui **alta maturidade funcional** — é raro receber uma demanda tão bem estruturada em módulos. No entanto, as seguintes lacunas precisam ser endereçadas antes da especificação técnica:

* **Plataforma e Ambiente (Lacuna Gravíssima):** O cliente não especifica o sistema operacional alvo (Windows 10/11? Linux? Ambos?). Isso impacta diretamente a escolha da stack tecnológica e do banco de dados local.
* **Banco de Dados Local (Lacuna Alta):** Dizer "banco de dados local" é vago. Precisa-se definir se será um banco embarcado (SQLite, Firebird) ou um SGBD instalável (PostgreSQL, MySQL). Impacta instalação, backup e performance.
* **Granularidade de Permissões (Lacuna Alta):** O cliente menciona "restrições de acesso ao sistema", mas não detalha o que cada perfil pode ou não fazer. Isso é crítico para definir a matriz de permissões.
* **Regras do Preço 2 (Lacuna Média):** Existe um "2º opção de preço no produto (venda a prazo ou atacado)". As regras de quando cada preço se aplica não estão definidas: é automático por tipo de venda? É seleção manual do vendedor? Depende de quantidade mínima?
* **Cálculo do DRE (Lacuna Média):** A definição "total de vendas menos total de compras no período" é uma simplificação contábil que ignora outros custos operacionais. Cabe alinhar se o cliente tem expectativa de incluir despesas fixas, impostos ou outros custos no cálculo de lucro.
* **Validade de Produtos (Lacuna Média):** O relatório "Validade de produto" aparece na seção de Relatórios > Estoque, mas não há campo de data de validade na seção de Cadastro > Produtos. É uma funcionalidade implícita que precisa ser confirmada.
* **Modelo de Licenciamento (Lacuna Média):** "Arquivo chave só pode ser utilizado uma vez" — como tratar cenários de reinstalação por falha de hardware, formatação ou troca de computador? Precisa de um processo de reativação.
* **Comissão sobre Vendas (Lacuna Baixa):** "Alteração de preço/comissão em massa" aparece em Utilitários, sugerindo que há comissionamento para vendedores. Mas a regra de comissão (percentual sobre venda? por produto? fixa?) não está descrita em nenhum lugar.
* **Formato de Nota de Venda A4 (Lacuna Baixa):** O cliente menciona "nota de venda A4 com logotipo", mas não detalha quais campos devem constar. Como não é documento fiscal, há total flexibilidade.
* **Conteúdo de Recibos e Cartas de Cobrança (Lacuna Baixa):** Formatos e campos esperados não foram descritos.
* **Concorrência e Conflito (Lacuna Baixa):** Com "apenas 1 computador", concorrência de banco não é problema. Mas com múltiplos usuários se alternando no mesmo terminal, o sistema precisa gerenciar sessões (login/logout) adequadamente.

## 4. Estratégia de Mitigação de Riscos (MVP)

Premissas que precisam ser validadas primeiro para evitar desperdício de escopo:

1. **Definir plataforma e banco ANTES de qualquer linha de código:** A escolha do SO e do banco determina toda a arquitetura. Sugere-se propor Windows como padrão (mercado dominante no varejo brasileiro de pequeno porte) com SQLite embarcado (zero-instalação, backup por cópia de arquivo), e validar com o cliente.
2. **Validar a persona de "1 pessoa por vez" vs "múltiplos logins":** O cliente quer múltiplos usuários mas apenas 1 por vez. Confirmar se há cenário futuro de rede local (múltiplos PDVs) que impactaria decisões de arquitetura.
3. **Priorizar o fluxo de venda (PDV) como espinha dorsal do MVP:** Se o PDV não funcionar perfeitamente (rápido, fluido, com suporte a código de barras e 2 formas de pagamento), o resto não importa. O MVP deve começar com: Cadastro de Produtos + Cadastro de Clientes + PDV + Fechamento de Caixa.
4. **Esclarecer as regras de crédito e juros no carnê ANTES de implementar o módulo financeiro:** O cliente menciona "acréscimo conforme número de parcelas". Validar se é juros simples, compostos, tabela price, ou taxa fixa por parcela. Isso impacta o cálculo de contas a receber.
5. **Confirmar escopo de relatórios obrigatórios para o MVP:** Nem todos os relatórios listados precisam estar na versão 1. Sugere-se priorizar: Vendas do dia, Fluxo de caixa, Contas a pagar/receber, Estoque mínimo.
6. **Tratar o backup como feature de segurança crítica desde o MVP:** Embora seja manual, precisa ser à prova de erros (validar integridade, não permitir sobrescrita acidental, guiar o usuário).
