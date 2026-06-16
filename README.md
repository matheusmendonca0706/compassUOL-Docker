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
