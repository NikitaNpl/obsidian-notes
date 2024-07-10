# SOLID 2024-07-09

_Теги_: #ПринципыРазработки #Разработка #Масштабируемость #ПО
## Для чего?

При написании кода, программисту необходимо прибегать к определенным правилам написания кода, чтобы код программы выглядел более лаконично, его было проще масштабировать и поддерживать.

Принципы **S.O.L.I.D** — это пять правил объектно-ориентированного проектирования классов, которые задают вышеописанную траекторию. Однако, помимо этого, он также может отлично ложиться и под [языки/фреймворки](SOLID%20%D0%B2%20REACT.md) с функциональной парадигмой программирования.
## Single Responsibility Principle (S) — принцип единой ответственности

Класс должен иметь только одну зону ответственности, поэтому у него должна быть только одна причина для изменения.

Более технически: принцип означает, что только изменение спецификации программного обеспечения (структура базы данных) может повлиять на изменение спецификации класса.
### Пример принципа единой ответственности 

В качестве примера разберем программу, которая выставляет счета за определенные книги.

```java
class Book {
	String name;
	String authorName;
	int year;
	int price;
	String isbn;

	public Book(String name, String authorName, int year, int price, String isbn) {
		this.name = name;
		this.authorName = authorName;
		this.year = year;
        this.price = price;
		this.isbn = isbn;
	}
}
```

В данном классе намерено обошлись без приватных полей, поскольку не хотели реализовывать getter-ы и setter-ы, что позволит сфокусироваться на дальнейшей логике.

Теперь реализуем класс `Invoice`, который содержит логику по созданию счета и вычислению итоговой стоимости.

```java
public class Invoice {
	private Book book;
	private int quantity;
	private double discountRate;
	private double taxRate;
	private double total;

	public Invoice(Book book, int quantity, double discountRate, double taxRate) {
		this.book = book;
		this.quantity = quantity;
		this.discountRate = discountRate;
		this.taxRate = taxRate;
		this.total = this.calculateTotal();
	}

	public double calculateTotal() {
	        double price = ((book.price - book.price * discountRate) * this.quantity);
		double priceWithTaxes = price * (1 + taxRate);
		return priceWithTaxes;
	}

	public void printInvoice() {
            System.out.println(quantity + "x " + book.name + " " +          book.price + "$");
            System.out.println("Discount Rate: " + discountRate);
            System.out.println("Tax Rate: " + taxRate);
            System.out.println("Total: " + total);
	}

        public void saveToFile(String filename) {
	// Creates a file with given name and writes the invoice
	}
}
```

В нашем классе содержится три метода:

- `calculateTotal` — метод, вычисляющий итоговую стоимость;
- `printInvoice` — метод, который отображает в консоль результат выставленного счета;
- `saveToFile` — метод, отвечающий за запись в файл результата выставленного счета.

*Основной вопрос: что плохого здесь происходит?*

Принцип единой ответственности гласит, что наш класс должен иметь только одну причину для изменения, и для рассматриваемого класса эта причина должна заключаться в изменении вычисления счета. Из этого следуют, что следующие методы нарушают принцип единой ответственности: 

- `printInvoice` — содержит в себе логику отображения результата, формат отображения может поменяться, поэтому мы не хотим смешивать логику отображения с бизнес логикой в одном классе;
- `saveToFile` — реализация метода может заключаться, как в использовании какой-либо БД, в вызове API или другие вещи, связанные с персистентными данными, поэтому связывание персистентной логикой с бизнес логикой также является распространенной ошибкой.

Для того, чтобы исправить найденные ошибки, достаточно создать два класса `InvoicePrinter` и `InvoicePersistence` и перенести в них методы.

```java
public class InvoicePrinter {
    private Invoice invoice;

    public InvoicePrinter(Invoice invoice) {
        this.invoice = invoice;
    }

    public void print() {
        System.out.println(invoice.quantity + "x " + invoice.book.name + " " + invoice.book.price + " $");
        System.out.println("Discount Rate: " + invoice.discountRate);
        System.out.println("Tax Rate: " + invoice.taxRate);
        System.out.println("Total: " + invoice.total + " $");
    }
}
```

```java
public class InvoicePersistence {
    Invoice invoice;

    public InvoicePersistence(Invoice invoice) {
        this.invoice = invoice;
    }

    public void saveToFile(String filename) {
        // Creates a file with given name and writes the invoice
    }
}
```

Теперь наш структура класса сохраняет рассматриваемый принцип единой ответственности. 

