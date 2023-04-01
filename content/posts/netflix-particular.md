+++ 
draft = false
date = 2023-04-01T09:41:05-03:00
title = "Criando uma netflix particular"
description = "descrição"
slug = "netflix-particular"
authors = ['guilherme']
tags = []
categories = []
externalLink = ""
series = []
+++

# Criando uma netflix particular


Que a Netflix é um excelente serviço todos sabemos, mas nem sempre temos acesso exatamente ao filme que queremos. Com isso assinamos outro serviço apenas para assistir aquela série que só está disponível naquela outra plataforma.  No final do mês acabamos assinando trocentos serviços e raramente voltamos para ver o que tem de novo, mas o pior na minha experiência é conseguir achar o filme e então perceber que tem que alugar para assistir 🤡.

Não seria interessante ter uma maneira que podermos ter nosso próprio serviço de streaming? De preferencia usando tecnologias open-source? Algo que contemplasse as seguintes features:

- [ ]  Acesso por vários dispositivos
- [ ]  Busca por filmes e caso não encontre baixa de forma automática.
- [ ]  Busca por séries e caso não encontre baixa também de forma automática.
- [ ]  Para séries, ficar monitorando o lançamento do novo episódio para assim que lançar baixar automaticamente.
- [ ]  Notificar toda vez que terminou de baixar a nova mídia.

Temos várias features necessárias, por isso quebraremos elas em pequenas coisas.

## Acesso por vários aparelhos

 Felizmente já existem várias alternativas para a reprodução de mídia como, por exemplo, o [Jellyfin](https://jellyfin.org)

{{< figure src="/images/media-server/jellyfin.png" alt="" >}}


Uma iniciativa open-source ótima, mas para meu caso de uso sinto falta uma feature que é essencial. A falta de aplicativos para TVs e videos games é algo que realmente me incomoda, ninguém merece ter que plugar um computador na sala , configurar a saída do áudio para TV, controlar com um mouse sem fio e ter que ficar levantando para operar, quero uma experiencia mais próxima da netflix possível.

Tendo em mente esses requisito temos a opção de usar o [plex](https://www.plex.tv). 

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%201.png)

Não é open source, mas é gratuito, tendo opções de assinatura se desejar ter acesso a outras features que entrarei em detalhes adiante.  Como a imagem acima mostra “Watch anytime, anywhere with the Plex app”, e isso é exatamente o que procuro, uma forma de poder iniciar uma série no computador, terminar na TV da sala, ou no video game. Sendo assim Plex será nossa escolha.

### Buscar

O Plex realmente é um produto ótimo, mas sinceramente não resolve tudo. Ele apenas é um serviço que disponibiliza os arquivos de mídia que você já tem e que organizou na pasta corretamente. Logo você  tem que manualmente copiar eles para a pasta, o que levanta outra pergunta. Como você consegue esses arquivos?

Gostaria de deixar bem claro que não apoio a pirataria, e que meu intuito é apenas disseminar o conhecimento. Feito o disclaimer, acredito que como a maioria de nos pobres mortais apela para sites de torrent. Uma simples busca no Google vai te levar para vários sites e por sua vez você é inundado de propaganda e links que te levam para outros sites e assim por diante, até você conseguir o maldito magnet link. Depois de um tempo talvez você descubra que existem sites que atuam como Indexadores desses links, talvez já tenha ouvido falar do ThePirateBay. Esses indexadores são uma forma mais rápida e direta que buscar por algo que  deseja.

Logo, temos uma alternativa que fazer uma lista de todos os indexadores de torrentes e ir em cada um e buscar pela que desejamos, mas sinceramente, não né. Felizmente existe um projeto chamado 

[Jackett](https://github.com/Jackett/Jackett)

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%202.png)

Nele você pode adicionar e vários indexadores de torrent e procurar em apenas um único lugar. O que ele faz é entrar nesses sites fazer a busca e te retornar os resultados. Apenas isso salva muito, mas muito tempo, invés de ter que ficar visitando site por site atrás do que quer. Mas nem tudo é perfeito, muitos desses indexadores usam tecnologias de captcha para evitar uso de robos (que é exatamente o que estamos fazendo). 

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%203.png)

Desse modo pode acontecer das suas pesquisas falharem, pois será necessário validar que o robozinho do jacket é um humano. Felizmente temos um modo de contornar isso usando o [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%204.png)

Pronto, dessa maneira conseguimos resolver o problema de buscar, o que funciona para filmes e séries que já foram lançados, mas não para as que ainda não foram. Ainda teremos que continuar monitorando manualmente, o que obviamente não é isso que queremos fazer.

