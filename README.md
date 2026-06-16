Aqui estão as alterações exatas que você precisa fazer em cada arquivo para transformar o código serial em uma solução concorrente segura usando semáforos, atendendo a todos os requisitos da especificação.
### 1. Modificações em Buffer.java
Você precisa adicionar os semáforos para controle de concorrência e capacidade máxima. Um semáforo atuará como *mutex* e os outros dois controlarão os espaços vazios e cheios.
**Adicione as importações e os atributos:**
```java
import java.util.concurrent.Semaphore;

// Dentro da classe Buffer:
private final Semaphore mutex = new Semaphore(1);
private final Semaphore empty = new Semaphore(100); [span_2](start_span)// Capacidade máxima de 100 itens[span_2](end_span)
private final Semaphore full = new Semaphore(0);

```
**Altere os métodos put e remove:**
```java
public void put(int value) throws InterruptedException {
    empty.acquire(); [span_3](start_span)[span_4](start_span)// Aguarda se o buffer estiver cheio[span_3](end_span)[span_4](end_span)
    mutex.acquire(); // Garante exclusão mútua

    data.add(value);
    System.out.println("Inserted: " + value + " | Buffer size: " + data.size());

    mutex.release();
    full.release(); [span_5](start_span)// Sinaliza que há um novo item disponível[span_5](end_span)
}

public int remove() throws InterruptedException {
    full.acquire(); [span_6](start_span)[span_7](start_span)// Aguarda se o buffer estiver vazio[span_6](end_span)[span_7](end_span)
    mutex.acquire();

    int value = data.remove(0);
    System.out.println("Removed: " + value + " | Buffer size: " + data.size());

    mutex.release();
    empty.release(); // Libera espaço no buffer

    return value;
}

```
### 2. Modificações em Consumer.java
O consumidor precisa rodar como uma thread e implementar a lógica condicional de pares e ímpares.
**Altere a assinatura da classe e o construtor:**
```java
class Consumer implements Runnable { // Adicionar implements Runnable
    private final Buffer buffer;
    private final int sleepTime;
    private final int id;
    private final boolean consumesEven; // Define se consome pares ou ímpares
    
    public Consumer(int id, Buffer buffer, int sleepTime, boolean consumesEven) {
        this.id = id;
        this.buffer = buffer;
        this.sleepTime = sleepTime;
        this.consumesEven = consumesEven;
    }

```
**Substitua o método process() pelo método run():**
```java
    @Override
    public void run() {
        while (true) {
            try {
                int item = buffer.remove();
                boolean isEven = (item % 2 == 0);

                [span_10](start_span)// Verifica se atende à condição do consumidor[span_10](end_span)
                if ((consumesEven && isEven) || (!consumesEven && !isEven)) {
                    System.out.println("Consumer " + id + " (Even: " + consumesEven + ") consumed item " + item);
                    Thread.sleep(sleepTime);
                } else {
                    [span_11](start_span)// Não atende à condição: reinserir no buffer[span_11](end_span)
                    buffer.put(item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }

```
### 3. Modificações em Producer.java
O produtor também precisa rodar como uma thread independente.
**Altere a assinatura da classe:**
```java
class Producer implements Runnable { // Adicionar implements Runnable

```
**Substitua o método produce() pelo método run() e ajuste as chamadas:**
```java
    @Override
    public void run() {
        for (int i = 0; i < maxItems; i++) {
            try {
                Thread.sleep(sleepTime);
                int item = (int) (Math.random() * 100);
                System.out.println("Producer " + id + " produced item " + item);
                buffer.put(item); // Agora pode lançar InterruptedException
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }

```
### 4. Modificações em Main.java
Em vez de chamar os métodos sequencialmente, você precisa instanciar e iniciar as Threads.
**Substitua os loops for de criação na função main por:**
```java
        // Iniciar Produtores
        for (int i = 1; i <= numProducers; i++) {
            Producer producer = new Producer(i, buffer, maxItemsPerProducer, producingTime);
            new Thread(producer, "Producer-" + i).start();
        }
        
        // Iniciar Consumidores (metade pares, metade ímpares)
        for (int i = 1; i <= numConsumers; i++) {
            boolean consumesEven = (i % 2 == 0); // Alterna entre consumidor par e ímpar
            Consumer consumer = new Consumer(i, buffer, consumingTime, consumesEven);
            new Thread(consumer, "Consumer-" + i).start();
        }

```
