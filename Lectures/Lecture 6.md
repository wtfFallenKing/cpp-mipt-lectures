# Указатели

## Содержание
- Взятие адреса, разыменование, определение указателя
- Операции над указателями
	- Целочисленная арифметика
	- Присваивание
	- Вычитание указателей одного типа
	- Сравнение указателей одного типа
- Указатели на указатели
- Некоторые утверждения про указатели, взятие адреса и разыменование
- Висячие указатели и прочие забавные UB
- void*
- Нулевой указатель
- Зачем это вообще нужно?
	- Пример

## Взятие адреса, разыменование, определение указателя

Для начала максимально упростим устройство хранения переменных в памяти: будем считать, что они хранятся в оперативной памяти, выделенной под программу. Значит, можем спросить номер ячейки, в которой лежит та или иная переменная. Для этого в С++ существует унарный оператор `&` (унарный амперсанд, взятие адреса, address), который по переменной возвращает ее адрес в памяти - число специального типа.

> [!NOTE]
>Разумеется, при разных запусках программы адрес одной и той же переменной может меняться.

Возникает вопрос: какой тип имеет результат операции взятия адреса от, например, переменной типа `T`. Ответ - `T*` (для дальнейших рассуждений запишем это в математической нотации - `& : T --> T*`).

>[!IMPORTANT]
>Переменная, хранящая адрес ячейки в памяти (то есть типа `T*`), называется указателем (pointer) на тип `T`. 

Существует и обратная операция к взятию адреса - унарный оператор `*` (унарная звездочка, разыменование, dereference). Грубо говоря, у нас есть адрес ячейки - хотим узнать, что под ним лежит. Таким образом, `* : T* --> T`.

>[!NOTE]
>Рассмотрим код
>```cpp
>int* p = &x;
>std::cout << *p << '\n';
>```
>`*` при `int` - часть названия типа. `*` в выводе - унарный оператор (с `&` будет такая же ситуация, но об этом позже).

>[!NOTE]
>`*` в названии типа привязывается к переменной, а не к строке объявления, то есть при объявлении `int* a, b, *c;` переменные `a` и `c` имеют тип `int*`, а `b` - `int`. Во избежании путаницы в этой и последующих лекциях используется codestyle, запрещающий объявление нескольких переменных в одной строке.

## Операции над указателями

### Целочисленная арифметика

Указатели можно складывать с целыми числами - `+ : (T*, int) --> T*`. При этом `+(p, n) = p + sizeof(p) * n`. Проще говоря, операция `p + n` над переменными `T* p` и `int n` равносильна сдвигу адреса, лежащего в `p`, на `n` шагов длины `sizeof(T)`. Аналогично определяются вычитание из указателя целого числа (шагаем в другую сторону), префиксные и постфиксные инкремент и декремент (указатель сдвинется так, как будто прибавили/вычли 1).

### Присваивание

Указателям можно присваивать адреса переменных, также существуют присваивающие арифметические операции:
```cpp
vector<int> v = {1, 2};
int* px;
int* py = &v[1];
px = &v[0];
px += 1; // px == py
px -= 1; // px == &v[0]
```

### Вычитание указателей одного типа

Указатели одного типа можно вычитать. При этом результат такой операции есть разность адресов, деленная на размер типа (грубо говоря, количество шагов длины `sizeof(T)`, которое необходимо пройти, чтобы добраться от вычитаемого до уменьшаемого.
```cpp
vector<int> v = {1, 2};
int* px = &v[0];
int* py = &v[1];
std::cout << px - py << '\n'; // -1
```

### Сравнение указателей одного типа

Для указателей одного и того же типа определены операции логического сравнения `>, >=, <, <=, ==, !=` (сравниваются как обычные числа)

## Указатели на указатели

Указатель - переменная, хранящаяся в памяти. Следовательно, можем применить к ней операцию взятия адреса. Так получаем типы `T**`, `T***`, ...
```cpp
int a = 0;
int* p = &a;
std::cout << p << '\n'; // некоторое 16-ричное число
int** pp = &p;
std::cout << pp << '\n'; // тоже 16-ричное число
std::cout << *pp << '\n'; // равносильно std::cout << p << '\n';
std::cout << *p << ' ' << **pp << '\n'; // равносильно std::cout << a << ' ' << a << '\n';
```

## Некоторые утверждения про указатели, взятие адреса и разыменование

Указатель, как правило, занимает 8 байт (вообще говоря, зависит от разрядности системы)

Для указателей операции взятия адреса и разыменования коммутируют: 
```cpp
int a;
int* p = &a;
std::cout << (*&p == &*p) << '\n'; // 1
```

Результат операции разыменования указателя - `lvalue`. Следовательно, конструкции типа `*p = 1;` имеют смысл. С другой стороны, результат операции взятия адреса - `rvalue`, поэтому конструкции типа `&a = 1;` приведет к `CE`

Операцию взятия адреса можно применять только к `lvalue`

## Висячие указатели и прочие забавные UB

>[!IMPORTANT]
>Висячим указателем называется указатель, который указывает на объект, время жизни которого закончилось

Чтение из висячего указателя и запись в него приводят к `UB`

```cpp
int x = 0;
int y = 1;
int* p = &x;
*++p; // UB, но, скорее всего, вернется 1
```

Можно насильно (сишными кастами) поменять тип указателя или привести число к указателю:
```cpp
int x = 1092847;
int* p = &x;
std::cout << (long long)p << '\n'; // не UB, но так делать не стоит
std::cout << (double*)x << '\n'; // не UB, но так делать тоже не стоит
std::cout << *(double*)x << '\n'; // UB, скорее всего, SegFault
```

## void*

Бывают ситуации, когда хотим сохранить адрес ячейки без указания типа переменной. В этом случае можно использовать `void*`. Для него не определены прибавление и вычитание целого числа, разыменование.

## Нулевой указатель

Чтобы задать начальное значение "пустого" указателя в `C++` существует специальное значение `nullptr` - нулевой указатель.

>[!NOTE]
>Вообще говоря, `nullptr` имеет тип `std::nullptr_t`, но он всегда неявно кастуется к любому конкретному типу указателя.

## Зачем это вообще нужно?

Исторически указатели нужны для того, чтобы можно было из функции менять переменные, которые в нее передали

### Пример

Хотим написать функцию, меняющую местами 2 переменные. Наивный (ничего не делающий) способ:
```cpp
void swap(int x, int y) {
	int t = x;
	x = y;
	y = t;
}
...
swap(x, y);
...
```
Он не сработает, так как при передаче аргументов в функцию создается их *локальная* копия.

Решение проблемы крайне простое - будем передавать адреса наших переменных:
```cpp
void swap(int* x, int* y) {
	int t = *x;
	*x = *y;
	*y = t;
}
...
swap(&x, &y);
...
```