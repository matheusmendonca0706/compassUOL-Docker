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
