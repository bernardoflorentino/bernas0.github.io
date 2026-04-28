---
title: "Trilha de Vulnerabilidades Server-Side - Portswigger academy"
date: 2026-04-28 00:00:00 -0300
categories: [write-up]
tags: [vulnerability, offsec, web]
---
<p>
<a href="assets\img\server-side-vuln-pts\hxh_burp_fade2.png" class="popup img-link"> 
<img src="assets\img\server-side-vuln-pts\hxh_burp_fade2.png" alt="Introduction Image" width="980" height="380" loading="lazy">
</a>
</p>
Write-up da trilha de Server-side Vulnerabilites da Portswigger, demonstração prática e explicação sobre os assuntos demonstrados nas aulas. \
*OBS:* Ficarei devendo a última parte sobre SQLi, pois estou planejándo escrever um post mais especifico e já incorporar tudo sobre ele, já que é um tema que já andei pesquisando bastante e tenho anotações sobre, obrigado pela atenção, boa leitura!

>Ferramentas que iremos utilizar nesses laborátorios: **BurpSuite + F12 (Inspecionar)**

<a href="https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice">
	Portswigger academy
</a>

## Path traversal
---

É um ataque de server-side onde é possível a exfiltração de dados a partir de uma mudança no caminho de uma chamada vulnerável no código.
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


## Access control
---

Access control é um controlador de sessões de usuarios para que sejá possível o usuario acessar somente sua determinada página web, mas se mal programa é possível acessar páginas de outros tal qual usuario administrativo.

### Vertical privilege escalation

Meio de ataque onde o usuario malicioso consegue acessar outro usuario privilegiado diretamente no qual ele não deveria, por exemplo um usuario sem acesso administrativo entrar ao painel administrativo e conseguir fazer alterações.

Um meio que um página pode estar vulnerável assim é através do robots.txt


### Request parameter

Ataque por meio da mudança um parametro em uma chamada sejá por meio de cookies, um campo escondido uma string em uma query.
```
<>
Por exemplo:
	...
	https://insecure-website.com/login/home.jsp?admin=true
	https://insecure-website.com/login/home.jsp?role=1
	...
<>	
```


### Lab: Horizontal privilege escalation

É um meio de ataque onde um por exemplo um colaborador que tem um ID, pode acessar a sessao/conta de outro usuario apenas trocando seus IDs.
Isso normalmente é resolvido passando um GUID unico para cada colaborador invés de um número subsequente e não deixando exposto essas informações nas chamadas de getUser() ou getInfoUser()


**Problema:**
>*"To solve the lab, find the GUID for `carlos`, then submit his API key as the solution.
>You can log in to your own account using the following credentials: `wiener:peter`"*

Acessando a conta, estamos de cara com nosso usuario e nossa API key então presumo que ao final devemos estar nessa página.
<a href="assets\img\server-side-vuln-pts\horizontal-1.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\horizontal-1.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Explorando o home:
Alguns posts estão disponíveis para ler dentro desse blog, lendo alguns procurei se tinha a princípio algum comentário mas não, inspecionando a página já notei por onde seria possível achar o GUID do usuário.

<a href="assets\img\server-side-vuln-pts\horizontal-2.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\horizontal-2.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

GUID do Administrador achado facilmente, mas não queremos esse, e sim o do **carlos**. então procurando algum post postado pelo carlos e é possível fazer a mesma coisa e achar o dele também.
Copiamos o ID e voltamos para a página onde queremos a API key, na URL é possível notar um parametro chamando o id do user, fazemos a mudança e pronto.

<a href="assets\img\server-side-vuln-pts\horizontal-3.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\horizontal-3.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>


### Lab: Horizontal to Vertical

Como já foi visto antes, agora podemos ir de um ataque horizontal e pegar um usuario com privilegios administrativos que vai deixar a gente fazer ainda mais coisas dentro de seu site.

**Problema:**
>*"To solve the lab, retrieve the administrator's password, then use it to delete the user `carlos`.
>You can log in to your own account using the following credentials: `wiener:peter`"*


Após realizar o login já é possível ver uma chamada na URL diretamente pelo nome do usuario, vamos tentar o mais básico primeiro acessar somente mudando esse parametro para administrador.

<a href="assets\img\server-side-vuln-pts\url-id.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\url-id.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Trocado o parametro

