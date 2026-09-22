## Fase 1 — Preparação

**Propósito:** garantir que o ambiente e o ponto de partida são confiáveis
antes de tirar qualquer conclusão sobre o alvo.

Esta é a fase que a pressa mais tenta pular, e é a que sustenta todo o
resto. Antes de investigar o alvo, preciso confiar na base a partir da qual
investigo. Uma conclusão tirada de um ambiente instável tem aparência de
dado, mas é ruído: se a conexão cai no meio de uma varredura, se o relógio
está dessincronizado, se sobrou resíduo de uma sessão anterior, os
resultados seguintes carregam essa contaminação silenciosamente. O risco
maior está nessa discrição. A ferramenta roda, devolve um resultado
plausível, e o problema de base só aparece horas depois, já com tempo
perdido seguindo uma pista falsa.

A preparação, então, tem menos a ver com montar ferramenta e mais com
estabelecer uma linha de confiança. Toda observação do alvo será
interpretada em cima dessa base.

**Perguntas-guia:**

- O canal até o alvo está de pé e estável ao longo de toda a operação, não
  apenas no primeiro segundo?
- O alvo está de fato respondendo, ou estou apenas assumindo que sim?
  Confirmar custa pouco e evita muito.
- O estado do ambiente está limpo e coerente? Sincronia de tempo, ausência
  de resíduo de sessões anteriores, ferramentas em estado conhecido.
- Se algo soar estranho adiante, esta base é sólida o bastante para eu
  descartá-la como causa de imediato?

## Fase 2 — Reconhecimento

**Propósito:** mapear toda a superfície que o alvo expõe antes de formular
qualquer hipótese de ataque.

O reconhecimento define o roteiro do restante do trabalho. Cada porta
aberta, cada serviço em execução e cada versão identificada é uma porta de
entrada em potencial, e o conjunto delas delimita o campo de jogo. Formar
uma teoria de ataque antes de completar esse mapa leva a fixar num caminho
sem saber se ele é o mais promissor, ou sequer viável.

A ordem correta vai do amplo ao detalhado. Primeiro uma visão geral do que
existe, depois o aprofundamento nos pontos que apareceram. Aprofundar cedo
demais num único serviço tem um custo de oportunidade: enquanto se investe
tempo cavando um alvo, outro serviço mais frágil pode ter passado
despercebido por falta de um olhar completo primeiro.

Vale resistir à tentação de eleger um favorito logo de início. Um serviço
mais familiar atrai a atenção naturalmente, mas familiaridade não é o mesmo
que explorabilidade. O serviço que eu entendo melhor nem sempre é o elo mais
fraco, e o mapa completo é o que revela qual é.

**Perguntas-guia:**

- O que o alvo expõe? Portas, serviços, versões, e qualquer detalhe que a
  identificação revele.
- Cobri a superfície toda, ou parei no primeiro achado interessante?
- Cada serviço encontrado sugere que tipo de interação? Um mesmo mapa pode
  abrir várias linhas de investigação em paralelo.
- Entre o que apareceu, o que tende a ser mais frágil, e não apenas o que me
  é mais familiar?

## Fase 3 — Enumeração

**Propósito:** extrair de cada ponto exposto o máximo de informação possível
antes de tentar qualquer ataque.

Se o reconhecimento responde "o que existe", a enumeração responde "o que
cada coisa me conta". É a fase mais determinante do resultado e, ao mesmo
tempo, a que mais gente atropela por ansiedade de partir logo para o ataque.
Quase toda exploração bem-sucedida foi preparada por uma enumeração
paciente, e quase todo beco sem saída nasce de uma enumeração apressada.

O ponto de partida é reconhecer que cada serviço tem uma forma própria de
ser interrogado. Antes de extrair informação, preciso saber como se conversa
com aquele serviço específico, qual cliente ou abordagem ele espera. Uma vez
estabelecida essa conversa, o objetivo é fazê-lo revelar o quanto der sem
esforço: versões, mensagens de identificação, conteúdo acessível
publicamente, comportamentos padrão que nunca foram ajustados. Muita coisa
que parece exigir um ataque elaborado está, na verdade, à vista de quem
pergunta direito.

A enumeração funciona melhor como um funil. Começo pela visão mais ampla que
o serviço oferece e vou estreitando em direção ao dado concreto, deixando
que cada informação encontrada defina a próxima pergunta. Um nome de usuário
descoberto muda o que faz sentido procurar em seguida; uma versão
identificada abre uma linha de investigação que antes não existia. O
processo é encadeado por natureza, e a disciplina está em segui-lo até o fim
em vez de saltar para uma conclusão na metade do caminho.

**Perguntas-guia:**

- Qual a forma natural de interagir com este serviço? Cada um tem a sua.
- O que ele revela sem resistência? Versão, banners, conteúdo público,
  configuração deixada no padrão.
- Estou indo do geral ao específico, ou pulei direto para o detalhe antes de
  entender o todo?
