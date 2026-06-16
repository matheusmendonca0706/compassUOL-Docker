lab7-e1.go

// Etapa 1: Produtor e Consumidor Simples
//
// Uma goroutine produtora gera valores aleatórios entre 0 e 100 (leituras de
// um sensor). Uma goroutine consumidora lê do canal e imprime apenas os
// valores acima de um limite pré-definido. Ambos rodam em loop infinito.
//
// Execução: go run lab7-e1.go   (encerrar com Ctrl+C)
package main

import (
	"fmt"
	"math/rand"
)

const limite = 50

// produtor simula um sensor: gera leituras infinitamente e as envia no canal.
func produtor(ch chan int) {
	for {
		v := rand.Intn(100) // valor entre 0 e 99
		ch <- v
	}
}

// consumidor lê do canal indefinidamente e imprime só o que passa do limite.
func consumidor(ch chan int) {
	for {
		v := <-ch
		if v > limite {
			fmt.Println("Leitura:", v)
		}
	}
}

func main() {
	rand.Seed(42)

	ch := make(chan int)

	go produtor(ch)
	// O consumidor roda no fluxo principal, mantendo o programa vivo.
	consumidor(ch)
}







lab7-e2.go



// Etapa 2: Produtor Finito
//
// O produtor gera apenas 10.000 valores aleatórios. Ao terminar, fecha o canal.
// O consumidor usa "range", que encerra automaticamente quando o canal é
// fechado e esvaziado, fazendo o programa terminar.
//
// Execução: go run lab7-e2.go
package main

import (
	"fmt"
	"math/rand"
)

const (
	limite      = 50
	numLeituras = 10000
)

func produtor(ch chan int) {
	for i := 0; i < numLeituras; i++ {
		ch <- rand.Intn(100)
	}
	close(ch) // sinaliza ao consumidor que não há mais dados
}

func main() {
	rand.Seed(42)

	ch := make(chan int)

	go produtor(ch)

	// range encapsula a verificação de fechamento do canal: termina sozinho
	// quando o canal é fechado e todos os valores foram consumidos.
	for v := range ch {
		if v > limite {
			fmt.Println("Leitura:", v)
		}
	}
}







lab7-e3.go



// Etapa 3: Múltiplos Sensores (Produtores)
//
// Dois sensores (duas goroutines produtoras), cada um gerando uma quantidade
// aleatória de valores. Um consumidor central único recebe de ambos e imprime
// apenas os valores acima do limite.
//
// Como há mais de um produtor, o canal não pode ser fechado por um deles
// isoladamente (um close seguido de send geraria panic). Usamos um
// sync.WaitGroup para fechar o canal só depois que os dois sensores acabarem.
//
// Execução: go run lab7-e3.go
package main

import (
	"fmt"
	"math/rand"
	"sync"
)

const limite = 50

func sensor(id int, ch chan int, wg *sync.WaitGroup) {
	defer wg.Done()

	n := rand.Intn(10000) // quantidade aleatória de leituras deste sensor
	for i := 0; i < n; i++ {
		ch <- rand.Intn(100)
	}
	fmt.Printf("[Sensor %d] gerou %d leituras\n", id, n)
}

func main() {
	rand.Seed(42)

	ch := make(chan int)
	var wg sync.WaitGroup

	wg.Add(2)
	go sensor(1, ch, &wg)
	go sensor(2, ch, &wg)

	// Fecha o canal quando AMBOS os sensores terminarem.
	go func() {
		wg.Wait()
		close(ch)
	}()

	// Consumidor central único.
	for v := range ch {
		if v > limite {
			fmt.Println("Leitura:", v)
		}
	}
}






lab7-e4.go


