+++ 
draft = false
date = 2023-04-01T09:41:05-03:00
title = "Criando uma netflix particular"
description = "Armazene e transmita seu próprio conteúdo de mídia"
slug = "netflix-particular"
authors = ['guilherme']
tags = ["docker"]
categories = []
externalLink = ""
series = []
+++

# Criando uma netflix particular


Que a Netflix é um excelente serviço todos sabemos, mas nem sempre temos acesso exatamente ao filme que queremos. Com isso assinamos outro serviço apenas para assistir aquela série que só está disponível naquela outra plataforma.  No final do mês acabamos assinando trocentos serviços e raramente voltamos para ver o que tem de novo, mas o pior na minha experiência é conseguir achar o filme e então perceber que tem que alugar para assistir 🤡.

Não seria interessante ter uma maneira de podermos ter nosso próprio serviço de streaming? De preferencia usando tecnologias open-source? Algo que contemplasse as seguintes features:

- [ ]  Acesso por vários dispositivos
- [ ]  Busca por filmes e caso não encontre baixa de forma automática.
- [ ]  Busca por séries e caso não encontre baixa também de forma automática.
- [ ]  Para séries, ficar monitorando o lançamento do novo episódio para assim que lançar baixar automaticamente.
- [ ]  Notificar toda vez que terminou de baixar a nova mídia.

Temos várias features necessárias, por isso quebraremos elas em pequenas coisas.

