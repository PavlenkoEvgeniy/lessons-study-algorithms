# Урок 10: Графы (Graphs) — Основы

## Зачем этот урок

**Проблемы из реальной жизни:**
- Найти кратчайший маршрут между городами
- Социальная сеть: "друзья друзей"
- Зависимости между задачами (что сделать сначала)

**Решение:** Графы! Универсальный способ представить связи.

---

## Что такое Граф?

**Граф** — это набор **вершин** (точек) и **рёбер** (линий), соединяющих эти точки.

```
Простая аналогия: Карта дорог

Вершины (города):     A, B, C, D, E
Рёбра (дороги):       A-B, A-C, B-D, C-D, D-E

    A --- B
    |     |
    C --- D --- E
```

---

## Термины

```
┌─────────────────┬─────────────────────────────────────────────┐
│ Термин          │ Значение                                    │
├─────────────────┼─────────────────────────────────────────────┤
│ Вершина (node)  │ Точка в графе                               │
│ Ребро (edge)    │ Связь между вершинами                       │
│ Путь (path)     │ Последовательность вершин                    │
│ Цикл (cycle)    │ Путь, начинающийся и заканчивающийся       │
│                 │ в одной вершине                             │
└─────────────────┴─────────────────────────────────────────────┘
```

---

## Виды графов

```
1. Неориентированный        2. Ориентированный (directed)
   (дорога двусторонняя)       (дорога с односторонним движением)
   
    A --- B                    A ──→ B
    |     |                    │     │
    C ─── D                    ↓     ↓
                               C ←── D

3. Взвешенный                4. Дерево (особый случай)
   (у дорог есть длина)          (нет циклов, связный)
   
    A --5-- B                   A
    |       |                  / \
    2       1                 B   C
    |       |                  \ /
    C --3-- D                   D---E
```

---

## Способы представления

### 1. Список смежности (Adjacency List)

**Каждая вершина хранит список своих соседей.**

```php
<?php

/**
 * Граф на списках смежности
 */
class Graph
{
    /** @var array<string, array<string>> Соседи каждой вершины */
    private array $adjacencyList = [];
    
    /**
     * Добавить вершину
     */
    public function addVertex(string $vertex): void
    {
        if (!isset($this->adjacencyList[$vertex])) {
            $this->adjacencyList[$vertex] = [];
        }
    }
    
    /**
     * Добавить ребро
     */
    public function addEdge(string $v1, string $v2, bool $directed = false): void
    {
        // Создаём вершины если их нет
        $this->addVertex($v1);
        $this->addVertex($v2);
        
        // Добавляем рёбра
        $this->adjacencyList[$v1][] = $v2;
        
        if (!$directed) {
            // Для неориентированного графа — обратное ребро
            $this->adjacencyList[$v2][] = $v1;
        }
    }
    
    /**
     * Получить соседей вершины
     */
    public function getNeighbors(string $vertex): array
    {
        return $this->adjacencyList[$vertex] ?? [];
    }
}
```

**Визуально:**
```
Граф:       A --- B
            |     |
            C --- D

Список смежности:

A: [B, C]
B: [A, D]
C: [A, D]
D: [B, C]
```

---

## Обход графов: BFS и DFS

### BFS — Поиск в ширину (Breadth-First Search)

**Идея:** Сначала посещаем всех соседей, потом соседей соседей.

**Аналогия:** Распространение слуха — сначала все узнают от источника, потом от тех, кто уже знает.

```php
<?php

/**
 * BFS — обход в ширину
 * Используем очередь!
 */
function bfs(Graph $graph, string $start): array
{
    $visited = [];        // Посещённые
    $queue = new SplQueue(); // Очередь
    $result = [];          // Порядок обхода
    
    $queue->enqueue($start);
    $visited[$start] = true;
    
    while (!$queue->isEmpty()) {
        $vertex = $queue->dequeue();
        $result[] = $vertex;
        
        // Добавляем всех непосещённых соседей
        foreach ($graph->getNeighbors($vertex) as $neighbor) {
            if (!isset($visited[$neighbor])) {
                $visited[$neighbor] = true;
                $queue->enqueue($neighbor);
            }
        }
    }
    
    return $result;
}
```

