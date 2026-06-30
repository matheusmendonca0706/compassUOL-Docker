Semelhanças
 Controle de fluxo: Ambas travam na condição de guarda, operam no buffer e sinalizam na saída.
 Exclusão mútua: Previnem race conditions garantindo acesso exclusivo à região crítica.
 Spurious wakeups: Ambas usam ⁠while⁠ (e não ⁠if⁠) para revalidar o estado após o wake-up.
 Comportamento externo: O resultado do bloqueio (buffer cheio/vazio) é o mesmo em ambos os cenários.
Diferenças
 Mecanismo do Lock: A Etapa 1 usa o monitor intrínseco do objeto (lock/unlock implícitos no bloco ⁠synchronized⁠). A Etapa 2 usa ⁠ReentrantLock⁠, exigindo chamadas explícitas de ⁠lock()⁠ e ⁠unlock()⁠ (este último sempre dentro de um ⁠finally⁠).
 Sinalização e Overhead: O ⁠synchronized⁠ tem uma fila de espera única. O ⁠notifyAll()⁠ gera overhead porque acorda todas as threads (produtores e consumidores), forçando reavaliações desnecessárias.
 Otimização de Fila: O ⁠ReentrantLock⁠ com ⁠Condition⁠ permite filas separadas (⁠notEmpty⁠ e ⁠notFull⁠). Isso possibilita o uso de ⁠signal()⁠ para acordar apenas a thread alvo (ex: produtor acorda apenas consumidor), poupando CPU.
 Flexibilidade vs. Verbosidade: A Etapa 2 é mais verbosa e manual, mas entrega recursos avançados de concorrência, como fairness (justiça na fila), ⁠tryLock()⁠ e espera passível de interrupção.
