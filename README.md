Aqui está uma versão mais simples e natural para o seu comments1.txt:
```text
1. Comportamento do tempo:
O tempo 'real' (o tempo do relógio mesmo) caiu quando coloquei mais threads, o que mostra que dividir o trabalho dá muito certo. Já o tempo 'user' subiu, o que é normal, porque ele soma o esforço de todos os núcleos do processador trabalhando juntos ao mesmo tempo.

2. Ponto de ganho e perda:
A maior melhora foi logo no começo, usando 2 e 4 threads. Mas chegou num ponto (ali pelas [X] threads) que parou de ajudar. O tempo 'real' travou e até começou a piorar depois disso.

3. Por que isso acontece:
No começo melhora muito porque cada pedaço da imagem é processado de forma independente, então os núcleos do computador conseguem trabalhar sem um atrapalhar o outro. 

Mas quando tem thread demais, o tiro sai pela culatra por dois motivos:
- O computador perde mais tempo organizando essa multidão de threads do que fazendo o trabalho de fato.
- Fica todo mundo tentando ler a imagem original e gravar a nova na memória RAM ao mesmo tempo, criando um grande "engarrafamento".

```
Basta trocar o [X] pelo número onde você notar que o tempo parou de cair!
