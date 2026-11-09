# Documentação técnica do Free Elétrica

A arquitetura implementada. Para desenvolvedores e para quem automatiza o programa. O pedido original está em
[ESPECIFICACAO.md](ESPECIFICACAO.md), e o histórico de decisões e limites em [free-eletrica-2.0.md](free-eletrica-2.0.md).

## Camadas

| Onde | Camada | O quê |
| :--- | :--- | :--- |
| `cadcraft/crates/free-eletrica` | L2 | O motor: pontos, famílias, circuitos, grafo de eletrodutos, NBR 5410, tags, tabela, materiais. Depende só de `cadcraft-doc`. |
| `cadcraft/crates/engine/src/cmd/eletrica.rs` | L4 | Os comandos, com forma JSON e prompt interativo. |
| `cadcraft/crates/ui-egui/src/eletrica.rs` | L5 | A interface: só lê o estado e chama comandos. |

Os pontos de registro no código do CADCraft estão em [arquitetura.md](arquitetura.md).

## Módulos do motor

| Módulo | Responsabilidade |
| :--- | :--- |
| `ponto` | Lê blocos inseridos como pontos elétricos, com os atributos e o XDATA |
| `familia` | `SystemDomain`, `FamilyCategory`, `RefAltura`, `CaixaFisica`, gravados no `BLOCK_RECORD` |
| `biblioteca` | Blocos base, inserção, modelo do ELEPONTO, alinhamento à parede, utilitários do editor de blocos |
| `circuito`, `editar` | Agrupamento em circuitos e todas as escritas no desenho, incluindo `processar_calculo` |
| `rota` | O grafo estrito 3D, `TipoEletroduto`, Dijkstra e a estimativa por Manhattan 3D |
| `eletroduto` | Traçados, caixa de passagem ao dividir um trecho, modelo do ELEELETRODUTO |
| `nbr5410`, `quadro`, `demanda` | Dimensionamento, `TipoCabo`, `Esquema`, `Curva`, configuração de cada quadro, grupos de DR, alimentador, presets e tabelas de demanda |
| `fiacao`, `tabela`, `unifilar`, `frontal`, `materiais`, `inspecao` | Tags, tabela, diagrama unifilar e vista frontal do quadro no desenho, lista de materiais e inspeção visual |
| `setup` | As configurações, nas preferências do programa: catálogos de cabos, perfis de eletroduto e tabelas de demanda próprias |
| `disciplina` | Elétrica (radial), Telecom (estrela) e Alarme de incêndio (laço classe A ou B, preparado): a central, os pontos e por onde o cabo de cada uma passa |
| `telecom` | Cabeamento NBR 14565: vincular e numerar os pontos ao rack, rotear em estrela, comprimento 3D com o limite de 90 m, eletroduto com 40 % de ocupação, tags e tabela do rack |

## O grafo de eletrodutos

- **Nós:** só blocos elétricos. **Arestas:** linhas, polilinhas abertas, arcos e splines abertas nas camadas de eletroduto.
- Cada ponta vale pelo handle gravado (`from_handle`, `to_handle`), se o bloco está no raio de captura; senão, pelo bloco
  mais perto no raio. Sem bloco, o eletroduto fica solto e é apontado.
- Cruzamento em X e derivação em T sem bloco não conectam.
- Cada trecho é uma entidade só: `LINE` no reto, `ARC` circular no arco (com desvios, arcos numa `LWPOLYLINE`) e `LWPOLYLINE`
  no 45°. Nunca `SPLINE`. Inserir um trecho não divide nem altera nenhum outro.
- Um bloco a até 5 cm do traçado de um eletroduto já é uma caixa: `eletroduto::encaixar` corta o trecho em `A -> bloco` e
  `bloco -> B`, mantendo a geometria (um arco vira dois arcos do mesmo círculo). Vale ao inserir, traçar, dimensionar,
  atualizar e ligar a inspeção. Clicar num bloco perto de um trecho não o corta.
- Módulos empilhados: cada tomada e cada tecla é um bloco. Um ponto de parede inserido a até 10 cm de outro com caixa 4x2
  (ou `eletrica.ponto.empilhar`) vai para o bloco `<nome>$EMP`, sem a caixa, encostado no topo do último símbolo, e grava
  `host_point_handle`. No grafo ele fica na caixa base. Nos materiais, a pilha conta uma caixa por altura.
