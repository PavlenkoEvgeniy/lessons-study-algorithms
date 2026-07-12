# Урок 5: Стеки и Очереди (Stacks and Queues)

## Зачем этот урок

**Стек** и **очередь** — это две простые, но очень важные структуры данных. Они используются везде: в браузере (кнопки назад/вперёд), в текстовых редакторах (отмена/повтор), в печати документов.

---

## Что такое Стек (Stack)?

**Стек** — это как стопка тарелок: кладёшь сверху, берёшь сверху.

```
Пример из жизни: стопка книг

     [Книга 5]  ← Последняя положенная (TOP)
     [Книга 4]
     [Книга 3]
     [Книга 2]
     [Книга 1]
```

**Принцип: LIFO** (Last In, First Out)
- Последний положил — первый взял

---

## Операции со стеком

```
┌──────────────┬──────────────────┬────────────────────────┐
│ Операция     │ По-английски      │ Что делает            │
├──────────────┼──────────────────┼────────────────────────┤
│ Положить     │ push             │ Добавляет сверху       │
│ Взять        │ pop              │ Забирает сверху        │
│ Посмотреть   │ peek (top)        │ Показывает верхний     │
│ Пуст?        │ isEmpty          │ Проверяет пустоту      │
└──────────────┴──────────────────┴────────────────────────┘
```

---

## Реализация стека на PHP

```php
<?php

/**
 * Стек на массиве
 */
class Stack
{
    private array $items = [];
    
    /**
     * Положить на верх — O(1)
     */
    public function push(mixed $item): void
    {
        $this->items[] = $item;
    }
    
    /**
     * Взять с верха — O(1)
     */
    public function pop(): mixed
    {
        if ($this->isEmpty()) {
            return null;
        }
        return array_pop($this->items);
    }
    
    /**
     * Посмотреть верхний — O(1)
     */
    public function peek(): mixed
    {
        if ($this->isEmpty()) {
            return null;
        }
        return $this->items[count($this->items) - 1];
    }
    
    /**
     * Проверить пустоту — O(1)
     */
    public function isEmpty(): bool
    {
        return empty($this->items);
    }
    
    /**
     * Размер — O(1)
     */
    public function size(): int
    {
        return count($this->items);
    }
}
```

---

## Визуализация: Как работает стек

```
Шаг 1: push("A")
┌───┐
│ A │ ← TOP
└───┘

Шаг 2: push("B")
┌───┐
│ B │ ← TOP
├───┤
│ A │
└───┘

Шаг 3: push("C")
┌───┐
│ C │ ← TOP
├───┤
│ B │
├───┤
│ A │
└───┘

Шаг 4: pop() → возвращает "C"
┌───┐
│ B │ ← TOP
├───┤
│ A │
└───┘

Шаг 5: peek() → возвращает "B" (не удаляя)
┌───┐
│ B │ ← TOP
├───┤
│ A │
└───┘
```

---

## Пример: Отмена действий (Ctrl+Z)

```php
<?php

/**
 * Система отмены действий
 */
class UndoManager
{
    private Stack $actions;
    
    public function __construct()
    {
        $this->actions = new Stack();
    }
    
    /**
     * Выполнить действие
     */
    public function doAction(string $action): void
    {
        echo "Выполнено: $action\n";
        $this->actions->push($action);
    }
    
    /**
     * Отменить последнее
     */
    public function undo(): void
    {
        if ($this->actions->isEmpty()) {
            echo "Нечего отменять\n";
            return;
        }
        
        $action = $this->actions->pop();
        echo "Отменено: $action\n";
    }
}

// Использование
$editor = new UndoManager();
$editor->doAction("Написал 'Привет'");
$editor->doAction("Написал 'Мир'");
$editor->doAction("Удалил 'Мир'");
$editor->undo();  // Отменяет "Удалил 'Мир'"
$editor->undo();  // Отменяет "Написал 'Мир'"
$editor->undo();  // Отменяет "Написал 'Привет'"
```

**Вывод:**
```
Выполнено: Написал 'Привет'
Выполнено: Написал 'Мир'
Выполнено: Удалил 'Мир'
Отменено: Удалил 'Мир'
Отменено: Написал 'Мир'
Отменено: Написал 'Привет'
```

---

## Что такое Очередь (Queue)?

**Очередь** — это как очередь в магазине: первый пришёл — первый обслужен.

```
Пример из жизни: очередь в кассу

Вход → [Человек 1] → [Человек 2] → [Человек 3] → Выход
       FRONT                                    REAR
       (первый)                                 (последний)
```

**Принцип: FIFO** (First In, First Out)
- Первый пришёл — первый ушёл

---

## Операции с очередью