- O que este achado me permite perguntar em seguida? Cada resposta deveria
  gerar a próxima pergunta.
- Já esgotei o que este serviço tem a oferecer, ou parei no primeiro dado
  útil?

## Fase 4 — Exploração

**Propósito:** converter em acesso concreto a informação reunida nas fases
anteriores.

Este é o momento que costuma ser tratado como o centro do trabalho, embora
na prática seja a consequência natural de uma enumeração bem feita. Quando a
informação foi levantada com cuidado, a exploração tende a se apresentar
quase por dedução. Quando o acesso custa a aparecer, o problema geralmente
não está aqui, e sim numa etapa anterior que foi apressada. Antes de forçar
um vetor difícil, vale voltar e checar se a enumeração realmente se esgotou.

A ordem de tentativa importa. O vetor mais barato vem primeiro: acesso
anônimo, credenciais padrão, uma configuração deixada aberta por descuido.
Recursos assim comprometem um alvo com uma fração do esforço de um ataque
elaborado, e são comuns o bastante para merecerem sempre a primeira
tentativa. Partir direto para o complexo quando o simples resolveria é
desperdício de tempo e ruído desnecessário.

O princípio mais útil desta fase é a desconfiança sobre o que o alvo aceita
como entrada. Todo campo, parâmetro ou ponto de dados que eu controlo é um
lugar onde o sistema pode estar confiando em mim mais do que deveria. A
pergunta central é se aquela entrada é tratada como texto inofensivo ou se,
sob a forma certa, ela consegue alterar o comportamento do sistema por trás.
Boa parte das vulnerabilidades de exploração nasce desse ponto: dados
fornecidos pelo usuário que o sistema interpreta com mais autoridade do que
deveria conceder.

Também é aqui que a mentalidade de cadeia se torna decisiva. Raramente uma
única falha entrega o alvo inteiro. O mais comum é uma sequência de
fraquezas modestas que, isoladas, pareceriam inofensivas, mas encadeadas
abrem o caminho completo. Um achado que não serve para nada sozinho costuma
ser o elo que conecta os outros, e descartá-lo cedo demais é perder a ponte
antes de vê-la. Pelo mesmo motivo, nenhum indicador isolado deveria ser
tratado como veredito: é a combinação de sinais que define se um caminho é
viável.

**Perguntas-guia:**

- Já tentei o vetor mais barato antes do mais complexo? Acesso anônimo,
  padrão, configuração aberta.
- Este ponto de entrada confia cegamente no que eu forneço? Onde há
  confiança, há vetor.
- Esta entrada é tratada como dado inerte, ou pode alterar o comportamento
  do sistema sob a forma certa?
- Estou buscando uma única falha decisiva quando o caminho pode ser uma
  cadeia de fraquezas menores?
- Este achado aparentemente inútil pode ser o elo que liga os outros?

## Fase 5 — Pós-exploração

**Propósito:** consolidar o acesso obtido e usá-lo como ponto de partida
para ir mais fundo.

Conseguir uma primeira entrada raramente é o fim do trabalho. Há uma
diferença importante entre ter tocado o sistema e ter controle sobre ele.
Ler um arquivo que não deveria estar acessível, capturar uma credencial em
trânsito ou obter uma resposta reveladora são conquistas reais, mas ainda
são passos intermediários. O objetivo desta fase é transformar esse contato
inicial em uma posição estável, de onde eu consiga operar com intenção em
vez de depender de uma brecha pontual.

A partir de qualquer acesso, o primeiro reflexo é perguntar o que ele
desbloqueia além de si mesmo. Uma credencial encontrada num lugar tende a
ser reaproveitada em outros, porque a reutilização é uma fraqueza humana
comum e persistente. Uma chave, um token ou até um padrão de nomeação podem
valer muito mais do que o contexto imediato em que apareceram. Todo material
recuperado merece ser examinado sob a pergunta de onde mais ele se encaixa.

Em seguida vem a questão do privilégio. O acesso inicial quase nunca é o
acesso máximo, e o trabalho passa a ser identificar o próximo degrau e o que
leva até ele. Esse movimento repete, em escala menor, todo o ciclo anterior:
uma nova rodada de enumeração a partir da posição recém-conquistada,
observando o que se tornou visível agora que antes não era. Cada nível de
acesso abre uma superfície nova, e a disciplina de reconhecer essa superfície
com o mesmo cuidado da primeira vez é o que separa um acesso raso de um
comprometimento completo.

Por fim, com a posição consolidada, resta localizar o objetivo concreto do
trabalho nos lugares onde ele costuma estar. Chegar até aqui com método faz
dessa etapa uma formalidade, não uma caçada.

**Perguntas-guia:**

- Passei de "toquei o sistema" para "tenho controle sobre ele", ou ainda
  dependo de uma brecha pontual?
- O que encontrei se reaproveita em outro lugar? Credenciais, chaves,
  padrões repetidos.
