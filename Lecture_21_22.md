###Лекции 21, 22

### Объединение типов
(Pascal --- вариантные записи)
Некоторый вариант структуры:
* Размеченные (дискриминированные)
* Неразмеченные
 
Дискриминант: 
* Code: Bar Code --- 4 int (NS, MC, PC, C), or QR Code --- string
* Event: Event Type --- Mouse click, Keyboard event...

Если в размеченном объединении убрать дискриминант, оно станет неразмеченным (так реализовано в С --- union, все элементы начинаются с одного и того же адреса). Каждый из этих типов начинается со своего short-поля (которое по факту можно считать дискриминантом объединения).
```C
struct sockaddr {
	short family; // указатель на реальный адрес приводить к этому типу
};
int bind(int sd,  struct sockaddr *pAddr);

/* Структура sockaddr_in описывает сокет для работы с протоколами IP. 
Значение поля sin_family всегда равно AF_INET. */
struct sockaddr_in {
	short sin_family;
	struct addr_in;
};
struct sockaddr_un {
	short sun_family; /* AF_UNIX */
	char path[108]; /* Pathname */
};
```
Описываем варианты дискриминанта с помощью конечного числа констант (дискриминант --- перечислимый тип).

> Самое страшное, конечно, это сексуальное насилие, а то, что там свидания --- это ничего, ничего (о разделении общежитий гз на мужские и женские).

```C
// "две дыры" в контроле типов (в Паскале таких нет):
T *ptr = (T*)malloc(sizeof(*ptr));
free(ptr); // можно без приведения типов
```
В Паскале могут быть неразмеченные объединния (и это "дыра").
(См. еще тип данных ADDRESS, Modula2).
В Обероне объединения убрали.
"Диспетчеризация по полю типа" --- плохой стиль ООП (проблема в том, что почти  любое содержательное действие, кроме разве что присваивания, с объектами таких иерархий должно происходить с помощью оператора выбора), надо использовать интерфейс виртуальных функций. В отдельных случаях приходится прибегать к диспетчеризации, например, если есть вложенный цикл с только виртуальными функциями, это замедляет работу программы, можно использовать объединение с полем типом, производительность вырастет.
Записи с вариантами из ЯП исчезли, так как считается, что наследование типов перекрывает необходимость в дискриминантах (и вообще это ненадежно). Например, в Ada при размещении записи в памяти, необходимо было размещать там и дискриминант (это снижало ненадежность, но не сильно). Мы могли использовать поля из другого типа объединения, и это никак не исправлялось.

Концепция записи с вариантами связана с оператором выбора. И хорошо бы привязывуать туда перечислимый тип данных.

#### Объединения в Swift

Записи появляются, если не нужно никакое наследование (нечего наследовать).
Это должно быть безопасно, нужны только размеченные объединения с перечислимым типом данных. Оказалось, что концепция перечислимого типа может покрыть эту необходимость.
```Swift
enum Planets {
	case mercury=1, // case намекает на использование оператора выбора
	case venus,
	case earth,
	...
}
let terra = Planets.earth; // (const) литералы перечисления - константы, область видимости которых - само перечисление
var p = Planets.venus; // переменная
p = .mars; // Planets можно убрать, если нет конфликтов перечислений (но . остается)
// Planets становится явно выводима из целочисленного тд
var pl:Planet =.venus;
switch pl {
	case .mars:  ...;
	case .venus: ...; // break не нужны
	...
	default: {...}
}
// let t = expr; - не нужно выносить и загрязнять область видимости
switch let t=expr { // компактно ссылаемся на эту метку t (и показываем, что это константа)
	case ...
}
enum ControlChars:AsciiCharacters {
	case CR = '\r',
	case LF = '\n',
	case TAB = '\t',
	...
}
```
```Swift
enum Planets:CaseIterable { // протокол
	case mercury,
	case venus,
	case earth,
	...
} // появится свойство  AllCases

for (p in Planets.AllCases) {
	print(p); // распечатать коллекцию всех имен
}
```
```Swift
enum Planets:String { // элементы станут строковыми константами
	case mercury=1,
	case venus,
	case earth="earth",
	...
}
```
```Swift
enum BarCode {
	case BC(Int, Int, Int, Int), // можем ассоциировать набор значений
	case QR(String)
} // запись с вариантами
var B:BarCode = .BC; // ОШИБКА!!! Надо задать ассоциированные значения
var B:BarCode = .BC(5, 56538, 36211, 3);
switch B {
	case .BC (let NS, let MC, let PC, let Check) : { // значения типизируются как ассоциированные поля, локализованы внутри этого оператора
		print(NS); // идентификаторы из другой ветки нельзя использовать
		print(MC);
		print(PC);
		print(Check);
	}
	// или
	// case .BC let(NS, MC, PC, Check): {...}
	case .QR let(S):
		print(S);	
}
```
> Как компилятор будет хранить --- не твое собачье дело, программист, ты там программируй.