- Comprimento da aresta = comprimento do traçado em planta + |cota - Z de A| + |cota - Z de B|. A cota é o pé-direito no
  teto, 0 no piso e o Z do bloco A na parede.
- Sem caminho até o quadro, o comprimento é estimado por Manhattan 3D e marcado com `~`.

## Separação de etapas

Lançar pontos, circuitos e eletrodutos só grava os dados do projeto. `eletrica.dimensionar` roteia, dimensiona, grava o
resultado de cada circuito em `DIM.<circuito>` e atualiza tags, tabelas e diagramas. Depois disso, um circuito cujo cálculo atual difere
do gravado aparece como desatualizado. A ficha e a inspeção mostram o cálculo atual como prévia, só com a janela ou a
inspeção abertas.

## Dados no DXF

Aplicativo do XDATA: `FREE_ELETRICA`, com pares `chave=valor` em grupos 1000. Os nomes antigos (`FREEELETRICA`, `CRAFTELE`)
continuam sendo lidos.

| Onde | Chaves |
| :--- | :--- |
| `BLOCK_RECORD` | `schema_ver`, `domain`, `category`, `height_ref`, `fixed_elevation_m`, `physical_box`, `power_va`, `power_factor`, `voltage_v`, `keys`, `demand_class`, `demand_factor` |
| Ponto (`INSERT`) | Atributos do bloco, incluindo `TIPO_CARGA` e `FATOR_DEMANDA`; `elevation_m` fora da legenda; `target_handles.<letra>` no interruptor |
| Quadro (`INSERT`) | `SISTEMA_QUADRO`, `TENSAO_NOMINAL`, `CABO_PADRAO_QUADRO`, `ALIMENTADO_POR`, `DISJ_GERAL`, `SECAO_GERAL`, `USAR_DPS`, `CLASSE_DPS`, `DISJ_DPS`, `METODO_QUADRO`, `MODULOS_TRILHO`, `BARRAMENTO_FASE`, `BARRAMENTO_NT` (de versões anteriores: neutro e terra ficam sempre nas laterais), `POLOS_GERAL`, `CURVA_GERAL`, `CABO_GERAL`, `TENSAO_DPS`, `IMAX_DPS`, `POLOS_DPS`; `DRIN.` e `DRSENS.` + grupo de DR (corrente e sensibilidade); `DESC.`, `SECAO.`, `DISJ.`, `CURVA.`, `CABO.`, `CABONOME.`, `ESQUEMA.`, `TENSAO.`, `FASE.`, `DR.`, `DIM.`, `METODO.` + circuito |
| Eletroduto | `schema_ver`, `from_handle`, `to_handle`, `install_elevation`, `conduit_type`, `conduit_profile`, `nominal_diameter_mm`, `manual_diameter_lock`, `passing_wires.<n>`, `tag_handle` |
| Tag de fiação | `schema_ver`, `parent_conduit_handle`, `anchor_x`, `anchor_y`, `user_offset_x`, `user_offset_y` |
| Ponto de telecom (`INSERT`) | `RACK_PAI` (handle do rack, no XDATA; `-` desvinculado à mão); `ID_PONTO` (o número da porta no rack), `TIPO_CABO` e `QTD_CABOS` no atributo ou no XDATA; `DESC_PORTA` (a descrição no quadro de telecom); `telecom_link_m` (comprimento do último cálculo) |
| Eletroduto de telecom | `telecom_cables.<n>` (`ID:cabo:quantidade;...`) e `nominal_diameter_mm` |

O tipo de linha e a cor do perfil ficam na própria entidade do eletroduto. O diagrama unifilar é o bloco
`UNIFILAR_<handle do quadro>`; a vista frontal, `QUADRO_FRONTAL_<handle do quadro>`; a tabela, `QUADRO_TABELA_<handle do quadro>`; a tabela do rack, `TELECOM_RACK_<handle do rack>`; a lista de materiais, `LISTA_MATERIAIS`; e as legendas, `LEGENDA_SIMBOLOS_ELETRICA` (com a simbologia do unifilar ao lado, `LEGENDA_SIMBOLOGIA_UNIFILAR`, e a de fiação e eletrodutos embaixo, `LEGENDA_FIACAO_ELETRODUTOS`) e `LEGENDA_SIMBOLOS_TELECOM`.