// Etapa 4: Canal Unidirecional e Bufferizado
//
// Mesma ideia da Etapa 3, mas:
//   - Os produtores recebem o canal como send-only  (chan<- int).
//   - O consumidor recebe o canal como receive-only (<-chan int).
//     Essas restrições são verificadas em tempo de compilação.
//   - O canal é bufferizado (buffer de 100), reduzindo o bloqueio entre
//     produtores e consumidor: o sender só bloqueia quando o buffer enche.
//
// Observação: um canal bidirecional (make(chan int, 100)) é convertido
// automaticamente para as direções restritas ao ser passado às funções.
//
// Execução: go run lab7-e4.go
package main

import (
	"fmt"
	"math/rand"
	"sync"
)

const (
	limite  = 50
	bufSize = 100
)

// sensor escreve no canal: parâmetro send-only.
func sensor(id int, out chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()

	n := rand.Intn(10000)
	for i := 0; i < n; i++ {
		out <- rand.Intn(100)
	}
	fmt.Printf("[Sensor %d] gerou %d leituras\n", id, n)
}

// consumidor lê do canal: parâmetro receive-only.
func consumidor(in <-chan int, done chan<- bool) {
	for v := range in {
		if v > limite {
			fmt.Println("Leitura:", v)
		}
	}
	done <- true
}

func main() {
	rand.Seed(42)

	ch := make(chan int, bufSize) // canal bufferizado
	done := make(chan bool)
	var wg sync.WaitGroup

	wg.Add(2)
	go sensor(1, ch, &wg)
	go sensor(2, ch, &wg)

	go func() {
		wg.Wait()
		close(ch)
	}()

	go consumidor(ch, done)

	<-done // espera o consumidor drenar todo o canal
}





lab7-e5.go




// Etapa 5: Vários Consumidores
//
// Evolução da Etapa 4 com múltiplas goroutines consumidoras. Cada consumidor
// processa valores recebidos do mesmo canal e imprime com sua identificação
// própria (ex.: "Consumidor 1 recebeu 87").
//
// Divisão de trabalho: todos os consumidores fazem "range" sobre o MESMO canal.
// O runtime de Go entrega cada valor a UM único consumidor (quem estiver pronto
// para receber). Não há duplicação: cada leitura é processada por exatamente um
// consumidor. A distribuição NÃO é igual nem determinística — depende do
// escalonamento das goroutines, então a contagem por consumidor varia entre
// execuções. Isso é, na prática, um pool de workers (fan-out).
//
// Sincronização:
//   - wgProd: fecha o canal quando os dois produtores terminam.
//   - wgCons: garante que main só termina depois que todos os consumidores
//     esvaziarem o canal (senão o programa poderia sair antes de imprimir tudo).
//
// Execução: go run lab7-e5.go
package main

import (
	"fmt"
	"math/rand"
	"sync"
)

const (
	limite          = 50
	bufSize         = 100
	numConsumidores = 3
)

func sensor(id int, out chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()

	n := rand.Intn(10000)
	for i := 0; i < n; i++ {
		out <- rand.Intn(100)
	}
	fmt.Printf("[Sensor %d] gerou %d leituras\n", id, n)
}

func consumidor(id int, in <-chan int, wg *sync.WaitGroup) {
	defer wg.Done()

	for v := range in {
		if v > limite {
			fmt.Printf("Consumidor %d recebeu %d\n", id, v)
		}
	}
}

func main() {
	rand.Seed(42)

	ch := make(chan int, bufSize)
	var wgProd sync.WaitGroup
	var wgCons sync.WaitGroup

	// Dois sensores (produtores).
	wgProd.Add(2)
	go sensor(1, ch, &wgProd)
	go sensor(2, ch, &wgProd)

	// Vários consumidores compartilhando o mesmo canal.
	wgCons.Add(numConsumidores)
	for i := 1; i <= numConsumidores; i++ {
		go consumidor(i, ch, &wgCons)
	}

	// Fecha o canal quando os produtores terminarem.
	go func() {
		wgProd.Wait()
		close(ch)
	}()

	// Espera todos os consumidores terminarem de processar.
	wgCons.Wait()
}



