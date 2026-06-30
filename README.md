import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Buffer {
    private static final int CAPACITY = 50;
    private final List<Integer> data = new ArrayList<>();
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public void put(int value) throws InterruptedException {
        lock.lock();
        try {
            while (data.size() == CAPACITY) {
                notFull.await();
            }
            data.add(value);
            System.out.println("Inserted: " + value + " | Buffer size: " + data.size());
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public int remove() throws InterruptedException {
        lock.lock();
        try {
            while (data.isEmpty()) {
                notEmpty.await();
            }
            int value = data.remove(0);
            System.out.println("Removed: " + value + " | Buffer size: " + data.size());
            notFull.signal();
            return value;
        } finally {
            lock.unlock();
        }
    }
}
