# Урок 4: Связные списки (Linked Lists)

## Зачем этот урок

Проблема массива: элементы должны лежать рядом в памяти.

```
Массив: [A][B][C][D][E] — все рядом, нужен непрерывный кусок памяти

А если места нет? Придётся перемещать весь массив!
```

**Связный список** решает эту проблему: элементы могут лежать где угодно, лишь бы знали, кто следующий.

---

## Что такое связный список?

**Связный список** — это набор "коробочек" (узлов), где каждая коробочка знает только о следующей.

**Простая аналогия: эстафета**

```
Бегун 1 → отдаёт палочку → Бегун 2 → отдаёт палочку → Бегун 3 → ... → Финиш
   ↑
Знает только следующего (куда бежать)
```

**Визуально:**
```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  Данные │    │  Данные │    │  Данные │    │  Данные │
│   "A"   │    │   "B"   │    │   "C"   │    │   "D"   │
├─────────┤    ├─────────┤    ├─────────┤    ├─────────┤
│След: ───┼────→│След: ───┼────→│След: ───┼────→│След: ───┼──→ null
└─────────┘    └─────────┘    └─────────┘    └─────────┘
   HEAD                                                         TAIL
```

Каждый узел содержит:
- **Данные** (значение)
- **Указатель на следующий** узел

---

## Реализация на PHP

### Шаг 1: Создаём узел

```php
<?php

/**
 * Узел связного списка
 */
class Node
{
    public mixed $value;  // Данные
    public ?Node $next;  // Ссылка на следующий (null если последний)
    
    public function __construct(mixed $value)
    {
        $this->value = $value;
        $this->next = null;
    }
}
```

### Шаг 2: Создаём список

```php
<?php

/**
 * Односвязный список
 */
class LinkedList
{
    private ?Node $head = null;  // Первый элемент
    private int $size = 0;        // Размер
    
    /**
     * Добавить в начало — O(1)
     */
    public function prepend(mixed $value): void
    {
        $newNode = new Node($value);
        $newNode->next = $this->head;  // Новый указывает на старый первый
        $this->head = $newNode;        // Голова теперь новый узел
        $this->size++;
    }
    
    /**
     * Добавить в конец — O(n)
     */
    public function append(mixed $value): void
    {
        $newNode = new Node($value);
        $this->size++;
        
        if ($this->head === null) {
            // Список пуст — новый узел и есть голова
            $this->head = $newNode;
            return;
        }
        
        // Идём до последнего узла
        $current = $this->head;
        while ($current->next !== null) {
            $current = $current->next;
        }
        
        // Последний указывает на новый
        $current->next = $newNode;
    }
    
    /**
     * Удалить первый — O(1)
     */
    public function removeFirst(): mixed
    {
        if ($this->head === null) {
            return null;
        }
        
        $value = $this->head->value;
        $this->head = $this->head->next;
        $this->size--;
        
        return $value;
    }
    
    /**
     * Поиск — O(n)
     */
    public function contains(mixed $value): bool
    {
        $current = $this->head;
        
        while ($current !== null) {
            if ($current->value === $value) {
                return true;
            }
            $current = $current->next;
        }
        
        return false;
    }
    
    /**
     * Размер — O(1)
     */
    public function size(): int
    {
        return $this->size;
    }
    
    /**
     * Преобразовать в массив — O(n)
     */
    public function toArray(): array
    {
        $result = [];
        $current = $this->head;
        
        while ($current !== null) {
            $result[] = $current->value;
            $current = $current->next;
        }
        
        return $result;
    }
}
```

### Шаг 3: Используем

```php
<?php

$list = new LinkedList();

// Добавляем элементы
$list->prepend("Первый");
$list->prepend("Второй");  // Теперь он первый
$list->append("Последний");

echo "Размер: " . $list->size() . "\n";  // 3

// Ищем
echo "Есть 'Второй': " . ($list->contains("Второй") ? "да" : "нет") . "\n";  // да

// Преобразуем в массив для просмотра
print_r($list->toArray());  // ["Второй", "Первый", "Последний"]

// Удаляем первый
$removed = $list->removeFirst();
echo "Удалили: $removed\n";  // "Второй"
```

---

## Сравнение: Массив vs Связный список

```
┌──────────────────────┬─────────────────┬─────────────────┐
│ Операция             │ Массив          │ Связный список  │
├──────────────────────┼─────────────────┼─────────────────┤
│ Доступ по индексу     │ O(1)            │ O(n)            │
│ Добавление в начало  │ O(n)           │ O(1) ✅         │
│ Добавление в конец   │ O(1)*          │ O(n)            │
│ Удаление из начала   │ O(n)           │ O(1) ✅         │
│ Удаление из конца    │ O(1)           │ O(n)            │
│ Поиск по значению    │ O(n)           │ O(n)            │
└──────────────────────┴─────────────────┴─────────────────┘
* амортизированно

Когда что выбрать:
- Массив: нужен быстрый доступ по индексу
- Связный список: часто добавляем/удаляем в начале
```

---

## Визуализация: Добавление в начало

**Было:**
```
HEAD → [A] → [B] → [C] → null
```

**Добавляем "NEW" в начало:**
```
Шаг 1: Создаём новый узел
        [NEW] → null

Шаг 2: Новый узел указывает на бывшую голову
        [NEW] → [A] → [B] → [C] → null

Шаг 3: Голова теперь NEW
        HEAD → [NEW] → [A] → [B] → [C] → null
```

