---
title: "Trilha de Vulnerabilidades Server-Side - Portswigger academy"
date: 2026-04-27 00:00:00 -0300
categories: [write-up]
tags: [vulnerability, offsec]
---
<p>
<a href="assets\img\server-side-vuln-pts\hxh_burp_fade2.png" class="popup img-link"> 
<img src="assets\img\server-side-vuln-pts\hxh_burp_fade2.png" alt="Introduction Image" width="980" height="380" loading="lazy">
</a>
</p>
Write-up da trilha de Server-side Vulnerabilites da Portswigger, demonstração prática e explicação sobre os assuntos demonstrados nas aulas.

>Ferramentas que iremos utilizar nesses laborátorios: **BurpSuite + F12 (Inspecionar)**

<a href="https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice">
	Portswigger academy
</a>

## Path traversal
---

É um ataque de server-side onde é possivel a exfiltração de dados a partir de uma mudança no caminho de uma chamada vulneravel no código.
```
<div class = images>
	...
	<img src = "getImage(image?filename=/123.png)">
	...
</div>	
```

Chamada do código buscando uma imagem normalmente em um diretorio /var/www/images
Agora como não há nenhuma sanitização ou defesa neste exemplo de código, podemos somente requisitar novamente essa imagem descendo os diretorios até a pasta root e pedir outra coisa invés dela.
```
<!-- Unix server like -->
	...
	<img src = "getImage(image?filename=/../../../etc/passwd)">
	...

<!-- Windows server like -->
	...
	<img src = "getImage(image?filename=\..\..\..\win\win.ini)"> <!-- aceita barra normal também -->
	...
```


## Acess control
---

Acess control é um controlador de sessões de usuarios para que seja possivel o usuario acessar somente sua determinada página web, mas se mal programa é possivel acessar páginas de outros tal qual usuario administrativo.

### Vertical privilege escalation

Meio de ataque onde o usuario malicioso consegue acessar outro usuario privilegiado diretamente, através de uma chamada ou mudança no url, com isso acessando todo o painel administrativo.

### Request parameter

Ataque por meio da mudança um parametro em uma chamada

### Horizontal privilege escalation

É um meio de ataque onde um por exemplo um colaborador que tem um ID, pode acessar a sessao/conta de outro usuario apenas trocando seus IDs.
Isso normalmente é resolvido passando um GUID unico para cada colaborador inves de um numero subsequente e não deixando exposto essas inforamações nas chamadas de getUser() ou getInfoUser()

Nesse exemplo é deixado uma vulnerabilidade para fazermos isso:\
>*"To solve the lab, find the GUID for `carlos`, then submit his API key as the solution.
>You can log in to your own account using the following credentials: `wiener:peter`"*

Acessando a conta, estamos de cara com nosso usuario e nossa API key então presumo que ao final devemos estar nessa página.
![[Pasted image 20260427105203.png]]

Explorando o home:
Alguns posts estão disponíveis para ler dentro desse blog, lendo alguns procurei se tinha a principio algum comentário mas não, inspecionando a página ja notei por onde seria possível achar o GUID do usuário.

![[Pasted image 20260427105343.png]]

GUID do Adminstrador achado facilmente, mas não queremos esse e sim do **carlos**. então procurando algum post postado pelo carlos e é possível fazer a mesma coisa e achar o dele também.
Copiamos o ID e voltamos para a página onde queremos a API key, na URL é possível notar um parametro chamando o id do user, fazemos a mudança e pronto.

![[Pasted image 20260427105709.png]]


### Horizontal to Vertical

Como já foi visto antes, agora podemos ir de um ataque horizontal e pegar um usuario com privilegios administrativos que vai deixar a gente fazer ainda mais coisas dentro de seu site.

Problema:\
>*"To solve the lab, retrieve the administrator's password, then use it to delete the user `carlos`.
>You can log in to your own account using the following credentials: `wiener:peter`"*


Após realizar o login ja é possivel ver uma chamada na URL diretamente pelo nome do usuario, vamos tentar o mais básico primeiro acessar somente mudando esse parametro para administrador.

![[Pasted image 20260427110447.png]]

Trocado o parametro

![[Pasted image 20260427111128.png]]

Acesso direto ao painel do usuario privilegiado, além de pelo inspecionar conseguimos pegar sua senha agora vamos deletar o usuario e continuar em nossa trilha.

![[Pasted image 20260427112114.png]]


# Authentication
---

Autenticação != Autorização
Autenticação é o site tentando verificar se você realmente é quem diz ser e a autorização é os recursos que essa conta vai ter no website, ver blogs, editar, ver dados dos usuarios et al.

-> terminar em casa

## SSRF (Server-side request forgery)
---

É um ataque onde o agente malicioso vai tentar se conectar diretamente ao server fazendo uma requisição, na parte vulnerável como nesse exemplo que ao invés de ser chamado uma API para buscar o estoque de items em site, ele vai retornar a pagina /admin liberando seu acesso.

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded 
Content-Length: 118 

stockApi=http://localhost/admin
```

Mas muitas vezes é read-only, sendo as modificações possiveis somente para usuarios autenticados e com autorização. Mas caso a pagina esteja rodando diretamente nesse servidor acessando o localmente vai fazer bypass em todas em barreiras do front-end.

### Basic SSRF in a local server

Problema:\
>"*This lab has a stock check feature which fetches data from an internal system.
>To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`"*

Para resolver isso vamos focar nessa dica, que ele passou no problema acessar o stock e mudar a requisição que ele faz nessa checagem.
Acessamos o serviço e procuramos a chamada dentro do codigo.

![[Pasted image 20260427132512.png]]

Achamos. Agora vamos colocar o localhost/admin, e acessar o painel administrativo.

![[Pasted image 20260427132825.png]]

Acesso feito agora só excluir o usuario que foi requisitado carlos.

![[Pasted image 20260427133006.png]]

Vixi nao deu certo por conta da autenticação, acabei esquecendo disso então vamos fazer na mão então igual os antigos faziam na idade das pedras.
Na URL é possivel notar que ainda está a chamada "delete?username=<>", vamos pegar essa chamada e colocar diretamente na chamada de stock onde fizemos a primeira requisição pois la vamos estar diretamente na máquina.

![[Pasted image 20260427133413.png]]

Agora sim, resolvido.

### Basic SSRF against another backend system

Problema: 
	*"This lab has a stock check feature which fetches data from an internal system.
	To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`."*