**Визуализация:**
```
Граф:       A
           / \
          B   C
         / \   \
        D   E   F

BFS от A: A → B → C → D → E → F
          ↑_______________
          Уровень за уровнем
```

---

### DFS — Поиск в глубину (Depth-First Search)

**Идея:** Идём как можно глубже, потом возвращаемся.

**Аналогия:** Лабиринт — идём до тупика, потом назад и пробуем другой путь.

```php
<?php

/**
 * DFS — обход в глубину
 * Используем стек (или рекурсию)!
 */
function dfs(Graph $graph, string $start): array
{
    $visited = [];
    $result = [];
    
    dfsRecursive($graph, $start, $visited, $result);
    
    return $result;
}

function dfsRecursive(Graph $graph, string $vertex, array &$visited, array &$result): void
{
    $visited[$vertex] = true;
    $result[] = $vertex;
    
    foreach ($graph->getNeighbors($vertex) as $neighbor) {
        if (!isset($visited[$neighbor])) {
            dfsRecursive($graph, $neighbor, $visited, $result);
        }
    }
}
```

**Визуализация:**
```
Граф:       A
           / \
          B   C
         / \   \
        D   E   F

DFS от A: A → B → D → E → C → F
          ↑___________________
          Сначала вглубь, потом обратно
```

---

## Сравнение BFS и DFS

```
┌─────────────────┬─────────────────────┬─────────────────────┐
│                 │ BFS                 │ DFS                 │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Расшифровка     │ Breadth-First       │ Depth-First        │
│                 │ Search              │ Search             │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Использует      │ Очередь (Queue)     │ Стек (Stack) или    │
│                 │                     │ рекурсию           │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Как работает    │ Уровень за уровнем   │ Вглубь до тупика,   │
│                 │                     │ потом назад         │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Кратчайший путь │ ✅ Да (от источника) │ ❌ Не гарантирует   │
│ (невзвеш. граф)│                     │                     │
├─────────────────┼─────────────────────┼─────────────────────┤
│ Память          │ Много (очередь)     │ Меньше (глубина)   │
└─────────────────┴─────────────────────┴─────────────────────┘
```

---

## Пример: Поиск друга в социальной сети

```php
<?php

/**
 * Поиск друга в социальной сети
 */
class SocialNetwork
{
    private Graph $graph;
    
    public function __construct()
    {
        $this->graph = new Graph();
    }
    
    /**
     * Добавить связь дружбы
     */
    public function addFriendship(string $person1, string $person2): void
    {
        $this->graph->addEdge($person1, $person2);
    }
    
    /**
     * Найти друга (BFS)
     */
    public function findFriend(string $start, string $target): ?array
    {
        $visited = [];
        $queue = new SplQueue();
        $parent = []; // Для восстановления пути
        
        $queue->enqueue($start);
        $visited[$start] = true;
        
        while (!$queue->isEmpty()) {
            $current = $queue->dequeue();
            
            if ($current === $target) {
                return $this->reconstructPath($parent, $target);
            }
            
            foreach ($this->graph->getNeighbors($current) as $neighbor) {
                if (!isset($visited[$neighbor])) {
                    $visited[$neighbor] = true;
                    $parent[$neighbor] = $current;
                    $queue->enqueue($neighbor);
                }
            }
        }
        
        return null; // Не найден
    }
    
    private function reconstructPath(array $parent, string $target): array
    {
        $path = [$target];
        $current = $target;
        
        while (isset($parent[$current])) {
            $current = $parent[$current];
            array_unshift($path, $current);
        }
        
        return $path;
    }
    
    /**
     * Получить всех друзей друга (2 уровня)
     */
    public function getFriendsOfFriends(string $person): array
    {
        $visited = [];
        $queue = new SplQueue();
        $friends = [];
        
        $queue->enqueue($person);
        $visited[$person] = true;
        
        $level = 0;
        
        while (!$queue->isEmpty() && $level < 2) {
            $size = $queue->count();
            
            for ($i = 0; $i < $size; $i++) {
                $current = $queue->dequeue();
                
                foreach ($this->graph->getNeighbors($current) as $neighbor) {
                    if (!isset($visited[$neighbor])) {
                        $visited[$neighbor] = true;
                        if ($level === 1) {
                            $friends[] = $neighbor;
                        }
                        $queue->enqueue($neighbor);
                    }
                }
            }
            
            $level++;
        }
        
        return $friends;
    }
}

// Использование
$network = new SocialNetwork();

// Строим сеть друзей
$network->addFriendship("Аня", "Борис");
$network->addFriendship("Аня", "Вика");
$network->addFriendship("Борис", "Глеб");
$network->addFriendship("Борис", "Даша");
$network->addFriendship("Вика", "Егор");
$network->addFriendship("Глеб", "Женя");
$network->addFriendship("Даша", "Зина");

echo "Путь от Ани до Егора:\n";
$path = $network->findFriend("Аня", "Егор");
echo implode(" → ", $path) . "\n";
// Вывод: Аня → Вика → Егор

echo "\nДрузья друзей Ани:\n";
$friends = $network->getFriendsOfFriends("Аня");
echo implode(", ", $friends) . "\n";
// Вывод: Глеб, Даша, Егор
```

