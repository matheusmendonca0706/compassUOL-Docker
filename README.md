public class ResourceCheckTask implements Runnable {

    private int resourceId;

    public ResourceCheckTask(int resourceId) {
        this.resourceId = resourceId;
    }

    @Override
    public void run() {
        String threadName = Thread.currentThread().getName();

        System.out.println("[" + threadName + "] Checking resource " + resourceId);

        try {
            Thread.sleep(1000); // simula tempo
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        System.out.println("[" + threadName + "] Resource " + resourceId + " OK");
    }
}





public class SimpleConcurrentSolutionV2 {

    public static void main(String[] args) {

        Thread[] threads = new Thread[5];

        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(
                new ResourceCheckTask(i),
                "Resource-Thread-" + i
            );
            threads[i].start();
        }

        // esperar todas terminarem
        for (int i = 0; i < 5; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        System.out.println("All resources checked.");
    }
}
