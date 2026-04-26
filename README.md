# Лабораторная работа №3: Структура данных
## Задание: 
<img width="1265" height="181" alt="image" src="https://github.com/user-attachments/assets/2cf48ecc-af8f-42e8-9600-1eb2528c8d59" />

## Реализация:

### Листинг программы:

``` python
import math

def compute_ordering(k: int):
    N = 1 << k  # 2^k
    matrix = [[0] * N for _ in range(N)]

    for y in range(N):
        for x in range(N):
            # Вычисляем Morton-код: чередование битов y и x, начиная со старшего бита y
            code = 0
            for i in range(k):
                # Извлекаем i-й бит (0 – младший)
                y_bit = (y >> i) & 1
                x_bit = (x >> i) & 1
                # Устанавливаем биты: y_bit на позицию (2*i+1), x_bit на позицию (2*i)
                code |= (y_bit << (2 * i + 1)) | (x_bit << (2 * i))
            matrix[y][x] = code + 1  # нумерация с 1
    return matrix

try:
    k = int(input("Введите k (количество клеток = 4^k): "))
    if k < 0:
        print("k должно быть неотрицательным.")
        exit(0)
    N = 1 << k
    print(f"\nРазмер квадрата: {N} x {N}\n")
    mat = compute_ordering(k)

    # Вывод матрицы (для больших N лучше записать в файл)
    for row in mat:
        print(' '.join(f"{num:4d}" for num in row))
except ValueError:
    print("Ошибка: введите целое число k.")
print("Автор: Кочаров Арсений Андреевич, группа: 090301-ПОВа-о25")

```
## Результат выполнения программы:
<img width="803" height="464" alt="image" src="https://github.com/user-attachments/assets/a85947ba-7d3f-4ff9-b392-217693505bf2" />
