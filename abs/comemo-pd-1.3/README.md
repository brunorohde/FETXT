# COMEMO-PD – Sistema de controle e memória modular para Pure Data Vanilla

---

Versão 1.3 (janeiro 2025)

https://github.com/brunorohde/comemo-pd

Copyleft 2025 Bruno Faria Rohde – GPLV3 license – http://www.gnu.org/licenses/gpl.txt

---

## Visão geral

O COMEMO-PD é um sistema de gestão de parâmetros para patchs de Pure Data (Pd) Vanilla, ou seja, sem dependência de bibliotecas externas. O pacote é composto por abstrações que funcionam de modo interligado, possibilitando receber e enviar dados dos parâmetros de um patch seguindo um padrão simples de comunicação, gerenciar a visualização do estado desses parâmetros em um ou mais objetos gráficos na interface, bem como armazenar seus dados em arquivos de texto para recuperação posterior.

Exemplos de projetos que utilizam o sistema são o [\[ARRAST_VJ\]](https://github.com/brunorohde/ARRAST_VJ "[ARRAST_VJ]") e [ESMERIL](https://github.com/brunorohde/ESMERIL "ESMERIL"). Durante o desenvolvimento desses projetos surgiu a necessidade de simplificar o processo de implementação e gestão de parâmetros com uma solução própria, dando origem às abstrações que foram então organizadas e documentadas no COMEMO-PD.

Um exemplo básico de uso é disponibilizado no repositório do projeto **(EXEMPLO.pd)** e será utilizado como ponto de partida, sem esgotar as configurações possíveis através de diferentes arranjos e da interação com recursos externos. Nas sessões seguintes serão comentadas cada uma das abstrações principais no que se refere a suas propriedades e utilização em contexto.

A implementação do COMEMO-PD em uma peça ou software em Pd segue procedimentos simples, que podem ser escalados e complexificados de acordo com a necessidade. No patch de exemplo visto na **FIGURA I** estão dispostos os principais módulos de maneira funcional, ilustrando os recursos descritos adiante.

*FIGURA I - Patch EXEMPLO.pd que demonstra uma implementação básica do COMEMO-PD com suas principais abstrações*


<p align="center">
	<img src="store/EXEMPLO.png" alt="EXEMPLO.pd" width="800"> 
</p>

Após o download do COMEMO-PD no repositório, o primeiro passo é declarar o pacote de abstrações como dependência em seu projeto, utilizando o objeto \[declare -path comemo-pd], considerando que a pasta baixada se encontra no mesmo diretório do patch. Uma vez definidos, mesmo que temporariamente, os parâmetros de controle do patch em questão, são necessárias as seguintes etapas básicas, executadas em qualquer ordem: declaração das abstrações \[param] respectivas a cada parâmetro; declaração da abstração \[memo] responsável pela memória do contexto; e padronização dos símbolos internos de envio e recebimento de dados nos elementos gráficos que representam os parâmetros. Utilizando apenas os métodos próprios dessas duas abstrações teremos um sistema funcional capaz do armazenamento de memórias em arquivos de texto, recurso que resolve a questão de memória para composições e patchs simples, de forma similar a outros projetos disponíveis na comunidade.

Para além dessa configuração básica, o sistema dispõe ainda de um conjunto de abstrações auxiliares – **\[memopack], \[memounpack], \[memolist], \[listbox], \[presets] e \[paramer]** – que permitem o agrupamento e gestão de memórias dispersas, facilitando especialmente o processo de desenvolvimento de novos softwares em Pd. Nessas situações, é comum que os parâmetros não sejam todos conhecidos de antemão e possam estar dispostos em blocos nas diferentes partes do código. Usando o COMEMO-PD, cada um desses blocos pode ser implementado independentemente a qualquer momento e ter sua memória local integrada num macrossistema de gestão. A última abstração incluída, **[paramer]**, também vem para simplificar a implementação geral do sistema em novos projetos e permitir uma gestão mais flexível dos parâmetros de cada memória, como veremos adiante.


## Abstração \[param]

Para cada parâmetro de uma composição ou programa em desenvolvimento deve ser declarada uma instância da abstração \[param], apresentada na **FIGURA II**, que atua como um atom multiuso, sendo seu elemento central um objeto \[list] utilizado como espaço genérico de memória local que permite o armazenamento de qualquer dos tipos básicos de dados no Pd – números, símbolos e listas. A abstração fornece métodos para acesso e edição quente e fria dos dados, tanto local quanto remotamente, seguindo padrões simples de comunicação que podem ser consultados diretamente em seu código ou arquivo de ajuda (param-help.pd). Quando o valor de um parâmetro é alterado ou solicitado pela interface, pelo patch ou pelo sistema de memória, o valor é enviado nas diferentes saídas locais e remotas, e também ocorre a comunicação com um ou mais objetos gráficos (GUI) respectivos ao tipo de dado, possibilitando o isolamento entre interface gráfica e engrenagem do patch, recurso frequentemente necessário.

*FIGURA II - Abstração \[param] que implementa métodos locais e remotos para gestão de um parâmetro em um contexto determinado*

<p align="center">
	<img src="store/param.png" alt="param.pd" height="800">
</p>

A gestão de elementos gráficos (GUI) que representam os parâmetros permite que sejam utilizadas múltiplas caixas de número em diferentes partes do código interno ou da interface relacionadas ao mesmo parâmetro sem conflito e com atualização instantânea em todo o sistema. O mesmo se aplica, evidentemente, a todos os elementos gráficos nativos do Pd, bem como externals que façam uso do mesmo padrão de comunicação via \[send] e \[receive]. Esse é um recurso bastante útil, por exemplo, quando um dado parâmetro precisa ser acessado e exibido de diferentes formas em diferentes partes de uma interface complexa.
Um exemplo detalhado do funcionamento pode ser testado no arquivo de ajuda da abstração, correspondente à **FIGURA III**.

*FIGURA III - Arquivo de ajuda param-help.pd que demonstra os detalhes de utilização da abstração \[param], incluindo seus argumentos de inicialização e os métodos locais e remotos para manipulação de dados no sistema*

<p align="center">
	<img src="store/param-help.png" alt="param-help.pd" width="800">
</p>

## Abstração \[memo]

Todas os \[param] de um contexto enviam seus valores para um \[memo] comum, que é um gerenciador de memória indexada utilizando o objeto \[text]. A abstração \[memo] possui métodos locais e remotos que permitem manipular diretamente seu conteúdo, bem como exportar e carregar arquivos com os dados de todos os parâmetros respectivos. Também é possível configurar um sistema simples de presets obtendo, com o método *dump*, uma lista com os dados na saída esquerda da abstração, a qual pode ser armazenada como mensagem simples e aberta posteriormente enviando o conteúdo de volta à entrada esquerda.
O gerenciador \[memo] normalmente é instanciado junto à declaração dos parâmetros, utilizando até três argumentos, sendo os dois primeiros obrigatórios: nome da memória, número de parâmetros gerenciados e, opcionalmente, nome do contexto de agrupamento superior. Seu funcionamento detalhado é apresentado no arquivo de ajuda correspondente, que pode ser visto na **FIGURA IV**.

*FIGURA IV - Arquivo de ajuda memo-help.pd que demonstra os detalhes de utilização da abstração \[memo], incluindo seus argumentos de inicialização e os métodos locais e remotos para controle de blocos de memória*

<p align="center">
	<img src="store/memo-help.png" alt="memo-help.pd" width="800">
</p>

## Abstrações utilitárias

Além das unidades básicas mencionadas acima, com as quais já é possível implementar mecanismos de presets eficientes, ao longo do tempo foram desenvolvidos alguns utilitários, listados abaixo, que permitem realizar a gestão de sistemas mais complexos, com diferentes graus e contextos de memória:

- \[memopack] e \[memounpack]: Servem para empacotar e desempacotar, respectivamente, listas com os dados de um conjunto de memórias \[memo], que são informadas através de uma lista com seus nomes.

- \[memolist]: Contém um mecanismo de coleta de dados já estabelecido para facilitar a implementação de grupos, e utiliza o terceiro argumento dos módulos \[memo] para defição de um contexto comum de agrupamento.

- \[listbox], \[listbox2] e \[listbox3]: Gestores de memória para listas com visualização e manipulação simples dos dados armazenados. São utilizados no sistema para, por exemplo, armazenar listas de saída de um \[memo] ou \[memopack] na implementação de um sistema de presets. Além de métodos comuns de edição de conteúdo, como copiar, colar, mover e apagar dados, também é possível exportar e carregar aquivos de texto com as informações armazenadas.

- \[presets], \[presets2] e \[presets3]: Como envelope final para o sistema, foram criadas as abstrações \[presets], que apenas empacotam as funcionalidades de um \[memolist] com uma \[listbox] utilizando também o contexto de agrupamento declarado no terceiro argumento dos \[memo]. A utilização de um gestor gráfico de dados [presets] pode ser vista e testada no arquivo **EXEMPLO.pd (FIGURA I)**.

- \[paramer]: Gerador de conjuntos compostos por vários \[param] e o respectivo \[memo], partindo da declaração dos elementos numa lista que pode ser editada e reordenada a qualquer momento. Essa ferramenta reduz muito o trabalho manual de programação a cada vez que novos parâmetros são incluídos, ou mesmo quando desejamos alterar a ordem de declaração dos mesmos, o que impacta na sequência de inicialização do sistema. A utilização do \[paramer] é detalhada e pode ser testada no arquivo de ajuda respectivo, apresentado na **FIGURA V**.

*FIGURA V - Arquivo de ajuda paramer-help.pd que demonstra os detalhes de utilização da abstração \[paramer] para implementação simplificada de estruturas de parâmetros e memórias*

<p align="center">
	<img src="store/paramer-help.png" alt="paramer-help.pd" width="800">
</p>

## Demais pastas (old, proto e store)

- old - arquivo de versões de desenvolvimento do projeto;
- proto - protótipo do sistema de memória utilizando apenas tabelas de números; contém abstrações que podem ser úteis para armazenamento simplificado de dados numéricos;
- store - diretório para armazenar memórias e imagens dos patchs de ajuda e exemplos.



---


## VERSÕES

1.3 (01/2025)

- Alteração no mecanismo de transições: acionado apenas quando carrega presets; qualquer mudança na interface cancela transição no parâmetro


1.2 (09/2024)

- Nova abstração [paramer] - gerador de conjuntos de parâmetros e memória a partir de lista que facilita implementação e gestão do sistema
- Mecanismo para lembrar última pasta navegada ao salvar ou abrir arquivos de memória - [memo] e todos [listbox] atualizados
- Atualização dos arquivos de ajuda (-help.pd)
- Atualização da documentação


1.1 (05/2022) 

- Atualização para Pd 0.52-2
- Suporte à inicialização de listas direto no [param] e edição com novo atom box


1.0 (09/2021)

- Lançamento do pacote e documentação no Github

