import java.util.ArrayList;
import java.util.List;

class Buffer {
    private static final int CAPACITY = 50;
    private final List<Integer> data = new ArrayList<>();

    public synchronized void put(int value) throws InterruptedException {
        while (data.size() == CAPACITY) {
            wait();
        }
        data.add(value);
        System.out.println("Inserted: " + value + " | Buffer size: " + data.size());
        notifyAll();
    }

    public synchronized int remove() throws InterruptedException {
        while (data.isEmpty()) {
            wait();
        }
        int value = data.remove(0);
        System.out.println("Removed: " + value + " | Buffer size: " + data.size());
        notifyAll();
        return value;
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

    public void run() {
        for (int i = 0; i < maxItems; i++) {
            try {
                Thread.sleep(sleepTime);
                int item = (int) (Math.random() * 100);
                System.out.println("Producer " + id + " produced item " + item);
                buffer.put(item);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}









class Consumer implements Runnable {
    private final Buffer buffer;
    private final int sleepTime;
    private final int id;

    public Consumer(int id, Buffer buffer, int sleepTime) {
        this.id = id;
        this.buffer = buffer;
        this.sleepTime = sleepTime;
    }

    public void run() {
        while (true) {
            try {
                int item = buffer.remove();
                System.out.println("Consumer " + id + " consumed item " + item);
                Thread.sleep(sleepTime);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}





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

        for (int i = 1; i <= numProducers; i++) {
            Producer producer = new Producer(i, buffer, maxItemsPerProducer, producingTime);
            new Thread(producer).start();
        }

        for (int i = 1; i <= numConsumers; i++) {
            Consumer consumer = new Consumer(i, buffer, consumingTime);
            new Thread(consumer).start();
        }
    }
}