```
┌──────────────┬──────────────────┬────────────────────────┐
│ Операция     │ По-английски      │ Что делает            │
├──────────────┼──────────────────┼────────────────────────┤
│ Встать в    │ enqueue          │ Добавляет в конец      │
│   конец      │                  │                        │
│ Выйти из    │ dequeue          │ Забирает из начала      │
│   начала     │                  │                        │
│ Посмотреть   │ front            │ Показывает первый      │
│   первый     │                  │                        │
│ Пуста?      │ isEmpty          │ Проверяет пустоту      │
└──────────────┴──────────────────┴────────────────────────┘
```

---

## Реализация очереди на PHP

```php
<?php

/**
 * Очередь на связном списке (для эффективности)
 */
class Node
{
    public mixed $value;
    public ?Node $next = null;
    
    public function __construct(mixed $value)
    {
        $this->value = $value;
    }
}

class Queue
{
    private ?Node $front = null;  // Первый в очереди
    private ?Node $rear = null;   // Последний в очереди
    private int $size = 0;
    
    /**
     * Встать в конец — O(1)
     */
    public function enqueue(mixed $item): void
    {
        $newNode = new Node($item);
        
        if ($this->rear === null) {
            // Очередь пуста — новый и есть первый и последний
            $this->front = $newNode;
            $this->rear = $newNode;
        } else {
            // Добавляем в конец
            $this->rear->next = $newNode;
            $this->rear = $newNode;
        }
        
        $this->size++;
    }
    
    /**
     * Выйти из начала — O(1)
     */
    public function dequeue(): mixed
    {
        if ($this->isEmpty()) {
            return null;
        }
        
        $value = $this->front->value;
        $this->front = $this->front->next;
        
        if ($this->front === null) {
            // Очередь стала пустой
            $this->rear = null;
        }
        
        $this->size--;
        return $value;
    }
    
    /**
     * Посмотреть первый — O(1)
     */
    public function front(): mixed
    {
        return $this->front?->value;
    }
    
    /**
     * Проверить пустоту — O(1)
     */
    public function isEmpty(): bool
    {
        return $this->front === null;
    }
    
    /**
     * Размер — O(1)
     */
    public function size(): int
    {
        return $this->size;
    }
}
```

---

## Визуализация: Как работает очередь

```
Шаг 1: enqueue("A")
FRONT → [A] ← REAR

Шаг 2: enqueue("B")
FRONT → [A] [B] ← REAR

Шаг 3: enqueue("C")
FRONT → [A] [B] [C] ← REAR

Шаг 4: dequeue() → возвращает "A"
FRONT → [B] [C] ← REAR

Шаг 5: dequeue() → возвращает "B"
FRONT → [C] ← REAR
```

---

## Пример: Очередь печати

```php
<?php

/**
 * Простая очередь печати
 */
class PrintQueue
{
    private Queue $queue;
    
    public function __construct()
    {
        $this->queue = new Queue();
    }
    
    /**
     * Добавить документ в очередь
     */
    public function submit(string $document): void
    {
        $this->queue->enqueue($document);
        echo "Документ '$document' добавлен в очередь\n";
    }
    
    /**
     * Напечатать следующий документ
     */
    public function printNext(): void
    {
        if ($this->queue->isEmpty()) {
            echo "Очередь печати пуста\n";
            return;
        }
        
        $document = $this->queue->dequeue();
        echo "Печатаем: $document\n";
    }
    
    /**
     * Посмотреть что будет печататься
     */
    public function peek(): void
    {
        if ($this->queue->isEmpty()) {
            echo "Очередь печати пуста\n";
        } else {
            echo "Следующий: " . $this->queue->front() . "\n";
        }
    }
}

// Использование
$printer = new PrintQueue();
$printer->submit("Отчёт.docx");
$printer->submit("Презентация.pptx");
$printer->submit("Письмо.pdf");

$printer->peek();  // Показать следующий

$printer->printNext();  // Печатаем
$printer->printNext();  // Печатаем
$printer->printNext();  // Печатаем
```

**Вывод:**
```
Документ 'Отчёт.docx' добавлен в очередь
Документ 'Презентация.pptx' добавлен в очередь
Документ 'Письмо.pdf' добавлен в очередь
Следующий: Отчёт.docx
Печатаем: Отчёт.docx
Печатаем: Презентация.pptx
Печатаем: Письмо.pdf
```

---

## Сравнение: Стек vs Очередь