---

## Когда использовать BFS, а когда DFS?

```
✅ BFS используй, когда:
   - Нужен кратчайший путь (невзвешенный граф)
   - Хочешь найти ближайших соседей
   - Граф "широкий" но не "глубокий"

✅ DFS используй, когда:
   - Нужно проверить существование пути
   - Хочешь найти любой путь
   - Граф "глубокий" но не "широкий"
   - Нужно топологическая сортировка
```

---

## Упражнения

### Легко
Создай граф из 4 городов и найди всех соседей одного из них.

### Средне
Напиши функцию, которая проверяет, есть ли путь между двумя вершинами (DFS или BFS).

### Сложно
Напиши функцию, которая находит количество компонент связности в графе.

---

## Решения

### Легко
```php
<?php

$graph = new Graph();
$graph->addEdge("Москва", "Тверь");
$graph->addEdge("Москва", "Рязань");
$graph->addEdge("Тверь", "Великий Новгород");
$graph->addEdge("Рязань", "Воронеж");

echo "Соседи Москвы:\n";
foreach ($graph->getNeighbors("Москва") as $neighbor) {
    echo "  - $neighbor\n";
}
```

### Средне
```php
<?php

function hasPath(Graph $graph, string $start, string $target): bool
{
    $visited = [];
    $stack = [$start];
    
    while (!empty($stack)) {
        $current = array_pop($stack);
        
        if ($current === $target) {
            return true;
        }
        
        if (!isset($visited[$current])) {
            $visited[$current] = true;
            
            foreach ($graph->getNeighbors($current) as $neighbor) {
                if (!isset($visited[$neighbor])) {
                    $stack[] = $neighbor;
                }
            }
        }
    }
    
    return false;
}
```

### Сложно
```php
<?php

function countComponents(Graph $graph): int
{
    $allVertices = array_keys($graph->getNeighbors("")); // Получить все вершины
    
    // Используем BFS для поиска компоненты
    $visited = [];
    $components = 0;
    
    foreach ($allVertices as $vertex) {
        if (!isset($visited[$vertex])) {
            // Новая компонента — запускаем BFS
            $queue = new SplQueue();
            $queue->enqueue($vertex);
            $visited[$vertex] = true;
            
            while (!$queue->isEmpty()) {
                $current = $queue->dequeue();
                
                foreach ($graph->getNeighbors($current) as $neighbor) {
                    if (!isset($visited[$neighbor])) {
                        $visited[$neighbor] = true;
                        $queue->enqueue($neighbor);
                    }
                }
            }
            
            $components++;
        }
    }
    
    return $components;
}
```

---

## Итоги

- **Граф** — вершины + рёбра
- **BFS** — идёт уровень за уровнем, использует очередь
- **DFS** — идёт вглубь, использует стек/рекурсию
- BFS находит кратчайший путь, DFS просто находит путь

**Следующий урок:** узнаем про кратчайший путь в графах с весами (алгоритм Дейкстры).
