# Урок 7: Деревья и Обходы (Trees and Traversals)

## Зачем этот урок

**Проблема:** Данные часто имеют иерархию (вложенность).

```
Пример: Папки на компьютере

Диск C:/
├── Программы/
│   ├── VSCode/
│   └── Git/
├── Документы/
│   ├── Работа/
│   │   └── Отчёт.pdf
│   └── Личное/
│       └── Фото.jpg
└── Музыка/
```

Как хранить такое? **Дерево** — идеально!

---

## Что такое Дерево?

**Дерево** — это структура данных, где каждый элемент (узел) может иметь несколько "детей", но только одного "родителя".

**Простая аналогия: Родословное древо**

```
                    Прадедушка (корень)
                    /         \
              Дедушка       Бабушка
              /     \        /     \
          Дядя   Папа    Тётя    Мама
                  |
                Я
```

**Термины:**
- **Корень (root)** — самый верхний узел (нет родителя)
- **Ребро (edge)** — связь между узлами
- **Лист (leaf)** — узел без детей
- **Высота (height)** — максимальная глубина
- **Глубина (depth)** — расстояние от корня

---

## Простое дерево на PHP

```php
<?php

/**
 * Узел дерева
 */
class TreeNode
{
    public string $value;
    public array $children = [];  // Список детей
    
    public function __construct(string $value)
    {
        $this->value = $value;
    }
    
    /**
     * Добавить ребёнка
     */
    public function addChild(TreeNode $child): void
    {
        $this->children[] = $child;
    }
}

/**
 * Дерево
 */
class Tree
{
    public ?TreeNode $root = null;
    
    public function __construct()
    {
        $this->root = null;
    }
}
```

---

## Пример: Структура папок

```php
<?php

// Создаём дерево папок
$tree = new Tree();

// Корень
$root = new TreeNode("Диск C:");
$tree->root = $root;

// Первый уровень
$programs = new TreeNode("Программы");
$documents = new TreeNode("Документы");
$music = new TreeNode("Музыка");

$root->addChild($programs);
$root->addChild($documents);
$root->addChild($music);

// Второй уровень
$vscode = new TreeNode("VSCode");
$git = new TreeNode("Git");
$programs->addChild($vscode);
$programs->addChild($git);

$work = new TreeNode("Работа");
$personal = new TreeNode("Личное");
$documents->addChild($work);
$documents->addChild($personal);

// Третий уровень
$report = new TreeNode("Отчёт.pdf");
$photo = new TreeNode("Фото.jpg");
$work->addChild($report);
$personal->addChild($photo);

// Визуализация
function printTree(TreeNode $node, string $prefix = ""): void
{
    echo $prefix . "📁 " . $node->value . "\n";
    foreach ($node->children as $child) {
        printTree($child, $prefix . "    ");
    }
}

printTree($tree->root);
```

**Вывод:**
```
📁 Диск C:
    📁 Программы
        📁 VSCode
        📁 Git
    📁 Документы
        📁 Работа
            📁 Отчёт.pdf
        📁 Личное
            📁 Фото.jpg
    📁 Музыка
```

---

## Два способа обхода деревьев

### 1. DFS (Depth-First Search) — поиск в глубину

**Идея:** Сначала идём вглубь (к потомкам), потом возвращаемся.

```
Порядок обхода (DFS сверху вниз):

Диск C:
    ├── Программы        ← Сначала сюда
    │   ├── VSCode
    │   └── Git
    ├── Документы
    └── Музыка

DFS: Диск C → Программы → VSCode → Git → Документы → Работа → Отчёт.pdf → Личное → Фото.jpg → Музыка
```

```php
<?php

/**
 * DFS — обход в глубину (Pre-order)
 */
function dfs(TreeNode $node): void
{
    echo $node->value . "\n";  // Посещаем узел
    
    foreach ($node->children as $child) {
        dfs($child);  // Рекурсивно идём в детей
    }
}

// Использование
dfs($tree->root);
```

### 2. BFS (Breadth-First Search) — поиск в ширину

**Идея:** Сначала обходим все узлы на одном уровне, потом переходим к следующему.

```
Порядок обхода (BFS уровень за уровнем):

Уровень 0: Диск C:
Уровень 1: Программы, Документы, Музыка
Уровень 2: VSCode, Git, Работа, Личное
Уровень 3: Отчёт.pdf, Фото.jpg

BFS: Диск C → Программы → Документы → Музыка → VSCode → Git → Работа → Личное → Отчёт.pdf → Фото.jpg
```

```php
<?php

/**
 * BFS — обход в ширину
 */
function bfs(Tree $tree): void
{
    $queue = [$tree->root];  // Используем очередь!
    
    while (!empty($queue)) {
        $node = array_shift($queue);  // Берём из начала
        echo $node->value . "\n";
        
        // Добавляем всех детей в конец
        foreach ($node->children as $child) {
            $queue[] = $child;
        }
    }
}

// Использование
bfs($tree);
```

---

## Визуализация: DFS vs BFS

```
Дерево:

        A
       /|\
      B C D
     /|   |
    E F   G

DFS (Pre-order): A → B → E → F → C → D → G
                 ↑_____________↑     ↑_____↑
                 Сначала вглубь      Потом обратно

BFS:               A
                  B C D              ← Уровень 1
                 E F   G             ← Уровень 2
                 
BFS: A → B → C → D → E → F → G
     ↑_________________________↑
     Уровень за уровнем
```