## Acesso por vários aparelhos

 Felizmente já existem várias alternativas para a reprodução de mídia como, por exemplo, o [Jellyfin](https://jellyfin.org)

{{< figure src="/images/media_server/jellyfin.png" alt="" >}}


Uma iniciativa open-source ótima, mas para meu caso de uso sinto falta uma feature que é essencial. A falta de aplicativos para TVs e videos games é algo que realmente me incomoda, ninguém merece ter que plugar um computador na sala, configurar a saída do áudio para TV, controlar com um mouse sem fio e ter que ficar levantando para operar, quero uma experiência mais próxima da netflix possível.

Tendo em mente esses requisito temos a opção de usar o [plex](https://www.plex.tv). 

{{< figure src="/images/media_server/plex.png" alt="" >}}

Não é open source, mas é gratuito, tendo opções de assinatura se desejar ter acesso a outras features que entrarei em detalhes adiante. Como a imagem acima mostra “Watch anytime, anywhere with the Plex app”, e isso é exatamente o que procuro, uma forma de poder iniciar uma série no computador, terminar na TV da sala, ou no videogame. Sendo assim Plex será nossa escolha.

### Buscar

O Plex realmente é um produto ótimo, mas sinceramente não resolve tudo. Ele apenas é um serviço que disponibiliza os arquivos de mídia que você já tem e que organizou na pasta corretamente. Logo você  tem que manualmente copiar eles para a pasta, o que levanta outra pergunta. Como você consegue esses arquivos?

Gostaria de deixar bem claro que não apoio a pirataria, e que meu intuito é apenas disseminar o conhecimento. Feito o disclaimer, acredito que como a maioria de nos pobres mortais apela para sites de torrent. Uma simples busca no Google vai te levar para vários sites e por sua vez você é inundado de propaganda e links que te levam para outros sites e assim por diante, até você conseguir o maldito magnet link. Depois de um tempo talvez você descubra que existem sites que atuam como Indexadores desses links, talvez já tenha ouvido falar do ThePirateBay. Esses indexadores são uma forma mais rápida e direta que buscar por algo que  deseja.

Logo, temos uma alternativa que fazer uma lista de todos os indexadores de torrentes e ir em cada um e buscar pela que desejamos, mas sinceramente, não né. Felizmente existe um projeto chamado 

[Jackett](https://github.com/Jackett/Jackett)

{{< figure src="/images/media_server/jacket.png" alt="" >}}

Nele você pode adicionar e vários indexadores de torrente e procurar em apenas um único lugar. O que ele faz é entrar nesses sites fazer a busca e te retornar os resultados. Apenas isso salva muito, mas muito tempo, invés de ter que ficar visitando site por site atrás do que quer. Mas nem tudo é perfeito, muitos desses indexadores usam tecnologias de captcha para evitar uso de robôs (que é exatamente o que estamos fazendo).

{{< figure src="/images/media_server/not_robot.png" alt="" >}}

Desse modo pode acontecer das suas pesquisas falharem, pois será necessário validar que o robozinho do jacket é um humano. Felizmente temos um modo de contornar isso usando o [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr).

{{< figure src="/images/media_server/flareSolverr.png" alt="" >}}

Pronto, dessa maneira conseguimos resolver o problema de buscar, o que funciona para filmes e séries que já foram lançados, mas não para as que ainda não foram. Ainda teremos que continuar monitorando manualmente, o que obviamente não é isso que queremos fazer.

### Monitorar

Para resolver o problema de monitoramento usaremos dois programas, uma para séries o [Sonarr](https://github.com/Sonarr/Sonarr) 

{{< figure src="/images/media_server/sonarr.png" alt="" >}}

E para filmes o [Radarr](https://github.com/Radarr/Radarr)

{{< figure src="/images/media_server/radarr.png" alt="" >}}

Os dois são muito parecidos e funcionam da mesma maneira. Basicamente fazem busca apenas por nomes e trazem todos os meta dados. Desse modo, quando você adiciona um novo filme, ele sabe dizer se o filme apenas foi anunciado, se já esta nos cinemas, ou se já foi lançado

{{< figure src="/images/media_server/radarr_calendar.png" alt="" >}}

O mesmo acontece com o Sonnar, mas especifico para séries, obviamente.

{{< figure src="/images/media_server/sonarr_calendar.png" alt="" >}}

Dessa maneira temos um modo de monitorar e saber quando o filme/serie estará disponível.

### Instalando os programas

Agora que temos todos os softwares necessários, devemos de alguma maneira ligar todos eles. Nesse ponto explicarei como inicializar usando sistema linux e contêineres, lembrando que para cada software tem uma sessão listando e explicando como instalar no site correspondente, algo que não focarei nessa parte.

Por se tratar de vários projetos não iremos levantar cada contêiner separadamente, iremos utilizar o docker-compose. Além disso, criei um repositório com um MakeFile para ajudar o setup, então vamos usar.

Clone o projeto usando git.

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

Antes de iniciar os serviços, vamos adicionar algumas informações para nossos programas. O Plex tem que de alguma forma saber que a instância do programa que você esta subindo lhe pertence, algo parecido quando você entrar pela primeira vez na sua conta da netflix em sua televisão. O plex disponibiliza esse site [https://www.plex.tv/claim/](https://www.plex.tv/claim/) assim que estiver logado ele gera um código.

{{< figure src="/images/media_server/plex_claim.png" alt="" >}}

Copie ele e altere o arquivo plex.env, no meu caso dessa maneira.

```bash
PLEX_CLAIM=claim-BqrTpZ9bxtiyxR8D6pNZ
```

Caso deseje mudar a pasta de download é só mudar no arquivo Makefile a variável `DOWNLOAD_FOLDER`

Rode o setup

```bash
make setup
```

O comando setup criar todas as pastas necessárias de configurações e a pasta download,  

Além disso, ele já via subir todos os contêineres.

## Configurando os programas

### Qbittorrent

Comecemos pelo `qbittorrent`, ele é o responsável por baixar os torrentes, acesse:

[http://localhost:6969](http://localhost:6969/)

{{< figure src="/images/media_server/qbittorrent.png" alt="" >}}

As credencias são:

Usuário : admin

Senha: adminadmin

Após entrar, alteraremos a pasta padrão para downloads, para isso clique no ícone da engrenagem na barra superior

{{< figure src="/images/media_server/q_config.png" alt="" >}}

A configuração da pasta fica nessa parte.

{{< figure src="/images/media_server/q_path.png" alt="" >}}

Altere de `/config/qBittorrent/downloads` para `/downloads` , altere também o management mode no modo automático e para as outras opções sempre marque `Relocate affected torrents`

{{< figure src="/images/media_server/q_config_1.png" alt="" >}}

role para baixo e clique em salvar.

{{< figure src="/images/media_server/q_save.png" alt="" >}}

Essas configurações são importantes, pois elas que jogaram os arquivos para a pasta correta baseado na categoria que cadastraremos no `radarr` e `sonarr`.


### Jacket

Para configurar o Jacket acesse [http://localhost:9117](http://localhost:9117/). Antes de adicionar novos indexes role para baixar até o final da página para configurarmos o `FlareSolverr.`

{{< figure src="/images/media_server/q_flamesolverr.png" alt="" >}}

Altere para [`http://localhost:8191`](http://192.168.0.190:8191/)  clique em `Apply server settings` e pronto. Feito isso podemos começar a adicionar nossos indices. Vá para o topo da página e clique em `+ Add indexer`

{{< figure src="/images/media_server/j_index.png" alt="" >}}

Selecione o índice que deseja adicionar e clique em `+`

{{< figure src="/images/media_server/j_select_index.png" alt="" >}}

Pronto o novo índice já deve estar disponível na listagem. Quanto mais índices você adicionar, mais chance você terá de encontrar o que deseja.

### Radar e Sonnar

Agora que temos o Jacket e o Qbittorrent configurados, precisamos unir a comunicação deles usando o Radarr para filmes e o Sonarr séries.

O sonarr é exatamente igual ao radarr , tem apenas um detalhe na categoria, logo você pode seguir os mesmos passos para configurar o cliente de torrente e os índices do jacket. Acesse usando o seguinte endereço [http://localhost:8989](http://localhost:8989/).

Dito isso, comecemos  com o `Radar` conectando o nosso cliente de torrente, o `qbittorrent` .

acesse [http://localhost:7878](http://localhost:7878/) e clique em `settings`, depois em `Download Clients` .

{{< figure src="/images/media_server/r_download_client.png" alt="" >}}

Clique no `+` e selecione qbittorrent

{{< figure src="/images/media_server/r_select_q.png" alt="" >}}

Vai abrir um página para preencher as credências:

Altere os seguintes campos:

- Name: Qbittorrent
- Port: 6969
- Username : admin
- Password: adminadmin
- Category: movies (caso esteja configurado o sonarr, use television)

Muito importante alterar o campo `Category` pois o qbittorrent separará os aquivos em pastas conforme a categoria. Assim teremos nossa biblioteca sincronizada entre todos os programas. 

{{< figure src="/images/media_server/r_form.png" alt="" >}}

Clique em `save` e pronto.

Feito isso configuraremos o Radarr com o Jacket, clique em `settings` → `Indexers`

{{< figure src="/images/media_server/r_index.png" alt="" >}}

Clique no `+` e selecione `Torznab`

{{< figure src="/images/media_server/r_torznab.png" alt="" >}}

Ele vai abrir esse formulário:

{{< figure src="/images/media_server/r_index_form.png" alt="" >}}

Nele você preencherá com os dados do índice que adicionou no Jackert, acesse ele e 

- Copie a url clicando em `copy Torznab Feed`
- Copie a API Key

{{< figure src="/images/media_server/j_index_copy.png" alt="" >}}

No meu caso ficou assim
{{< figure src="/images/media_server/r_index_form_filled.png" alt="" >}}

Feito isso clique em `test` se tudo der certo aparecerá um check verdinho e clique em salvar. Caso tenha adicionado mais índices tera que fazer essa parte para cada um (um saco, estou tentando automatizar isso).

### Plex

Agora configuremos as pastas para plex conseguir enxergar nossa  biblioteca.

Acesse [http://localhost:32400](http://127.0.0.1:32400/) , ele pedirá para entrar sua conta, preencha os dados necessário e prossiga.

{{< figure src="/images/media_server/plex_settings.png" alt="" >}}

Acesse a Libraries no menu lateral

{{< figure src="/images/media_server/plex_lib_menu.png" alt="" >}}

Clique em `add Library` , selecione Movies, clique em avançar e selecione para a pasta `data\movies`

{{< figure src="/images/media_server/plex_movie_folder.png" alt="" >}}


Feito isso repita o processo para `Tv shows`, mas selecione `data\tvshows`

{{< figure src="/images/media_server/plex_tvshows_folder.png" alt="" >}}

Efetuado isso temos tudo configurado!

### Baixando o primeiro filme

Acesse o radar e clique em `Add new` procure pelo filme que deseja baixar

{{< figure src="/images/media_server/new_movie.png" alt="" >}}

Selecione a pasta, clique em `add new path` e selecione `dowloads/movies` esse passo se dará apenas dessa vez, nos próximos apenas selecionar a pasta. Feito isso clique em `Add new`
 

{{< figure src="/images/media_server/new_movie_settings.png" alt="" >}}

Assim que adicionar o vai verificar que esse filme já foi lançado e não se encontra no seu HD. Logo ele chamará os índices que você cadastrou e caso algum índice encontre algo ele vai devolver o magnet link, e então repassará para o cliente de torrente que por sua vez baixará o arquivo.

Caso ele encontre no menu lateral em queue vai aparecer que o filme esta sendo baixado

{{< figure src="/images/media_server/download_info.png" alt="" >}}

Se quiser mais detalhes você pode acessar o `qbittorrent` em [http://localhost:6969](http://localhost:6969/)

## Baixando a primeira série:

Após ter configurado o sonarr, acesse [http://localhost:8989/](http://localhost:8989/add/new). O Processo é igual ao radarr, mas note que a primeira vez que for adicionar terá que selecionar a pasta `/downloads/television` 

{{< figure src="/images/media_server/sonarr_folder.png" alt="" >}}

Clique em Add e pronto, sua série já começará a ser monitorada.

## Consumindo as mídias baixadas.

Assim que o filme tiver completado acesse o plex o filme deve aparecer.

{{< figure src="/images/media_server/plex_index.png" alt="" >}}

Como seu servidor esta ligado na internet você poderá instalar o app na sua TV ou celular e assistir de onde quiser. Além disso o plex é esperto para identificar que sua tv esta na mesma rede local e concede acesso localmente, evitando gargalho de ter que sair da rede interna.

{{< figure src="/images/media_server/plex_dashboard.png" alt="" >}}

Uma coisa legal do raddar e sonarr é a possibilidade de ser notificado via telegram, mas isso é para um futuro update desse post.

Pronto temos configurado nosso servidor de mídia, sempre que quisermos adicionar um filme temos apenas que acessar o `radarr` e para séries o `sonarr`.

Lembrando que é sempre recomendado usar uma VPN se proteger, caso tenha interesse em aplicar a VPN apenas no qbitorrent de uma olhada do readme do projeto.

[https://github.com/binhex/arch-qbittorrentvpn](https://github.com/binhex/arch-qbittorrentvpn)

Caso tenha algum problema não deixe de abrir um issue no github do projeto

[https://github.com/guibeira/media-server](https://github.com/guibeira/media-server)
