### Лекции 27, 28

### Функциональные типы данных

*C++, C#, Java:*
* Глобальные функции (статические функции-члены)
* Функции-члены (нестатические)

*C++:*
```C++
int x;
class A {
public:
	static int g;
	int i;
	int k;
	void foo();
	static void bar();
	void q();
};
...
int *pi;
pi = &x;
pi = &A::g; // указатель на статический член - просто как указатель на такую переменную
int A::g = 1;
*pi = 2;
int A::*piA;
piA = &A::i;
piA = &A::k;
A a;
// &a.i - можно
// a.*piA
piA = &A::i;
pi = &a.i;
// a.*piA ~ *pi
pvf = &A::bar;
void (A::*pf)();
pf = &A::foo;
pf = &A::q;
a.*pf();
// информацию о том, виртуальная ли функция, надо хранить с указателем
```

```C++
// find_if(it1, it2, f) - ?
class Functor {
	T _a;
public:
	bool operator()(T x) {return x == _a;}
	Functor(a) {_a = a;}
};
int arr[N];
int k;
find_if(begin(arr), end(arr), Functor(k));
// it =?= end(arr)
```

```C++
#include <functional>
// std::function<R(a1, a2, ..)>
std::function<void()> func;
func = &A::bar();
func = &A::foo();

class VoidFunc {
public:
	void operator()(){..}
};
VoidFunc f;
func = &f;
func(); // вызов
```
```C++
int add(int a, int b) {return a + b;}
inr sub(int a, int b) {return a - b;}
std::function<int(int, int)> f;
f = add;
std::function<int(int)> f1 = std::bind(f, 2, 10); // add
auto foo = std::bind(add, _1, 5); // std::function<int(int)> 
foo(5);
auto bar = std::bind(sub, _1, 8);
bar(25); // первый аргумент sub, а вторым будет 8, результат 17
bar = std::bind(sub, _2, 8);
bar(25); // -17 (25 вместо второго аргумента)
```

```C++
int b = 25;
auto f = []; // lambda - анонимный функтор
auto f = [&b](int a) {return b += a;};
f(1);// 26
f(2);// 27
...
class ...{
	auto f1 = [this](int a) {int k...}
	// глобальные переменные не входят в замыкание
	// неявные ссылки на глобальные переменные запрещены
};
```

```C++
int arr[10];
int k;
find_if(begin(arr), end(arr), [k](int x) {return x == k;});
```
*C#:*
```C#
type fii = function(i, j : integer) : integer;
```
*C#:*
```C#
// все есть класс
//System Delegate
//	MulticastDelegate
pvf cb;
class X {
	delegate void pvf();
	void foo() {..}
	static void bar() {..}
}
class Y {
	public void g() {..}
	public static void gg() {..}
}
void xxx() {
	cb = new pvf(foo);
	Y y;
	cb += new pvf(bar);
	cb += new pvf(Y.gg);
	cb += new pvf(Y.g);
	cb(); // все зарегистрированные делегаты будут вызваны
	
```
Делегаты не являются отдельным типом данных. Зато используется для оповещения о событиях (подписка: +=, отписка: -=,  рассылка - вызов).
```C#
public event pvf myEv; // на этот экзамепляр: изнутри применимы все операции, извне (из другого класса) только += и -=.
myEv = new pvf(X.foo)
myEv();

public event pvf cb {
	add{..}
	remove{..}
	// можно добавить свою семантику в терминах += и -=
	// () нельзя переопределять
}
```
**Обобщения**
*C#:*
```C#
class X<..>
delegate TR Func<T, TR> (T x);
// уже никто не пишет delegate
void MyMethod(Func<int,int, int> f) {
	int i = f(0, 1);
	...
}
EventHandler<T> // T -> EventArgs
static int add(int x, int y) {return x+y;}
Func<int, int, int> f = add;
Action<T1, T2, ..., TN>
Func<int, int> adder(int n) { return add(1, n); } // будет сообщение об ошибке, так как возвращается выражение типа int, а надо возвращать функцию от одного параметра
Func<int, int> adder(int n) {
	Func<int, int> res = int delegate(int k) { return add(k, n); }
	return res;
}
var add5 = adder(5);
int i = add5(18);
```

```C#
//лямбда выражение (не делегат, а в некотором смысле генератор экземпляров делегатского типа. непонятно, какой тип возвращается
x => x+1; // список аргументов => выражение
(x, y) => x + y
(x, y) => {return x + y;} // можно писать в блоках

Func<int, int, double> f = (x, y)=>x+y;
void M(Func<int, int> adder);
M(x=>x+1);
```
*C++:*
```C++
class Outer {
public:
	class Inner{..}
	static int i;
};
Outer::Inner in;
Outer::i = -1;
// а в Java и С# понятие статического класса появляется
```
*Java:*
```Java
import <имя_пакета>.* // внутри класс или интерфейс
System.IO.Path
static class Path
```
```Java
class WI {
	string s = "Hello";
	public Button MakeButton() {
		Button res = new Button{
			@override
			protected void OnClick() {
				caption = s;
			}
		}();
		return res;
	}
}

interface ISomething {
	void foo;
	int bar(int);
	int count;
	int key; set;
	// данных у интерфейса не может быть, это только функции
	// для события надо указать методы add и remove
	event pvf cb; add; remove;
}
```