#### Рекурсивные перечисления
Внутри ссылается само на себя.
*Swift:*
Дерево арифметических выражений 
```Swift
enum ArExpr {
	case Add(ArExpr, ArExpr), 
	case Mult(ArExpr, ArExpr),
	case Val(Int) // здесь рекурсия кончается
}
let two:ArExpr = .Val(2), five:ArExpr = .Val(5), three:ArExpr =.Val(3);
let t1:ArExpr = .Add(three, five);
let t2:ArExpr = .Mult(two, t1);
...
// переключатель, функция evaluate(e:ArExpr)
{
	switch e {
		case .Val let(v): 
			return v;
		case .Mult let(l, r):
			return evaluate(l) * evaluate(r);
		case .Add let(l, r):
			return evaluate(l) + evaluate(r);
		// default не нужен
	}
}	
```

### Объединение типов
Тип данных и его роль
```C
void process(T *p); // совместить несколько  типов данных в одном Т
```
* Необъектные ЯП: можно описать только с помощью неразмеченных объединений => диспетчеризация по полю типа.
* Объектные ЯП: наследование (но в какой-то степени тоже как диспетчеризация, см. пример)
```C++
class BarCodeBase {
public:
	BarCodeType bcType; // поле типа
	BarCodeBase(BarCodeType b) : bcType(b) {}
};
class QRCode: public BarCodeBase {
public:
	string s;
	QRCode() : BarCodeBase(QRCODE_TYPE) {}
};
class BarCode : publc BarCodeBase {
public:
	int numberSystem,
	    manufactureCode,
	    productCode,
	    check;
	BarCode(int ns, int mc, int pc, int c) 
	    : 
	    numberSystem(ns),
	    manufactureCode(mc),
	    productCode(pc),
	    check(c),
	    BarCodeBase(BARCODE_TYPE) 
	{}	
};
// диспетчеризация по полю типа, плохой стиль
void Process(BarCodeBase *pBC) {
	switch(pBC->bcType) {
	case QRCODE_TYPE: {
		QRCode *pQR = (QRCode *)pBC;
		cout << pQR->s;
		break;
	}
	case BARCODE_TYPE: {
		BarCode *pB = (BarCode *)pBC;
		cout << pB->numberSystem;
		...
		break;
	}
	}
}
// можно статически привести в нужном case 
BarCode *pB = static_cast<BarCode *>(pBC);
// для полиморфных классов есть dynamic_cast, пусть есть virtual функция
void Process(BarCodeBase *pBC) {
	BarCode *pB = dynamic_cast<BarCode*>(pBC);
	if (pB) {
		cout << pB->numberSystem;
	} else {
		QRCode *pQR = dynamic_cast<QRCode*>(pBC);
		cout << pQR->s;
	}
}
// введем virtual print() в BarCodeBase (стремиться нужно к такому коду):
void Process(BarCodeBase *pBC) {
	pBC->print(); // тогда нужно заранее определить набор базовых действий и поместить его в базовый класс. Это невозможно, например, при использовании API OS
}
```
> Я нигде никаких дефолтов, извините за клоачный язык, не ставлю.
> 
*Немного дополнений:*
Словари: Go, Swift, Python --- добавлены в базис языка.

```Swift
let s:String = "A"; // immutable
var s1:String = "B"; // mutable
```

### Операторы

Изменение функцией своих параметров --- побочный эффект.
Изменение функцией глобальных переменных --- нежелательный побочны эффект.
> "GOTO statement considered harmful"

Все арифметико-логические команды --- это выражения (expressions).

Операторы:
* перехода (команды управления последовательностью вычислений)
* пересылки
* ввода/вывода

В Fortran раньше был единственный оператор управления:
```Fortran
IF(e) M1, M2, M3 // три метки - <0, ==0, >0 (изначально для трехадресных машин)
```
И один оператор цикла с фиксированным числом повторений:
```Fortran
DO 5 I=1,3
...
5 CONTINUE
```
Оператор перехода (их было много):
```Fortran
GOTO 5
```
Основные операторы циклов:
* while B do S
*  repeat s1; ...; sN until B
* for i:= l1 to l2 do S