### Monitorar

Para resolver o problema de monitoramento usaremos dois programas, uma para séries o [Sonarr](https://github.com/Sonarr/Sonarr) 

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%205.png)

E para filmes o [Radarr](https://github.com/Radarr/Radarr)

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%206.png)

Os dois são muito parecidos e funcionam da mesma maneira. Basicamente fazem busca apenas por nomes e trazem todos os meta dados. Desse modo quando você adiciona um novo filme ele sabe dizer se o filme apenas foi anunciado, se já esta nos cinemas, ou se já foi lançado. 

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%207.png)

O mesmo acontece com o Sonnar, mas especifico para séries obviamente.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%208.png)

Dessa maneira temos um modo de monitorar e saber quando o filme/serie estará disponível.

### Instalando os programas

Agora que temos todos os softwares necessários, devemos de alguma maneira ligar todos eles. Nesse ponto explicarei como inicializar usando sistema linux e contêineres, lembrando que para cada software tem uma sessão listando e explicando como instalar no site correspondente, algo que não focarei nessa parte.

Por se tratar de vários projetos não iremos levantar cada contêiner separadamente, iremos utilizar o docker-compose. Além disso criei um [repositório](https://github.com/guibeira/media-server) com os um MakeFile para ajudar o setup, então vamos usar-lo.

Clone o projeto usando git

```bash
git clone https://github.com/guibeira/media-server.git
```

Entre na pasta

```bash
cd media-server

```

A estrutura da pasta vai se encontrar dessa maneira:

```bash
media-server
├── Makefile
├── docker-compose.yml
├── plex.env
└── qbittorrentvpn.env
```

Antes de iniciar os serviços, vamos adicionar algumas informações para nossos programas. O Plex tem que de alguma forma saber que a instância do programa que você esta subindo lhe pertence, algo parecido quando você vai logar pela primeira vez na sua conta da netflix em sua televisão. O plex disponibiliza esse site [https://www.plex.tv/claim/](https://www.plex.tv/claim/) assim que estiver logado ele gera um código.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%209.png)

copie ele e altere o arquivo plex.env, no meu caso dessa maneira.

```bash
PLEX_CLAIM=claim-BqrTpZ9bxtiyxR8D6pNZ
```

Caso deseje mudar a pasta de download é só mudar no arquivo Makefile a variável `DOWNLOAD_FOLDER`

Rode o setup

```bash
make setup
```

O comando setup criar todas as pastas necessárias de configurações e a pasta download,  

Além disso ele já via subir todos os contêineres.

## Configurando os programas

### Qbittorrent

Comecemos pelo `qbittorrent`, ele é o responsável por baixar os torrentes, acesse:

[http://localhost:6969](http://localhost:6969/)

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2010.png)

As credencias são:

Usuário : admin

Senha: adminadmin

Após entrar, alteraremos a pasta padrão para donwloads, para isso clique no ícone da engrenagem na barra superior.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2011.png)

A configuração da pasta fica nessa parte.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2012.png)

Altere de `/config/qBittorrent/downloads` para `/downloads` , altere também o management mode no modo automático e para as outras opções sempre marque `Relocate affected torrents`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2013.png)

role para baixo e clique em salvar.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2014.png)

Essa configurações são importantes pois elas que jogaram os arquivos para a pasta correta baseado na categoria que cadastraremos no `radarr` e `sonarr`.

Pronto! 

### Jacket

Para configurar o Jacket acesse [http://localhost:9117](http://localhost:9117/). Antes de adicionar novos indexes role para baixar até o final da página para configurarmos o `FlareSolverr.`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2015.png)

Altere para [`http://localhost:8191`](http://192.168.0.190:8191/)  clique em `Apply server settings` e pronto. Feito isso podemos começar a adicionar nossos indices. Vá para o topo da página e clique em `+ Add indexer`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2016.png)

Selecione o índice que deseja adicionar e clique em `+`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2017.png)

Pronto o novo índice já deve estar disponível na listagem. Quanto mais índices você adicionar, mais chance você terá de encontrar o que deseja.

### Radar e Sonnar

Agora que temos o Jacket e o Qbittorrent configurados, precisamos unir a comunicação deles usando o Radarr para filmes e o Sonarr séries.

O sonarr é exatamente igual ao radarr , tem apenas um detalhe na categoria, logo você pode seguir os mesmos passos para configurar o cliente de torrente e os índices do jacket. Acesse usando o seguinte endereço [http://localhost:8989](http://localhost:8989/).

Dito isso, comecemos  com o `Radar` conectando o nosso cliente de torrente, o `qbittorrent` .

acesse [http://localhost:7878](http://localhost:7878/) e clique em `settings`, depois em `Download Clients` .

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2018.png)

Clique no `+` e selecione qbittorrent

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2019.png)

Vai abrir um página para preencher as credências:

Altere os seguintes campos:

- Name: Qbittorrent
- Port: 6969
- Username : admin
- Password: adminadmin
- Category: movies (caso esteja configurado o sonarr, use television)

Muito importante alterar o campo `Category` pois o qbittorrent separará os aquivos em pastas conforme a categoria. Assim teremos nossa biblioteca sincronizada entre todos os programas. 

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2020.png)

