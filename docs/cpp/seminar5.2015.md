# Семинар 5 (02.10.2015)

## ! Обновление (07.10.2015)

Обратите внимание на [результаты компиляции и запуска тестов](tasks-acceptance)

## Типичные ошибки

Обращайте внимание не только на ошибки, выдаваемые компилятором, но и на **предупреждения (warnings)**.
Поищите, как в вашей рабочей среде настроить компилятор так, чтобы он выдавал предупреждения.
Например, при использовании *gcc* (`g++`) под Linux или при использовании *mingw* в Windows,
вывод всех предупреждений включается опцией `-Wall`:
```
g++ -std=c++14 -Wall source1.cpp source2.cpp ...
```

Многое из того, что сообщает компилятор указывает на логические ошибки в коде.
Например:
```
В функции-члене «LinkedList::iterator& LinkedList::iterator::operator++()»:
предупреждение: присваивание, используемое как логическое выражение, рекомендуется  [-Wparentheses]
     if (this->element = NULL)
                             ^
```
Скорее всего, **вместо присваивания** (`=`) в этой строке подразумевалось **сравнение** (`==`).
В таком виде блок кода идущий после `if (...)` никогда не будет выполнен.

Ещё пример:
```
В функции-члене «LinkedList::iterator LinkedList::iterator::operator++(int)»:
предупреждение: control reaches end of non-void function [-Wreturn-type]
 }
 ^
```
**Нет возвращаемого значения** при выходе из не-void функции,
что будет делать программа при вызове этого метода скорее всего неопределено.

**Не используйте функцию `memcpy`** для копирования объектов, для этого есть оператор присваивания (`operator=`)
и возможность его перегрузить.

При **копировании итератора** (с использованием констуктора `iterator(const iterator &)` или оператора присваивания
`iterator & operator=(const iterator&)`) не нужно копировать узел (_Node_), на который указывает итератор.
Например, если `iterator` содержит поля:
```
class iterator {
	Node * node;
}
```
то при копировании итератора нужно просто скопировать указатель:
```
iterator(const iterator & other): node(other.node) {
}

iterator & operator=(const iterator & other) {
	this->node = other.node;
}
```
Кстати, в точности то же самое сделает `default`:
```
iterator(const iterator & other): node(other.node) = default;
iterator & operator=(const iterator & other) = default;
```
Объяснение достаточно простое: итератор не владеет "узлом" с данными, на который он указывает.
Данными владеет класс `LinkedList`.
По этой же причине при деструктор итератора (`~iterator()`) не должен удалять "узел".

**Операторы `+` и `+=`**. Оператор `+` не изменяет содержимое ни одного из двух аргументов
и возвращает новый объект. Пример для чисел:
```
int x = 10;
int y = 15;
int z = x + y;
// сожержимое x и y не изменилось. z содержит новое значение.
```
Оператор `+=` изменяет содержимое первого аргумента, но не трогает второе:
```
int x = 10;
int y = 15
x += y;
// x содержит новое значение, значение y не изменилось.
```

Теперь к классам. Обратите внимание на сигнатуру операторов:
```
	List operator+(const List & other) const;
	List& operator+=(const List & other);
```
`operator+` возвращает `List`, то есть новый объект, `operator+=` возвращает ссылку `List&`.
Семантически это "ссылка на самого себя".
Что это значит? Это значит, то можно написать следующий код:
```
string x = "one";
(x += "two") += "three";
cout << x << endl;
// резутьтат: onetwothree
```
Если бы `operator+=` возвращал новый объект (`List`) вместо ссылки (`List&`),
то при вызове `+= "three"` менялось содержимое не `x`, а этого нового объекта и результат был бы `onetwo`.

Можно определить оператор `+` через `+=` и конструктор копирования:
```
List& operator+=(const List & other) {
	for (List::iterator it = other.begin(); it != other.end(); ++it) {
		this->push_back(*it);
	}
	return *this;
}

List operator+(const List & other) const {
	List result(*this); // создаём копию "этого" списка
	result += other; // добавляем к копии "другой" список
	return result; // возвращаем результат
}
```

**Сравнение итераторов.** В коде часто встречается конструкция `it != list.end()`.
Как правильно сравнить два итератора?
Во-первых, нужно помнить, что итераторы можно копировать.
Во-вторых, что значит "два итератора равны"?
В случае со списком это означает, что два итератора указывают на один и тот-же узел.
Как понять, что итераторы указывают на один и тот же узел?
Использовать "хитрый хак" - сравнить два указателя :)
```
bool operator==(const iterator & other) const {
	return this->node == other.node;
}
bool operator!=(const iterator & other) const {
	return !((*this) == other);
}
```

Как узнать, что `iterator++` дошел до конца списка?
Если внутри итератора хранить длину списка и позицию, как в классе [Str](seminar2/),
но для связного списка это не будет работать.
Например:
```C++
struct iterator {
	Node * node;
	int len;
	int pos;
};

int main() {
	List l{1, 2};
	List::iterator it = l.begin();
	l.push_back(3);
	cout << *it << endl; // 1
	++it;
	cout << *it << endl; // 2
	++it;
	cout << *it << endl; // Должно быть 3, но it не знает, что длина списка изменилась
}
```
Можно сказать "но я же могу использовать ссылку на длину, следующим образом:"
```C++
class List {
	// ...
	int len
	// ...
};

struct iterator {
	Node * node;
	int & len; // len указывает на List::len
	int pos;
};
```
но тогда я предложу следующий вариант, который всё испортит:
```C++
int main() {
	List l{1, 2};
	List::iterator it = l.begin();
	l.push_front(0);
	l.pop_back();
	// Длина списка не изменилась, но изменилась относительная позиция
}
```

