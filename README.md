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
