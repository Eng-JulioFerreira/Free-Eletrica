# Novidades do Free Elétrica

## v0.4.0

### Circuitos e comandos

- **Criar / editar circuito:** mini-barra com Criar novo | Editar circuito, quadro, número, descrição, esquema e tensão.
  Os pontos clicados ficam em destaque na planta e nada é gravado até confirmar. No modo editar, o circuito vem da lista
  ou de um clique num ponto dele: azul o que já era, verde o que entra, vermelho o que sai.
- **Criar / editar comando** (`ELECOMANDO` contínuo): a próxima letra livre já vem pronta e nunca repete no projeto (ou no
  quadro); depois do `z`, `aa`, `bb`... ou `a1`, `b1`.... Editar um comando existente pela lista ou por um clique num ponto
  dele.
- **O tipo do interruptor manda:** simples só sozinho; num grupo, dois paralelos nas pontas e os intermediários no meio.
  Um grupo que não fecha é recusado.
- **Tensão por ponto:** atributo `TENSAO` nas famílias e nos pontos, com a do quadro quando falta.

### Dimensionamento

- **Dimensionar projeto** em quatro etapas: diagnóstico, circuitos, diagrama unifilar e vista frontal.
- **Diagnóstico** com os avisos de cada circuito e do quadro (tensão divergente por ponto, tensão fora das do quadro,
  alimentador...) e o botão **Ir para o ponto**, que centraliza e destaca.
- **Alimentador:** o disjuntor geral leva o cabo junto, na menor seção com Iz ≥ In (63 A em B1/PVC sobe de 10 para
  16 mm²). A seção pode ser travada à mão numa lista que mostra o Iz de cada seção; travada pequena demais, o aviso traz
  **Corrigir alimentador para ... mm²**.
- **DPS:** classe, tensão Uc, Imax e o disjuntor prévio escolhido entre 10 e 63 A (antes ficava preso em 40 A).
  DR com corrente e sensibilidade por grupo.

### Prancha e materiais

- **Fonte dos textos:** estilo `FREE_ELETRICA`, Arial por padrão, ou qualquer fonte instalada no computador, com busca.
  Os acentos aparecem nas tabelas, no unifilar, na vista frontal, nas legendas e nas tags. O diâmetro sai como `Ø25`.
- **Tags de retorno** só com a letra, sem apóstrofo e sem sobreposição.
- **Eletroduto que sobe e que desce** (`ELE-SOBE`, `ELE-DESCE`), e a simbologia de fiação e eletrodutos na legenda,
  com a do unifilar ao lado.
- **Lista de materiais:** cabos por isolação, seção e cor, com os rolos de 100 m; **Inserir Tabela na Prancha (DXF)** e
  **Exportar para Excel (.csv)**.

## v0.3.0

Primeira versão também para Linux, como AppImage.

### Elétrica

- **Paleta na aba Instalações** dos Conjuntos de ferramentas, com o seletor Elétrica | Telecom, e quadro de cargas acoplado
  embaixo do desenho.
- **Pontos de luz da NBR 5444** (teto e arandela), caixa octogonal de teto e empilhamento de qualquer família na caixa 4x2 ou
  4x4, com busca na mini-barra de pontos.
- **Interruptores paralelos e intermediários** num mesmo comando, com os retornos certos na fiação.
- **Eletroduto na borda das luminárias**; nos outros blocos, até o ponto de inserção. Tracejado curto em metros para a planta
  em 1:50.
- **Dimensionamento:** métodos de instalação A1 a F (tabelas 36 a 39), critério do disjuntor pelo Ib ou pelo Iz, neutro e PE
  pelas tabelas 48 e 58, disjuntor geral e seção do alimentador travados à mão, DPS opcional com classe e disjuntor próprio.
- **Grupos de DR** com um clique e **seleção múltipla** (Ctrl, Shift, Ctrl+A) para editar vários circuitos de uma vez.
- **Prancha:** tabelas em 1 unidade = 1 metro, unifilar com quadro de simbologia, vista frontal do quadro nos trilhos DIN
  (pente ou espinha de peixe, neutro e terra em cima ou nas laterais), legendas elétrica e de telecom.
- **Materiais:** disjuntores por polos e curva, DR, DPS, barramentos e o quadro com a capacidade em módulos DIN.
- **Demanda:** de fábrica, só a tabela oficial da Equatorial; as outras concessionárias o projetista cadastra.

### Telecom (NBR 14565)

- Pontos vinculados ao rack e numerados pela porta, cabeamento em estrela (um cabo inteiro de cada ponto até o rack),
  comprimento 3D com alerta acima de 90 m e eletroduto com 40 % de ocupação.
- Tags de cabeamento com a quantidade, o tipo de cabo e o diâmetro, e Quadro de Telecomunicações acoplado, no padrão do quadro
  de cargas.
- Tabela do rack e cabos, terminações e portas na lista de materiais.

### Documentação

- Guia de Início Rápido (`docs/GETTING_STARTED.md`) com o passo a passo e a referência dos comandos.

## v0.2.0

Primeira versão pronta para uso no Windows: o pacote `.zip` com o executável, os manuais e as licenças.
