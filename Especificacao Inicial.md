# Especificação Inicial

## Tema

Sistema de Gestão de Produção de Mobiliário por Medida

## Identificação dos Autores

| Nº Aluno | Prática | Nome | Email |
|---|---|---|---|
| 2025140643 | P2 | Gabriel Costa | a2025140643@isec.pt |
| 2025157930 | P1 | Daniel Ramada | a2025157930@isec.pt |
| 2025138938 | P1 | Afonso Santos | a2025138938@isec.pt |
| 2026115033 | P8 | Gustavo Leite | a2026115033@isec.pt |

## Identificação do Esforço

| Tarefa | Gabriel Costa | Daniel Ramada | Gustavo Leite | Afonso Santos |
|---|---|---|---|---|
| Análise da situação atual | 25% | 25% | 25% | 25% |
| Modelo ER | 37.5% | 37.5% | 0% | 25% |
| Modelo Físico E/R | [a preencher] | [a preencher] | [a preencher] | [a preencher] |

## Objetivo

O objetivo deste projeto é implementar uma base de dados relacional que suporte todo o ciclo de vida de uma encomenda de mobiliário por medida, desde o pedido do cliente e o orçamento até à entrega, à faturação e ao pagamento. Pretende-se, em concreto:

- registar clientes, orçamentos, encomendas, faturas, entregas e pagamentos numa única estrutura ligada;
- descrever cada móvel pelas suas medidas e por uma árvore de componentes (peças, sub-peças, matérias-primas e ferragens);
- controlar o stock por armazém e as compras a fornecedores, distinguindo as compras feitas para uma encomenda concreta das compras preventivas;
- acompanhar as fases de produção de cada produto, com as máquinas usadas, o material consumido e o trabalho dos empregados;
- garantir, através de restrições e regras de negócio, a coerência dos dados (stock, custos, datas, estados e horários).

## Situação Atual

A gestão da produção de mobiliário por medida na oficina é feita de forma inteiramente manual, sem qualquer sistema informático de suporte. Um único produto, por exemplo uma secretária customizada, obriga a conjugar componentes de origens totalmente diferentes: uma estrutura metálica encomendada a um fornecedor externo, ferragens adquiridas por catálogo e um tampo de madeira maciça cortado, lixado e envernizado na própria oficina.

O circuito de uma encomenda decorre hoje da seguinte forma. O cliente faz um pedido e a oficina responde com um orçamento calculado de forma aproximada, com base na experiência do responsável e em preços de materiais que podem estar desatualizados. Depois de aprovado, o pedido passa a encomenda, que pode incluir vários móveis (por exemplo, uma secretária e uma estante). Cada móvel é produzido por fases (corte, montagem, lixagem, verniz, entre outras), em máquinas diferentes e por vários empregados. Os materiais saem do stock dos armazéns ou, quando não existem, são comprados a fornecedores: umas vezes especificamente para essa encomenda, outras de forma preventiva, para repor stock. No final há a entrega, a fatura e o pagamento do cliente, e a oficina paga ainda aos fornecedores e aos empregados.

Todo este circuito é registado em apontamentos em papel, folhas de cálculo isoladas e comunicação direta com fornecedores, sem ligação entre si. O acompanhamento de que materiais foram usados em que encomenda, de que fornecedor veio cada lote e de em que fase se encontra cada produto depende inteiramente da memória dos funcionários. Com vários produtos em simultâneo, este método torna-se insustentável à medida que o volume de trabalho cresce.

## Problemas / Melhorias

O principal problema é a perda de controlo do processo produtivo e a falta de rastreabilidade. Concretamente:

