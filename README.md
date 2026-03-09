Letra A 
import sys

def main():
    n, a, b, c = map(int, sys.stdin.readline().split())

    dp = [-10**9] * (n + 1)
    dp[0] = 0

    for i in range(1, n + 1):
        if i >= a:
            dp[i] = max(dp[i], dp[i - a] + 1)
        if i >= b:
            dp[i] = max(dp[i], dp[i - b] + 1)
        if i >= c:
            dp[i] = max(dp[i], dp[i - c] + 1)

    print(dp[n])

if __name__ == "__main__":
    main()


LETRA C

import sys
sys.setrecursionlimit(1000000)

def build_post(pre_l, pre_r, in_l, in_r):
    if pre_l >= pre_r:
        return []

    root = preorder[pre_l]
    idx = pos[root]
    left_size = idx - in_l

    left = build_post(pre_l + 1, pre_l + 1 + left_size, in_l, idx)
    right = build_post(pre_l + 1 + left_size, pre_r, idx + 1, in_r)

    return left + right + [root]


def main():
    global preorder, pos

    input = sys.stdin.readline

    n = int(input())
    preorder = list(map(int, input().split()))
    inorder = list(map(int, input().split()))

    pos = {v:i for i,v in enumerate(inorder)}

    post = build_post(0, n, 0, n)

    print(*post)

if __name__ == "__main__":
    main()

LETRA D 

import sys

def is_sorted(a, l, r):
    for i in range(l, r-1):
        if a[i] > a[i+1]:
            return False
    return True


def merge(a, l, mid, r):
    left = a[l:mid]
    right = a[mid:r]

    i = j = 0
    k = l

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            a[k] = left[i]
            i += 1
        else:
            a[k] = right[j]
            j += 1
        k += 1

    while i < len(left):
        a[k] = left[i]
        i += 1
        k += 1

    while j < len(right):
        a[k] = right[j]
        j += 1
        k += 1


def mergesort(a, l, r):
    global calls
    calls += 1

    if r - l <= 1:
        return

    if is_sorted(a, l, r):
        return

    mid = (l + r) // 2

    mergesort(a, l, mid)
    mergesort(a, mid, r)

    merge(a, l, mid, r)


def main():
    global calls

    input = sys.stdin.readline

    n = int(input())
    arr = list(map(int, input().split()))

    calls = 0

    mergesort(arr, 0, n)

    print(calls)


if __name__ == "__main__":
    main()


LETRA B

import sys

def main():
    n = int(sys.stdin.readline())

    notas = [100, 20, 10, 5, 1]

    total = 0

    for nota in notas:
        total += n // nota
        n %= nota

    print(total)

if __name__ == "__main__":
    main()







Ótima observação. Vou revisar a implementação inteira da atividade garantindo que ela esteja 100% alinhada com o slide do professor:

Principais regras do slide:
	•	Template Method → public final
	•	Métodos comuns → private
	•	Métodos gancho (hooks) → protected abstract
	•	Subclasses → protected override
	•	Ordem do algoritmo fixa  ￼

Agora segue a implementação revisada corretamente.

⸻

1️⃣ Classe abstrata (Template Method)

Arquivo:

ValidadorDocumento.java

public abstract class ValidadorDocumento {

    // Template Method
    public final void validar() {
        abrirArquivo();
        verificar();
        analisar();
        finalizarValidacao();
    }

    // Métodos comuns
    private void abrirArquivo() {
        System.out.println("Abrindo arquivo...");
    }

    private void finalizarValidacao() {
        System.out.println("Validação finalizada.");
    }

    // Métodos gancho (hooks)
    protected abstract void verificar();

    protected abstract void analisar();
}

✔ template public final
✔ métodos comuns private
✔ hooks protected abstract

⸻

2️⃣ Implementação para PDF

Arquivo:

ValidadorPDF.java

public class ValidadorPDF extends ValidadorDocumento {

    @Override
    protected void verificar() {
        System.out.println("Verificando permissões do PDF (extração de texto permitida)");
    }

    @Override
    protected void analisar() {
        System.out.println("Convertendo PDF para texto...");
        System.out.println("Aplicando expressões regulares para detectar CPF e nome...");
    }
}

✔ hooks sobrescritos como protected

⸻

3️⃣ Implementação para Imagem

Arquivo:

ValidadorImagem.java

public class ValidadorImagem extends ValidadorDocumento {

    @Override
    protected void verificar() {
        System.out.println("Verificando resolução mínima da imagem...");
    }

    @Override
    protected void analisar() {
        System.out.println("Extraindo texto da imagem...");
        System.out.println("Aplicando expressões regulares para detectar CPF e nome...");
    }
}


⸻

4️⃣ Classe principal (Bootstrap)

Arquivo que já existe no projeto:

Bootstrap.java

public class Bootstrap {

    public static void main(String[] args) {

        System.out.println("Projeto de Software");

        ValidadorDocumento pdf = new ValidadorPDF();
        ValidadorDocumento imagem = new ValidadorImagem();

        System.out.println("\nValidação de PDF:");
        pdf.validar();

        System.out.println("\nValidação de Imagem:");
        imagem.validar();
    }
}


⸻

5️⃣ Estrutura final do projeto

Dentro de src/main/java:

src/main/java
│
├── Bootstrap.java
├── ValidadorDocumento.java
├── ValidadorPDF.java
└── ValidadorImagem.java


⸻

6️⃣ Ordem garantida pelo Template Method

Execução:

validar()
 ├─ abrirArquivo()
 ├─ verificar()
 ├─ analisar()
 └─ finalizarValidacao()

Essa ordem não pode ser alterada pelas subclasses, pois o método é final.

⸻

7️⃣ UML correta para entregar no papel

Desenhe exatamente assim:

            <<abstract>>
        ValidadorDocumento
--------------------------------
+ validar()
# verificar()
# analisar()
- abrirArquivo()
- finalizarValidacao()

            ▲
       ┌────┴────┐
   ValidadorPDF  ValidadorImagem
--------------------------------
# verificar()    # verificar()
# analisar()     # analisar()

Legenda UML:

+ public
# protected
- private


⸻

8️⃣ Checklist de correção (baseado no slide)

Critério	Status
classe abstrata	✅
template method	✅
template public final	✅
ordem fixa	✅
métodos comuns private	✅
hooks protected	✅
subclasses implementam hooks	✅
herança correta	✅


⸻

✅ Agora a implementação está 100% alinhada com o slide do professor.

⸻

💡 Se quiser, posso também te mostrar um detalhe de modelagem que provavelmente o professor espera no diagrama UML da atividade (e que quase toda a turma erra).
