letra a 
def main():
    t = int(input())
    
    for _ in range(t):
        n, k = map(int, input().split())
        a = list(map(int, input().split()))
        
        ouro = 0
        cont = 0
        
        for x in a:
            if x >= k:
                ouro += x
            elif x == 0 and ouro > 0:
                ouro -= 1
                cont += 1
        
        print(cont)


if __name__ == "__main__":
    main()

LETRA B 

def main():
    a = int(input())
    b = int(input())
    c = int(input())
    
    ans = max(
        a + b + c,
        a + b * c,
        a * b + c,
        a * b * c,
        (a + b) * c,
        a * (b + c)
    )
    
    print(ans)


if __name__ == "__main__":
    main()

  LETRA C 

  def main():
    n = int(input())
    arr = list(map(int, input().split()))
    
    # exemplo de lógica comum: contar positivos
    cont = 0
    for x in arr:
        if x > 0:
            cont += 1
    
    print(cont)


if __name__ == "__main__":
    main()


  LETRA D

  import itertools

def main():
    s1 = input().strip()
    s2 = input().strip()

    target = 0
    for c in s1:
        if c == '+':
            target += 1
        else:
            target -= 1

    pos = 0
    q = 0

    for c in s2:
        if c == '+':
            pos += 1
        elif c == '-':
            pos -= 1
        else:
            q += 1

    ways = 0

    for comb in itertools.product([1, -1], repeat=q):
        cur = pos + sum(comb)
        if cur == target:
            ways += 1

    total = 2 ** q
    print(ways / total)


if __name__ == "__main__":
    main()
