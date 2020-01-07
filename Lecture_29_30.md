### Лекция 29, 30

### Функции в Java 
* 1995 -- 2014: не было функций, были методы (невозможны без класса).
* 1999: в Java2 понятия внутреннего и анонимного класса (их методы уже были замыканиями). В методы внутреннего класса могли входить ссылки на внешние переменные.
```Java
class Bar {
	int _mem;
	void foo(int i) {
		int k;
		Button b = new Button {
			void OnClick() {//тут могут быть ссылки на i, k и _mem}
			// каждый объект внутреннего класса имеет ссылку на внешний класс
		}
	}
};
```
* 2005: обобщения (классы и интерфейсы)
```Java
interface ISomething {
	void foo();
	int bar(int i);
}
class CoClass implements ISomething {
	public void foo() {..}
	public int bar(int i) {..}
}

CoClass c = new CoClass();
ISomething f = c; // интерфейсы наследуемы множественным образом, так как наследуется таблица виртуальных методов
f.foo();
```
* функциональный интерфейс --- интерфейс, в котором есть только одна функция
```Java
@FunctionalInterface // необязательно, но если добавить, то компилятор будет ругаться при попытке добавить второй метод в такой интерфейс
interface IFoo {
	int bar(int i);
}
// @override - функция переопределяется (должна замещать, полностью совпадать по прототипу)
IFoo i = (int x)->{return x+1;};
System.out.println(i.bar(5)); // вывод - 6
```
```Java
interface Function<T, R> {
	R apply(T x);
}
interface BiFunction<T1, T2, R> {
	R apply(T1 x, T2 y);
}

interface UnaryOperator<T> {
	T apply(T x);
}
interface BinaryOperator<T> {
	T apply(Tx, Ty);
}
// в Java интерфейсы не перегружаются!
interface Predicate<T> {
	bool test(T x);
}

interface Consumer<T> {
	void accept(T x);
}
interface Supplier<T> {
	T get();
}

interface Compare<T> {
	bool compare(T x, Ty);
}
```
* default-methods
```Java
interface Function<T, R> {
	R apply(T x);
	default
	T compose(T x);
	default
	T andThen(R x);
}
Function<T, R> f = (T x)->{return r;// r типа R};
Function<Integer, String> f1 = new Function<Integer, String>() {
	String apply(Integer i) {
		String.valueOf(i);
	}
}; // хоть и интерфейс, но тут будет анонимный внутренний класс
// можно записать и в виде лямбды, будет короче
Function<Integer, String> f2 = (Integer i)->{return String.valueOf(i);};
// можно и методы
Function<Integer, String> f3 = String::valueOf;
f3.apply(2);
```
#### Java.  Ссылки на  методы
```Java
// статические
Function<T, R> f1 = Class::method; // method должен удовлетворять сигнатуре f.apply (Function - произвольный интерфейс)
// нестатический
//Class:method
BiFunction<Class, T, R> f2 = Class::method;
Class c;
R r = f2.apply(c, t);
// ссылка первого рода (самим надо передавать this)
Function<T, R> f3 = c::method; // c - объект класса, даст ссылку на this, метод нестатический
R r = f3.apply(t);
// ссылка на конструктор
Class::new; // может удовлетворять практически любому интерфейсу. если конструктор умолчания, то удовлетворяет Supplier
```
```Java
/* ? f = ?? */
Locale lRu = f.apply("ru", "RU");
Locale lEn = f.apply("en", "UK");
// варианты f
// 1
BiFunction<String, String, Locale> f = new BiFunction<String, String, Locale>() {
	Locale apply(String lang, String country) {
		return new Locale(lang, country);
	}
}; // обертка вокруг конструктора
// 2
BiFunction<String, String, Locale> f = (lang, country)->new Locale(lang, country);
// 3
BiFunction<String, String, Locale> f = Locale::new;
```
#### Потоки в Java
```Java
// Stream API
List<int> l = new List<int>();
l.stream().max((i, j)-> i > j);// max - терминальный метод, как и min(), sum()
// еще у потоков есть методы average(), count() и другие
// sum и average - варианты метода reduce
l.stream().filter(x->x>0).sorted().map(System::println); // filter оставит только числа >0, sorted - терминальный метод, сортировка, map распечатает с помощью println
// иначе нужно было бы делать
for (int x: l) {/*filter*/} // получится какая-то последовательность l1
for (int x: l1) {/*sort*/}
for (int x: l1) {/*print*/}

// тип данных optional
Optional<T> // либо пусто, либо какое-то значение
Optional<Integer>
// вместо Stream можно сделать parallelStream, тогда операции будут применяться параллельно
//.reduce(initVal, BinaryOperator<T> f)
l.stream().reduce(0, (x, y)-> x + y); // так можно записать sum через reduce
```
### Go
```Go
// go-рутины
go func(...) /*тип*/ {...} (/*вызов с аргументами*/);
c := make(chan int, 0)
c<-0 // ввод в канал
<-c // чтение из канала
```
```Go
// числа Фибоначчи через каналы
package main
import {"fmt"}
func fib(n int, c chan int) /*нет типа*/ {
	x, y := 0, 1 
	for i := 0; i < n; i++ {
		c<-x // вывод в канал, зависает, пока кто-то не считает (похоже на yield в python, сопрограмма)
		fmt.Println("number ")
		x, y = y, x+y
		
	}
	close(c)
}
//fib будет работать как сопрограмма за счет нулевой емкости канала
func main() {
	c := make(chan int, 0) // 0 по умолчанию, канал нулевой емкости
	go fib(10, c)
	// выполнение идет дальше асинхронно
	for i := 0; i < 10; i++ {
		x := <-c // будет ждать ввода
		fmt.Println(x)
	}
}
```

