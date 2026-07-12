# Урок 8: Бинарное Дерево Поиска (Binary Search Tree)

## Зачем этот урок

**Проблема:** Есть отсортированные числа. Как искать быстро?

```
Отсортированный массив: [1, 5, 10, 15, 20, 25, 30]
Искать 20?

Линейный поиск: O(n) — проверить 7 элементов
Бинарный поиск: O(log n) — проверить 3 элемента (1, 20, 30)

Но в массиве бинарный поиск требует случайного доступа по индексу...

А если данные добавляются/удаляются часто?
Массив — плохо, нужно что-то лучше!
```

**Решение:** Бинарное Дерево Поиска (BST). Бинарный поиск + гибкость дерева.

---

## Что такое BST?

**Бинарное дерево поиска** — это дерево, где:
- У каждого узла **максимум 2 ребёнка** (левый и правый)
- **Левый ребёнок** меньше родителя
- **Правый ребёнок** больше родителя

```
Пример BST:

            15
           /  \
          10   20
         / \   / \
        5  12 17 25

Правило:
- Всё левее 15: меньше 15  (5, 10, 12)
- Всё правее 15: больше 15 (20, 17, 25)
```

**Это позволяет искать как бинарный поиск, но с возможностью легко добавлять/удалять!**

---

## Узел BST

```php
<?php

/**
 * Узел бинарного дерева
 */
class BSTNode
{
    public int $value;
    public ?BSTNode $left = null;   // Меньше
    public ?BSTNode $right = null;  // Больше
    
    public function __construct(int $value)
    {
        $this->value = $value;
    }
}
```

---

## Реализация BST

```php
<?php

/**
 * Бинарное дерево поиска
 */
class BinarySearchTree
{
    private ?BSTNode $root = null;
    
    /**
     * Добавить элемент
     * Сложность: O(log n) в среднем, O(n) в худшем
     */
    public function insert(int $value): void
    {
        $newNode = new BSTNode($value);
        
        if ($this->root === null) {
            $this->root = $newNode;
            return;
        }
        
        $this->insertNode($this->root, $newNode);
    }
    
    private function insertNode(BSTNode $current, BSTNode $newNode): void
    {
        if ($newNode->value < $current->value) {
            // Идём налево
            if ($current->left === null) {
                $current->left = $newNode;
            } else {
                $this->insertNode($current->left, $newNode);
            }
        } else {
            // Идём направо
            if ($current->right === null) {
                $current->right = $newNode;
            } else {
                $this->insertNode($current->right, $newNode);
            }
        }
    }
    
    /**
     * Поиск элемента
     * Сложность: O(log n) в среднем, O(n) в худшем
     */
    public function contains(int $value): bool
    {
        return $this->searchNode($this->root, $value);
    }
    
    private function searchNode(?BSTNode $node, int $value): bool
    {
        if ($node === null) {
            return false;
        }
        
        if ($value === $node->value) {
            return true;  // Нашли!
        } elseif ($value < $node->value) {
            return $this->searchNode($node->left, $value);   // Ищем слева
        } else {
            return $this->searchNode($node->right, $value);  // Ищем справа
        }
    }
    
    /**
     * Найти минимум
     */
    public function findMin(): ?int
    {
        if ($this->root === null) {
            return null;
        }
        
        $node = $this->root;
        while ($node->left !== null) {
            $node = $node->left;
        }
        
        return $node->value;
    }
    
    /**
     * Найти максимум
     */
    public function findMax(): ?int
    {
        if ($this->root === null) {
            return null;
        }
        
        $node = $this->root;
        while ($node->right !== null) {
            $node = $node->right;
        }
        
        return $node->value;
    }
}
```

---

## Визуализация: Вставка элементов

**Добавляем: 15, 10, 20, 5, 12, 17, 25**