- Qual o próximo nível de privilégio, e o que me leva até ele?
- A partir desta nova posição, o que ficou visível que antes não estava? Cada
  acesso pede uma nova rodada de enumeração.

## Fase 6 — Documentação

**Propósito:** registrar o processo enquanto ele ainda está fresco,
tratando o registro como parte do trabalho e não como um acréscimo
opcional.

Documentar costuma ser deixado para depois, e "depois" quase sempre
significa com metade dos detalhes já perdidos. O momento de registrar é
durante e logo após, quando o raciocínio ainda está vivo e as razões de cada
decisão continuam claras. Um passo que parecia óbvio no calor da resolução
se torna um mistério poucos dias depois, e reconstruí-lo de memória custa
mais do que teria custado anotá-lo na hora.

O que merece registro não são os comandos em si, mas o raciocínio que levou
a eles. Qualquer pessoa consulta a sintaxe de uma ferramenta; o que tem valor
é o porquê de cada passo ter sido o próximo, como um resultado foi
interpretado, que hipótese uma saída abriu ou fechou. Esse é o conteúdo que
transforma um registro de solução em um registro de pensamento, e é também o
que mais ensina na releitura.

A evidência entra para sustentar a narrativa nos pontos decisivos, não para
ilustrar cada etapa. Um bom registro captura os momentos que mudaram o rumo
do trabalho e descarta o resto, mantendo o foco no fio do raciocínio.

Fechar o ciclo é a última função desta fase. Vale nomear o que este trabalho
deixou de aprendizado transferível, o princípio que sobrevive à máquina
específica e passa a fazer parte do repertório para a próxima.

**Perguntas-guia:**

- Estou registrando enquanto o raciocínio está vivo, ou adiando para quando
  os detalhes já terão esfriado?
- Capturei o porquê de cada decisão, ou apenas os comandos que executei?
- A evidência que guardei sustenta os momentos decisivos, ou só acumula
  telas sem foco?
- O que este trabalho me deixou de transferível para o próximo?

## Apêndice — A ponte Blue Team

Todo este documento descreve o lado ofensivo. O apêndice existe para fazer a
virada que dá sentido a ele dentro de uma trajetória de defesa: cada ação
executada contra um alvo deixa rastro, e reconhecer esse rastro é o trabalho
de quem defende. Praticar o ataque com atenção é também aprender a
antecipar o que o defensor deveria ver. As duas perspectivas descrevem o
mesmo evento a partir de lados opostos da mesa.

O princípio central é que nenhuma ação ofensiva é silenciosa por natureza.
Ela pode passar despercebida por falta de visibilidade, mas a atividade em si
gera sinal. O trabalho defensivo consiste em garantir que esse sinal seja
capturado na origem, chegue até um ponto central de análise e seja
correlacionado com outros para formar uma imagem coerente. Uma detecção
falha quase sempre por uma destas três razões, e vale saber distinguir qual:
o evento nunca foi gerado, foi gerado mas não coletado, ou foi coletado mas
não correlacionado. Cada fase ofensiva ilumina um desses pontos.

**Reconhecimento e enumeração.** A varredura de um alvo produz um volume de
conexões incomum, muitas vezes contra portas que nenhum uso legítimo tocaria,
concentradas numa janela curta de tempo. Esse padrão é uma das assinaturas
mais reconhecíveis de atividade hostil em estágio inicial. Do lado defensivo,
a questão é se o ambiente registra e destaca esse comportamento, ou se ele se
perde no ruído do tráfego normal.

**Exploração.** Quando uma entrada controlada pelo atacante altera o
comportamento de um sistema, isso tende a se manifestar em desvios do padrão
esperado: erros de aplicação incomuns, requisições malformadas, sequências
de acesso que não correspondem a um uso legítimo. A telemetria da própria
aplicação e do sistema que a hospeda é onde esse desvio aparece, desde que
esteja sendo coletada com granularidade suficiente para revelá-lo.

**Pós-exploração.** É a fase que costuma gerar a evidência mais valiosa,
porque envolve autenticação e uso de credenciais. Um acesso bem-sucedido a
partir de uma origem inesperada, uma credencial usada em um contexto que foge
do seu padrão histórico, um privilégio elevado logo após um evento de acesso:
esses são sinais de alto valor justamente porque descrevem o atacante já
operando por dentro. A reutilização de credenciais, tão útil para o ataque, é
também um dos comportamentos mais detectáveis quando há visibilidade de
autenticação entre sistemas.

**A leitura que fecha o ciclo.** Resolver uma máquina do lado ofensivo e
depois percorrer cada passo perguntando "que rastro isto deixou, e o
ambiente o veria?" é um exercício que fortalece as duas competências ao mesmo
tempo. Entender o ataque torna a defesa mais precisa, porque revela onde
olhar; entender a defesa torna o ataque mais consciente, porque revela o que
se está expondo. Essa dupla visão é o que este projeto busca desenvolver.
