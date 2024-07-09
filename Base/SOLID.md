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