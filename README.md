Análise do Laboratório 7 - Produtor-Consumidor Condicional

1. Estrutura da Implementação Concorrente
A versão serial foi transformada em um sistema concorrente seguro utilizando Semáforos para evitar condições de corrida. A sincronização do buffer foi feita com três semáforos:
- 'mutex' (permite apenas 1 thread acessar a lista por vez).
- 'empty' (iniciado com 100, bloqueia produtores se o buffer atingir a capacidade máxima).
- 'full' (iniciado com 0, bloqueia consumidores se o buffer estiver vazio).

Para atender ao requisito de Consumidor Condicional, os consumidores testam a paridade do item (Par ou Ímpar). Se a condição não for atendida, o item é devolvido ao buffer e a thread realiza uma micropausa (sleep) para ceder a CPU, evitando um "Livelock" onde o mesmo consumidor ficaria pegando e devolvendo o mesmo item infinitamente.

Para garantir um encerramento limpo (evitando que o programa "trave" no final), utilizamos a técnica de "Poison Pill". Após a finalização de todos os produtores, a thread principal insere sinais de parada (-1) no buffer, avisando aos consumidores que o trabalho acabou.

2. Análise dos Cenários de Execução

- Cenário A (Produção mais rápida que o Consumo):
Ao configurar um tempo de produção menor que o tempo de consumo, ou ao usar mais produtores do que consumidores, o buffer enche rapidamente. O semáforo 'empty' atua travando os produtores, que ficam ociosos esperando que os consumidores processem os itens e liberem espaço.

- Cenário B (Consumo mais rápido que a Produção):
Ao configurar um tempo de consumo menor, ou usar mais consumidores, o buffer passa a maior parte do tempo vazio. Os consumidores disputam os poucos itens gerados e passam a maior parte do tempo bloqueados no semáforo 'full', aguardando o trabalho dos produtores.

- Cenário C (Desbalanceamento de Condição - Ex: Apenas 1 Consumidor):
Caso o sistema rode com apenas 1 consumidor (ex: ímpar), os itens da paridade oposta (pares) nunca são removidos definitivamente. Eles são devolvidos ao buffer repetidamente até ocuparem todos os 100 espaços. Nesse momento, o sistema entra em Deadlock, pois os produtores não conseguem inserir novos itens e o consumidor ímpar só encontra itens pares. Isso demonstra a necessidade de manter sempre representantes dos dois tipos de consumidores (Par e Ímpar) rodando em conjunto.