As configurações do Setup, incluindo PE compartilhado, FCA fixo, catálogos, perfis e tabelas de demanda próprias, ficam nas
preferências do programa, não no DXF.

### Fonte dos textos

Tudo o que o programa escreve no desenho (tabelas, unifilar, vista frontal, legendas, tags, lista de materiais e os
atributos dos blocos de fábrica) usa o estilo de texto `FREE_ELETRICA`. A fonte dele é a de `fonteTextos` nas configurações
(o nome do arquivo: `arial.ttf` é o padrão; `calibri.ttf`, `tahoma.ttf`, `isocp.shx` ou qualquer TrueType/OpenType
instalada, listada por `eletrica.fontes`). Os valores de versões anteriores (`arial`, `isocp`...) continuam valendo; `simplex`,
vetorial e sem acentos, volta como Arial. Só o nome do arquivo vai para o DXF, nenhuma fonte vem com o programa. Cada
gerador e o ELEREGEN acertam o estilo, e os textos já desenhados mudam junto. O diâmetro das tags é o `Ø` (U+00D8), que
toda fonte tem; o `%%c` do CAD vira `⌀` (U+2300), que falta em Arial, Calibri e Tahoma.

### Letras de comando

O `COMANDO` de um interruptor guarda um comando por tecla: `ab` são as teclas `a` e `b` (como sempre), e depois do `z`
um comando é a mesma letra repetida (`aa`, `bbb`) ou letra e número (`a1`), separados por vírgula num interruptor de várias
teclas (`aa,b`). A próxima letra olha todos os interruptores e luminárias do escopo das configurações (`escopoComandos`:
`global` ou `porQuadro`; `sufixoComandos`: `letrasRepetidas` ou `letraNumero`). Gravar um comando (`editar::aplicar_comando`)
tira a letra de quem a tinha no escopo e ficou fora da seleção: é assim que um comando existente é editado. Os papéis
(`switch_role.<letra>`) vêm do tipo do bloco (`editar::papeis`): simples só sozinho; num grupo, dois paralelos nas pontas
(o primeiro clicado é o início) e os intermediários no meio; o grupo que não fecha é recusado. Um bloco sem tipo segue a
posição.

O ELECOMANDO e o ELECIRCUITO montam a seleção fora do desenho (`circuito::SelecaoComando`, `circuito::SelecaoCircuito`,
por desenho) e só gravam ao confirmar (`editar::aplicar_comando`, `editar::aplicar_circuito`). No modo Editar, o primeiro
clique num ponto que já tem comando (ou circuito) carrega o grupo dele; depois cada clique tira ou põe. Gravar um circuito
põe os marcados nele e tira os desmarcados que ele tinha; a descrição, o esquema e a tensão vão para o quadro junto.

### Diagnóstico do quadro

`eletrica.diagnostico` junta a inspeção do quadro (pontos sem circuito, fora da rede, queda, ocupação, eletrodutos soltos)
com os avisos de cada circuito e do quadro, cada um no ponto a que se refere: a tensão divergente em cada ponto que
diverge do circuito, a tensão fora das do quadro, os outros avisos do circuito no primeiro ponto dele e os do quadro
(alimentador, fases, DPS) no próprio quadro.

O disjuntor geral travado com a seção automática leva o cabo do alimentador à menor seção com Iz ≥ In (5.3.4.1); com a
seção também travada e pequena demais, o alimentador avisa e sugere a menor que comporta (`secaoSugerida`).

### Lista de materiais

Os cabos são medidos condutor a condutor em cada trecho de eletroduto (planta e desníveis até os blocos), vezes a folga de
materiais, e agrupados por (tipo de cabo, seção, função): cada fase que o circuito usa no quadro (R, S, T) na cor dela, o
neutro azul-claro (NBR 5410, 6.1.5.3.1), os retornos que passam no trecho e o condutor de proteção verde ou verde-amarelo
(6.1.5.3.2), um só por trecho quando é compartilhado. As cores das fases e do retorno ficam nas configurações. Um circuito
sem eletrodutos até o quadro entra pelo percurso estimado, sem os retornos, com aviso.

`eletrica.materiais.exportar` grava a lista como planilha CSV para o Excel em português: UTF-8 com BOM, `;` entre as
colunas, vírgula decimal e CRLF, com as colunas Categoria, Item / Descrição, Isolação / Especificação, Cor / Função,
Unidade e Quantidade. O arquivo fica sempre em `.csv`.