---

## Три вида DFS обхода

### Pre-order: Узел → Дети

```php
<?php

function preOrder(TreeNode $node): void
{
    if ($node === null) return;
    
    echo $node->value . " ";      // 1. Посещаем узел
    foreach ($node->children as $child) {
        preOrder($child);          // 2. Дети
    }
}
```

### Post-order: Дети → Узел

```php
<?php

function postOrder(TreeNode $node): void
{
    if ($node === null) return;
    
    foreach ($node->children as $child) {
        postOrder($child);          // 1. Сначала дети
    }
    echo $node->value . " ";      // 2. Потом узел
}
```

### In-order: Левый ребёнок → Узел → Правый ребёнок
(Только для бинарных деревьев — будет в следующем уроке)

---

## Сравнение DFS и BFS

```
┌─────────────────────┬──────────────────┬──────────────────┐
│                     │ DFS               │ BFS               │
├─────────────────────┼──────────────────┼──────────────────┤
│ Использует          │ Стек (рекурсия)   │ Очередь          │
│ Память              │ O(h) — высота     │ O(w) — ширина    │
│ Подходит для        │ Глубокие деревья  │ Поиск уровнями   │
│ Примеры             │ Пути в играх      │ Кратчайший путь  │
└─────────────────────┴──────────────────┴──────────────────┘
```

---

## Практический пример: Поиск файла

```php
<?php

/**
 * Поиск файла в дереве папок
 */
function findFile(TreeNode $folder, string $filename): ?string
{
    // Проверяем текущий узел
    if ($node->value === $filename) {
        return $filename;
    }
    
    // Ищем в детях (DFS)
    foreach ($node->children as $child) {
        $result = findFile($child, $filename);
        if ($result !== null) {
            return $result;
        }
    }
    
    return null;
}

// BFS для поиска кратчайшего пути
function findFileBFS(Tree $tree, string $filename): ?array
{
    $queue = [[
        'node' => $tree->root,
        'path' => [$tree->root->value]
    ]];
    
    while (!empty($queue)) {
        $current = array_shift($queue);
        $node = $current['node'];
        $path = $current['path'];
        
        // Проверяем
        if ($node->value === $filename) {
            return $path;
        }
        
        // Добавляем детей
        foreach ($node->children as $child) {
            $queue[] = [
                'node' => $child,
                'path' => array_merge($path, [$child->value])
            ];
        }
    }
    
    return null;
}

// Использование
$path = findFileBFS($tree, "Отчёт.pdf");
if ($path) {
    echo "Найден: " . implode(" → ", $path) . "\n";
    // Вывод: Найден: Диск C: → Документы → Работа → Отчёт.pdf
}
```

---

## Когда использовать обходы?

```
✅ DFS хорошо, когда:
   - Нужно обойти всё дерево
   - Хочешь найти любой путь (не обязательно кратчайший)
   - Дерево глубокое, но не широкое

✅ BFS хорошо, когда:
   - Нужен кратчайший путь от корня
   - Хочешь найти что-то на минимальной глубине
   - Дерево широкое, но не глубокое
```

---

## Упражнения

### Легко
Напиши функцию, которая считает количество узлов в дереве.

### Средне
Напиши функцию, которая находит максимальную глубину дерева (height).

### Сложно
Напиши функцию, которая проверяет, есть ли путь от корня до листа, сумма значений узлов которого равна заданному числу.

---

## Решения

### Легко
```php
<?php

function countNodes(?TreeNode $node): int
{
    if ($node === null) {
        return 0;
    }
    
    $count = 1;  // Текущий узел
    
    foreach ($node->children as $child) {
        $count += countNodes($child);  // Плюс дети
    }
    
    return $count;
}
```

### Средне
```php
<?php

function treeHeight(?TreeNode $node): int
{
    if ($node === null) {
        return 0;
    }
    
    $maxChildHeight = 0;
    
    foreach ($node->children as $child) {
        $childHeight = treeHeight($child);
        if ($childHeight > $maxChildHeight) {
            $maxChildHeight = $childHeight;
        }
    }
    
    return $maxChildHeight + 1;  // +1 для текущего узла
}
```

### Сложно
```php
<?php

function hasPathWithSum(TreeNode $node, int $targetSum, int $currentSum = 0): bool
{
    if ($node === null) {
        return false;
    }
    
    $newSum = $currentSum + (int)$node->value;
    
    // Если лист и сумма совпала
    if (empty($node->children) && $newSum === $targetSum) {
        return true;
    }
    
    // Проверяем детей
    foreach ($node->children as $child) {
        if (hasPathWithSum($child, $targetSum, $newSum)) {
            return true;
        }
    }
    
    return false;
}
```

---

## Итоги

- **Дерево** — структура с родителем и детьми
- **DFS** — идём вглубь, использует стек
- **BFS** — идём в ширину, использует очередь
- Оба обхода за **O(n)**, где n — количество узлов

**Следующий урок:** узнаем про бинарное дерево поиска (BST) — дерево с особыми свойствами для быстрого поиска.
