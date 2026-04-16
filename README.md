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



-----


public class SimpleConcurrentSolutionV2

import java.lang.Thread;
import java.lang.Runnable;
import java.util.Random;

public class SimpleConcurrentSolutionV2 {

    // 🔥 Agora cada task verifica UM recurso
    private static class ResourceCheckTask implements Runnable {

        private static final Random random = new Random();
        private int resourceId;

        // ✅ recebe o ID do recurso
        public ResourceCheckTask(int resourceId) {
            this.resourceId = resourceId;
        }

        @Override
        public void run() {
            Thread currentThread = Thread.currentThread();

            System.out.println("[" + currentThread.getName() + "] Verificando Recurso " + resourceId);

            try {
                int sleepTime = 1000 + random.nextInt(2000);
                System.out.println("[" + currentThread.getName() + "] Duração: " + sleepTime + "ms");
                Thread.sleep(sleepTime);
            } catch (InterruptedException e) {
                System.err.println("[" + currentThread.getName() + "] interrompida.");
                Thread.currentThread().interrupt();
                return;
            }

            System.out.println("[" + currentThread.getName() + "] Recurso " + resourceId + " OK");
        }
    }

    // mesma tarefa de logs
    private static Runnable logSetupTask = new Runnable() {
        @Override
        public void run() {
            Thread currentThread = Thread.currentThread();

            System.out.println("[" + currentThread.getName() + "] INÍCIO: Logs");

            try {
                Thread.sleep(4000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }

            System.out.println("[" + currentThread.getName() + "] FIM: Logs");
        }
    };

    public static void main(String[] args) {

        Thread mainThread = Thread.currentThread();
        System.out.println("[" + mainThread.getName() + "] INÍCIO");

        // Thread de logs
        Thread tLogs = new Thread(logSetupTask, "Log-Thread");
        tLogs.start();

        // 🔥 Criando 5 threads (uma por recurso)
        Thread[] resourceThreads = new Thread[5];

        for (int i = 0; i < 5; i++) {
            resourceThreads[i] = new Thread(
                new ResourceCheckTask(i + 1),
                "Resource-Thread-" + (i + 1)
            );
            resourceThreads[i].start();
        }

        // 🔥 Esperar todas terminarem
        try {
            tLogs.join();

            for (int i = 0; i < 5; i++) {
                resourceThreads[i].join();
            }

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        System.out.println("[" + mainThread.getName() + "] FIM");
    }
}