```
┌────────────────────┬──────────────────┬──────────────────┐
│                    │ Стек (Stack)     │ Очередь (Queue)  │
├────────────────────┼──────────────────┼──────────────────┤
│ Принцип           │ LIFO             │ FIFO             │
│                   │ Последний → Первый│ Первый → Первый   │
├────────────────────┼──────────────────┼──────────────────┤
│ Аналогия из жизни │ Стопка тарелок   │ Очередь в магазин│
├────────────────────┼──────────────────┼──────────────────┤
│ Где используется  │ Отмена действий   │ Очередь задач    │
│                   │ Скобки в коде    │ Шаблоны в печати │
│                   │ Вызовы функций   │ BFS в графах     │
└────────────────────┴──────────────────┴──────────────────┘
```

---

## Когда что использовать?

```
✅ СТЕК используй, когда:
   - Нужно отменять действия (Ctrl+Z)
   - Нужно проверить парные скобки
   - Нужно идти "назад" (история браузера)

✅ ОЧЕРЕДЬ используй, когда:
   - Нужно обрабатывать в порядке поступления
   - Очередь печати, задач, запросов
   - Алгоритм поиска в ширину (BFS)
```

---

## Упражнения

### Легко
Напиши функцию, которая проверяет, сбалансированы ли скобки в строке:
```php
isBalanced("({[]})");  // true
isBalanced("({[}]");   // false
```
*Подсказка: используй стек!*

### Средне
Реализуй очередь с приоритетом, где можно добавлять элементы с приоритетом, а извлекаться будет всегда элемент с наибольшим приоритетом.

### Сложно
Напиши функцию, которая определяет, является ли одна строка анаграммой другой (используй стек или очередь).

---

## Решения

### Легко
```php
function isBalanced(string $s): bool
{
    $stack = [];
    $open = "({[";
    $close = ")}]";
    $pairs = [")" => "(", "}" => "{", "]" => "["];
    
    for ($i = 0; $i < strlen($s); $i++) {
        $char = $s[$i];
        
        if (strpos($open, $char) !== false) {
            // Открывающая — кладём в стек
            $stack[] = $char;
        } elseif (strpos($close, $char) !== false) {
            // Закрывающая — проверяем пару
            if (empty($stack) || end($stack) !== $pairs[$char]) {
                return false;
            }
            array_pop($stack);
        }
    }
    
    return empty($stack);
}
```

### Средне (простая версия на массиве)
```php
<?php

class PriorityQueue
{
    private array $items = [];
    
    public function enqueue(mixed $item, int $priority): void
    {
        $this->items[] = ["item" => $item, "priority" => $priority];
        // Сортируем по приоритету (больший приоритет первый)
        usort($this->items, fn($a, $b) => $b["priority"] <=> $a["priority"]);
    }
    
    public function dequeue(): mixed
    {
        return array_shift($this->items)["item"] ?? null;
    }
    
    public function isEmpty(): bool
    {
        return empty($this->items);
    }
}

$pq = new PriorityQueue();
$pq->enqueue("Важно!", 3);
$pq->enqueue("Срочно!", 5);
$pq->enqueue("Обычно", 1);

echo $pq->dequeue(); // "Срочно!"
echo $pq->dequeue(); // "Важно!"
```

### Сложно
```php
function isAnagram(string $s1, string $s2): bool
{
    if (strlen($s1) !== strlen($s2)) {
        return false;
    }
    
    $s1Queue = new Queue();
    $s2Queue = new Queue();
    
    // Кладём все буквы в очереди
    for ($i = 0; $i < strlen($s1); $i++) {
        $s1Queue->enqueue($s1[$i]);
        $s2Queue->enqueue($s2[$i]);
    }
    
    // Сравниваем: перемещаем из одной очереди в другую и обратно
    for ($i = 0; $i < strlen($s1); $i++) {
        $char = $s1Queue->dequeue();
        
        // Ищем эту букву во второй очереди
        $found = false;
        $temp = new Queue();
        
        while (!$s2Queue->isEmpty()) {
            $c = $s2Queue->dequeue();
            if ($c === $char && !$found) {
                $found = true; // Нашли пару
            } else {
                $temp->enqueue($c); // Сохраняем для возврата
            }
        }
        
        // Возвращаем обратно
        while (!$temp->isEmpty()) {
            $s2Queue->enqueue($temp->dequeue());
        }
        
        // Ставим обработанную букву обратно
        $s1Queue->enqueue($char);
        
        if (!$found) {
            return false;
        }
    }
    
    return true;
}
```

---

## Итоги

- **Стек (LIFO):** последний пришёл — первый ушёл. Кнопка "назад".
- **Очередь (FIFO):** первый пришёл — первый ушёл. Очередь в магазин.
- Обе структуры поддерживают основные операции за **O(1)**

**Следующий урок:** узнаем про хеш-таблицы — самую быструю структуру для поиска.