<a href="assets\img\server-side-vuln-pts\horizontal-to-vertical-1.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\horizontal-to-vertical-1.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Acesso direto ao painel do usuario privilegiado, além de pelo inspecionar conseguimos pegar sua senha agora vamos deletar o usuario e continuar em nossa trilha.

<a href="assets\img\server-side-vuln-pts\horizontal-to-vertical-2.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\horizontal-to-vertical-2.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>


# Authentication
---

Autenticação != Autorização
Autenticação é o site tentando verificar se você realmente é quem diz ser e a autorização é os recursos que essa conta vai ter no website, ver blogs, editar, ver dados dos usuarios et al.
Normalmente ataques de brute-force são uteis, ainda mais quando existe um padrão e é possível fazer uma wordlist personalizada para a atuação. 

## SSRF (Server-side request forgery)
---

É um ataque onde o agente malicioso vai tentar se conectar diretamente ao server fazendo uma requisição, na parte vulnerável como nesse exemplo que ao invés de ser chamado uma API para buscar o estoque de items em site, ele vai retornar a pagina /admin liberando seu acesso.

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded 
Content-Length: 118 

stockApi=http://localhost/admin
```

Mas muitas vezes é read-only, sendo as modificações possiveis somente para usuarios autenticados e com autorização. Mas caso a pagina estejá rodando diretamente nesse servidor acessando o localmente vai fazer bypass em todas em barreiras do front-end.

### Lab: Basic SSRF in a local server

**Problema:**
>"*This lab has a stock check feature which fetches data from an internal system.
>To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`"*

Para resolver isso vamos focar nessa dica, que ele passou no problema acessar o stock e mudar a requisição que ele faz nessa checagem.
Acessamos o serviço e procuramos a chamada dentro do código.

<a href="assets\img\server-side-vuln-pts\ssrf-local-sv-1.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\ssrf-local-sv-1.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Achamos. Agora vamos colocar o localhost/admin, e acessar o painel administrativo.

<a href="assets\img\server-side-vuln-pts\ssrf-local-sv-2.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\ssrf-local-sv-2.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Acesso feito agora só excluir o usuario que foi requisitado carlos.

<a href="assets\img\server-side-vuln-pts\ssrf-local-sv-3.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\ssrf-local-sv-3.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Vixi não deu certo por conta da autenticação, acabei esquecendo disso então vamos fazer na mão então igual os antigos faziam na idade das pedras.
Na URL é possível notar que ainda está a chamada "delete?username=<>", vamos pegar essa chamada e colocar diretamente na chamada de stock onde fizemos a primeira requisição pois la vamos estar diretamente na máquina.

<a href="assets\img\server-side-vuln-pts\ssrf-local-sv-4.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\ssrf-local-sv-4.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Agora sim, resolvido.

### Lab: Basic SSRF against another backend system

**Problema:** 
	*"This lab has a stock check feature which fetches data from an internal system.
	To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`."*

Aqui vamos fazer igualmente o outro, utilizando o F12 para inspecionar a chamada do stockApi e ver os ips que temos para trabalhar.

<a href="assets\img\server-side-vuln-pts\ssrf-backend-sv-1.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\ssrf-backend-sv-1.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>


Agora vamos precisar passar um scan nessa rede e descobrir qual vai conseguir acessar, mandando a mesma requisição para os 254 hosts disponiveis da rede, para isso é possível utilizar os payloads já prontos feitos pelo burpsuite passando a requisição e o range de ip que temos para scanear, a partir disso é possível achar o ip que deu 200 como resposta e acessa-lo.

```
Request:
"http://192.168.0.$1$:8080/" até ".254"
	|
	V
Procure por:
	HTTP 1.0 200 OK ou 404 -> Altere para /admin
```

Agora só acessar e deletar usando a mesma técnica de antes de passar o delete direto na URL e voltar para o painel de admin.


# File upload

É uma vulnerabilidade ocasionada pela falta de validação do arquivo upado no web-site, sem validação do tipo, nome, tamanho et al.  Isso liberado pode se tornar um shell remoto em uma unica falha de validação de arquivo, aumentar as despesas se for colocado um arquivo gigante já que armazenamento custa dinheiro e o usuario pode utilizar o quanto ele quiser.
O pior caso é realmente a de um shell remoto, que é vindo de arquivos .py .php ou jáva. com isso é possível ter dominio de todo o server.
```
Exemplo:

