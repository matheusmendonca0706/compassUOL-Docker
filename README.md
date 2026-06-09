import java.io.*;
import java.util.*;
import java.util.concurrent.Semaphore;

public class FileSimilarity {

    static long totalSum = 0;
    static int N;

    // Semáforos para controle de concorrência conforme especificação e slides
    static Semaphore multiplex;    // Controle de admissão (N/2)
    static Semaphore mutexSum;     // Exclusão mútua para totalSum
    static Semaphore mutexBuffer;  // Exclusão mútua para a fila do produtor/consumidor
    static Semaphore mutexStack;   // Exclusão mútua para o Stack
    static Semaphore items;        // Sinalização: itens disponíveis no buffer (inicia em 0)

    static class FileData {
        String name;
        List<Long> chunks;
        FileData(String name, List<Long> chunks) {
            this.name = name;
            this.chunks = chunks;
        }
    }

    // Buffer de comunicação clássico e o Stack solicitado
    static Queue<FileData> buffer = new LinkedList<>();
    static Stack<FileData> historyStack = new Stack<>();

    static class Producer implements Runnable {
        private String filePath;

        public Producer(String file){
            this.filePath = file;
        }

        @Override
        public void run() {
            try {
                // Multiplex: limita a N/2 leituras simultâneas
                multiplex.acquire();
                List<Long> fingerprint = fileSum(filePath);
                multiplex.release();

                // Mutex: protege a inserção no buffer compartilhado
                mutexBuffer.acquire();
                buffer.add(new FileData(filePath, fingerprint));
                mutexBuffer.release();

                // Sinalização: avisa o consumidor que há um novo item
                items.release();

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                // Aguarda a sinalização de que há itens disponíveis
                items.acquire();

                // Mutex: protege a remoção do buffer
                mutexBuffer.acquire();
                FileData current = buffer.poll();
                mutexBuffer.release();

                // Mutex: protege a cópia e inserção no Stack
                List<FileData> pastFiles;
                mutexStack.acquire();
                pastFiles = new ArrayList<>(historyStack);
                historyStack.push(current);
                mutexStack.release();

                // Calcula similaridade com os arquivos que já estavam no Stack
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
        
        // Inicialização dos semáforos baseada nos conceitos da aula
        multiplex = new Semaphore(Math.max(1, N / 2)); 
        mutexSum = new Semaphore(1);
        mutexBuffer = new Semaphore(1);
        mutexStack = new Semaphore(1);
        items = new Semaphore(0); // Sinalização inicializa em 0
        
        Thread[] producers = new Thread[N];
        Thread[] consumers = new Thread[N];

        // Dispara todos os Produtores e Consumidores
        for (int i = 0; i < N; i++) {
            producers[i] = new Thread(new Producer(args[i]));
            consumers[i] = new Thread(new Consumer());
            producers[i].start();
            consumers[i].start(); 
        }
        
        // Barreira final para o main aguardar o processamento
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
                
                // Exclusão Mútua na soma global
                mutexSum.acquire();
                totalSum += sum;
                mutexSum.release();
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
