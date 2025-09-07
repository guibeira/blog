
---
date: 2025-07-09
author: guibeira
tags: rust,self-host
card_image: https://private-user-images.githubusercontent.com/10093193/486531251-e88f084b-47b8-467b-a5c6-d64327805792.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTcyNjU4OTIsIm5iZiI6MTc1NzI2NTU5MiwicGF0aCI6Ii8xMDA5MzE5My80ODY1MzEyNTEtZTg4ZjA4NGItNDdiOC00NjdiLWE1YzYtZDY0MzI3ODA1NzkyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MDclMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTA3VDE3MTk1MlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWY2ZDVjNTE3ZGY1MzBkODY3ZTEyZTM1YTNhNDI1MzQzZDFlOGFlYWU4ODY4YWI4ZjgwYmI1N2FhYTBlOTYwMjEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.cvx_2cTNHJ-7ARa_F0x0mOD25C0SxIDp-d1N1zTh5lA
banner_image: media/posts/wakezilla/banner.png
extra:
  mermaid: true
  mermaid_theme: default
---


# Economizando energia no self-hosting, Wake-on-LAN e Rust


### Introdução

Há algum tempo, comecei a explorar o mundo do self-hosting, e como isso é viciante, você sempre se pega pensando em quais novos serviços poderia hospedar. Tenho uma máquina bem simples, um Intel I3 de quarta geração, com uma placa de vídeo RTX 1650 de 4gb, ou seja, não consome muita energia. 
Sabendo que minha placa era subutilizada, decidi instalar o [Ollama](https://ollama.com), uma ferramenta que permite rodar modelos de IA localmente, e após conseguir testar o Ollama, logo percebi que 4gb não era suficiente para rodar os modelos mais recentes. 

### Upgrade de hardware

Com esse novo problema, agora tinha a desculpa perfeita para fazer o upgrade da minha outra máquina, a que eu uso para jogos. Após muita pesquisa, acabou que consegui um bom negócio em uma RX7900xtx, e com isso, agora tenho 24gb para rodar os modelos mais recentes. Mas me surpreendi com seu consumo, facilmente consumindo mais de 300 watts. O que levantou um alerta, deixar essa máquina ligada 24/7 não seria nada eficiente em termos de consumo de energia.  

### Ideia inicial

E se eu tivesse uma forma de ligar a máquina apenas quando eu precisasse? Precisaria de outro dispositivo para gerenciar, poderia usar um Rapberry pi para fazer isso, assim poderia deixar o pi ligado 24/7 , visto que não consumiria muita energia, e ele ligaria e desligaria a máquina que consome mais energia. 

### Wake-on-LAN

Pensando nisso, comecei a pesquisar formas de ligar minha máquina remotamente, e foi aí que descobri o Wake-on-LAN, ou simplesmente WoL. Após configurar minha placa mãe e o sistema operacional, consegui ligar minha máquina remotamente usando este simples comando:

```bash
wakeonlan <MAC_ADDRESS>
```

Devido ao modo como o WoL funciona, ele envia um pacote mágico na rede local, ou seja, você precisa estar na mesma rede para conseguir ligar a máquina, mas tudo bem, um problema a menos. Agora temos a habilidade de ligar a máquina remotamente, isso me levou a outra pergunta. Quando eu preciso ligar o servidor? A resposta é simples, quando eu precisar acessar os serviços que estão rodando nele, como o Ollama, ou qualquer outro serviço que eu queira hospedar.


### Interceptando o tráfego

A maioria dos serviços usa alguma porta específica, como o 11434 para o Ollama, onde ele abre uma conexão TCP, pensei em usar um proxy reverso para interceptar o tráfego, e quando necessário, ligar o servidor, e assim que o servidor estiver online, redirecionar o tráfego para ele, perfeito, agora teremos a habilidade de ligar o servidor remotamente quando necessário. 

```mermaid
sequenceDiagram
    participant User as Usuário
    participant Proxy as Proxy Reverso (Wakezilla)
    participant Server as Servidor (Ollama - porta 11434)

    User->>Proxy: Requisição TCP (porta 11434)
    Proxy->>Server: Verifica se está online
    alt Servidor OFF
        Proxy->>Server: Envia Wake-on-LAN (ligar servidor)
        Server-->>Proxy: Servidor inicializado
    end
    Proxy->>Server: Redireciona tráfego
    Server-->>Proxy: Resposta
    Proxy-->>User: Retorna dados

```

### Quando desligar o servidor? 

Agora que temos a habilidade de ligar o servidor remotamente, precisamos pensar em quando desligar o servidor.
Não quero deixar o servidor ligado 24/7, então pensei, já que estamos interceptando o tráfego, podemos monitorar, e quando não houver mais nenhum requeste, desligar o servidor. Assim, adicionamos a configuração de requestes por minuto, e quando não houver mais requisições, podemos desligar o servidor.


### Como fazer isso?

Após pesquisar um pouco, não encontrei muitas opções que fizessem exatamente o que eu queria, então decidi criar minha própria solução. Visto que, de qualquer maneira, precisaria configurar a máquina destino com algum tipo de software para receber o comando de desligar, decidi que fazer o mais simples possível, uma cli que abre um servidor web, e quando recebe uma requisição HTTP, sem autenticação (por enquanto), desliga a máquina, aproveitei e adicionei um  health check, para que o proxy reverso consiga verificar se a maquina está online ou não.


### Wakezilla

Com isso em mente, desenvolvi o Wakezilla, uma ferramenta simples que faz exatamente isso: intercepta o tráfego, e quando necessário, liga o servidor usando WoL, e quando não houver mais tráfego, desliga o servidor. 
Tudo isso de forma simples, usando Rust, e com um binário único, sem dependências externas, tornando fácil de usar em qualquer lugar.

![wakezilla](media/posts/wakezilla/output.gif)

### Projeto open source
O projeto está disponível no GitHub, e qualquer contribuição é bem-vinda, seja para adicionar novas funcionalidades ou para melhorar a documentação.
Se quiser testar, basta seguir as instruções no README do projeto, e caso tenha alguma dúvida, pode abrir uma issue que eu terei o maior prazer em ajudar.
O link do projeto é esse: [Wakezilla](https://github.com/guibeira/wakezilla)

