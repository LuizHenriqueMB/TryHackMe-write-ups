<h1> CTF Daily Bugle | TryHackMe</h1>

- Máquina: Daily Bugle
- Dificuldade: Difícil
- Plataforma: TryHackMe

<h1>Introdução</h1>
O CTF “Daily Bugle” tem como objetivo comprometer uma conta do Joomla CMS via SQLi, quebra de hash e escalação de privilégios via yum.

---

<h1>Reconhecimento</h1>
Iniciamos com uma enumeração do host, utilizando o Nmap a fim de identificar serviços expostos e portas ativas.

```
nmap -sS -sVC -p- 10.201.6.75 --open -T4
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*mNxmyC5JEUQjvmuS2UJpHw.png" width="800" alt="Print 01">

Ao visitarmos a aplicação, nos deparamos com uma noticia do jornal Daily Bugle noticiando que o Spider-Man, havia roubado um banco.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*ngFRtLOLf_9-Z5dwvmbvAw.png" width="800" alt="Print 02">

Em seguida, realizamos uma enumeração de diretórios em busca de arquivos e diretórios ocultos utilizando a ferramenta ffuf

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u <http://10.201.6.75/FUZZ> -t 150
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*kMyQMbENdLR4259ZajIi9Q.png" width="800" alt="Print03">

Ao identificarmos o diretório `/administrator`, navegamos até esse diretório onde notamos que era uma sessão de login do CMS joomla, que exigia credenciais válidas não sendo possível usarmos credenciais padrões.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*B8zvIdrYIqXW_PNHNKW8BQ.png" width="800" alt="Print04">

Em seguida, para identificarmos a versão do Joomla utilizamos a ferramenta WhatWeb:

```
whatweb -a 3 http://10.66.131.128/administrator/
```
<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*5lgIsix3cnImWwwKsk-9NQ.png" width="800" alt="Print05">

Com a versão do `CMS`, descobrimos que essa versão possui a vulnerabilidade `CVE-2017-8917` que permite que um atacante execute comandos `SQL Injection` sem a necessidade de estar autenticado.

<h1>Explorando a Vulnerabilidade</h1>
Para explorarmos essa vulnerabilidade utilizamos a ferramenta joomblah :

```
python3 joomblah.py http://10.66.131.128 
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*Bh36E2Ari3QllGcGSeEMwQ.png" width="800" alt="Print06">

Ao obtermos uma hash Bcrypt no campo ‘`Found user`’, utilizamos a ferramenta Hashcat para quebrar essa hash, e obtivermos a senha do usuário `jonah`.

```
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```
<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*DeuPQhVSzH6Hl1YP1u9bqw.png" width="800" alt="Print07">

Com as credenciais válidas, conseguimos obter acesso ao joomla e posteriormente ao acessarmos a seção templates nos deparamos com dois modelos de template

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*uRNobswz9VDt9PiY-4-N_w.png" width="800" alt="Print08">

Em seguida, acessamos o template `protostar` e após uma análise, identificamos que poderiamos criar um novo arquivo em “New File”, e instalar uma `reverse shell`, colando o código da shell (PentestMonkey):

<img  src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*wVqSxx4RBwekhqjygRAAhQ.png" width="800"  alt="Print09">

Em seguida, realizamos uma requisição através da `URL`, a `shell` foi estabelecida.

```
http://<machine-ip>/templates/protostar/<nome-do-arquivo>.php
```
<img  src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*NWqENIeQWc-ll9d3Hrx6Fg.png" width="800"  alt="Print10">

<h1>Exploração - Obtendo a user flag</h1>
Em seguida, tentei acessar o diretório do usuário jjameson, mas não tinha permissão para acessá-lo
