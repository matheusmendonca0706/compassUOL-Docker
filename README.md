Direto ao ponto: o que está acontecendo é um **livelock** (uma espécie de loop infinito onde o sistema trabalha sem sair do lugar) e ele está sendo causado por dois fatores principais agindo juntos.
Aqui está a explicação do problema e como consertar agora mesmo.
### 1. A Causa do "Loop Infinito"
 * **O Consumidor Egoísta (Falta de Pausa):** A especificação exige que o item seja reinserido no buffer caso não atenda à condição do consumidor. No entanto, do jeito que o código está, se o consumidor devolve o item pro buffer no bloco else, ele não faz nenhuma pausa. A thread imediatamente recomeça o loop while, pega o **mesmo item** de novo, testa, falha e devolve. Fica fazendo isso milhares de vezes por segundo, travando a CPU.
 * **O Teste com 1 Consumidor:** Se você rodou o script de teste padrão do lab (bash run.sh 1 100 150 1 150), ele cria apenas **1 consumidor**. Pela nossa lógica do Main, esse consumidor i=1 será um Consumidor Ímpar. Se o produtor gerar um número Par, esse número **nunca** será consumido. O consumidor ímpar vai rejeitar e devolver para o buffer eternamente.
### 2. A Solução (Código)
Para corrigir isso, precisamos dar uma pequena "respirada" (sleep) na thread quando ela rejeita um item, permitindo que a outra thread (o consumidor do tipo oposto) tenha a chance de pegar esse item no buffer.
Altere o bloco else dentro do run() no seu arquivo **Consumer.java**:
```java
    @Override
    public void run() {
        while (true) {
            try {
                int item = buffer.remove();
                boolean isEven = (item % 2 == 0);

                // Verifica se atende à condição do consumidor
                if ((consumesEven && isEven) || (!consumesEven && !isEven)) {
                    System.out.println("Consumer " + id + " (Even: " + consumesEven + ") consumed item " + item);
                    Thread.sleep(sleepTime);
                } else {
                    // Não atende à condição: reinserir no buffer
                    buffer.put(item);
                    
                    // FIX: Adicionar uma pequena pausa para evitar o Livelock e 
                    // ceder a CPU para o outro consumidor pegar o item!
                    Thread.sleep(10); 
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }

```
### 3. Como testar corretamente
Lembre-se que para o Produtor-Consumidor Condicional funcionar sem travar itens no buffer, você **precisa** garantir que haja pelo menos um consumidor Par e um Ímpar rodando.
Ao rodar o script no terminal, passe 2 ou um número par para a quantidade de consumidores (o quarto argumento da chamada):
**Execute com 2 consumidores (1 Par e 1 Ímpar):**
```bash
bash run.sh 1 100 150 2 150

```
Faça essa alteração no sleep(10) e rode com 2 consumidores. O loop maluco vai sumir e os logs do terminal farão sentido. Pode compilar e rodar!
