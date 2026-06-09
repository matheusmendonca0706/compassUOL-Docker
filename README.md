WebStatsSemaphoreMain


import java.util.Random;
import java.util.concurrent.Semaphore;

// Classe que mantem as estatisticas (versao protegida por Semaforo)
class WebStatsSem {
    private long totalAccess = 0;
    private int totalPurchases = 0;
    private int totalFailures = 0;
    private int totalNothing = 0;
    private int onlineUsers = 0;

    // Semaforo binario (valor inicial 1) usado como mutex para garantir
    // exclusao mutua no acesso as regioes criticas (os contadores compartilhados).
    private final Semaphore mutex = new Semaphore(1);

    // Usuario acessa o sistema
    public void access() {
        try {
            mutex.acquire();              // entra na regiao critica
            try {
                totalAccess++;
                onlineUsers++;
            } finally {
                mutex.release();          // sai da regiao critica
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // Usuario realiza uma compra
    public void purchase() {
        try {
            mutex.acquire();
            try {
                totalPurchases++;
            } finally {
                mutex.release();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // Ocorreu uma falha
    public void failure() {
        try {
            mutex.acquire();
            try {
                totalFailures++;
            } finally {
                mutex.release();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // Usuario nem compra nem falha
    public void nothing() {
        try {
            mutex.acquire();
            try {
                totalNothing++;
            } finally {
                mutex.release();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // Usuario faz logout
    public void logout() {
        try {
            mutex.acquire();
            try {
                onlineUsers--;
            } finally {
                mutex.release();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // Impressao das estatisticas atuais
    public void printStats() {
        System.out.println("========= Estatisticas do Sistema =========");
        System.out.println("Total de Acessos: " + totalAccess);
        System.out.println("Total de Compras: " + totalPurchases);
        System.out.println("Total de Falhas: " + totalFailures);
        System.out.println("Total de acessos sem compras ou falhas: " + totalNothing);
        System.out.println("Usuarios Online: " + onlineUsers);
        System.out.println("=======================================================");
    }
}

// Classe que simula acoes de um usuario no sistema
class UserSimulationSem implements Runnable {
    private WebStatsSem stats;
    private Random random;

    public UserSimulationSem(WebStatsSem stats) {
        this.stats = stats;
        this.random = new Random();
    }

    @Override
    public void run() {
        try {
            // Usuario acessa o sistema
            stats.access();

            // Simula tempo navegando
            Thread.sleep(random.nextInt(300));

            // Decide se faz compra, falha ou apenas navega
            int action = random.nextInt(3); // 0 = compra, 1 = falha, 2 = nada
            if (action == 0) {
                stats.purchase();
            } else if (action == 1) {
                stats.failure();
            } else {
                stats.nothing();
            }

            // Simula tempo antes de logout
            Thread.sleep(random.nextInt(200));

            // Usuario sai do sistema
            stats.logout();

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

// Classe principal que executa a simulacao concorrente (controle por Semaforo)
public class WebStatsSemaphoreMain {
    public static void main(String[] args) {

        if (args.length < 1) {
            System.err.println("Usage: java WebStatsSemaphoreMain number_users");
            System.exit(1);
        }

        int numUsers = Integer.valueOf(args[0]); // quantidade de threads (usuarios simultaneos)

        WebStatsSem stats = new WebStatsSem();
        Thread[] users = new Thread[numUsers];

        // Criacao e inicializacao das threads
        for (int i = 0; i < numUsers; i++) {
            users[i] = new Thread(new UserSimulationSem(stats));
            users[i].start();
        }

        // Aguarda todas as threads terminarem
        for (int i = 0; i < numUsers; i++) {
            try {
                users[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        // Imprime estatisticas finais (agora corretas e consistentes)
        stats.printStats();
    }
}







WebStatsAtmVarMain



import java.util.Random;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.AtomicInteger;

// Classe que mantem as estatisticas (versao com Variaveis Atomicas)
class WebStatsAtm {
    // Cada contador e' uma variavel atomica: as operacoes de incremento e
    // decremento sao executadas de forma atomica (sem condicao de corrida),
    // sem necessidade de lock explicito.
    private final AtomicLong totalAccess = new AtomicLong(0);
    private final AtomicInteger totalPurchases = new AtomicInteger(0);
    private final AtomicInteger totalFailures = new AtomicInteger(0);
    private final AtomicInteger totalNothing = new AtomicInteger(0);
    private final AtomicInteger onlineUsers = new AtomicInteger(0);

    // Usuario acessa o sistema
    public void access() {
        totalAccess.incrementAndGet();
        onlineUsers.incrementAndGet();
    }

    // Usuario realiza uma compra
    public void purchase() {
        totalPurchases.incrementAndGet();
    }

    // Ocorreu uma falha
    public void failure() {
        totalFailures.incrementAndGet();
    }

    // Usuario nem compra nem falha
    public void nothing() {
        totalNothing.incrementAndGet();
    }

    // Usuario faz logout
    public void logout() {
        onlineUsers.decrementAndGet();
    }

    // Impressao das estatisticas atuais
    public void printStats() {
        System.out.println("========= Estatisticas do Sistema =========");
        System.out.println("Total de Acessos: " + totalAccess.get());
        System.out.println("Total de Compras: " + totalPurchases.get());
        System.out.println("Total de Falhas: " + totalFailures.get());
        System.out.println("Total de acessos sem compras ou falhas: " + totalNothing.get());
        System.out.println("Usuarios Online: " + onlineUsers.get());
        System.out.println("=======================================================");
    }
}

// Classe que simula acoes de um usuario no sistema
class UserSimulationAtm implements Runnable {
    private WebStatsAtm stats;
    private Random random;

    public UserSimulationAtm(WebStatsAtm stats) {
        this.stats = stats;
        this.random = new Random();
    }

    @Override
    public void run() {
        try {
            // Usuario acessa o sistema
            stats.access();

            // Simula tempo navegando
            Thread.sleep(random.nextInt(300));

            // Decide se faz compra, falha ou apenas navega
            int action = random.nextInt(3); // 0 = compra, 1 = falha, 2 = nada
            if (action == 0) {
                stats.purchase();
            } else if (action == 1) {
                stats.failure();
            } else {
                stats.nothing();
            }

            // Simula tempo antes de logout
            Thread.sleep(random.nextInt(200));

            // Usuario sai do sistema
            stats.logout();

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

// Classe principal que executa a simulacao concorrente (controle por variaveis atomicas)
public class WebStatsAtmVarMain {
    public static void main(String[] args) {

        if (args.length < 1) {
            System.err.println("Usage: java WebStatsAtmVarMain number_users");
            System.exit(1);
        }

        int numUsers = Integer.valueOf(args[0]); // quantidade de threads (usuarios simultaneos)

        WebStatsAtm stats = new WebStatsAtm();
        Thread[] users = new Thread[numUsers];

        // Criacao e inicializacao das threads
        for (int i = 0; i < numUsers; i++) {
            users[i] = new Thread(new UserSimulationAtm(stats));
            users[i].start();
        }

        // Aguarda todas as threads terminarem
        for (int i = 0; i < numUsers; i++) {
            try {
                users[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        // Imprime estatisticas finais (agora corretas e consistentes)
        stats.printStats();
    }
}