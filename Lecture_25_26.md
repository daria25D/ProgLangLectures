### Лекции 25, 26

### Передача управления
**Сопрограмма**
* Для подпрограмм:
Главная программа (caller) и подпрограмма (callee). Главная программа сохраняет в стеке адрес возврата (стековый фрейм).
* В сопрограммах:
Нет главной программы, нет ключевых слов call и return, есть resume. Но это не многопоточность, хоть речь и идет о потоках управления. Round Robin. С точки зрения реализации сопрограмма должна хранить свое состояние (адрес возобновления, стек, регистры и тд).
Modula 2:
```Modula 2
PROCEDURE NEWPROCESS
	(P: PROC; N: CARDINAL; VAR C: ADDRESS); // COROUTINE; N - размер рабочей области
PROCEDURE RESUME
	(VAR C1: ADDRESS);// в С1 запоминается адрес памяти, в которой запомнено состояние сопрограммы С1, управление передается С2
```
```
// MODULE M[4];
DEFINITION MODULE M[4];
	TYPE SIGNAL
	PROCEDURE WAIT(VAR S: SIGNAL);
	PROCEDURE SEND(VAR S: SIGNAL);
	PROCEDURE INIT(VAR S: SIGNAL);
```
```
DEFINITION MODULE [4]; // типичный интерфейс
IMPLEMENTATION MODULE [4]; // модули реализации, могли быть откомпилированы после модуля определения.
```
m.mod -> m.obj
m.def -> m.sym

*C#:*
```C#
Interface IEnumerable/*<T>*/ {
	IEnumerator/*<T>*/ GetEnumerator();
}
Interface IEnumerator {
	Object/* T*/ Current; get;
	bool MoveNext();
	void Reset();
}

// foreach (T x in C)
IEnumerator i = C.GetEnumerator();
while (i.MoveNext()) {
	x = (T)/**/i.Current;
	...
}
```
```C#
class Iterator: IEnumerator<T> {
	T []arr;
	int index;
	Iterator(T []a) {
		arr = a;
		index = -1;
	}
	bool MoveNext() {
		index++;
		return index < arr.length;
	}
	void Reset() {
		index = -1;
	}
	T Current {
		get { return arr[index]; }
	}
}
class Collection: IEnumerable {
	T []arr = new T[3];
	public IEnumerator<T> GetEnumerator() {
		return new Iterator(arr);
	}
}
```
```C#
IEnumerator<T> GetEnumerator() {
	int index = 0;
	while (index < arr.length) {
		yield return arr[index];
		index++;
	}
	yield break;
}
```
*Python:*
```python
#python2
xrange(1, 1000) # генератор
range(1, 1000) # список
#python3
range(1, 100) # генератор
def fib(n):
	a = 1
	b = 1
	i = 1
	while i < n:
		yield a
		a, b = b, a+b
		i += 1

g = fib(100) # g - генератор
next(g) # прогон до следующего yield
...
# после 100 GeneratorExit exception
def gen(l):
	i = 0
	while len(l) > i:
		yield l[i]
		i += 1

def sum():
	count = 0
	while True:
		count += (yield) # yield-выражение
		print(count)
s = sum()
next(s)
s.send(5) # 5 в качестве yield-выражения, напечатано 5
s.send(8) # напечатается 13
```

### Передача параметров

1.  Переменный список параметров
	*  Параметры по умолчанию (но их все равно конечное число)
```C++
int f(int x=0, int y=1);
f();
f(1);
f(1, 2);
```

2. Позиционный способ указания параметров
```
f(,,,val1, ,, val2)
```
3.  Ключевой способ указания параметров (указание имени)
```
f(value1: val1, value2: val2)
func name_func(label:name:type, ...) -> type
func sub(offset:ofs:int) -> int
sub(offset:1)
```
*C#:*
```C#
void foo(int a, params double[] d);
void foo(int a, double[] d); // разные функции!
foo(0, 1.0, 2.0, 3.0); // соберется в массив и передастся в первую функцию
foo(0, new double[]{1.0, 2.0, 3.0}); // явное задание массива double
void foo(params Object[] o); // переменный список параметов любого типа
// Java: void foo(Object... o)
```
*Swift:*
```Swift
func foo(_ a: Int, _ d: Double...) {
    print(a, d) // 1 [2.0, 3.0, 4.0]
}

foo(1, 2, 3, 4)
```
*Java и C#:*
```Java
Object o = Integer(5); // автораспаковка
Int32 i = (Int32)o;
int ii = i;
Object o1 = 5; // автоупаковка
```

### Функциональные типы данных

```C
typedef void (*pvf)(); // pvf() - тогда должен быть указатель
// void (*pvf())() - функция, возвращающая указатель на функцию типа void
void bar();
pvf foo = bar; // *foo = bar
```
*Ada:*
```Ada
TYPE PPC = PROCEDURE(VAR: INTEGER, REAL):REAL;
```
Замыкание
```Pascal
procedure P(i:integer);
	var j, k: integer;
	procedure InnerP(l:integer);
		begin:
			..k, j, l..
		end;
	begin:
		InnerP;
	end;
```

*Python:*
```python
def inc1(): # замыкание
	n = 0
	def f():
		n += 1
		return n
	return f

def incNbyM(N, M): # замыкание
	n = N
	def f():
		n += M
		return n
	return f

def add(x, y):
	return x+y

def add1(x): # не замыкание
	return add(x, 1)

def add1_1(): # замыкание
	def adder1(x):
		return add(x, 1)
	return adder1

def add1_2():
	return LAMBDA(x): add(x, 1)

inc1 = add1_1()
inc1(5)
```
*JavaScript:*
```JavaScript
function add(x, y) {return x+ y}
function add1()
{
	return x => add(x, 1) // или вот так:
	// function(x) {return add(x, 1)} // анонимная функция
}
```
*С++:*
```C++
class X {
public:
	static void f(x);
	void g() {..}
	int j;
};
typedef void (*pvf)();
pvf foo;
foo = X::f;
foo = X::g; // ошибка, g - функция-член, для нее отдельный вид указателя
int X::*p; // указатель на член класса X типа int (должен быть static), чтобы можно было обратиться не через экземпляр класса:
/* int X::*pp = &X::j; static int j in class X, указатель не совсем указатель, а скорее смещение от начала полей класса*/
X a;
p = &X::j;
a.*p; // обращение к члену j
X *pa;
pa->*p;
```