**Код:**
```php
$newNode = new Node("NEW");
$newNode->next = $this->head;  // Указывает на старую голову
$this->head = $newNode;         // Новая голова
```

**Время: O(1)** — просто поменяли две ссылки!

---

## Визуализация: Добавление в конец

**Было:**
```
HEAD → [A] → [B] → [C] → null
```

**Добавляем "NEW" в конец:**
```
Шаг 1: Идём до последнего узла
        HEAD → [A] → [B] → [C] → null
                              ↑
                          Здесь мы

Шаг 2: Последний узел указывает на новый
        HEAD → [A] → [B] → [C] → [NEW] → null
```

**Код:**
```php
$current = $this->head;
while ($current->next !== null) {
    $current = $current->next;  // Идём пока есть следующий
}
$current->next = new Node("NEW");  // Последний указывает на новый
```

**Время: O(n)** — нужно пройти весь список!

---

## Практический пример: Браузерная история (назад/вперёд)

```php
<?php

/**
 * Упрощённая история браузера
 * forward и back — как кнопки в браузере
 */
class BrowserHistory
{
    private ?Node $backStack = null;  // Кнопка "назад"
    private ?Node $forwardStack = null; // Кнопка "вперёд"
    
    /**
     * Посетить страницу — добавляем в "назад", очищаем "вперёд"
     */
    public function visit(string $url): void
    {
        // Новая страница — старый "вперёд" не нужен
        $this->forwardStack = null;
        
        // Добавляем текущую в стек "назад"
        $newNode = new Node($url);
        $newNode->next = $this->backStack;
        $this->backStack = $newNode;
    }
    
    /**
     * Назад
     */
    public function back(): ?string
    {
        if ($this->backStack === null || $this->backStack->next === null) {
            return null;
        }
        
        // Берём текущую страницу
        $current = $this->backStack->value;
        
        // Перемещаем в "вперёд"
        $newForward = new Node($current);
        $newForward->next = $this->forwardStack;
        $this->forwardStack = $newForward;
        
        // Убираем из "назад"
        $this->backStack = $this->backStack->next;
        
        return $this->backStack ? $this->backStack->value : null;
    }
    
    /**
     * Вперёд
     */
    public function forward(): ?string
    {
        if ($this->forwardStack === null) {
            return null;
        }
        
        $current = $this->forwardStack->value;
        
        // Перемещаем в "назад"
        $newBack = new Node($current);
        $newBack->next = $this->backStack;
        $this->backStack = $newBack;
        
        // Убираем из "вперёд"
        $this->forwardStack = $this->forwardStack->next;
        
        return $current;
    }
}

// Использование
$browser = new BrowserHistory();
$browser->visit("google.com");
$browser->visit("github.com");
$browser->visit("stackoverflow.com");

echo $browser->back() . "\n";  // github.com
echo $browser->back() . "\n";  // google.com
echo $browser->forward() . "\n"; // github.com
```

---

## Когда использовать связный список?

```
✅ ХОРОШО:
   - Часто добавляем/удаляем в начале
   - Не нужен доступ по индексу
   - Хотим экономить память (нет заранее выделенного места)

❌ ПЛОХО:
   - Нужен быстрый доступ по индексу
   - Часто нужен поиск
   - Хочешь узнать длину быстро (O(n) для обычного списка)
```

---

## Упражнения

### Легко
Напиши функцию, которая считает количество элементов в связном списке (уже есть size, но напиши свой метод).

### Средне
Добавь в класс LinkedList метод `get(int $index)`, который возвращает элемент по индексу. Если индекс за пределами — вернуть null.

### Сложно
Напиши функцию `reverse(LinkedList $list)`, которая разворачивает связный список:

```
Было: [1] → [2] → [3] → null
Стало: [3] → [2] → [1] → null
```

---

## Решения

### Легко
```php
public function countElements(): int
{
    $count = 0;
    $current = $this->head;
    
    while ($current !== null) {
        $count++;
        $current = $current->next;
    }
    
    return $count;
}
```

### Средне
```php
public function get(int $index): mixed
{
    if ($index < 0) {
        return null;
    }
    
    $current = $this->head;
    $i = 0;
    
    while ($current !== null) {
        if ($i === $index) {
            return $current->value;
        }
        $current = $current->next;
        $i++;
    }
    
    return null;
}
```

### Сложно
```php
public function reverse(): void
{
    $prev = null;
    $current = $this->head;
    
    while ($current !== null) {
        $next = $current->next;  // Запоминаем следующий
        $current->next = $prev;  // Разворачиваем ссылку
        $prev = $current;         // prev — теперь текущий
        $current = $next;          // Идём дальше
    }
    
    $this->head = $prev;  // Последний prev — новая голова
}
```

**Визуализация:**
```
Шаг:    [1] → [2] → [3] → null
prev   cur  next
null    ↓
       [1]→ null    (развернули 1→2)

Шаг:    null ← [1]    [2] → [3] → null
       prev  cur  next
              null ← [2]   (развернули 2→1)

Шаг:    null ← [1] ← [2]    [3] → null
              prev  cur  next
                     null ← [3]  (развернули 3→2)

Результат: null ← [1] ← [2] ← [3]  ← prev
                  ↑
               это голова!
```

---

## Итоги

- **Связный список** — узлы со ссылками на следующий
- Добавление/удаление в начале — **O(1)**
- Доступ по индексу — **O(n)**
- Полезен когда часто работаем с началом списка

**Следующий урок:** узнаем про стеки и очереди — структуры на основе массивов и списков.
