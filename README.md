Como você me pediu respostas rápidas: o seu código está travando porque os **Consumidores estão entrando em espera infinita (deadlock) no final da execução**.
Como o while(true) do consumidor nunca acaba, quando os produtores terminam de gerar os itens, os consumidores ficam parados na linha full.acquire() esperando por um número que nunca vai chegar. O terminal fica "pendurado".
Para corrigir isso, precisamos de uma técnica clássica de sistemas concorrentes chamada **Poison Pill (Pílula de Veneno) / Sinal de Parada**. A thread Main vai esperar os produtores terminarem e, em seguida, enviar um item falso (como o número -1) para o buffer. Quando o consumidor ler esse -1, ele sabe que é hora de encerrar.
Faça exatamente estas duas alterações:
### 1. Atualize a classe Main.java
Vamos guardar as threads dos produtores, aguardar eles terminarem com join() e inserir o sinal de parada no final.
```java
public class Main {
    // Adicione o throws InterruptedException na assinatura da main
    public static void main(String[] args) throws InterruptedException {
        if (args.length != 5) {
            System.out.println("Use: java Main <num_producers> <max_items_per_producer> <producing_time> <num_consumers> <consuming_time>");
            return;
        }
        
        int numProducers = Integer.parseInt(args[0]);
        int maxItemsPerProducer = Integer.parseInt(args[1]);
        int producingTime = Integer.parseInt(args[2]);
        int numConsumers = Integer.parseInt(args[3]);
        int consumingTime = Integer.parseInt(args[4]);
        
        Buffer buffer = new Buffer();
        
        // Array para guardar as threads dos produtores
        Thread[] producers = new Thread[numProducers];
        
        for (int i = 1; i <= numProducers; i++) {
            Producer producer = new Producer(i, buffer, maxItemsPerProducer, producingTime);
            producers[i-1] = new Thread(producer, "Producer-" + i);
            producers[i-1].start();
        }
        
        for (int i = 1; i <= numConsumers; i++) {
            boolean consumesEven = (i % 2 == 0); 
            Consumer consumer = new Consumer(i, buffer, consumingTime, consumesEven);
            new Thread(consumer, "Consumer-" + i).start();
        }

        // NOVO: A thread principal aguarda todos os produtores terminarem o trabalho
        for (Thread p : producers) {
            p.join();
        }

        // NOVO: Produtores terminaram. Enviar um sinal de parada (-1) para CADA consumidor sair do loop
        for (int i = 0; i < numConsumers; i++) {
            buffer.put(-1);
        }
    }
}

```
### 2. Atualize o loop da classe Consumer.java
Adicione o trecho de código para verificar se o item puxado é o nosso sinal de parada -1.
```java
    @Override
    public void run() {
        while (true) {
            try {
                int item = buffer.remove();
                
                // NOVO: Se receber o sinal de parada, sai do loop e encerra a Thread
                if (item == -1) {
                    System.out.println("Consumer " + id + " recebeu sinal de parada e foi encerrado.");
                    break;
                }

                boolean isEven = (item % 2 == 0);

                // Verifica se atende à condição do consumidor
                if ((consumesEven && isEven) || (!consumesEven && !isEven)) {
                    System.out.println("Consumer " + id + " (Even: " + consumesEven + ") consumed item " + item);
                    Thread.sleep(sleepTime);
                } else {
                    // Não atende à condição: reinserir no buffer e dar tempo pro outro consumidor pegar
                    buffer.put(item);
                    Thread.sleep(10); 
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }

```
**Importante:** Lembre-se de rodar o código com pelo menos **2 consumidores** (um número par no script) para que haja um Consumidor Par e um Consumidor Ímpar. Se você rodar com 1 só, o código vai travar porque metade dos números nunca será consumida.
Comando correto de exemplo:
bash run.sh 1 100 10 2 10