## Open-Closed Principle (O) — принцип открытости-закрытости

Класс должен быть закрыт для изменения, но открыт для расширения. Когда меняется текущее поведение класса, все такие изменения сказываются на всех системах, работающих с данным классом.

Обычно добавление новой функциональности без изменения уже существующего класса достигается путем интерфейсов и абстрактных классов.
### Пример принципа открытости-закрытости

Предположим, нам была задача, чтобы все счета могли сохранятся и в базе данных, и в виде файлов. Просто решение выглядит следующим образом:

```java
public class InvoicePersistence {
    Invoice invoice;

    public InvoicePersistence(Invoice invoice) {
        this.invoice = invoice;
    }

    public void saveToFile(String filename) {
        // Creates a file with given name and writes the invoice
    }

    public void saveToDatabase() {
        // Saves the invoice to database
    }
}
```

Данный подход некорректный, поскольку при добавлении нового подхода к сохранению информации о счете, нам бы пришлось изменять уже существующий класс, что противоречит принципу открытости-закрытости.

Попробуем решить выявленную проблему через интерфейсы.

```java
interface InvoicePersistence {

    public void save(Invoice invoice);
}
```

```java
public class DatabasePersistence implements InvoicePersistence {

    @Override
    public void save(Invoice invoice) {
        // Save to DB
    }
}
```

```java
public class FilePersistence implements InvoicePersistence {

    @Override
    public void save(Invoice invoice) {
        // Save to file
    }
}
```

Теперь, если нам необходимо добавить новый способ сохранения счета, нам достаточно создать новый класс с имплементацией нашего общего интерфейса.

## Liskov Substitution Principle (L) — принцип подстановки Барбары Лисков

Если в коде заменить *Базовый класс* на его *Наследника*, то программа должна работать по-прежнему корректно, поскольку в *Наследнике* есть все операции, которые были в *Базовом классе*. В *Базовый класс* нужно выносить только общую логику, которую наследники будут реализовывать. 

В случаях, когда *Наследник* не способен выполнить те же действия, что и *Базовый класс*, возникает риск появления ошибок. Если же *Наследник* не удовлетворяет этим требованиям, значит, он сильно отличается от *Базового класса* и нарушает данный принцип.
### Пример принципа подстановки Барбары Лисков

Допустим, у нас есть класс `Invoice`, в котором есть три метода на просмотр остатка на счете, пополнение счета и оплата.

```java
public class Invoice {
	public BigDecimal balance(String numberAccount) {
		// logic
		return bigDecimal;    
	}   
	
	public void refill(String numberAccount, BigDecimal sum) {
		//logic    
	}
	
	public void payment(String numberAccount, BigDecimal sum) {
	//logic    
	}
}
```

По задаче, нам необходимо написать еще два класса: зарплатный счет и депозитный счет, при этом, зараплатный счет должен поддерживать все операции, представленные в базовом классе, а депозитный счет не должен поддерживать проведение оплаты.

```java
public class SalaryInvoice extends Invoice {
	@Override
	public BigDecimal balance(String numberAccount) {
		//logic
		return bigDecimal;
	}
	
	@Override
	public void refill(String numberAccount, BigDecimal sum) {
		//logic
	}    
	
	@Override
	public void payment(String numberAccount, BigDecimal sum) {
		//logic
	}
}
```

```java
public class DepositInvoice extends Invoice {
	@Override
	public BigDecimal balance(String numberAccount) {
		//logic
		return bigDecimal;
	}
	
	@Override
	public void refill(String numberAccount, BigDecimal sum) {
		//logic
	}    
	
	@Override
	public void payment(String numberAccount, BigDecimal sum) {
		throw new UnsopportedOperationException("Operation not supported");
	}
}
```

Если же после этого в коде программы везде, где использовался класс `Invoice` заменить на его *Наследника* `SalaryInvoice`, то программа продолжит работать нормально, поскольку в классе `SalaryInvoice` доступны все операции, что и в *Базовом классе*.

Если же попробуем сделать такое же с классом `DepositInvoice`, то программа начнет в некоторых моментах работать неправильно, поскольку при вызове метода `payment()` будет выбрасываться исключение. Таким образом, мы нарушили принцип подстановки Барбары Лисков.

Для того, чтобы следовать принципу подстановки Барбыры Лисков, необходимо в *Базовом классе* выносить только общую логику, характерную для классов *Наследников*, которые будут ее реализовывать.

