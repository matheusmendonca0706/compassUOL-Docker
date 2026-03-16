LETRA A
t = int(input())

for _ in range(t):
    n = int(input())
    
    p = list(map(int, input().split()))
    s = input()
    
    p = [x-1 for x in p]
    
    visitado = [False] * n
    resposta = [0] * n
    
    for i in range(n):
        if not visitado[i]:
            
            ciclo = []
            atual = i
            
            while not visitado[atual]:
                visitado[atual] = True
                ciclo.append(atual)
                atual = p[atual]
            
            pretos = 0
            for v in ciclo:
                if s[v] == '0':
                    pretos += 1
            
            for v in ciclo:
                resposta[v] = pretos
    
    print(*resposta)

LETRA B 

# entrada
n, t = map(int, input().split())
a = list(map(int, input().split()))

pos = 1

while pos < t:
    pos = pos + a[pos - 1]

# saída
if pos == t:
    print("YES")
else:
    print("NO")

LETRA C 

import heapq

# entrada
n, m = map(int, input().split())

grafo = [[] for _ in range(n + 1)]

for _ in range(m):
    a, b, c = map(int, input().split())
    grafo[a].append((b, c))

# dijkstra
INF = 10**18
dist = [INF] * (n + 1)
dist[1] = 0

fila = [(0, 1)]

while fila:
    d, u = heapq.heappop(fila)

    if d > dist[u]:
        continue

    for v, w in grafo[u]:
        if dist[u] + w < dist[v]:
            dist[v] = dist[u] + w
            heapq.heappush(fila, (dist[v], v))

# saída
for i in range(1, n + 1):
    print(dist[i], end=" ")


LETRA D 
def find(pai, x):
    if pai[x] != x:
        pai[x] = find(pai, pai[x])
    return pai[x]

def union(pai, rank, a, b):
    ra = find(pai, a)
    rb = find(pai, b)
    
    if ra != rb:
        if rank[ra] < rank[rb]:
            pai[ra] = rb
        else:
            pai[rb] = ra
            if rank[ra] == rank[rb]:
                rank[ra] += 1
        return True
    return False


while True:
    
    m, n = map(int, input().split())
    
    if m == 0 and n == 0:
        break
    
    arestas = []
    total = 0
    
    for _ in range(n):
        x, y, z = map(int, input().split())
        arestas.append((z, x, y))
        total += z
    
    arestas.sort()
    
    pai = list(range(m))
    rank = [0] * m
    
    mst = 0
    
    for peso, u, v in arestas:
        if union(pai, rank, u, v):
            mst += peso
    
    economia = total - mst
    
    print(economia)





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
    calls = 1

    if r - l <= 1:
        return calls

    mid = (l + r) // 2

    calls += mergesort(a, l, mid)
    calls += mergesort(a, mid, r)

    merge(a, l, mid, r)

    return calls


def main():
    n = int(input())
    arr = list(map(int, input().split()))

    result = mergesort(arr, 0, n)

    print(result)


if __name__ == "__main__":
    main()
