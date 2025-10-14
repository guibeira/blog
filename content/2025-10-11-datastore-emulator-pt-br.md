
---
date: 2025-10-11
author: guibeira
tags: rust,datastore,google-cloud
extra:
  mermaid: true
  mermaid_theme: default
---


# Desenvolvendo um Emulador Alternativo para o Google Datastore: Como Resolvi o Problema de Memória e Corrupção de Dados


### Introdução: O Problema com o Datastore Emulator

Quem trabalha com o Google Datastore em ambientes locais provavelmente já se deparou com a seguinte situação: o emulador começa leve, mas, com o tempo, vira um [monstro](https://github.com/googleapis/python-datastore/issues/582#event-15802729926) de memória. E o pior, ele adora corromper seus arquivos de dados quando você menos espera.

No nosso time, o Datastore é parte crítica do stack. Apesar de ser um banco NoSQL poderoso, o emulador local não acompanhava nosso ritmo. Com dumps grandes, a performance despencava e o risco de corromper o banco aumentava. A cada novo dia de desenvolvimento, a mesma rotina: limpar, restaurar, rezar para não quebrar de novo.


### Tentativas de Solução 

Inicialmente, diminuímos o tamanho do backup, o que funcionava por um período, mas logo o problema aparecia novamente. A outra alternativa seria usar um banco real para cada desenvolvedor ou, em último caso, criar nosso próprio emulador — o que pareceu inicialmente uma ideia desafiadora, mas fascinante.

### Engenharia Reversa: Entendendo as APIs e Protobufs

Diante da decisão de criar um emulador alternativo, comecei pelo ponto mais importante: entender como o Datastore se comunica.

Felizmente, o Google disponibiliza os [protobufs](https://github.com/googleapis/googleapis/blob/c98457cd51f80e56daf7de102ed8d4c347ada663/google/datastore/v1/datastore.proto), utilizados na API do Datastore. Isso inclui as mensagens, serviços e métodos expostos pela API gRPC padrão, como:
- Lookup
- RunQuery
- BeginTransaction
- Commit
- Rollback
- AllocateIds

Com as interfaces em mãos, comecei a implementar um emulador próprio. A ideia foi criar uma conexão gRPC que imitasse o comportamento do Datastore. Comecei com operações básicas, como Lookup, sempre hardcode, e gradualmente, fui implementando as demais operações, sempre hard code, para entender o fluxo. Em dado momento, já tinha todos os métodos implementados, mas sempre retornando dados estáticos, nesse momento, decidi que era hora de pensar em como armazenar os dados.


### Principais Decisões de Design

In-Memory First: A prioridade era desempenho e simplicidade. Ao manter tudo na RAM, evitei lock de disco e I/O intensivo. Isso por si só eliminou a maioria dos problemas de corrupção e vazamento.

Salvamento no shutdown: Ao desligar o emulador, ele automaticamente persiste os dados para um arquivo datastore.bin. Isso garante que o estado local não seja perdido entre sessões. Temos o risco de perder dados se o processo for morto abruptamente, mas isso é um trade-off aceitável para nós, visto que esse emulador é apenas para desenvolvimento local.

### Garantindo Compatibilidade

Para garantir que nosso emulador fosse fiel ao original, fiz testes lado a lado: subi o emulador padrão e o meu emulador, criei dois clientes, um para cada emulador, executei a mesma sequência de operações comparando os resultados. Cada teste verifica uma feature específica, como inserção, consulta com filtros, transações, etc. Obviamente, não temos como cobrir 100% dos casos, mas cobri o essencial para o meu fluxo de trabalho. Isso ajudou muito a discobrir bugs e inconsistências. Inclusive, foi nesse momento que percebi que ao fazer uma consulta e a quantidade de items for maior que o limite, o emulador vai fazer paginação, e o cliente vai agregar todos os resultados.
  Com o avançar dos testes, percebi que o emulador original tinha algumas limitações, algumas operações não eram suportadas by design, como operações de ["IN", "!=", e "NOT-IN"](https://cloud.google.com/datastore/docs/tools/datastore-emulator#known_issues). Nesse momento, decidi que usaria também um [banco real](https://github.com/guibeira/datastore-emulator/tree/main/tests) para testes mais complexos, o que se mostrou essencial para garantir a compatibilidade, dado as limitações do emulador original.


### Importação e Exportação de Dumps

   Outro ponto importante foi a capacidade de importar dumps do datastore. E isso é uma feature excencial no meu setup de desenvolvimento, visto que não posso criar tudo do zero. Felizmente, o formato do dump é simples, basicamente um arquivo com várias entidades serializadas em protobuf. Felizmente outra pessoa já tinha feito a engenharia reversa do formato, como pode conferir nesse repositório [dsbackups](https://github.com/remko/dsbackups), oque me ajudou bastante a entender o formato. Com isso, implementei a feature de importação, e deixei de lado a exportação, visto que não é algo que eu precise no momento.

   A importação é feita em background, e depois de algumas melhorias demora em torno de 5 segundos para importar um dump com 150k entidades, bem melhor que 10 minutos do emulador original.

### Ok Funciona, mas quão rapido é?

Já com o emulador funcional, me perguntei, o quão rápido ele era comparado ao original? O intuito do desenvolvimento era apenas resolver o problema de memória e corrupção, mas se fosse mais rápido, seria um bônus. Visto que o emulador original é escrito em Java, e o meu em Rust, eu esperava uma diferença significativa. Para medir, criei um script que executa uma série de operações (inserção, consulta, atualização, exclusão) em ambos os emuladores e mede o tempo total gasto. Os resultados foram surpreendentes: meu emulador foi consistentemente mais rápido em todas as operações. Em alguns casos, como single insert a diferença era de até 50x.


```bash
python benchmark/test_benchmark.py --num-clients 30 --num-runs 5

--- Benchmark Summary ---

Operation: Single Insert
  - Rust (30 clients, 5 runs each):
    - Total time: 0.8413 seconds
    - Avg time per client: 0.0280 seconds
  - Java (30 clients, 5 runs each):
    - Total time: 48.1050 seconds
    - Avg time per client: 1.6035 seconds
  - Verdict: Rust was 57.18x faster overall.

Operation: Bulk Insert (50)
  - Rust (30 clients, 5 runs each):
    - Total time: 9.5209 seconds
    - Avg time per client: 0.3174 seconds
  - Java (30 clients, 5 runs each):
    - Total time: 163.7277 seconds
    - Avg time per client: 5.4576 seconds
  - Verdict: Rust was 17.20x faster overall.

Operation: Simple Query
  - Rust (30 clients, 5 runs each):
    - Total time: 2.2610 seconds
    - Avg time per client: 0.0754 seconds
  - Java (30 clients, 5 runs each):
    - Total time: 29.3397 seconds
    - Avg time per client: 0.9780 seconds
  - Verdict: Rust was 12.98x faster overall.

```


Okay, mas e a memória?

```bash
docker stats

CONTAINER ID   NAME                        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O        PIDS
b44ea75d665b   datastore_emulator_google   0.22%     939.2MiB / 17.79GiB   5.16%     2.51MB / 2.57MB   1.93MB / 332kB   70
aa0caa062568   datastore_emulator_rust     0.00%     18.35MiB / 17.79GiB   0.10%     2.52MB / 3.39MB   0B / 0B          15

```
O emulador original, após rodar o benchmark já consumia quase 1GB de RAM. Enquanto o meu, consumia apenas 18MB. Isso é uma diferença enorme.


Interessante não é mesmo, se quise rodar o benchmark na sua máquina, [aqui](https://github.com/guibeira/datastore-emulator/tree/main/benchmark) está as instruções.



### Conclusão e Próximos Passos

O resultado final foi um binário com cerca de 10MB de tamanho, mais rápido e muito mais eficiente em termos de memória e CPU. Tenho absoluta certeza que o código tem muitas partes que poderia ser melhoradas, por isso se manja de Rust, , por favor mande um PR. Mas dado o que existia antes, estou muito satisfeito

Um passo importante para ter paridade de features é a implementação de endpoints HTTP para facilitar o uso de clientes web como o [dsasmin](https://github.com/remko/dsadmin). Isso está na minha lista de próximos passos, assim como melhorar a cobertura de testes e adicionar mais features conforme necessário. 

Se quiser testar, o projeto está disponível no GitHub: [Datastore Emulator](https://github.com/guibeira/datastore-emulator)
