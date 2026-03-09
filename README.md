LETRA C 

def build_post(preorder, inorder):
    if not preorder:
        return []

    root = preorder[0]
    idx = inorder.index(root)

    left = build_post(preorder[1:1+idx], inorder[:idx])
    right = build_post(preorder[1+idx:], inorder[idx+1:])

    return left + right + [root]


def main():
    n = int(input())

    preorder = list(map(int, input().split()))
    inorder = list(map(int, input().split()))

    postorder = build_post(preorder, inorder)

    print(*postorder)


if __name__ == "__main__":
    main()

LETRA D 


calls = 0


def sorted_segment(a, l, r):
    for i in range(l, r-1):
        if a[i] > a[i+1]:
            return False
    return True


def merge(a, l, m, r):
    left = a[l:m]
    right = a[m:r]

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

    if sorted_segment(a, l, r):
        return

    m = (l + r) // 2

    mergesort(a, l, m)
    mergesort(a, m, r)

    merge(a, l, m, r)


def main():
    global calls

    n = int(input())
    arr = list(map(int, input().split()))

    mergesort(arr, 0, n)

    print(calls)


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
