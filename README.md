Indo direto ao ponto: o seu código não zera no final porque está sofrendo de um **Livelock (Inanição/Starvation)**.
Isso acontece pela combinação de três fatores na sua implementação:
 1. **Semáforos Injustos (Non-Fair):** No Java, quando você instancia um semáforo como new Semaphore(0), ele é criado no modo "injusto". Isso significa que não há fila. Quando o consumidor rejeita um item, ele dá release() no filledSlots e, como essa thread já está com a CPU, ela dá a volta no loop while e faz o acquire() novamente antes que a thread do outro consumidor consiga "acordar". Ele fica rodando em falso, infinitamente.
 2. **Thread.yield() é ineficiente:** Na JVM moderna, o yield() é apenas uma "dica" para o escalonador do Sistema Operacional. Na prática, ele é frequentemente ignorado, e a thread não cede o processamento, agravando o livelock do item anterior.
 3. **Número ímpar de consumidores:** Pela lógica do seu Main, se você passou 1 no argumento de consumidores, o sistema cria apenas um OddConsumer. Logo, todos os números pares gerados pelo produtor ficarão encalhados no buffer para sempre, pois não existe ninguém para consumi-los.
### Como corrigir agora mesmo
Faça estas duas alterações simples:
**1. Em Buffer.java**, force os semáforos a serem "Justos" (Fair), passando true no construtor. Isso obriga a JVM a respeitar uma fila (FIFO), garantindo que o consumidor que está dormindo seja o próximo a pegar o acquire().
```java
    // mutex: exclusao mutua sobre a estrutura 'data'
    private final Semaphore mutex = new Semaphore(1, true);
    // emptySlots: numero de posicoes livres
    private final Semaphore emptySlots = new Semaphore(MAX_SIZE, true);
    // filledSlots: numero de itens disponiveis
    private final Semaphore filledSlots = new Semaphore(0, true);

```
**2. Em Consumer.java**, troque o yield() por um bloqueio real (sleep) de alguns milissegundos para forçar uma troca de contexto da CPU.
```java
                int item = buffer.remove(this::accepts); // bloqueia se vazio
                if (item == -1) {
                    // Cede a vez com uma pausa real para o outro consumidor pegar o item
                    Thread.sleep(5); 
                    continue;
                }

```
Feito isso, garanta que está executando o código com um número **par** de consumidores (ex: 2, 4) para que existam consumidores pares e ímpares rodando ao mesmo tempo. O buffer irá zerar corretamente e a execução encerrará de forma limpa.