```
Шаг 1: insert(15)
        [15]
        
Шаг 2: insert(10) — 10 < 15, идём налево
        [15]
         /
       [10]
       
Шаг 3: insert(20) — 20 > 15, идём направо
        [15]
         / \
       [10] [20]
       
Шаг 4: insert(5) — 5 < 15 → налево → 5 < 10 → налево
        [15]
         / \
       [10] [20]
       /
     [5]
     
Шаг 5: insert(12) — 12 < 15 → налево → 12 > 10 → направо
        [15]
         / \
       [10] [20]
       / \
     [5] [12]
     
Шаг 6: insert(17) — 17 > 15 → направо → 17 < 20 → налево
        [15]
         / \
       [10] [20]
       / \   /
     [5] [12] [17]
     
Шаг 7: insert(25) — 25 > 15 → направо → 25 > 20 → направо
        [15]
         / \
       [10] [20]
       / \   / \
     [5] [12] [17] [25]
```

---

## Визуализация: Поиск

**Ищем 12 в дереве:**

```
Сравниваем 12 с 15: 12 < 15 → идём НАЛЕВО
        [15]
         / \
  ───→ [10] [20]    ← сюда
       / \
     [5] [12]
     
Сравниваем 12 с 10: 12 > 10 → идём НАПРАВО
       [10]
       / \
    [5] [12]        ← сюда! Нашли 12!

Шагов: 2 (вместо 7 в линейном поиске)
```

**Ищем 18 в дереве:**

```
       [15]
        / \
      [10] [20]
      / \   /
    [5] [12] [17]

18 > 15 → направо
18 < 20 → налево
но там только 17, 17 ≠ 18
Лист → не найдено
```

---

## Три вида обхода BST

### 1. In-order (симметричный): Левый → Узел → Правый

**Особенность:** Выводит элементы **в отсортированном порядке!**

```php
<?php

function inOrder(?BSTNode $node): void
{
    if ($node === null) return;
    
    inOrder($node->left);     // 1. Левое поддерево
    echo $node->value . " ";  // 2. Узел
    inOrder($node->right);    // 3. Правое поддерево
}
```

```
Дерево:         In-order: 5 10 12 15 17 20 25
        [15]          ↑
       /  \           Отсортировано!
     [10] [20]        (Лево → Узел → Право)
     / \   /
   [5] [12] [17]
```

### 2. Pre-order: Узел → Левый → Правый

```php
<?php

function preOrder(?BSTNode $node): void
{
    if ($node === null) return;
    
    echo $node->value . " ";  // 1. Узел
    preOrder($node->left);    // 2. Левое
    preOrder($node->right);   // 3. Правое
}
```

```
Pre-order: 15 10 5 12 20 17 25
```

### 3. Post-order: Левый → Правый → Узел

```php
<?php

function postOrder(?BSTNode $node): void
{
    if ($node === null) return;
    
    postOrder($node->left);   // 1. Левое
    postOrder($node->right);  // 2. Правое
    echo $node->value . " ";  // 3. Узел
}
```

```
Post-order: 5 12 10 17 25 20 15
```

---

## Пример: Словарь на BST

```php
<?php

/**
 * Словарь (список дел) на BST
 */
class TodoDictionary
{
    private BSTNode $root;
    private bool $hasRoot = false;
    
    public function add(string $task, int $priority): void
    {
        // Приоритет: меньше = важнее
        $newNode = new class($task, $priority) extends BSTNode {
            public string $task;
            public function __construct(string $task, int $priority) {
                parent::__construct($priority);
                $this->task = $task;
            }
        };
        
        if (!$this->hasRoot) {
            $this->root = $newNode;
            $this->hasRoot = true;
            return;
        }
        
        $this->insertTask($this->root, $newNode);
    }
    
    private function insertTask(BSTNode $current, $newNode): void
    {
        if ($newNode->value < $current->value) {
            if ($current->left === null) {
                $current->left = $newNode;
            } else {
                $this->insertTask($current->left, $newNode);
            }
        } else {
            if ($current->right === null) {
                $current->right = $newNode;
            } else {
                $this->insertTask($current->right, $newNode);
            }
        }
    }
    
    /**
     * Показать задачи по приоритету (in-order)
     */
    public function showAll(): void
    {
        echo "Задачи по приоритету:\n";
        $this->inOrderPrint($this->root);
    }
    
    private function inOrderPrint(?BSTNode $node): void
    {
        if ($node === null) return;
        
        $this->inOrderPrint($node->left);
        echo "  [{$node->value}] {$node->task}\n";
        $this->inOrderPrint($node->right);
    }
}

// Использование
$todos = new TodoDictionary();

$todos->add("Купить хлеб", 3);      // Не срочно
$todos->add("Позвонить маме", 1);   // Очень срочно
$todos->add("Оплатить счета", 2);    // Срочно
$todos->add("Прочитать книгу", 5);   // Совсем не срочно

$todos->showAll();
```

