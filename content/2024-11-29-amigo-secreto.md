---
date: 2024-11-29
author: guibeira
tags: rust,webassembly,yew,actix
projects: amigo-secreto
---

# Amigo secreto em Rust

Sempre no final do ano eu e meus amigos nos reunimos para fazer um amigo secreto/oculto, e de uns anos para cá tem ficado cada vez mais difícil reunir todo mundo para fazer o sorteio. Então, me recordo de um amigo sugerir uma plataforma para fazer o sorteio, achei muito legal, mas assim que acessei o link, notei que pedia todos, juro, todos os seus dados para fazer um simples sorteio. Fiquei me questionando a necessidade de ter, por exemplo, meu CPF para cadastro. E após ler as letrinhas miúdas, percebi que eles vendiam os dados para terceiros.

Com uma breve busca no GitHub encontrei uma opção que usava Django. Como tenho familiaridade com esse framework, foi fácil subir localmente e usando um serviço de tunelamento, no caso Ngrok para disponibilizar para meus amigos. Fizemos o sorteio e foi muito divertido, mas percebi ter um pequeno bug, algo que outro colega desenvolvedor já tinha apontado. O sorteio não pode pegar os participantes de forma aleatória, por arriscar não ter uma sequência em que o próximo pode pegar alguém que já começou, acabando com a graça que é ter alguém para revelar.

Com isso em mente, decidi criar um amigo secreto em Rust. Por se tratar de um problema simples, foquei em fazer primeiro a engine, que conteria todas as regras.
Após isso, decidiria como faria o restante, se seria uma CLI, uma API, ou uma aplicação web.


## Separação de responsabilidades

Com minha engine pronta, pensei em criar um bom e velho serviço web que já retorna o HTML renderizado do servidor. Depois de alguns testes, iria usar [HTMX](https://htmx.org) para trazer um pouco de dinamismo para a aplicação, mas como eu queria realmente aprender algo novo, me perguntei por que não usar Rust também para o frontend? E foi aí que encontrei o [Yew](https://yew.rs), um framework front-end em Rust inspirado em React, que me permitiria fazer tudo em Rust, e foi isso que fiz.

Agora eu tinha uma engine que fazia o sorteio, uma API em Actix que interagi com a engine, e um frontend em Yew que consumia a API, e tudo isso em Rust.


## Indo além

Minha aplicação já estava pronta, mas tinha um problema, apenas pessoas com conhecimento técnico iriam conseguir usar. Na minha roda de amigos, eu sou a pessoa que 
conseguiria subir esse serviço, mas e se eu quisesse compartilhar com outras pessoas? Como faria para qualquer pessoa poder usar? O projeto teria que ter um Ngrok embutido para automaticamente disponibilizar o endereço do sorteio para os participantes.

Não queria fazer algo relacionado com scripts, queria apenas um binário que, ao ser executado, já disponibilizasse o endereço para os participantes, e foi aí que me lembrei do [bore](https://github.com/ekzhang/bore). Depois de alguns testes, vi que poderia embutir o projeto e assim conseguir gerar um binário que, ao ser executado, já disponibilizasse o endereço para os participantes.

Tudo lindo, maravilhoso, mas tinha um pequeno problema, o bore não aceita requisições https, e sempre que você compartilha um link http, o seu navegador automaticamente redireciona para https, e como o bore não aceita https, o link não funcionaria. Eu poderia ajustar o NGINX para redirecionar para http, mas eu não sou o dono do bore.pub, logo não tenho acesso ao NGINX.

## Criando o próprio servidor de tunelamento

Felizmente o bore tem a feature de self host, o que me permitiria criar minhas regras de redirecionamento, e foi isso que fiz. Logicamente, não usando meu home server, e sim uma máquina na nuvem, que após várias tentativas de configuração do NGINX, decidir fazer o mais simples. Todas as requisições iram conter a porta como parâmetro, e o servidor iria redirecionar para a porta correta, ficando desta maneira.
* http://meu-servidor.com/1234 -> http://meu-servidor.com:1234

Assim, eu apenas gero o link com a porta, e o servidor irá redirecionar para a porta correta. Com isso, os participantes conseguem compartilhar o link corretamente, agora finalmente consegui atingir todos os objetivos, projeto finalizado.

## Conclusão. 

Esse projeto pequeno foi uma forma de aprender novas tecnologias, e usar Rust no front-end é algo que até o momento não é nada simples, pelo menos foi a minha experiência com Yew, ainda quero experimentar outras opções como o [Leptos](https://leptos.dev) e o [Dioxus](https://dioxuslabs.com), mas isso é outra história. Já no backend a conversa é outra, com o tempo fica cada vez mais fácil entender os motivos do porquê o borrow checker está reclamando, e com isso você consegue fazer projetos cada vez mais complexos.
Se ficou curioso para ver o projeto, clique [aqui](https://github.com/guibeira/secret-santa). Caso use Windows, na aba de releases tem um binário para você testar, e por favor, deixe uma estrela no projeto.
