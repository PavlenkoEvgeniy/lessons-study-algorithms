# Урок 11: Кратчайший Путь — Алгоритм Дейкстры

## Зачем этот урок

**Проблема:** Есть карта городов с расстояниями между ними. Как найти кратчайший маршрут из Москвы в Санкт-Петербург?

```
    Москва
     /|\
    / | \
200 /  |  \ 300
   /   |   \
Тверь  Рязань Воронеж
   \   |   /
    \  |  /
     \ | /
      \ |/
   Великий Новгород ← Санкт-Петербург (600 от Москвы через Великий Новгород)
   
Варианты:
- Москва → Тверь → Великий Новгород → Санкт-Петербург: 200 + 300 + 600 = 1100 км
- Москва → Рязань → Воронеж: ... (другой путь)
```

**Решение:** Алгоритм Дейкстры — находит кратчайший путь в графе с весами.

---

## Отличие от BFS

```
BFS:     Ищет путь с МИНИМУМОМ РЁБЕР (все рёбра = 1)
Дейкстра: Ищет путь с МИНИМУМОМ СУММЫ ВЕСОВ (рёбра разные)

Это РАЗНЫЕ задачи!

Граф с весами:        Путь с мин. рёбрами:     Путь с мин. весом:
    A --5-- B              A → B → C              A → D → C
    |       |             (2 ребра)              (2 + 1 = 3)
    2       1
    |       |
    D --1-- C              стоимость = 2          стоимость = 3
```

---

## Идея алгоритма Дейкстры

**Принцип:** На каждом шаге выбираем вершину с **минимальным известным расстоянием**.

```
Шаг 1: Начинаем с Москвы (расстояние 0)
        Все остальные = "неизвестно" (бесконечность)

Шаг 2: Из Москвы можем попасть:
        - Тверь за 200
        - Рязань за 300
        
        Выбираем Тверь (200 < 300)

Шаг 3: От Твери можем попасть:
        - Великий Новгород за 200 + 300 = 500
        (это лучше чем "неизвестно")
        
        Выбираем Рязань (300 < 500)

Шаг 4: От Рязани можем попасть:
        - Воронеж за 300 + 400 = 700
        
        Выбираем Великий Новгород (500 < 700)

...и так далее
```

---

## Реализация на PHP

```php
<?php

/**
 * Алгоритм Дейкстры
 * Находит кратчайшие пути от одной вершины
 */
class Dijkstra
{
    /** @var array<string, array<string, int>> Граф (список смежности с весами) */
    private array $graph = [];
    
    /**
     * Добавить ребро
     */
    public function addEdge(string $from, string $to, int $weight): void
    {
        // Неориентированный граф
        $this->graph[$from][$to] = $weight;
        $this->graph[$to][$from] = $weight;
    }
    
    /**
     * Найти кратчайшие пути от source до всех вершин
     */
    public function shortestPaths(string $source): array
    {
        $distances = [];  // Расстояния
        $visited = [];    // Посещённые
        $previous = [];   // Предыдущие для восстановления пути
        
        // Инициализация
        foreach (array_keys($this->graph) as $vertex) {
            $distances[$vertex] = PHP_INT_MAX;
            $previous[$vertex] = null;
        }
        $distances[$source] = 0;
        
        // Главный цикл
        while (true) {
            // Находим непосещённую вершину с мин. расстоянием
            $minVertex = null;
            $minDist = PHP_INT_MAX;
            
            foreach ($distances as $vertex => $dist) {
                if (!$visited[$vertex] && $dist < $minDist) {
                    $minDist = $dist;
                    $minVertex = $vertex;
                }
            }
            
            // Если не осталось достижимых вершин — выходим
            if ($minVertex === null) {
                break;
            }
            
            $visited[$minVertex] = true;
            
            // Пропускаем если расстояние = бесконечность
            if ($distances[$minVertex] === PHP_INT_MAX) {
                continue;
            }
            
            // Обновляем расстояния до соседей
            foreach ($this->graph[$minVertex] ?? [] as $neighbor => $weight) {
                if ($visited[$neighbor]) {
                    continue;
                }
                
                $newDist = $distances[$minVertex] + $weight;
                
                if ($newDist < $distances[$neighbor]) {
                    $distances[$neighbor] = $newDist;
                    $previous[$neighbor] = $minVertex;
                }
            }
        }
        
        return $distances;
    }
    
    /**
     * Кратчайший путь между двумя вершинами
     */
    public function shortestPath(string $from, string $to): ?array
    {
        $distances = $this->shortestPaths($from);
        
        if ($distances[$to] === PHP_INT_MAX) {
            return null; // Путь не существует
        }
        
        // Восстанавливаем путь
        $path = [];
        $current = $to;
        
        while ($current !== null) {
            array_unshift($path, $current);
            $current = $previous[$current] ?? null;
        }
        
        return $path;
    }
}
```