Clique em `save` e pronto.

Feito isso configuraremos o Radarr com o Jacket, clique em `settings` → `Indexers`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2021.png)

Clique no `+` e selecione `Torznab`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2022.png)

Ele vai abrir esse formulário:

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2023.png)

Nele você preencherá com os dados do índice que adicionou no Jackert, acesse ele e 

- Copie a url clicando em `copy Torznab Feed`
- Copie a API Key

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2024.png)

No meu caso ficou assim

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2025.png)

Feito isso clique em `test` se tudo der certo aparecerá um check verdinho e clique em salvar. Caso tenha adicionado mais índices tera que fazer essa parte para cada um (um saco, estou tentando automatizar isso).

### Plex

Agora configuremos as pastas para plex conseguir enxergar nossa  biblioteca.

Acesse [http://localhost:32400](http://127.0.0.1:32400/) , ele pedirá para entrar sua conta, preencha os dados necessário e prossiga.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2026.png)

Acesse a Libraries no menu lateral

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2027.png)

Clique em `add Library` , selecione Movies, clique em avançar e selecione para a pasta

`data\movies`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2028.png)

Feito isso repita o processo para `Tv shows`, mas selecione `data\tvshows`

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2029.png)

Efetuado isso temos tudo configurado!

### Baixando o primeiro filme

Acesse o radar e clique em `Add new` procure pelo filme que deseja baixar

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2030.png)

selecione a pasta clique em `add new path` e selecione `dowloads/movies` esse passo se dará apenas dessa vez, nos proximos apenas selecionar a pasta. Feito isso clique em `Add new`
 

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2031.png)

Assim que adicionar o vai verificar que esse filme já foi lançado e não se encontra no seu HD. Logo ele chamará os indices que você cadastrou e caso algum indice encontre algo ele vai devolver o magnet link, e então repassará para o cliente de torrent que por sua vez baixará o arquivo.

Caso ele encontre no menu lateral em queue vai aparecer que o filme esta sendo baixado

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2032.png)

Se quiser mais detalhes você pode acessar o `qbittorrent` em [http://localhost:6969](http://localhost:6969/)

## Baixando a primeira série:

Depois de ter configurado, acesse [http://localhost:8989/](http://localhost:8989/add/new). O Processo é igual ao radarr, mas diferente note que a primeira vez que foi adicionar terá que selecionar a pasta `/downloads/television` 

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2033.png)

Clique em Add e pronto, sua série já começará a ser monitorada.

## Consumindo as mídias baixadas.

Assim que o filme tiver completado acesse o plex o filme deve aparecer.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2034.png)

Como seu servidor esta ligado na internet você poderá instalar o app na sua TV ou celular e assistir de onde quiser. Além disso o plex é esperto para identificar que sua tv esta na mesma rede local e concede acesso localmente, evitando gargalho de ter que sair da rede interna.

![Untitled](Criando%20uma%20netflix%20particular%20aeb2863131064cdb9df9f62db2d048f8/Untitled%2035.png)

Uma coisa legal do raddar e sonarr é a posibilidade de ser notificado via telegram, mas isso é para um futuro update desse post

Pronto temos configurado nosso servidor de midia, sempre que quisermos adicionar um filme temos apenas que acessar o `radarr` e para séries o `sonarr`.

Lembrando que é sempre recomendado usar uma VPN se proteger, caso tenha interesse em aplicar a VPN apenas no qbitorrent de uma olhada do readme do projeto.

[https://github.com/binhex/arch-qbittorrentvpn](https://github.com/binhex/arch-qbittorrentvpn)

Caso tenha algum problema não deixe de abrir um issue no github do projeto

[https://github.com/guibeira/media-server](https://github.com/guibeira/media-server)