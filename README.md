import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class FileSimilarity {

    static long totalSum = 0;
    static Semaphore mutex;
    static Semaphore multiplex;
    static int N;

    // Estrutura auxiliar para guardar os dados do arquivo
    static class FileData {
        String name;
        List<Long> chunks;
        FileData(String name, List<Long> chunks) {
            this.name = name;
            this.chunks = chunks;
        }
    }

    // Fila bloqueante: conecta produtor e consumidor em tempo real
    static BlockingQueue<FileData> readyQueue = new LinkedBlockingQueue<>();
    
    // Stack substituindo o HashMap, como você pediu
    static Stack<FileData> historyStack = new Stack<>();

    static class Producer implements Runnable {
        private String filePath;

        public Producer(String file){
            this.filePath = file;
        }

        @Override
        public void run() {
            try {
                // Controle de admissão N/2
                multiplex.acquire();
                List<Long> fingerprint = fileSum(filePath);
                multiplex.release();

                // Assim que termina de ler, já envia para os consumidores
                readyQueue.put(new FileData(filePath, fingerprint));
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                // Pega um arquivo recém-processado da fila (espera se estiver vazia)
                FileData current = readyQueue.take();
                List<FileData> pastFiles;
                
                // Sincroniza o Stack rapidinho só para copiar os anteriores e se adicionar
                synchronized(historyStack) {
                    pastFiles = new ArrayList<>(historyStack);
                    historyStack.push(current);
                }

                // Calcula a similaridade concorrentemente fora do bloco synchronized!
                for (FileData past : pastFiles) {
                    float similarityScore = similarity(current.chunks, past.chunks);
                    
                    synchronized(System.out) {
                        System.out.printf("Similarity between %s and %s: %.5f%%\n", 
                                current.name, past.name, (similarityScore * 100));
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length < 2) {
            System.err.println("Usage: java FileSimilarity filepath1 filepath2 filepathN");
            System.exit(1);
        }

        N = args.length;
        multiplex = new Semaphore(Math.max(1, N / 2));
        mutex = new Semaphore(1);
        
        Thread[] producers = new Thread[N];
        Thread[] consumers = new Thread[N];

        // Dispara Produtores e Consumidores AO MESMO TEMPO
        for (int i = 0; i < N; i++) {
            producers[i] = new Thread(new Producer(args[i]));
            consumers[i] = new Thread(new Consumer());
            producers[i].start();
            consumers[i].start(); 
        }
        
        for (int i = 0; i < N; i++) {
            producers[i].join();
            consumers[i].join();
        }

        System.out.println("Total sum: " + totalSum);
    }

    private static List<Long> fileSum(String filePath) throws IOException, InterruptedException {
        File file = new File(filePath);
        List<Long> chunks = new ArrayList<>();
        try (FileInputStream inputStream = new FileInputStream(file)) {
            byte[] buffer = new byte[100];
            int bytesRead;
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                long sum = sum(buffer, bytesRead);
                chunks.add(sum);
                
                mutex.acquire();
                totalSum += sum;
                mutex.release();
            }
        }
        return chunks;
    }

    private static long sum(byte[] buffer, int length) {
        long sum = 0;
        for (int i = 0; i < length; i++) {
            sum += Byte.toUnsignedInt(buffer[i]);
        }
        return sum;
    }

    private static float similarity(List<Long> base, List<Long> target) {
        int counter = 0;
        List<Long> targetCopy = new ArrayList<>(target);

        for (Long value : base) {
            if (targetCopy.contains(value)) {
                counter++;
                targetCopy.remove(value);
            }
        }

        return (float) counter / base.size();
    }
}
