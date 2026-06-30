Semelhancas:
- Ambas resolvem o problema do buffer limitado (capacidade 50) com a mesma
  logica: enquanto a condicao de guarda nao for satisfeita a thread espera,
  depois opera sobre o buffer e por fim sinaliza as outras threads.
- As duas garantem exclusao mutua na regiao critica (acesso a lista
  compartilhada), evitando condicoes de corrida.
- Em ambas a condicao de guarda e testada com while (e nao if), para
  reavaliar o estado apos acordar e tratar wake-ups espurios.
- As duas bloqueiam o consumidor quando o buffer esta vazio e o produtor
  quando o buffer esta cheio, e produzem o mesmo comportamento externo
  independente da quantidade de produtores e consumidores.

Diferencas:
- Etapa 1 (synchronized) usa o lock intrinseco do proprio objeto (monitor)
  e uma unica fila de espera implicita (wait/notifyAll). Como existe so um
  conjunto de espera, notifyAll() acorda TODAS as threads bloqueadas
  (produtores e consumidores), e cada uma reavalia sua guarda, gerando
  despertares desnecessarios.
- Etapa 2 (ReentrantLock + Condition) usa um lock explicito com DUAS
  condicoes separadas (notFull e notEmpty). Assim e possivel sinalizar
  apenas o grupo relevante: o produtor faz signal em notEmpty (acorda um
  consumidor) e o consumidor faz signal em notFull (acorda um produtor).
  Isso otimiza a solucao, pois evita acordar threads que nao podem
  prosseguir e usa signal() no lugar de notifyAll().
- Na Etapa 1 a aquisicao e liberacao do lock sao implicitas (entrar/sair do
  metodo synchronized). Na Etapa 2 o lock e adquirido e liberado
  explicitamente com lock()/unlock(), exigindo o unlock() no bloco finally
  para garantir a liberacao mesmo em caso de excecao.
- O ReentrantLock + Condition oferece mais flexibilidade (multiplas
  condicoes, possibilidade de fairness, tryLock e espera interrompivel),
  ao custo de um codigo mais verboso e da responsabilidade de liberar o
  lock manualmente.