---

## Пошаговая визуализация

```
Граф:

        A
       /|\
      / | \
     4  2  5
    /  / \  \
   B  C   D  E
   | /|\   |
   2 | | | 3
    \|/ \ |
     F   G

Начинаем с A, ищем кратчайшие пути до всех вершин.

Состояние: (расстояние от A, предыдущая вершина)

Шаг 1: A = 0 (источник)
        B = 4 (через A)
        C = 2 (через A)
        D = 5 (через A)
        E = бесконечность
        F = бесконечность
        G = бесконечность

Шаг 2: Выбираем C (расстояние 2 — минимум!)
        Обновляем:
        B = min(4, 2+3) = 4 (не изменилось)
        D = min(5, 2+1) = 3 (улучшилось!)
        F = min(∞, 2+2) = 4 (новый!)
        
        D = 3 (через C)
        F = 4 (через C)

Шаг 3: Выбираем D (расстояние 3 — минимум!)
        Обновляем:
        E = min(∞, 3+3) = 6 (новый!)
        G = min(∞, 3+2) = 5 (новый!)

Шаг 4: Выбираем B или F (расстояние 4 — минимум!)
        ...и так далее

Результат:
A → A: 0
A → B: 4
A → C: 2
A → D: 3
A → E: 6
A → F: 4
A → G: 5
```

---

## Пример: Навигация

```php
<?php

$dijkstra = new Dijkstra();

// Добавляем города и расстояния
$dijkstra->addEdge("Москва", "Тверь", 170);
$dijkstra->addEdge("Москва", "Рязань", 200);
$dijkstra->addEdge("Тверь", "Великий Новгород", 300);
$dijkstra->addEdge("Рязань", "Воронеж", 400);
$dijkstra->addEdge("Великий Новгород", "Санкт-Петербург", 150);
$dijkstra->addEdge("Воронеж", "Ростов-на-Дону", 350);
$dijkstra->addEdge("Санкт-Петербург", "Хельсинки", 400);

echo "=== Кратчайшие пути от Москвы ===\n\n";

$paths = $dijkstra->shortestPaths("Москва");

foreach ($paths as $city => $distance) {
    if ($distance === PHP_INT_MAX) {
        echo "$city: недостижим\n";
    } else {
        echo "$city: $distance км\n";
    }
}

echo "\n=== Маршрут Москва → Санкт-Петербург ===\n";

$path = $dijkstra->shortestPath("Москва", "Санкт-Петербург");

if ($path) {
    echo implode(" → ", $path) . "\n";
    
    // Посчитать общее расстояние
    $total = 0;
    for ($i = 0; $i < count($path) - 1; $i++) {
        $from = $path[$i];
        $to = $path[$i + 1];
        $total += $dijkstra->getWeight($from, $to);
    }
    echo "Расстояние: $total км\n";
}
```

**Вывод:**
```
=== Кратчайшие пути от Москвы ===

Тверь: 170 км
Рязань: 200 км
Великий Новгород: 470 км
Воронеж: 600 км
Санкт-Петербург: 620 км
Ростов-на-Дону: 950 км
Хельсинки: 1020 км

=== Маршрут Москва → Санкт-Петербург ===
Москва → Тверь → Великий Новгород → Санкт-Петербург
Расстояние: 620 км
```

---

## Добавление метода getWeight

```php
/**
 * Получить вес ребра (для диагностики)
 */
public function getWeight(string $from, string $to): int
{
    return $this->graph[$from][$to] ?? PHP_INT_MAX;
}
```

---

## Сравнение: BFS vs Дейкстра

```
┌─────────────────┬─────────────────────┬─────────────────────┐
│                 │ BFS                 │ Дейкстра            │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Когда использовать│ Все рёбра = 1       │ Рёбра имеют веса    │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Сложность        │ O(V + E)            │ O(V²) простая       │
│                 │                     │ реализация           │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Результат        │ Мин. количество     │ Мин. сумма весов    │
│                 │ рёбер               │                     │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Пример           │ Кратчайший путь     │ Кратчайший маршрут  │
│                 │ в метро (все станции│ по дорогам (разные  │
│                 │ равны)              │ расстояния)         │
└─────────────────┴─────────────────────┴─────────────────────┘
```

---

## Важное ограничение

**Алгоритм Дейкстры работает ТОЛЬКО с положительными весами!**

```
❌ Если есть отрицательные веса — Дейкстра МОЖЕТ давать неправильный ответ

✅ Если могут быть отрицательные веса — нужен алгоритм Беллмана-Форда
   (он сложнее, но проверяет отрицательные циклы)
```

