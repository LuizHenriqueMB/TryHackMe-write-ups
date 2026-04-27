<h1> CTF Daily Bugle | TryHackme</h1>

- Máquina: Daily Bugle
- Dificuldade: Difícil
- Plataforma: TryHackMe

<h1>Introdução</h1>
O CTF “Daily Bugle” tem como objetivo comprometer uma conta do Joomla CMS via SQLi, quebra de hash e escalação de privilégios via yum.

---

<h1>Reconhecimento</h1>
Iniciei com uma enumeração do host, utilizando o Nmap a fim de identificar serviços expostos e portas ativas.

```sql
nmap -sS -sVC -p- 10.201.6.75 --open -T4
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*mNxmyC5JEUQjvmuS2UJpHw.png" width="1000" alt="Print 01">

Ao visitarmos a aplicação, nos deparamos com uma noticia do jornal Daily Bugle noticiando que o Spider-Man, havia roubado um banco.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*ngFRtLOLf_9-Z5dwvmbvAw.png" width="1000" alt="Print 02aaaaaaaaaaaaaaaaaaaa">

Em seguida, realizamos uma enumeração de diretórios em busca de arquivos e diretórios ocultos utilizando a ferramenta ffuf

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u <http://10.201.6.75/FUZZ> -t 150
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*kMyQMbENdLR4259ZajIi9Q.png" width="1000" alt="Print03">

Ao acessar o diretório `/administrator` identificamos o CMS joomla, que exigia credenciais válidas!

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*B8zvIdrYIqXW_PNHNKW8BQ.png" width="1000" alt="Print04">

Para identificar a versão do Joomla utilizei a ferramenta WhatWeb:

```
whatweb -a 3 http://10.66.131.128/administrator/
```
<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*5lgIsix3cnImWwwKsk-9NQ.png" width="1000" alt="Print05">

Com a versão do `CMS`, identifiquei a vulnerabilidade `CVE-2017-8917` que permite que um atacante execute comandos `SQL Injection` sem a necessidade de estar autenticado.

<h1>Explorando a Vulnerabilidade</h1>
Para explorar essa vulnerabilidade utilizei a ferramenta joomblah:

```
python3 joomblah.py http://10.66.131.128 
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*Bh36E2Ari3QllGcGSeEMwQ.png" width="1000" alt="Print06">

Ao obter uma hash Bcrypt no campo ‘`Found user`’, utilizei a ferramenta Hashcat para quebrar essa hash, e obtive a senha do usuário `jonah`