**Вывод:**
```
Задачи по приоритету:
  [1] Позвонить маме
  [2] Оплатить счета
  [3] Купить хлеб
  [5] Прочитать книгу
```

---

## BST vs Массив vs Связный список

```
┌──────────────────────┬─────────────────┬─────────────────┬─────────────────┐
│ Операция             │ Массив          │ Связный список  │ BST             │
├──────────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Поиск (отсортир.)    │ O(log n)        │ O(n)            │ O(log n)        │
│ Вставка             │ O(n)*           │ O(1)            │ O(log n)        │
│ Удаление            │ O(n)            │ O(n)            │ O(log n)        │
│ Min/Max             │ O(1)/O(1)       │ O(n)/O(1)       │ O(log n)/O(1)   │
│ Обход (сорт.)        │ O(n)            │ O(n log n)**    │ O(n)            │
└──────────────────────┴─────────────────┴─────────────────┴─────────────────┘
* нужен сдвиг элементов
** сначала отсортировать

BST объединяет преимущества:
- Быстрый поиск как в массиве
- Гибкие вставки/удаления как в списке
```

---

## Визуализация: Почему BST — O(log n)?

```
Если в дереве 31 узел (полное бинарное дерево):

Уровень 0:     [ ]              — 1 узел   (корень)
Уровень 1:    / \               — 2 узла
Уровень 2:   /   \              — 4 узла
Уровень 3:  /     \             — 8 узлов
Уровень 4: o       o            — 16 узлов (листья)

Всего: 1 + 2 + 4 + 8 + 16 = 31 узел
Высота: 5 уровней

Поиск: максимум 5 сравнений (log₂(31) ≈ 5)

31 элемент:
- Массив (линейный): до 31 сравнения
- BST: до 5 сравнений!
```

---

## Упражнения

### Легко
Напиши функцию, которая считает количество листьев в BST.

### Средне
Напиши функцию, которая проверяет, является ли дерево сбалансированным (разница высот левого и правого поддеревьев не более 1).

### Сложно
Напиши функцию, которая находит "второй по величине" элемент в BST.

---

## Решения

### Легко
```php
<?php

function countLeaves(?BSTNode $node): int
{
    if ($node === null) {
        return 0;
    }
    
    // Лист — это узел без детей
    if ($node->left === null && $node->right === null) {
        return 1;
    }
    
    return countLeaves($node->left) + countLeaves($node->right);
}
```

### Средне
```php
<?php

function isBalanced(?BSTNode $node): bool
{
    return checkHeight($node) !== -1;
}

function checkHeight(?BSTNode $node): int
{
    if ($node === null) {
        return 0;
    }
    
    $leftHeight = checkHeight($node->left);
    if ($leftHeight === -1) return -1;  // Левое поддерево несбалансировано
    
    $rightHeight = checkHeight($node->right);
    if ($rightHeight === -1) return -1;  // Правое поддерево несбалансировано
    
    // Проверяем баланс
    if (abs($leftHeight - $rightHeight) > 1) {
        return -1;  // Несбалансировано
    }
    
    return max($leftHeight, $rightHeight) + 1;
}
```

### Сложно
```php
<?php

function secondLargest(?BSTNode $node, int &$count): ?BSTNode
{
    if ($node === null || $count >= 2) {
        return null;
    }
    
    // Идём вправо (там большие значения)
    $result = secondLargest($node->right, $count);
    if ($result !== null) {
        return $result;
    }
    
    // Обрабатываем текущий узел
    $count++;
    if ($count === 2) {
        return $node;
    }
    
    // Идём влево
    return secondLargest($node->left, $count);
}
```

---

## Итоги

- **BST** — дерево, где левые < узел < правые
- Поиск, вставка, удаление: **O(log n)** в среднем
- In-order обход выдаёт **отсортированную последовательность**
- Полезен когда данные часто меняются и нужен быстрый поиск

**Следующий урок:** узнаем про алгоритмы сортировки — как упорядочивать данные.