---

## Упражнения

### Легко
Создай граф городов и найди кратчайший путь между двумя.

### Средне
Модифицируй алгоритм, чтобы он запоминал не только расстояние, но и время в пути (если средняя скорость 60 км/ч).

### Сложно
Реализуй алгоритм Беллмана-Форда, который работает с отрицательными весами.

---

## Решения

### Легко
```php
<?php

$dijkstra = new Dijkstra();
$dijkstra->addEdge("A", "B", 4);
$dijkstra->addEdge("A", "C", 2);
$dijkstra->addEdge("B", "D", 2);
$dijkstra->addEdge("C", "D", 5);
$dijkstra->addEdge("D", "E", 1);

$path = $dijkstra->shortestPath("A", "E");
echo implode(" → ", $path); // A → B → D → E
```

### Средне
```php
<?php

class DijkstraWithTime
{
    private array $graph = [];  // [from][to] = distance
    
    public function addRoad(string $from, string $to, int $distance): void
    {
        $this->graph[$from][$to] = $distance;
        $this->graph[$to][$from] = $distance;
    }
    
    /**
     * Кратчайший путь с учётом времени (60 км/ч)
     */
    public function fastestPath(string $from, string $to): array
    {
        $distances = [];  // расстояния
        $times = [];      // время в часах
        $visited = [];
        
        foreach (array_keys($this->graph) as $v) {
            $distances[$v] = PHP_INT_MAX;
            $times[$v] = PHP_INT_MAX;
        }
        $distances[$from] = 0;
        $times[$from] = 0;
        
        while (true) {
            $minVertex = null;
            $minDist = PHP_INT_MAX;
            
            foreach ($distances as $v => $d) {
                if (!$visited[$v] && $d < $minDist) {
                    $minDist = $d;
                    $minVertex = $v;
                }
            }
            
            if ($minVertex === null) break;
            $visited[$minVertex] = true;
            
            foreach ($this->graph[$minVertex] ?? [] as $neighbor => $dist) {
                if ($visited[$neighbor]) continue;
                
                $newDist = $distances[$minVertex] + $dist;
                
                if ($newDist < $distances[$neighbor]) {
                    $distances[$neighbor] = $newDist;
                    $times[$neighbor] = $newDist / 60; // 60 км/ч
                }
            }
        }
        
        return [
            'distance' => $distances[$to],
            'time_hours' => round($times[$to], 2)
        ];
    }
}

$nav = new DijkstraWithTime();
$nav->addRoad("Москва", "Тверь", 170);
$nav->addRoad("Тверь", "Санкт-Петербург", 470);

$result = $nav->fastestPath("Москва", "Санкт-Петербург");
echo "Расстояние: {$result['distance']} км\n";
echo "Время в пути: {$result['time_hours']} ч\n";
```

### Сложно (Беллман-Форд)
```php
<?php

/**
 * Алгоритм Беллмана-Форда
 * Работает с отрицательными весами
 */
class BellmanFord
{
    private array $edges = [];  // Список всех рёбер
    private array $vertices = [];
    
    public function addEdge(string $from, string $to, int $weight): void
    {
        $this->edges[] = ['from' => $from, 'to' => $to, 'weight' => $weight];
        $this->vertices[$from] = true;
        $this->vertices[$to] = true;
        
        // Для неориентированного
        $this->edges[] = ['from' => $to, 'to' => $from, 'weight' => $weight];
    }
    
    public function shortestPaths(string $source): array
    {
        $distances = [];
        $vertices = array_keys($this->vertices);
        
        foreach ($vertices as $v) {
            $distances[$v] = PHP_INT_MAX;
        }
        $distances[$source] = 0;
        
        // Релаксация всех рёбер V-1 раз
        $vCount = count($vertices);
        for ($i = 0; $i < $vCount - 1; $i++) {
            foreach ($this->edges as $edge) {
                $from = $edge['from'];
                $to = $edge['to'];
                $weight = $edge['weight'];
                
                if ($distances[$from] !== PHP_INT_MAX
                    && $distances[$from] + $weight < $distances[$to]) {
                    $distances[$to] = $distances[$from] + $weight;
                }
            }
        }
        
        return $distances;
    }
}
```

---

## Итоги

- **Дейкстра** находит кратчайший путь с учётом весов
- Работает только с **положительными** весами
- Сложность: **O(V²)** для простой реализации (можно улучшить с приоритетной очередью)
- Основа навигации, сетевой маршрутизации, игр

**Следующий урок:** узнаем про динамическое программирование — мощный метод решения сложных задач.
