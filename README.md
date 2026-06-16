import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Semaphore;
import java.util.function.IntPredicate;

class Buffer {
    private static final int MAX_SIZE = 100;

    private final List<Integer> data = new ArrayList<>();

    // mutex: exclusao mutua sobre a estrutura 'data'
    private final Semaphore mutex = new Semaphore(1);
    // emptySlots: numero de posicoes livres (comeca cheio de permissoes)
    private final Semaphore emptySlots = new Semaphore(MAX_SIZE);
    // filledSlots: numero de itens disponiveis (comeca em zero)
    private final Semaphore filledSlots = new Semaphore(0);

    // Produtor: bloqueia enquanto o buffer estiver cheio.
    public void put(int value) throws InterruptedException {
        emptySlots.acquire();   // espera por uma posicao livre
        mutex.acquire();        // entra na regiao critica
        data.add(value);
        System.out.println("Inserted: " + value + " | Buffer size: " + data.size());
        mutex.release();        // sai da regiao critica
        filledSlots.release();  // sinaliza que ha um item disponivel
    }

    // Consumidor condicional: bloqueia enquanto o buffer estiver vazio.
    // Remove um item e o consome caso satisfaca a condicao; caso contrario,
    // reinsere o item no buffer. Retorna o valor consumido ou -1 quando o item
    // inspecionado nao atendeu a condicao.
    //
    // A inspecao (remove -> testa -> consome/reinsere) e feita sob o mesmo mutex,
    // sem qualquer 'acquire' bloqueante dentro da regiao critica. Isso garante
    // que um produtor nao roube a posicao liberada durante uma eventual
    // reinsercao, evitando deadlock.
    public int remove(IntPredicate condition) throws InterruptedException {
        filledSlots.acquire();  // espera por um item
        mutex.acquire();        // entra na regiao critica
        int value = data.remove(0);
        if (condition.test(value)) {
            System.out.println("Removed: " + value + " | Buffer size: " + data.size());
            mutex.release();
            emptySlots.release();   // liberou uma posicao
            return value;
        }
        // Item nao atende a condicao deste consumidor: devolve ao buffer.
        data.add(value);
        System.out.println("Reinserted: " + value + " | Buffer size: " + data.size());
        mutex.release();
        filledSlots.release();      // o item continua disponivel
        return -1;
    }

    // Tamanho atual do buffer (leitura segura). Usado apenas para o encerramento
    // ordenado pela thread principal.
    public int size() throws InterruptedException {
        mutex.acquire();
        int s = data.size();
        mutex.release();
        return s;
    }
}









abstract class Consumer implements Runnable {
    protected final Buffer buffer;
    private final int sleepTime;
    protected final int id;

    public Consumer(int id, Buffer buffer, int sleepTime) {
        this.id = id;
        this.buffer = buffer;
        this.sleepTime = sleepTime;
    }

    // Condicao que define quais itens este consumidor pode consumir.
    protected abstract boolean accepts(int value);

    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                int item = buffer.remove(this::accepts); // bloqueia se vazio
                if (item == -1) {
                    // O item inspecionado nao atendia a condicao e foi reinserido.
                    // Cede a vez para dar chance a outras threads e tenta de novo.
                    Thread.yield();
                    continue;
                }
                System.out.println("Consumer " + id + " consumed item " + item);
                Thread.sleep(sleepTime);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}

// Consome apenas numeros pares.
class PairConsumer extends Consumer {
    public PairConsumer(int id, Buffer buffer, int sleepTime) {
        super(id, buffer, sleepTime);
    }

    @Override
    protected boolean accepts(int value) {
        return value % 2 == 0;
    }
}

// Consome apenas numeros impares.
class OddConsumer extends Consumer {
    public OddConsumer(int id, Buffer buffer, int sleepTime) {
        super(id, buffer, sleepTime);
    }

    @Override
    protected boolean accepts(int value) {
        return value % 2 != 0;
    }
}







class Producer implements Runnable {
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
                buffer.put(item); // bloqueia se o buffer estiver cheio
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
}








import java.util.ArrayList;
import java.util.List;

public class Main {
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

        // Cria e inicia os produtores (cada um em sua propria thread).
        List<Thread> producers = new ArrayList<>();
        for (int i = 1; i <= numProducers; i++) {
            Thread t = new Thread(new Producer(i, buffer, maxItemsPerProducer, producingTime), "Producer-" + i);
            producers.add(t);
            t.start();
        }

        // Cria e inicia os consumidores, alternando entre os dois tipos:
        // ids impares -> OddConsumer (impares); ids pares -> PairConsumer (pares).
        // Consumidores sao threads daemon para que a JVM possa encerrar quando os
        // produtores terminarem (eles podem ficar bloqueados aguardando itens).
        List<Thread> consumers = new ArrayList<>();
        for (int i = 1; i <= numConsumers; i++) {
            Consumer consumer = (i % 2 == 0)
                    ? new PairConsumer(i, buffer, consumingTime)
                    : new OddConsumer(i, buffer, consumingTime);
            Thread t = new Thread(consumer, consumer.getClass().getSimpleName() + "-" + i);
            t.setDaemon(true);
            consumers.add(t);
            t.start();
        }

        // Aguarda todos os produtores terminarem de produzir.
        for (Thread p : producers) {
            p.join();
        }
        System.out.println("All producers finished. Draining buffer...");

        // Aguarda o consumo dos itens restantes. Se o buffer parar de diminuir,
        // significa que os tipos de consumidor ativos nao conseguem consumir o
        // que sobrou (ex.: so ha um tipo de consumidor) -> encerra mesmo assim.
        int previous = -1;
        int stableRounds = 0;
        while (true) {
            int current = buffer.size();
            if (current == 0) {
                break;
            }
            if (current == previous) {
                if (++stableRounds >= 5) {
                    System.out.println("Remaining items cannot be consumed by the active consumer types: " + current);
                    break;
                }
            } else {
                stableRounds = 0;
            }
            previous = current;
            Thread.sleep(100);
        }
        System.out.println("Done.");
    }
}