Вообще, самым лучшим решением будет не хранить длину списка (?!).
Как понять, что итератор дошёл до конца списка?
Очень просто: если у текущего узла нет следующего, значит это конец:

```C++
struct iterator {
	Node * node;
};

iterator & operator++() {
	if (this->node->next) {
		this->node = this->node->next;
	}
	return *this;
}
```

**Что возвращает метод end()?**
Метод `end()` возвращает итератор, указывающий на фиктивный элемент (узел), являющийся _первый после последнего реального элемента в списке_.
Покажу на примере. Допустим, список содержит элементы 1, 2 и 3.
Тогда список будет выглядеть примерно следующим образом:
```
        begin()                            end()
           |                                |
           V                                V
        iterator                          iterator
           |                                |
           V                                V
NULL  <-  node  <->  node  <->  node  <->  node -> NULL
           |          |          |
           V          V          V
           1          2          3
```

В самом классе `List` достаточно иметь два поля: `Node * _begin` и `Node * _end`,
при этом `_end` всегда будет указывать на узел с "первым после последнего элементом",
а `_begin` - на узел с первым элементом.

**Как тогда должен выглядеть пустой список?**
`_end` должен указывать на "фиктивный" узел, а `_begin` должен указывать на `_end`:
```C++
class List {
	// ...
private:
	Node * _begin;
	Node * _end;
};

List::List() {
	_end = new Node();
	_begin = _end;
}
```

Не забывайте при добавлении и удалении узлов **перестраивать указатели соседних узлов**.

**Нельзя возвращать ссылки** (точно так же, как и указатели) **на локальные переменные**
из методов, функций.
Например, следующий код будет неправильным:
```
int & getReference() {
	int x = 13;
	return x;
}
```
Компилятор будет (должен) выдавать предупреждение для такого кода:
```
В функции-члене «int& getReference()»:
предупреждение: возвращена ссылка на локальную переменную «x» [-Wreturn-local-addr]
   int x = 13;
       ^
```

В следующем коде та же самая ошибка:
```
class Node {
	// ...
public:
	value_type & getValue();
	// ...
};

class List {
	// ...
public:
	value_type & front() {
		value_type result = this->_begin->getValue(); // создаётся локальная переменная
		return result; // ошибка: возвращается ссылка на локальную переменную
	}
private:
	Node * _begin;
};
```
Использовать _статические переменные_ для решения этой проблемы **не нужно**.
В данном случае можно написать просто:
```
	value_type & front() {
		return this->_begin->getValue();
	}
```
Или, если нужно использовать локальную переменную, то объявить её как ссылку:
```
	value_type & front() {
		value_type & result = this->_begin->getValue();
		return result;
	}
```

**Из константного метода нельзя вызвать неконстантный метод.**
Также нельзя вызвать неконстантный метод _поля_ класса.
Объяснение простое: const-методы не должны изменять состояние объекта.
Поэтому следующий код работать не будет:
```
class Node {
	// ...
public:
	value_type & getValue();
	// ...
};

class List {
	// ...
public:
	const value_type & front() const {
		return this->_begin->getVanue(); // ошибка: вывзов non-const метода из const
	}
private:
	Node * _begin;
};
```
В таком случае можно объявить два метода `getValue` - const и не-const:
```
class Node {
	// ...
public:
	value_type & getValue();
	const value_type & getValue() const;
	// ...
};
```
Тогда из const-метода `front` будет вызвать const-метод `getValue`.
Этот подход используют коллекции из стандартной библиотеки, например метод
[operator\[\]](http://www.cplusplus.com/reference/vector/vector/operator%5B%5D/) класса _std::vector_.
В случае, если от метода `getValue` требуется только доступ "на чтение", можно оставить только const-версию метода.

**Обн. 07.10.2015**
Важно, чтобы **код соответствовал спецификации**.
В частности, _публичные_ методы должны называться так, как указано в задании
и иметь точно такую же сигнатуру (если метод обозначен как `const`, то `const` убирать нельзя).
Самый простой способ этого добиться - полностью скопировать объявление класса `LinkedList`
из текста задания в заголовочный файл.
Константные итераторы традиционно называются `const_iterator`.

## Пожелания

**Структуры с стиле C**. Вместо
```C++
typedef struct _MyStruct {
	int some_field;
} MyStruct;
```
можно писать просто
```C++
struct MyStruct {
	int some_field;
};
```
Поскольку ключевое слово `struct` так же, как слово `class`, определяет класс, но с доступом `public` к членам класса по-умолчанию, последняя запись аналогична:
```C++
class MyStruct {
public:
	int some_field;
};
```

**Имена файлов**.
Старайтесь придерживаться следующей схемы разбиения кода на модули (файлы).
Каждому классу с именем `SomeClass` должна соответствовать пара файлов
`SomeClass.h` и `SomeClass.cpp`.
Вложенные классы (например `SomeClass::iterator`) можно определять в тех же файлах.
функцию `main()` лучше всего располагать в отдельном файле `main.cpp`.
Тесты располагаются в отдельном файле (файлах), например: `SomeClass_test.cpp` - функции для тестирования класса `SomeClass`.

Таким образом, код первой задачи можно разбить на 4 файла:
`LinkedList.h`, `LinkedList.cpp`, `LinkedList_test.cpp` и `main.cpp`.

_Продолжение следует_