# CTF Overpass2 | TryHackMe

- Máquina: Overpass2
- Dificuldade: Fácil
- Medium: https://medium.com/@henrique.mb/ctf-overpass2-tryhackme-22dd4e3b50ad

## Introdução
O Overpass foi hackeado! A equipe do SOC (Paradox, parabéns pela promoção) notou atividade suspeita durante um turno da noite enquanto analisava os dados do sistema e conseguiu capturar os pacotes de dados no momento do ataque. Você consegue descobrir como o invasor entrou e invadir o servidor de produção da Overpass?

## Análise do PCAP

Iniciamos a análise do arquivo `.pcap` enviado pela equipe de SOC utilizando a ferramenta `Wireshark`. Para isso aplicamos o filtro de requisição `http.request.method == "POST"` com o objetivo de filtrar todas as requisições `POST` feitas pelo atacante, onde nos possibilitou de identificar a `URL`  da página utilizada pelo atacante `/development/` e o payload foi carregado na página `upload.php`. 

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*jBeW948VXe5d3H5BAvxJcA.png" width="1000" alt=Print01>

Em seguida, encontramos o payload utilizado pelo atacante para criar um shell reverso.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*5ny02jGopUAmh7w88-f-7A.png" width="1000" alt="Print02">

Como o atacante usou a porta `4242` para a conexão via netcat, filtramos todas as requisições feitas por essa porta através do filtro `ip.addr == 192.168.170.145 && tcp.port == 4242`.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*FPIf3tgYBEitkkqHt-Jfkw.png" width="1000" alt="Print03">

Em seguida, conseguimos visualizar todos os comandos usados pelo atacante e  identificar qual foi a senha utilizada pelo atacante para tentar escalar privilégio alternando para o usuário `james`.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*53kWSYap0fgaWXEC6B-Nmw.png" width="1000" alt="Print04">

Nesse mesmo log, encontramos um `backdoor SSH` utilizado pelo atacante para estabelecer persistência no sistema.

```
https://github.com/NinjaJc01/ssh-backdoor
```

Em seguida, identificamos que o atacante acessou o arquivo `/etc/passwd` em busca de outros usuários:

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*992yfyh8hAsls17kfyMJ3Q.png" width="1000" alt="Print05">

Então copiamos os hashes do pacote para utilizar a ferramenta `John The Ripper` para ver quantas senhas podem ser quebradas utilizando a wordlist `Fasttrack`. 

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*7t_AlXarJMVJkv9vM4YGwA.png" width="1000" alt="Print06">

Tirando o usuário `james` conseguimos quebrar `4` senhas.

## Análise do código

Realizamos uma análise mais profunda no `backdoor` e descobrimos que ele possui um hash padrão.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*dENHvcrODZL_hy5UyXCtdQ.png" width="1000" alt="Print07">

E também possui um `salt` fixo presente no código

<img src="https://miro.medium.com/v2/resize:fit:828/format:webp/1*PgQ_dNOLehRmF8uQmZq0yQ.png" width="1000" alt="Print08">

Olhando novamente para o arquivo `.pcap` encontramos o hash utilizado pelo atacante

```
6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*gtrKHMRsOBmpU3vevtiYiQ.png" width="1000" alt="Print09">

Em seguida, com o hash e o SALT enviando para o arquivo **hash.txt** , utilizamos a ferramenta **Hashcat** para descobrir a senha utilizada pelo atacante

```
hashcat -m 1710 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

Senha encontrada:

```
november16
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*e4GYELUkFvXvOQzCEhs7Iw.png" width="1000" alt="Print10">

## Ataque

Agora que o incidente foi investigado, a Paradox precisa de alguém para  assumir o controle do servidor de produção do Overpass novamente.

Iniciei com uma enumeração de portas tcp, utilizando a ferramenta Nmap:

```
nmap -p- -min-rate 1600 -sVC 10.65.139.36 -Pn --open
```

Encontramos as portas:

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*cX0cm_3hVxwULs9pfMc9Iw.png" width="1000" alt="Print11">

Acessando a aplicação web nos deparamos com uma mensagem implantada pelo atacante

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*ZrSJRzi_Ug4sJZ57Z04XpA.png" width="1000" alt="Print12">

Para ganhar acesso ao sistema, acessamos o serviço SSH pela porta `2222`

```
ssh james@10.65.139.36 -oHostKeyAlgorithms=+ssh-rsa -p 2222
```

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*J_PIchlywYtX9wUerBHLGg.png" width="1000" alt="Print13">

Assim que conseguimos acesso, obtemos a user flag no diretório do usuário james

<img src="https://miro.medium.com/v2/resize:fit:786/format:webp/1*pvnKTNZ0tM_coYIywJdKxQ.png" width="1000" alt="Print14">

Para conseguir a root flag, ainda no diretório do usuário `James`, usamos o comando `ls -la` onde encontramos um arquivo oculto com  o bit SUID definido.

<img src="https://miro.medium.com/v2/resize:fit:750/format:webp/1*4juPSLVuGQav0pXsS19GYA.png" width="1000" alt="Print15">

Em seguida, executamos o arquivo `.suid_bash` com o comando abaixo que nos permitiu escalar privilégio root e obter a root flag.

<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*uZlQnpJxfr2aRjMS5VxK4A.png" width="1000" alt="Print16">