1. **Falta de rastreabilidade de materiais e lotes.** É muito difícil saber que material foi usado em que encomenda e qual o fornecedor que entregou um lote com defeito, o que impede a oficina de identificar rapidamente todas as encomendas afetadas.
2. **Estado da produção desconhecido.** Não é fácil saber em que fase se encontra um produto, nem se uma encomenda está realmente pronta para entrega.
3. **Stock desatualizado e disperso.** As contagens são manuais e o material está em vários armazéns, sem visão global. A falta de material só é descoberta em plena produção, o que leva a compras em duplicado ou a fases que não podem começar.
4. **Compras desligadas das encomendas.** Não se distingue a compra feita para uma encomenda concreta da compra preventiva para stock, nem se sabe quando o material chega, pelo que os atrasos não são previstos.
5. **Orçamentos sem base de custos.** O preço de um móvel é definido sem somar o custo real dos seus componentes, havendo o risco de vender abaixo do custo.
6. **Mão de obra mal registada.** As horas trabalhadas são anotadas em papel, não se sabe quem executou cada fase e o mesmo empregado pode ficar registado em dois trabalhos ao mesmo tempo.
7. **Informação comercial e financeira dispersa.** Orçamentos, faturas, entregas e pagamentos de clientes, fornecedores e empregados não estão ligados à encomenda a que dizem respeito.

A melhoria passa pela implementação de uma base de dados que estruture o cruzamento destes dados, ligando diretamente as encomendas aos produtos, aos materiais, aos fornecedores e aos empregados responsáveis.

## Soluções Propostas

A solução assenta numa base de dados relacional com 18 entidades e 22 relacionamentos, organizados em cinco grupos:

- **Pessoas:** CLIENTE, EMPREGADO, FORNECEDOR.
- **Produto:** PRODUTO. Todos os móveis são feitos por medida, descritos pelas suas medidas (altura, largura, profundidade), pelo acabamento e pelo preço.
- **Materiais:** COMPONENTE, ARMAZEM.
- **Ciclo comercial:** ORCAMENTO, ENCOMENDA, FATURA, ENTREGA, ENCOMENDA_FORNECEDOR, PAGAMENTO_CLIENTE, PAGAMENTO_FORNECEDOR, PAGAMENTO_EMPREGADO.
- **Produção:** FASE_PRODUCAO, TIPO_FASE, TRABALHO, MAQUINA.

O SAP PowerDesigner não representa atributos multivalor, por isso, no modelo desenhado nessa ferramenta, os telefones dos clientes e dos fornecedores e as especialidades dos empregados passam a entidades próprias (TELEFONE_CLIENTE, TELEFONE_FORNECEDOR e ESPECIALIDADE_EMPREGADO). Nesse modelo há, assim, 21 entidades e 25 relacionamentos.

A ENCOMENDA é o centro do modelo: é a lista de todos os produtos que o cliente pretende e liga-se ao cliente, ao empregado que a vendeu, ao orçamento, à fatura, ao pagamento e à entrega. Cada problema identificado tem resposta no modelo:

| Problema | Solução proposta |
|---|---|
| 1. Rastreabilidade | O lote, a quantidade e o preço de cada compra ficam registados nos relacionamentos de abastecimento, ligando fornecedor, componente e, nas compras dirigidas, a encomenda. Permite listar as encomendas afetadas por um lote defeituoso. |
| 2. Estado da produção | Cada produto tem as suas fases de produção, com datas e estado, e a encomenda tem o seu próprio estado (Em orçamento, Aprovada, Em produção, Pronta, Entregue). |
| 3. Stock disperso | O stock é guardado por componente e por armazém, e cada fase regista o material que consome, com a respetiva quantidade. |
| 4. Compras desligadas | Dois ternários separam a compra dirigida a uma encomenda da compra preventiva para stock, e cada compra tem data de entrega e estado. |
| 5. Orçamentos sem custos | Cada produto tem uma árvore de componentes com quantidades e cada compra regista o preço unitário, o que permite calcular o custo e compará-lo com o preço. |
| 6. Mão de obra | Cada trabalho regista o empregado, a fase, o dia e as horas de início e de fim, e cada empregado pode ter um chefe de equipa. |
| 7. Informação dispersa | Orçamento, fatura, pagamento e entrega ligam-se diretamente à encomenda, e as compras e os pagamentos a fornecedores e a empregados ligam-se às respetivas entidades. |

A nível estrutural, o modelo suporta:

- **Relacionamentos 1:1:** ENCOMENDA com ORCAMENTO, FATURA, PAGAMENTO_CLIENTE e ENTREGA; ENCOMENDA_FORNECEDOR com PAGAMENTO_FORNECEDOR.
- **Relacionamentos 1:N:** CLIENTE e EMPREGADO (vendedor) com ENCOMENDA; ENCOMENDA com PRODUTO; PRODUTO e TIPO_FASE com FASE_PRODUCAO; FASE_PRODUCAO e EMPREGADO com TRABALHO; FORNECEDOR com ENCOMENDA_FORNECEDOR; EMPREGADO com PAGAMENTO_EMPREGADO.
- **Relacionamentos M:N:** PRODUTO com COMPONENTE; FASE_PRODUCAO com COMPONENTE (material consumido) e com MAQUINA; COMPONENTE com ARMAZEM (com o stock como atributo).
- **Relacionamentos recursivos:** COMPONENTE compõe COMPONENTE (M:N, a árvore de componentes) e EMPREGADO supervisiona EMPREGADO (1:N, chefe de equipa).
- **Relacionamentos ternários (M:N:P):** `abastece`, entre ENCOMENDA, COMPONENTE e ENCOMENDA_FORNECEDOR, para a compra dirigida a uma encomenda; `reabastece`, entre ENCOMENDA_FORNECEDOR, COMPONENTE e ARMAZEM, para a compra preventiva de reposição de stock.
- **Entidade fraca:** FASE_PRODUCAO depende de PRODUTO, sendo identificada pelo produto e pelo número da fase.
- **Participações obrigatórias:** por exemplo, uma ENCOMENDA tem sempre orçamento e pelo menos um produto; cada PRODUTO tem pelo menos uma fase e utiliza pelo menos um componente; cada TRABALHO pertence a uma fase e a um empregado.

## Lista de Termos Técnicos

- **Árvore de Componentes:** estrutura hierárquica que define as peças, sub-peças e matérias-primas necessárias para construir um produto final.
- **Componente:** qualquer material ou peça usada na produção (madeira, ferragem, verniz, pé metálico, peça composta), classificado por tipo e unidade de medida.
- **Produto por medida:** móvel produzido segundo as medidas pedidas pelo cliente, com fases de produção e lista de componentes próprios.
- **Encomenda:** pedido do cliente, que reúne todos os produtos pretendidos e passa pelos estados Em orçamento, Aprovada, Em produção, Pronta e Entregue.
- **Fase de Produção:** passo de tratamento ou montagem de um produto (corte, montagem, lixagem, verniz), de um determinado tipo de fase, com datas e estado.
- **Registo de Trabalho:** período de trabalho de um empregado numa fase, com dia e horas de início e de fim.
- **Abastecimento Dirigido (ternário):** compra a um fornecedor de um componente destinado a uma encomenda específica.
- **Reabastecimento Preventivo (ternário):** compra a um fornecedor de um componente para repor o stock de um armazém, sem encomenda de cliente associada.
- **Lote:** conjunto de material recebido numa mesma compra, identificado para efeitos de rastreabilidade.
- **Stock:** quantidade disponível de um componente num armazém.

## 5 Regras de Negócio

1. **Validação de Stock:** o stock de um componente num armazém não pode assumir valores negativos após a alocação ou o consumo numa fase de produção.
2. **Custo do Produto:** o preço de um produto tem de ser obrigatoriamente superior à soma do custo de compra de todos os componentes da sua árvore, tendo em conta as quantidades. O custo de compra de um componente é o preço unitário da sua compra mais recente; se o componente ainda não foi comprado, usa-se o seu custo de referência.
3. **Cronologia de Produção:** a data de início de uma fase de montagem ou tratamento não pode ser anterior à data de entrega, pelo fornecedor, do material associado.
4. **Estado da Encomenda:** uma encomenda apenas pode transitar para o estado "Pronta" se a totalidade dos produtos nela incluídos tiverem todas as suas fases de produção com o estado "Concluída".
5. **Exclusividade de Alocação de Tempo:** um empregado não pode ter registos de trabalho sobrepostos, no mesmo dia e no mesmo bloco de horas, em fases de produtos diferentes.
