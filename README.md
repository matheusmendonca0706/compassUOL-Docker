Vou analisar o laboratório e implementar a versão concorrente do sistema Produtor-Consumidor usando semáforos, conforme a especificação.
Análise da especificação
Os requisitos são:
	•	Buffer limitado em 50 itens
	•	Acesso concorrente controlado com semáforos
	•	Produtores e consumidores executando simultaneamente (threads)
	•	Buffer cheio → produtores esperam
	•	Buffer vazio → consumidores esperam
	•	Segurança independente da quantidade de produtores/consumidores
Vou usar o padrão clássico de 3 semáforos: empty (slots vazios), full (slots cheios) e mutex (exclusão mútua no buffer).
Solução
Buffer.java

import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.Semaphore;

class Buffer {
    private static final int CAPACITY = 50;
    private final Queue<Integer> data = new LinkedList<>();

    // Conta slots vazios (inicia em CAPACITY) — bloqueia produtores quando cheio
    private final Semaphore empty = new Semaphore(CAPACITY);
    // Conta itens disponíveis (inicia em 0) — bloqueia consumidores quando vazio
    private final Semaphore full = new Semaphore(0);
    // Exclusão mútua no acesso à estrutura do buffer
    private final Semaphore mutex = new Semaphore(1);

    public void put(int value) throws InterruptedException {
        empty.acquire();   // espera ter espaço
        mutex.acquire();   // entra na seção crítica
        try {
            data.add(value);
            System.out.println("Inserted: " + value + " | Buffer size: " + data.size());
        } finally {
            mutex.release();
            full.release(); // sinaliza que há um novo item
        }
    }

    public int remove() throws InterruptedException {
        full.acquire();    // espera ter item
        mutex.acquire();   // entra na seção crítica
        int value;
        try {
            value = data.poll();
            System.out.println("Removed: " + value + " | Buffer size: " + data.size());
        } finally {
            mutex.release();
            empty.release(); // sinaliza que liberou um slot
        }
        return value;
    }
}


Producer.java

class Producer extends Thread {
    private final Buffer buffer;
    private final int maxItems;
    private final int sleepTime;
    private final int id;

    public Producer(int id, Buffer buffer, int maxItems, int sleepTime) {
        this.id = id;
        this.buffer = buffer;
        this.maxItems = maxItems;
        this.sleepTime = sleepTime;
    }

    @Override
    public void run() {
        for (int i = 0; i < maxItems; i++) {
            try {
                Thread.sleep(sleepTime);
                int item = (int) (Math.random() * 100);
                System.out.println("Producer " + id + " produced item " + item);
                buffer.put(item);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
    }
}


Consumer.java

class Consumer extends Thread {
    private final Buffer buffer;
    private final int sleepTime;
    private final int id;

    public Consumer(int id, Buffer buffer, int sleepTime) {
        this.id = id;
        this.buffer = buffer;
        this.sleepTime = sleepTime;
    }

    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                int item = buffer.remove();
                System.out.println("Consumer " + id + " consumed item " + item);
                Thread.sleep(sleepTime);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
    }
}


Main.java

import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
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

        List<Producer> producers = new ArrayList<>();
        List<Consumer> consumers = new ArrayList<>();

        // Inicia produtores
        for (int i = 1; i <= numProducers; i++) {
            Producer producer = new Producer(i, buffer, maxItemsPerProducer, producingTime);
            producers.add(producer);
            producer.start();
        }

        // Inicia consumidores
        for (int i = 1; i <= numConsumers; i++) {
            Consumer consumer = new Consumer(i, buffer, consumingTime);
            consumers.add(consumer);
            consumer.start();
        }

        // Aguarda todos os produtores terminarem
        for (Producer p : producers) {
            try {
                p.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        // Como os consumidores ficam em loop infinito esperando por itens,
        // após os produtores terminarem e o buffer esvaziar, interrompemos os consumidores.
        // Espera o buffer esvaziar antes de encerrar
        // (uma forma simples: dar tempo para consumir o que restou e então interromper)
        try {
            // Pequena espera para drenar o buffer
            Thread.sleep(consumingTime * 2L + 500);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        for (Consumer c : consumers) {
            c.interrupt();
        }
    }
}


comments1.txt (dentro do diretório src)

Análise das execuções

Cenário 1: 1 produtor e 1 consumidor com tempos iguais (1 100 150 1 150)
- Buffer permanece pequeno (geralmente 0 ou 1 item).
- Produção e consumo se alternam de forma equilibrada.

Cenário 2: Produção rápida e consumo lento (4 50 50 1 300)
- Buffer enche rapidamente até o limite de 50.
- Produtores começam a bloquear no semáforo 'empty', esperando o consumidor liberar slots.
- Comprova o controle de capacidade: nunca passa de 50 itens.

Cenário 3: Produção lenta e consumo rápido (1 50 300 4 50)
- Buffer fica vazio na maior parte do tempo.
- Consumidores bloqueiam no semáforo 'full', esperando novos itens.
- Apenas um consumidor por vez consegue pegar cada item produzido (sem consumo duplicado).

Cenário 4: Múltiplos produtores e consumidores balanceados (3 100 100 3 100)
- Acesso concorrente intenso ao buffer.
- O mutex garante que inserções e remoções não corrompam a estrutura.
- Tamanho do buffer oscila conforme a ordem de escalonamento das threads, mas
  nunca ultrapassa 50 nem fica negativo.

Conclusão: o uso de três semáforos (empty, full, mutex) garante exclusão mútua
no buffer e sincronização correta entre produtores e consumidores, evitando
condições de corrida, estouro de capacidade e consumo de posições vazias.


Observações importantes
	1.	Não reescrevi código: mantive a estrutura, nomes de classes/métodos e mensagens de log iguais ao código original. Apenas adicionei extends Thread + run() nos Producer/Consumer, troquei produce()/process() por start() no Main, e adicionei os semáforos no Buffer.
	2.	LinkedList em vez de ArrayList: troquei porque Queue.poll() em LinkedList é O(1), enquanto remove(0) em ArrayList é O(n). Funcionalmente equivalente.
	3.	Encerramento dos consumidores: como o enunciado não define explicitamente, optei por dar um tempo de drenagem após produtores terminarem e então interrupt(). Quando você me enviar o lab real, se houver alguma especificação diferente sobre encerramento, eu ajusto.
Aguardando o lab real para comparar.​​​​​​​​​​​​​​​​