Os blocos `ELE-SOBE` e `ELE-DESCE` (eletroduto que sobe e que desce, NBR 5444: círculo com a seta saindo ou entrando) são
nós do grafo como uma caixa (`TIPO_PONTO = CAIXA DE PASSAGEM`), sem caixa física na lista de materiais; o que sobe fica
na altura do teto e o que desce no piso.

## Comandos

Todos respondem em JSON pelo canal de controle, pelo `cadcraft-cli` e pelo MCP. Os parâmetros exatos de cada um estão no
próprio `CommandSpec`.

| Grupo | Comandos |
| :--- | :--- |
| Pontos | `eleponto`, `eletrica.ponto.modelo`, `eletrica.ponto.altura`, `eletrica.ponto.tensao`, `eletrica.ponto.demanda`, `eletrica.ponto.excluir`, `eletrica.blocos`, `eletrica.bootstrap` |
| Famílias | `eletrica.familia.ler`, `.definir`, `.geometria`, `.atributos`, `elefamilia_conector` |
| Atributos | `eletrica.atributos`, `eletrica.atributo.definir` |
| Circuitos | `elecircuito`, `eleremoverponto`, `elecomando`, `eletrica.comando.selecao` (a mini-barra do ELECOMANDO), `eletrica.circuito.selecao` (a do ELECIRCUITO), `eletrica.circuito.criar`, `.definir`, `.descrever`, `.juntar`, `.excluir`, `.alvo` |
| Eletrodutos | `eleeletroduto`, `eletrica.eletroduto.modelo`, `.dividir`, `.definir`, `eletrica.eletrodutos` |
| Cálculo | `eletrica.dimensionar` (com `unifilar` e `frontal`), `eletrica.diagnostico`, `eletrica.rotear`, `eletrica.sobrescrever`, `eletrica.quadro`, `eletrica.dr.definir`, `eletrica.dr.excluir`, `eletrica.quadro.equilibrar`, `eletrica.cargas`, `eletrica.demanda.tabelas`, `.salvar`, `.excluir`, `eletrica.setup`, `eletrica.fontes` |
| Documentação | `elefiacao` (com `ancora`: direto no eletroduto; com `tag` também: linha de chamada), `eletrica.fiacao.modo`, `eleregen`, `elequadro_tabela`, `eletrica.unifilar`, `eleunifilar`, `eletrica.quadro.frontal`, `elequadro_frontal`, `eletrica.documentacao.alvo` (o quadro ou rack da mini-barra), `elelegenda`, `eletrica.materiais`, `eletrica.materiais.exportar`, `elemateriais_tabela`, `eletrica.inspecionar`, `eletrica.analisar` |
| Telecom | `televincular`, `telerotear`, `eletrica.telecom`, `eletrica.telecom.ponto`, `telefiacao` (os mesmos modos do `elefiacao`), `teletabela`, `telelegenda` |

Comandos que só existem na interface, porque abrem janelas: `ELETRICA`, `ELECONFIG`, `ELEPONTOS`, `ELEBLOCOS`, `ELEATRIBUTOS`,
`ELECARGAS`, `ELEROTEAR`, `ELEDIMENSIONAR`, `ELEMATERIAIS`, `ELECONFERIR`, `ELEINSPECAO`, `TELEVDI`. `ELETRICA` abre a aba
Installations das Tool Sets; `ELECARGAS` e `ELEROTEAR` acoplam o quadro de cargas embaixo do desenho, e `TELEVDI`, o quadro VDI /
rack no mesmo lugar, até ser fechado.

## Compilar e empacotar

O ambiente de compilação e os portões de qualidade estão em [CLAUDE.md](../CLAUDE.md).

| Comando | O quê |
| :--- | :--- |
| `scripts\package_release.ps1` | Compila em release e gera `dist/FreeEletrica/` e `dist/FreeEletrica-v<versão>-win64.zip` |
| `cargo xtask dist --versao 0.3.0` | O mesmo, dentro de `cadcraft/`, com o ambiente carregado |
| `scripts/package_appimage.sh` | Num Linux x86_64: compila em release e gera `dist/FreeEletrica-v<versão>-x86_64.AppImage` |
| Workflow `Release` (`.github/workflows/release.yml`) | Gera os dois pacotes no GitHub Actions (Windows e Ubuntu 22.04) e cria a Release como rascunho; é disparado à mão |