Web-shell apenas com 1 linha -
	<?php echo file_get_contents('/path/to/target/file'); ?>

Mais refinado:
	<?php echo system($_GET['command']); ?>
```

### Lab 1:
**Problema:**
>*"To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
>You can log in to your own account using the following credentials: `wiener:peter`"*

Efetuamos o login com as nossas credenciais, upload de avatar agora está e como ele já disse não tem nehuma validação de tipo ou qualquer defesa contras fulnerabilidade de upload.
Criamos um arquivo .php em nosso proprio sistema e colocams um simples web-shell puxando as informações que queremos do usuario e a adicionamos.
<a href="assets\img\server-side-vuln-pts\file-upload-lab1-1.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\file-upload-lab1-1.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

No burp utilizando proxy -> http histoy é possível que realmente funcionou
<a href="assets\img\server-side-vuln-pts\file-upload-lab1-2.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\file-upload-lab1-2.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Agora so recarregar a página e procurar pela chamada buscando no exploit e ver a resposta do site.
<a href="assets\img\server-side-vuln-pts\file-upload-lab1-3.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\file-upload-lab1-3.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

## Explorando defesas fracas

Falha de tipo de arquivos:
É quando há uma validação no type-content verificando se é igual o mime type da requisição assim fazendo a validação o problema é que a gente consegue mudar esse mime type dentro do burp repeater e bypassar essa defesa

### Lab 2 (Web shell upload via content-type):

**Problema:**
>*"This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.
>To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
>You can log in to your own account using the following credentials: `wiener:peter`"*

Vamos utilizar novamente o Burp repeater para ver oque está acontecendo, fiz o upload do exploit sendo ele agora um .jpg, para conseguir até o servidor onde agora está armazenado lembrando que sempre que a gente recarregar a página tem um GET puxando as informações dessa "imagem".
<a href="assets\img\server-side-vuln-pts\file-upload-lab2-1.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\file-upload-lab2-1.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Para que funcione e não sejá apenas um blank de informações vamos precisar ou alterar seu content-type ou mais fácil somente alterar o nome pois a verificação está no front-end e não no servidor.
<a href="assets\img\server-side-vuln-pts\file-upload-lab2-2.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\file-upload-lab2-2.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Fazendo a troca já conseguimos visualizar a resposta correta do /secret que queriamos.
<a href="assets\img\server-side-vuln-pts\file-upload-lab2-3.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\file-upload-lab2-3.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

## OS Command Injection
OS Command Injection, é quando fazemos um ataque procurando acessar o servidor onde está aplicação e e com isso exfiltrar dados, normalmente usando comandos de linux ou windows server para isso.

### Lab:

**Problema:**
>*"This lab contains an OS command injection vulnerability in the product stock checker.
>The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.
>To solve the lab, execute the `whoami` command to determine the name of the current user."*

Vamos utilizar o burp repeater, entramos em qualquer página e vemos oque está acontecendo, vejo um POST no stock o único, então vou verificar e acabo achando um meio por onde vamos conseguir realizar o ataque.
<a href="assets\img\server-side-vuln-pts\os-ci-lab-1.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\os-ci-lab-1.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Mudando os parametros aqui não ia dar certo utilizar o & coloquei | (pipe), para passar o comando que foi requisitado no problema.
<a href="assets\img\server-side-vuln-pts\os-ci-lab-2.webp" class="popup img-link">
  <img src="assets\img\server-side-vuln-pts\os-ci-lab-2.webp" alt="Introduction Image" 
  style="max-width: 100%; height: auto;" loading="lazy">
</a>

Pronto está feito o sorvetinho, proximo.


# Considerações

Uma trilha muito bem desenvolvida, o mais interessante da academia é que realmente da pra por a mão na massa logo após a explicação e isso faz fixar muito mais o conteúdo, considero um ponto extramamente positivo além de que eles não pegam em sua mão e te ensinam onde achar a resposta tem que quebrar um pouquinho a cabeça, particurlamente prefiro assim do que outras plataformas onde basta copiar e colar do texto a resposta mas ainda tem dicas e solução da propria comunidade que é disponibilizado se você precisar de um empurrãozinho.
Falo também que o BurpSuite em sua funcionalidade para vulnerabilidade web, é a melhor ferramenta que já usei simples, fácil e eficiente. Gostaria de usar a versão pro, para saber como funciona alguns payloads e dns proprios deles que devem facilitar ainda mais.

