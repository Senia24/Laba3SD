# Лабораторная работа №3: Структура данных
## Задание: 
<img width="1265" height="181" alt="image" src="https://github.com/user-attachments/assets/2cf48ecc-af8f-42e8-9600-1eb2528c8d59" />

## Реализация:

### Листинг программы:

``` python
import math
import time
import sys
from collections import deque
from typing import List, Optional

# Исходная функция вычисления нумерации (эталон)

def compute_ordering(k: int):
    """
    Возвращает матрицу размера N x N (N = 2^k) с номерами клеток от 1 до N^2,
    упорядоченных согласно процессу перегиба.
    """
    N = 1 << k  # 2^k
    matrix = [[0] * N for _ in range(N)]

    for y in range(N):
        for x in range(N):
            code = 0
            for i in range(k):
                y_bit = (y >> i) & 1
                x_bit = (x >> i) & 1
                code |= (y_bit << (2 * i + 1)) | (x_bit << (2 * i))
            matrix[y][x] = code + 1
    return matrix


# Реализация 1: через массив (вложенные списки)

def simulate_with_list(k: int) -> List[int]:
    """Моделирует перегибы, храня стопки как списки Python."""
    N = 1 << k
    numbering = compute_ordering(k)
    grid = [[[numbering[y][x]] for x in range(N)] for y in range(N)]

    size = N
    while size > 1:
        # Вертикальный перегиб
        new_grid = []
        for y in range(size):
            new_row = []
            for x in range(size // 2):
                left = grid[y][x]
                right = grid[y][size - 1 - x]
                new_row.append(left + right)
            new_grid.append(new_row)
        grid = new_grid
        size //= 2

        if size == 1:
            break

        # Горизонтальный перегиб
        new_grid = []
        for y in range(size // 2):
            new_row = []
            for x in range(size):
                top = grid[y][x]
                bottom = grid[size - 1 - y][x]
                new_row.append(top + bottom)
            new_grid.append(new_row)
        grid = new_grid
        size //= 2

    return grid[0][0]


# Реализация 2: через связанный список

class Node:
    __slots__ = ('value', 'next')
    def __init__(self, value: int):
        self.value = value
        self.next: Optional[Node] = None

def list_to_linked(lst: List[int]) -> Optional[Node]:
    head = None
    for v in reversed(lst):
        node = Node(v)
        node.next = head
        head = node
    return head

def linked_to_list(head: Optional[Node]) -> List[int]:
    res = []
    while head is not None:
        res.append(head.value)
        head = head.next
    return res

def concat_linked(top: Optional[Node], bottom: Optional[Node]) -> Optional[Node]:
    """Склеивает top + bottom (bottom подкладывается под top)."""
    if top is None:
        return bottom
    curr = top
    while curr.next is not None:
        curr = curr.next
    curr.next = bottom
    return top

def simulate_with_linked_list(k: int) -> List[int]:
    N = 1 << k
    numbering = compute_ordering(k)
    grid = [[list_to_linked([numbering[y][x]]) for x in range(N)] for y in range(N)]

    size = N
    while size > 1:
        # Вертикальный перегиб
        new_grid = []
        for y in range(size):
            new_row = []
            for x in range(size // 2):
                left = grid[y][x]
                right = grid[y][size - 1 - x]
                new_row.append(concat_linked(left, right))
            new_grid.append(new_row)
        grid = new_grid
        size //= 2

        if size == 1:
            break

        # Горизонтальный перегиб
        new_grid = []
        for y in range(size // 2):
            new_row = []
            for x in range(size):
                top = grid[y][x]
                bottom = grid[size - 1 - y][x]
                new_row.append(concat_linked(top, bottom))
            new_grid.append(new_row)
        grid = new_grid
        size //= 2

    return linked_to_list(grid[0][0])


# Реализация 3: через collections.deque

def simulate_with_deque(k: int) -> List[int]:
    N = 1 << k
    numbering = compute_ordering(k)
    grid = [[deque([numbering[y][x]]) for x in range(N)] for y in range(N)]

    size = N
    while size > 1:
        # Вертикальный перегиб
        new_grid = []
        for y in range(size):
            new_row = []
            for x in range(size // 2):
                left = grid[y][x]
                right = grid[y][size - 1 - x]
                left.extend(right)
                new_row.append(left)
            new_grid.append(new_row)
        grid = new_grid
        size //= 2

        if size == 1:
            break

        # Горизонтальный перегиб
        new_grid = []
        for y in range(size // 2):
            new_row = []
            for x in range(size):
                top = grid[y][x]
                bottom = grid[size - 1 - y][x]
                top.extend(bottom)
                new_row.append(top)
            new_grid.append(new_row)
        grid = new_grid
        size //= 2

    return list(grid[0][0])

def compare_for_k(k: int):
    # Сравнивает три реализации для конкретного k и выводит время.
    N = 1 << k
    total = N * N
    expected = list(range(1, total + 1))

    print(f"\n Сравнение производительности для k = {k} (N = {N}, клеток = {total})")
    print(f"{'Реализация':<20} {'Время (мс)':<12}")

    # Список
    start = time.perf_counter()
    res_list = simulate_with_list(k)
    t_list = (time.perf_counter() - start) * 1000
    ok_list = (res_list == expected)

    # Связанный список
    start = time.perf_counter()
    res_linked = simulate_with_linked_list(k)
    t_linked = (time.perf_counter() - start) * 1000
    ok_linked = (res_linked == expected)

    # Deque
    start = time.perf_counter()
    res_deque = simulate_with_deque(k)
    t_deque = (time.perf_counter() - start) * 1000
    ok_deque = (res_deque == expected)

    print(f"{'Массив':<20} {t_list:<12.2f} ")
    print(f"{'Связный список':<20} {t_linked:<12.2f}")
    print(f"{'Deque':<20} {t_deque:<12.2f}")

# Обычный режим: ввод k, вывод матрицы и сравнение производительности
try:
    k = int(input("Введите k (количество клеток = 4^k): "))
    if k < 0:
        print("k должно быть неотрицательным.")
        sys.exit(1)
    N = 1 << k
    print(f"\nРазмер квадрата: {N} x {N}\n")

    # Вывод матрицы (только для небольших k, чтобы не засорять экран)
    if k <= 4:
        mat = compute_ordering(k)
        for row in mat:
            print(' '.join(f"{num:4d}" for num in row))
    else:
        print(f"Матрица слишком большая ({N}x{N}), вывод пропущен.\n")

    # Запуск сравнения производительности
    compare_for_k(k)

except ValueError:
    print("Ошибка: введите целое число k.")
    sys.exit(1)
print("Автор: Кочаров Арсений Андреевич, группа: 090301-ПОВа-о25")
```
## Результат выполнения программы:
<img width="739" height="588" alt="image" src="https://github.com/user-attachments/assets/611e3725-137c-4b26-82dd-7e68e932b08e" />