```Go
import {"fmt", "time"}
func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		  case <-tick:
			fmt.Println("tick")
		  case <-boom:
			fmt.Println("boom")
			return
		  default:
			// есть дефолт, тогда не будет зависать, если ни в один канал ничего не вывели
			fmt.Println("...")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

### Введение в обобщенное программирование

generics
C++, C#, Java поддерживают
Go не поддерживает

**Обобщенное программирование:**
Статическая параметризация типов (как правило во время трансляции)
С++ самый мощный с точки зрния generics.

1. Надежность
	Гарантируется, что у контейнера только один тип данных
2. Эффективность 
	Введение обобщенных коллекций в C# (не надо поддерживать гомоморфность коллекций), у Java такой эффективности нет

#### C++
* Определение шаблона (класса, функции)
	* Формальные параметры
* Конкретизация шаблона
	* Фактические параметры
* Специализация шаблона
	* Частичная (все равно приводит к какому-то шаблону, более специализированному)
	* Полная

STL контейнер -> итератор <- алгоритм

```C++
template<class T, int size, void (*pfn)()> class X {
	T::iterator; // компилятор ничего не может сказать о Т, кроме того, что это класс
	// iterator - имя либо статического члена (тогда все ок), может фигурировать имя класса. компилятор считает, что более частая ситуация - статический член, поэтому надо:
	typename T::iterator;
	function<void()>
};
```
```C++
std::basic_string<char> // через typedef определена как string
// конкретизация для функций определялась как перегрузка
// нельзя было написать шаблон (так как в шаблон входит возвращаемое значение)
template<typename T>
T get() {..}
exp(1.0);
exp(1.0f);
// сейчас можно явным образом указать и задать шаблон
get<x>();
exp<double(1);
```
```C++
// специализация
// полная (1):
template<class T>
class X {..};
// частичная (2):
template<class T>
class X<T*> {..};
// у функций нет частичной чпециализации (этого можно достигнуть через специализацию параметров)
X<int> x; // (1)
X<int*> x1; // (2)
X<const char*> x2; // (2)
// (1)
template<class X, class Y>
class T{..};
// (2)
template<class X>
class T<X, int> {..};// не имеет отношения к классу Т (1) выше, интерфейсы могут быть разные
//template<>
class T<int, int> {..}; // (3)
T<int, double>; // полная (1)
T<double, int>; // частичная (2)
T<int, int>; // полная (3)
```
SFINAE --- substitution failure is not an error
```C++
template<int N> 
struct Fact {
	enum { res = Fact<N-1>::res*N; };
};
template<>
struct Fact<0> {
	enum { res = 1; };
};
int main() {
	cout << Fact<10>::res; // выдаст факториал 10
}
```
```C++
template<int A[], int N>
struct Sum {
	enum { res = A[N-1] + sum<A, N-1>; };
};
// для 0 тоже надо
int a[] = {1, 3, -6, 17, ..};
constexpr
	//...
```