```java
public class Invoice {
	public BigDecimal balance(String numberAccount) {
		// logic
		return bigDecimal;    
	}   
	
	public void refill(String numberAccount, BigDecimal sum) {
		//logic    
	}
}
```

```java
public class DepositInvoice extends Invoice {
	@Override
	public BigDecimal balance(String numberAccount) {
		//logic
		return bigDecimal;
	}
	
	@Override
	public void refill(String numberAccount, BigDecimal sum) {
		//logic
	}    
}
```

```java
public class PaymentInvoice extends Invoice {
	public void payment(String numberAccount, BigDecimal sum) {
		//logic
	}
}
```

```java
public class SalaryInvoice extends PaymentInvoice {
	@Override
	public BigDecimal balance(String numberAccount) {
		//logic
		return bigDecimal;
	}
	
	@Override
	public void refill(String numberAccount, BigDecimal sum) {
		//logic
	}    
	
	@Override
	public void payment(String numberAccount, BigDecimal sum) {
		//logic
	}
}
```

В заключении, принцип подстановки Барбары Лисков заключается в правильном использовании наследования.

## Interface Segregation Principle (I) — принцип разделения интерфейса

Клиенты не должны зависеть от интерфейсов, которые они используют. Большие интерфейсы следует разбивать на интерфейсы поменьше. Так, клиенты смогут использовать только те интерфейсы, которым им необходимы. Это делает менее связанный код, уменьшает зависимости между элементами системы и упрощает изменения в коде.
### Пример принципа разделения интерфейса

```java
interface Robot {
	void move();
	void speak();
	void fly();
}
```

В данном примере, интерфейс написан не совсем удачно. Например, роботы-дроны могут использовать метод `fly()` и не использовать методы `move()` и `speak()`, но при этом он все равно должен полностью реализовать интерфейс, который включает эти методы. Таким образом произойдет нарушение принципа разделения интерфейсов.

Чтобы исправить выявленную проблему, необходимо разделить наш исходный интерфейс на несколько малых.

```java
interface Movable { void move(); }
interface Speakable { void speak(); }
interface Flyable { void fly(); }

class Robot implements Movable, Speakable {
	@Override
	public void move() { 
		/*Сложная логика движения*/ 
	}    
	
	@Override
	public void speak() {
		/*Сложная логика произнесения фразы*/ 
	}
}

class Drone implements Flyable {
	@Override
	public void fly() {
		/*Сложная логика полета*/
	}
}
```

## Dependency Inversion Principle (D) — принцип инверсии зависимостей

Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модуля должны зависеть от абстракции. Абстракции в свою очередь не должны зависеть от реализации. Реализация должна зависеть от абстракции.
### Пример принципа инверсии зависимостей

Допустим, у нас имеется страница магазина и мы должны решить вопросы с проведением оплаты. Поначалу это просто небольшой магазин, где оплата происходит только за наличные. Создадим классы по описанной логике.

```java
public class Cash {
	public void doTransaction(BigDecimal amount) {
		//logic
	}
}
```

```java
public class Shop {
	private Cash cash;
	
	public Shop(Cash cash) {
		this.cash = cash;
	}
	
	public void doPayment(Object order, BigDecimal amount) {       
		cash.doTransaction(amount);
	}
}
```

Казалось бы выглядит хорошо, но в данном примере мы уже нарушили принцип инверсии зависимостей, поскольку тесно связали оплату наличными к нашему магазину. В случае расширения и добавления нового способа оплаты, нам придется переписывать и изменять достаточно много кода. В представленном коде, модуль верхнего уровня `Cash` тесно связан с модулем нижнего уровня `Shop`, а нужно чтобы оба уровня зависили от абстракции.

Попытаемся исправить найденную проблему через имплементацию интерфейса.

```java
public interface Payments {
	void doTransaction(BigDecimal amount);
}

public class Cach implements Payments {
	@Override
	public void doTransaction(BigDecimal amount) {
		// logic
	}
}

public class BankCard implements Payments {
	@Override
	public void doTransaction(BigDecimal amount) {
		// logic
	}
}
```

Теперь, надо немного переписать реализацию класса `Shop`.

```java
public class Shop {
	private Payments payments;
	
	public Shop(Payments payments) {
		this.payments = payments;
	}
	
	public void doPayment(Object order, BigDecimal amount) {       
		payments.doTransaction(amount);
	}
}
```

В текущем решении магазин слабо связан с системой оплаты и зависит от абстракции, и нам уже особо не важно, каким способом оплаты будут пользоваться.