# CTF Game Zone | TryHackMe

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/0*2w1d5x5fMiQzqrTS.jpg" width="900" alt="Icon Game Zone">

- Máquina: Game Zone
- Dificuldade: Facíl 
- Plataforma: TryHackMe
- Medium: https://medium.com/@henrique.mb/ctf-game-zone-tryhackme-37d57c7991d6

# Introdução
O CTF “Game Zone”, tem como objetivo a exploração da vulnerabilidade SQL Injection.

---
# Reconhecimento
Iniciando com Iniciando com uma enumeração de portas de comunicação com a ferramenta Nmap, pelo comando:

```
nmap -sVC --open -p- 10.64.184.171 -Pn --min-rate=1600
```

Retornando as ***portas***:

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*ZzLdRYT79AReWGrr9342QA.png" width="900" alt="Print01">

Em seguida, acessamos a aplicação web através da porta HTTP 80 onde nos deparamos com um site de venda de jogos:

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*iaDdinE8kBs0NnEVMoN90w.png" width="900" alt="Print02">

Um fato interessante é que o personagem na capa do site é o “agent47” da franquia de jogos Hitman, e utilizamos no campo de login um payload de SQLi para fazer o bypass da sessão:

```
' OR '1'='1'-- -
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*wygDTr-ykEoSZ138iBTJww.png" width="900" alt="Print03">

Com isso conseguimos bypassar o login, sendo redirecionados ao `portal.php`

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*R474FAYBHhBifYFnedm5ZQ.png" width="900" alt="Print04">

Optamos por utilizar o SQLMap para automatizar o processo de exploração de SQLi, salvando a requisição interceptada via Burp Suite no arquivo `burp_request.txt`:

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*q-6CDqbj3cSJl2G5410smw.png" width="900" alt="Print05">

Em seguida, encontramos a Hash contendo a senha do usuário `agent47`:

```
sqlmap -r burp_request.txt --dbms=mysql --dump --batch
```

<img src="https://miro.medium.com/v2/resize:fit:828/format:webp/1*GBFQ74i9BqGTPlJA6EnUPg.png" witdh="900" alt="Print06">

Para quebrar a hash utilizamos o John The Ripper, obtendo a senha:

```
john hash.txt -w=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*O14ee_5Jpe1hGgBNJtOY_Q.png" witdh="900" alt="Print07">

## Obtendo a user flag
Para obter a user flag, acessamos o serviço de SSH com a credencial obtida:

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*zZBRgU4dzOE2TE8r5WZOxw.png" width="900d" alt="Print08">

Obtemos a flag:

<img src="https://miro.medium.com/v2/resize:fit:786/format:webp/1*uvYfIFcEmdtNAiPEFwxlRw.png" width="900" alt="Print09">

## Obtendo a root flag
Utilizamos o comando `ss -tulpn` para descobrir quantas conexões TCP estão em execução:

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*2QjWtMeU6ByX5bAlw6DV2Q.png" width="900" alt="Print10">

Ao identificarmos que o serviço em execução na porta `10000` está bloqueado por uma regra de firewall externa, utilizamos um túnel SSH para expor a porta: 

```
ssh -L 10000:localhost:10000 agent47@10.64.184.171
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*bxoSF9GzgIZNcDBUHgd8fQ.png" width="900" alt="Print11">

Após isso, acessamos o navegador e digitamos o `localhost:10000` para acessar o servidor web recém exposto que identificamos por se tratar de um `webmin`:

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*KyK3Hy4PM0JO7lKJnlbEIQ.png" width="900" alt="Print12">

Para descobrir a versão do CMS, fizermos o login com a credencial do agent47 e todas as informação apareceram:

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*_VOSIWbQ1KPDBO0qw8FtxQ.png" width="900" alt="Print13">

Com a versão do `CMS` em mãos utilizamos o `Metasploit` para encontrar um payload que seria executado na máquina:

```
exploit(unix/webapp/webmin_show_cgi_exec)
```

<img src="https://miro.medium.com/v2/resize:fit:4800/format:webp/1*hamIGtgbj6quurwlak1RiQ.png" width="900" alt="Print14">

Ao rodarmos o exploit conseguimos acesso como usuário `root`:

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*Fqv7fpfpmahdzwfK_97xyA.png" width="900" alt="Print15">

Em seguida, conseguimos a root flag na raiz do servidor:

<img src="https://miro.medium.com/v2/resize:fit:640/format:webp/1*2fOe3g5aEfIMpivA04WRUQ.png" width="900" alt="Print16">