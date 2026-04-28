---
title: "Aula de pentest"
date: 2026-04-25 00:00:00 -0300
categories: [security]
tags: [security, vulnerability]
---

Primeiro gostaria de agradecer a você que está lendo, esse é meu primeiro post de muitos e que aqui vai ser uma biblioteca e um porfólio de todos meus aprendizados, que você leve algo com você, espero que goste.


Eu adoro segurança ofensiva e tentar achar falhas, até semana passada, todo que tinha visto sobre pentest foi por fora da faculdade e por pura curiosidade e vontade propria. Mas voltando semana passada tive a primeira aula de pentest depois de 2 anos de curso e 4 longos semestres, aconteceu, nesse caso ansiedade foi nos seus mais altos níveis.

Trabalho simples:
    *"ESP32, varrer, achar, entrar, explorar" simples servidor web que o professor tinha preparado para testar nossas habilidades naquele momento.*

Mas no momento que estava varrendo a rede junto com meu grupo, detectei algo estranho um salto gigante de ip na rede, decidi saciar minha curiosidade rodei um scan básico com nmap, vejo algo **porta 80 aberta**, para todos os fãs de CTFs (eu) ali era uma mina de ouro mas claro tomando todos os cuidados já que era uma rede corporativa, fiz o acesso, caí diretamente no painel administrativo do switch local, avisado o reponsável da aula, decidimos ir mais a fundo com permissão.
Logo testei o mais obvio, senha padrão, entrei.
Lá conseguimos ver todas as configurações e com direitos a fazer tudo que era possível dentro de um switch, com isso, fomos diretamento contatar os responsáveis e recebemos **reconhecimento por ter achado uma falha**, dopamina a mil com essa frase.

## Sentimento
---

Aqueceu e acalmou meu coração ver meus colegas e eu, se juntando e finalmente fazendo algo acontecer na prática e ver como realmente funciona achar uma falha por mais boba que seja.
A felicidade estampada na cara de todos, *"Depois de 2 anos, finalmente, isso que eu gosto"* - my friend.

E aquilo deu um gás e acendeu o fogo de voltar a aprender ainda mais, sentimento que realmente a gente pode conseguir fazer isso e trabalhar com isso que gostamos, não está tão distante.