# _designPattern_with_example
NOTE: this is not a bible. In this repo each design pattern has different example. so that we can better understand the design and best use cases.

# SRP

> Voilate The Rule

```java

package playground;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.PrintStream;
import java.net.URL;
import java.util.ArrayList;

class Journal {
	private ArrayList<String> entries = new ArrayList<>();
	private static int count = 0;
	
	public void addEntry(String text) {
		entries.add(" "  + (count++) + " : " + text);
	}
	
	public void removeEntry(int index) {
		entries.remove(index);
	}
	/*************************************/
	/* These methods should not belong to this class. this are persistence methods and belong to difference class
		REMEMBER: "separation of concern"
	*/
	/*************************************/
	public void saveToFile(Journal journal, String filename, boolean overwrite) 
			throws FileNotFoundException {
		try (PrintStream out = new PrintStream(filename)) {
			out.println(toString());
		}
	}
	
	public void load(String filename) {}
	public void load(URL url) {}
	/*************************************/
	@Override
	public String toString() {
		return String.join(System.lineSeparator(), entries);
	}
}

public class Main {

	public static void main(String[] args) throws Exception {
		Journal j = new Journal();
		j.addEntry("I kissed my wife.");
		j.addEntry("Me and my wife ate pizza togather");
		System.out.println(j);
	}

}

```
> Right Way

```java

package playground;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.PrintStream;
import java.util.ArrayList;

class Journal {
	private ArrayList<String> entries = new ArrayList<>();
	private static int count = 0;
	
	public void addEntry(String text) {
		entries.add(" "  + (count++) + " : " + text);
	}
	
	public void removeEntry(int index) {
		entries.remove(index);
	}
		
	@Override
	public String toString() {
		return String.join(System.lineSeparator(), entries);
	}
}


class Persistence<ANYTYPE> {
	public void saveToFile(ANYTYPE doc, String filename, boolean overwrite) 
			throws FileNotFoundException {
		if(overwrite || new File(filename).exists()) {
			try (PrintStream out = new PrintStream(filename)) {
				out.println(doc.toString());
			}
		}
	}
	
	// public ANYTYPE load(String filename) {}
	// public ANYTYPE load(URL url) {}
}

public class Main {

	public static void main(String[] args) throws Exception {
		Journal j = new Journal();
		j.addEntry("I kissed my wife.");
		j.addEntry("Me and my wife ate pizza togather");
		System.out.println(j);
		
		Persistence<Journal> p = new Persistence<Journal>();
		String filename = "c:\\temp\\journal.txt";
		p.saveToFile(j, filename, true);
		
		Runtime.getRuntime().exec("notepad.exe " + filename);
		
	}

}


```

# OCP

`` Open Close Principle With Specification Pattern which is not GoF Design Pattern. It's an enterprise pattern. OCP + Specifications``

> Voilating The Rule Code

```java

package playground;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

enum Color {
	RED,
	BLUE,
	GREEN
}

enum Size {
	SMALL,
	MEDIUM,
	LARGE,
	YUGE
}

class Product {
	public String name;
	public Color color;
	public Size size;
	
	public Product(String name, Color color, Size size) {
		super();
		this.name = name;
		this.color = color;
		this.size = size;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Color getColor() {
		return color;
	}

	public void setColor(Color color) {
		this.color = color;
	}

	public Size getSize() {
		return size;
	}

	public void setSize(Size size) {
		this.size = size;
	}
}

class ProductFilter {
	
	public Stream<Product> filterByColor(List<Product> products, Color color) {
		return products.stream().filter(p -> p.color == color);
	}
	
	public Stream<Product> filterBySize(List<Product> products, Size size) {
		return products.stream().filter(p -> p.size == size);
	}
	
	public Stream<Product> filterBySizeAndColor(List<Product> products, Size size, Color color) {
		return products.stream().filter(p -> p.size == size && p.color == color);
	}
	
	// we have to modify this file again and again if any requirements comes
	// which is not recommended in OCP
}


public class Main {

	// Open Close Principle With Specification Pattern which is not
	// GoF Design Pattern. It's an enterprise pattern
	
	// OCP + Specifications
	
	public static void main(String[] args) {
		Product apple = new Product("Apple", Color.GREEN, Size.SMALL);
		Product tree = new Product("Tree", Color.GREEN, Size.SMALL);
		Product house = new Product("House", Color.BLUE, Size.LARGE);
		
		List<Product> products = Arrays.asList(apple, tree, house);
		ProductFilter pf = new ProductFilter();
		
		System.out.println("Green products (old):");
		pf.filterByColor(products, Color.GREEN)
		.forEach(p -> System.out.println(
				" - " + p.name + " is green"));

	}

}

```

> The Right Way

```java

package playground;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

enum Color {
	RED,
	BLUE,
	GREEN
}

enum Size {
	SMALL,
	MEDIUM,
	LARGE,
	YUGE
}

class Product {
	public String name;
	public Color color;
	public Size size;
	
	public Product(String name, Color color, Size size) {
		super();
		this.name = name;
		this.color = color;
		this.size = size;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Color getColor() {
		return color;
	}

	public void setColor(Color color) {
		this.color = color;
	}

	public Size getSize() {
		return size;
	}

	public void setSize(Size size) {
		this.size = size;
	}
}

/* we will create two new interfaces */
	
interface Specification<T> {
	boolean isSatisfied(T item);
}

interface Filter<T> {
	Stream<T> filter(List<T> items, Specification<T> spec);
}

class ColorSpecification implements Specification<Product> {

	private Color color;
	
	public ColorSpecification(Color color) {
		this.color = color;
	}
	
	@Override
	public boolean isSatisfied(Product item) {
		return item.color == color;
	}
	
}

class SizeSpecification implements Specification<Product> {

	private Size size;
	
	public SizeSpecification(Size size) {
		this.size = size;
	}
	
	@Override
	public boolean isSatisfied(Product item) {
		return item.size == size;
	}
	
}

class BetterFilter implements Filter<Product> {

	@Override
	public Stream<Product> filter(List<Product> items, Specification<Product> spec) {
		return items.stream().filter(p -> spec.isSatisfied(p));
	}
	
}

class AndSpecification<T> implements Specification<T> {

	private Specification<T> left;
	private Specification<T> right;
		
	public AndSpecification(Specification<T> left, Specification<T> right) {
		super();
		this.left = left;
		this.right = right;
	}

	@Override
	public boolean isSatisfied(T item) {
		return left.isSatisfied(item) && right.isSatisfied(item);
	}
	
}

/* ************** */

public class Main {

	// Open Close Principle With Specification Pattern which is not
	// GoF Design Pattern. It's an enterprise pattern
	
	// OCP + Specifications
	
	public static void main(String[] args) {
		Product apple = new Product("Apple", Color.GREEN, Size.SMALL);
		Product tree = new Product("Tree", Color.GREEN, Size.SMALL);
		Product house = new Product("House", Color.BLUE, Size.LARGE);
		
		List<Product> products = Arrays.asList(apple, tree, house);

		System.out.println("Green products (new):");
		BetterFilter bf = new BetterFilter();
		bf.filter(products, new ColorSpecification(Color.GREEN)).forEach(p -> 
				System.out.println(" - " + p.name + " is green"));
		
		System.out.println("Large blue items:");
		bf.filter(products, new AndSpecification<>(
				new ColorSpecification(Color.BLUE),
				new SizeSpecification(Size.LARGE)
			)).forEach(p -> 
			System.out.println(" - " + p.name + " is green"));
	}

}


```

# LSP (Liskov Substitution Principle)

```java

package playground;

class Rectangle {
	protected int width, height;

	public Rectangle() {

	}

	public Rectangle(int width, int height) {
		this.width = width;
		this.height = height;
	}

	public int getWidth() {
		return width;
	}

	public void setWidth(int width) {
		this.width = width;
	}

	public int getHeight() {
		return height;
	}

	public void setHeight(int height) {
		// here we simply set the height without modifying any other properties
		this.height = height;
	}

	public int getArea() {
		return width * height;
	}

	@Override
	public String toString() {
		return "Rectangle{" + "width=" + width + ", height=" + height + "}";
	}
}

class Square extends Rectangle {
	public Square() {
	}

	public Square(int size) {
		width = height = size;
	}

	// this below method violating THE Liskov of substitution principle. (LSP)

	@Override
	public void setWidth(int width) {
		super.setWidth(width);
		super.setHeight(width);
	}

	// this is very bed setter because it make sense for rectangle but not square
	// rectangle
	// even though square is also a rectangle.

	@Override
	public void setHeight(int height) {
		// this method implementation is wrong and violating LSP.
		super.setHeight(height);
		super.setWidth(height);
	}

	// solution:-
	public boolean isSquare() {
		return height == width;
	}
}

// this is also a solution to create A factory class
class RectangleFactory {
	public static Rectangle newRectangle(int width, int height) {
		return new Rectangle(width, height);
	}

	public static Rectangle newSquare(int side) {
		return new Rectangle(side, side);
	}
}

public class Main {

	/*
	 * DEFINITION:-
	 * 
	 * The Liskov Substitution Principle (LSP): functions that use pointers to base
	 * classes must be able to use objects of derived classes without knowing it.
	 * ... The Liskov Substitution Principle is a way of ensuring that inheritance
	 * is used correctly.
	 * 
	 */

	/*
	 * What we want to achieve
	 * 
	 * its all about making sure if you have a method such as 'useIt(Rectangle r)' which takes
	 * a Rectangle it works correctly even if your pass some Derived class of
	 * Rectangle or some inheritor of Rectangle.
	 * 
	 * So, How can you improve the situation solution would be we don't need a
	 * square class at all, but we make a detection whether a Rectangle is Square or
	 * not
	 * 
	 * goto: solution comment
	 * 
	 */

	static void useIt(Rectangle r) {
		int width = r.getWidth();
		r.setHeight(10);
		// area = width * 10;
		System.out.println("Expected area of " + (width * 10) + ", got " + r.getArea());
	}

	public static void main(String[] args) {
		Rectangle rc = new Rectangle(2, 3);
		useIt(rc);
		// Expected area of 20, got 20
		// this works fine. but wont work if we use a derived class

		// Below Code Violating The LSP. because the way we are using setter in square
		// class
		// is WRONG.
		Rectangle sq = new Square();
		sq.setWidth(5);
		useIt(sq);
		// Expected area of 50, got 100 <- unexpected dude :(
	}

}




```


## Cheat Sheet

![cheatSheet](https://i.pinimg.com/originals/02/32/6a/02326ab87918f1ac3fa8305310919176.jpg)

# strategy design pattern

``
In computer programming, the strategy pattern (also known as the policy pattern) is a behavioral software design pattern that enables selecting an algorithm at runtime. Instead of implementing a single algorithm directly, code receives run-time instructions as to which in a family of algorithms to use.
``

[Strategy Design Pattern](https://en.wikipedia.org/wiki/Strategy_pattern)

![image](https://res.cloudinary.com/dcalvdelc/image/upload/v1565777382/Strategy_Design_Pattern.png)

## Example One

> ScoreAlgoBase 

```java

public interface ScoreAlgoBase {
	// $1 Step To Make This Interface
	// each Concrete Class Has To Implement These Method's With Their Own Implementation.
	public int calculateScore(int taps, int multiplier);
	public void ShoutClassName();
	public boolean Destroy();
	public float totalScore();
	public void incressScore(int points);
}

```

> ScoreBoard

```java

public class ScoreBoard {
	// 2. This is the second Class
	// we'll create this class instance on Main Class because we can use this class member 
	// ScoreAlgoBase and call method based on the implementation.
	public ScoreAlgoBase _scoreAlgoBase;
	
	public void _showScore(int taps, int multiplier) {
		System.out.println(_scoreAlgoBase.calculateScore(taps, multiplier) + " ");
	}
	
	public void _shoutOut() {
		_scoreAlgoBase.ShoutClassName();
	}
	
	public boolean _destroy() {
		return _scoreAlgoBase.Destroy();
	}
	
	public float _totalScore() {
		return _scoreAlgoBase.totalScore();
	}
	
	public void _incressScore(int points) {
		_scoreAlgoBase.incressScore(points);
	}
	
}

```

> Balloon

```java

public class Balloon implements ScoreAlgoBase {
	// These are the main implementation of Methods. which is later call by ScoreBoard.

	public boolean isAlive = true;
	public float totalScore = 0;
	
	@Override
	public int calculateScore(int taps, int multiplier) {
		totalScore += (taps * multiplier) - 20;
		return (taps * multiplier) - 20;
	}
	
	@Override
	public void ShoutClassName() {
		// TODO Auto-generated method stub
		System.out.println(this.getClass().getSimpleName());	
	}
	
	@Override
	public boolean Destroy() {
		// TODO Auto-generated method stub
		isAlive = false;
		return true;
	}
	
	@Override
	public float totalScore() {
		// TODO Auto-generated method stub
		return this.totalScore;
	}
	
	@Override
	public void incressScore(int points) {
		// TODO Auto-generated method stub
		this.totalScore += points;	
	}
}


```

> Clown

```java

public class Clown implements ScoreAlgoBase {
	// These are the main implementation of Methods. which is later call by ScoreBoard.
	
	public boolean isAlive = true;
	public float totalScore = 0;
	
	@Override
	public int calculateScore(int taps, int multiplier) {
		totalScore += (taps * multiplier) - 10;
		return (taps * multiplier) - 10;
	}
	
	@Override
	public void ShoutClassName() {
		// TODO Auto-generated method stub
		System.out.println(this.getClass().getSimpleName());	
	}
	
	@Override
	public boolean Destroy() {
		// TODO Auto-generated method stub
		isAlive = false;
		return true;
	}
	
	@Override
	public float totalScore() {
		// TODO Auto-generated method stub
		return this.totalScore;
	}
	
	@Override
	public void incressScore(int points) {
		// TODO Auto-generated method stub
		this.totalScore += points;	
	}
}


```

> SquareBalloon

```java

public class SquareBaloon implements ScoreAlgoBase {
	// These are the main implementation of Methods. which is later call by ScoreBoard.
	
	public boolean isAlive = true;
	public float totalScore = 0;
	
	@Override
	public int calculateScore(int taps, int multiplier) {
		totalScore += (taps * multiplier) + 40;
		return (taps * multiplier) + 40;
	}
	
	@Override
	public void ShoutClassName() {
		// TODO Auto-generated method stub
		System.out.println(this.getClass().getSimpleName());	
	}
	
	@Override
	public boolean Destroy() {
		// TODO Auto-generated method stub
		isAlive = false;
		return true;
	}
	
	@Override
	public float totalScore() {
		// TODO Auto-generated method stub
		return this.totalScore;
	}
	
	@Override
	public void incressScore(int points) {
		// TODO Auto-generated method stub
		this.totalScore += points;
	}
}

```

> Main.java

```java

public class Main {

	public static void main(String[] args) {
		ScoreBoard scoreBoard = new ScoreBoard();
		
		System.out.print("Ballon Tap Score: ");
		scoreBoard._scoreAlgoBase = new Balloon(); // <- Put public abstract int calculateScore(int taps, int multiplier); of Balloon Class
		scoreBoard._showScore(10, 5);			  // Because it's extends scoreAlgoBase abstract class
		scoreBoard._destroy();
		System.out.println(scoreBoard._totalScore());
		scoreBoard._incressScore(10);
		scoreBoard._incressScore(10);
		scoreBoard._incressScore(10);
		scoreBoard._incressScore(10);
		System.out.println(scoreBoard._totalScore());
		
		System.out.print("Clown Tap Score: ");
		scoreBoard._scoreAlgoBase = new Clown();
		scoreBoard._showScore(10, 5);
		scoreBoard._destroy();
		System.out.println(scoreBoard._totalScore());
		scoreBoard._incressScore(10);
		scoreBoard._incressScore(10);
		System.out.println(scoreBoard._totalScore());
		
		System.out.print("SquareBalloon Tap Score: ");
		scoreBoard._scoreAlgoBase = new SquareBaloon();
		scoreBoard._showScore(10, 5);
		scoreBoard._destroy();
		System.out.println(scoreBoard._totalScore());
		scoreBoard._incressScore(10);
		scoreBoard._incressScore(10);
		scoreBoard._incressScore(10);
		scoreBoard._incressScore(10);
		System.out.println(scoreBoard._totalScore());
	}

}

```

## Example Two


![ex_02](https://res.cloudinary.com/dcalvdelc/image/upload/v1565802936/Strategy_Design_Pattern_2.png)

> Payment <<interface>>
	
```java

package _2019.Kind1.ex_01.controllers;

public interface Payment {
	public boolean pay(int amount);
}

```

> OnPayment <<interface>>
	
```java

package _2019.Kind1.ex_01.controllers;

public interface OnPayment<T> {
	public void onSuccess(boolean what);
	public void onError(boolean what);
}

```

> OnPaymentWithPrinter <<interface>>
	
```java

package _2019.Kind1.ex_01.controllers;

public interface OnPaymentWithPrinter<T> extends OnPayment<T> {
	public void onSuccess(Printer<T> printer);
}

```

> BillBehavior <<interface>>
	
```java

package _2019.Kind1.ex_01.controllers;

public interface BillBehavior<T> {
	public void printBill(T[] items);
}

```

> Printer.java

```java

package _2019.Kind1.ex_01.controllers;

public class Printer<T> {
	public BillBehavior<T> behavior;
	
	public void printBill(T[] billList) {
		behavior.printBill(billList);
	}
}

```

> ShoppingCart.java

```java

package _2019.Kind1.ex_01.controllers;

import java.util.ArrayList;

import _2019.Kind1.ex_01.models.Product;

public class ShoppingCart {
	ArrayList<Product> products = new ArrayList<>();
	
	public ShoppingCart() {}
	
	public ShoppingCart(Product[] products) {
		for (Product product: products) {
			this.products.add(product);
		}
	}
	
	public boolean addProduct(Product product) {
		this.products.add(product);
		return true;
	}
	
	public boolean removeProduct(Product product) {
		this.products.remove(product);
		return true;
	}
	
	public boolean clearCart() {
		this.products.clear();
		return true;
	}
	
	public boolean Pay(Payment payment, OnPaymentWithPrinter<Product> onPaymentPrinter) {
		int total = 0;
		
		for(Product product: this.products) {
			total += product.getPrice();
		}
		
		onPaymentPrinter.onSuccess(new Printer<Product>());
		// onPayment.onError(false);
		
		return payment.pay(total);
	}
}


```

> Product.java

```java

package _2019.Kind1.ex_01.models;

public class Product {
	private String name;
	private int price;
	
	public Product(String name, int price) {
		this.name = name;
		this.price = price;
	}
	
	public String getName() { return this.name; }
	public int getPrice() { return this.price; }
}

```

> Paypal.java

```java

package _2019.Kind1.ex_01.models;

import _2019.Kind1.ex_01.controllers.Payment;

public class Paypal implements Payment {
	String email;
	String password;
	
	public Paypal(String email, String password) {
		this.email = email;
		this.password = password;
	}
	
	@Override
	public boolean pay(int amount) {
		System.out.println("Email " + email + ",\nPaying " + amount + " with " + this.getClass().getSimpleName());
		return true;
	}
}

```

> CreditCard.java

```java

package _2019.Kind1.ex_01.models;

import _2019.Kind1.ex_01.controllers.Payment;

public class CreditCard implements Payment {
	String name;
	String cardNumber;
	
	public CreditCard(String name, String cardNumber) {
		this.name = name;
		this.cardNumber = cardNumber;
	}
	
	@Override
	public boolean pay(int amount) {
		System.out.println("Name " + name + ",\nPaying " + amount + " with " + this.getClass().getSimpleName());
		return true;
	}
}

```

> BillWithPrice.java

```java

package _2019.Kind1.ex_01.models;

import _2019.Kind1.ex_01.controllers.BillBehavior;

public class BillWithPrice implements BillBehavior<Product>{
	@Override
	public void printBill(Product[] products) {
		int counter = 1;
		System.out.println("===========================");
		for(Product product: products) {
			System.out.println(counter++ + ". " + product.getName() + "\t" + product.getPrice());
		}
		System.out.println("===========================");
	}
}

```

> BillWithoutPrice.java

```java

package _2019.Kind1.ex_01.models;

import _2019.Kind1.ex_01.controllers.BillBehavior;

public class BillWithoutPrice implements BillBehavior<Product> {
	@Override
	public void printBill(Product[] products) {
		int counter = 1;
		System.out.println("===========================");
		for(Product product: products) {
			System.out.println(counter++ + ". " + product.getName());
		}
		System.out.println("===========================");
	}
}

```

> Main.java


```java

package _2019.Kind1.ex_01;

import _2019.Kind1.ex_01.controllers.OnPayment;
import _2019.Kind1.ex_01.controllers.OnPaymentWithPrinter;
import _2019.Kind1.ex_01.controllers.Printer;
import _2019.Kind1.ex_01.controllers.ShoppingCart;
import _2019.Kind1.ex_01.models.BillWithPrice;
// import _2019.Kind1.ex_01.models.BillWithPrice;
import _2019.Kind1.ex_01.models.BillWithoutPrice;
import _2019.Kind1.ex_01.models.CreditCard;
import _2019.Kind1.ex_01.models.Paypal;
import _2019.Kind1.ex_01.models.Product;

public class Main {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Product[] products = {
					new Product("Shampoo", 45), 
					new Product("Lighter", 12),
					new Product("Fridge", 45499)
				};
		
		ShoppingCart shoppingCart = new ShoppingCart(products);
		shoppingCart.Pay(new CreditCard("GauravGupta", "123456789"), new OnPaymentWithPrinter<Product>() {
			
			@Override
			public void onSuccess(boolean what) {
				// TODO Auto-generated method stub
				
			}
			
			@Override
			public void onError(boolean what) {
				// TODO Auto-generated method stub
				
			}
			
			@Override
			public void onSuccess(Printer<Product> printer) {
				// TODO Auto-generated method stub
				printer.behavior = new BillWithPrice();
				printer.printBill(products);
			}
		});
		
		/*
		 * 
		 * 
			new OnPayment() {
				
				@Override
				public void onSuccess(boolean what) {
					// TODO Auto-generated method stub
					System.out.println("Payment Done. Thanks Visit Again");
					shoppingCart.clearCart();
					
					Printer<Product> printer = new Printer<>();
					// printer.behavior = new BillWithPrice();
					printer.behavior = new BillWithoutPrice();
					printer.printBill(products);
				}
				
				@Override
				public void onError(boolean what) {
					// TODO Auto-generated method stub
					System.out.println("No Error Found!");
				}
			}
		 * 
		 * 
		 * */
	}
}

```

# Observer Design Pattern

``The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods.``

![umlObserver](https://res.cloudinary.com/dcalvdelc/image/upload/v1565871481/Observer_Design_Pattern.png)

> Observer - Interface

```java

package _2019.Observer.ex_01.controllers;

public interface Observer {
	public void update();
	public void setSubject(Subject subject);
}


```

> Subject - Interface

```java
package _2019.Observer.ex_01.controllers;

public interface Subject {
	public void register(Observer observer);
	public void unregister(Observer observer);
	public void notifyObserver();
	
	public Object getUpdate(Observer observer); 
}
```

> EmailTopic.java

```java

package _2019.Observer.ex_01.models;

import java.util.ArrayList;
import java.util.List;

import _2019.Observer.ex_01.controllers.Observer;
import _2019.Observer.ex_01.controllers.Subject;

public class EmailTopic implements Subject {

	List<Observer> observers;
	String message;
	
	public EmailTopic() {
		this.observers = new ArrayList<>();
	}
	
	@Override
	public void register(Observer observer) {
		// TODO Auto-generated method stub
		if(observer == null) throw new NullPointerException("Null object/Observer");
		
		if(!observers.contains(observer))
			observers.add(observer);
		
	}

	@Override
	public void unregister(Observer observer) {
		// TODO Auto-generated method stub
		observers.remove(observer);
	}

	@Override
	public void notifyObserver() {
		// TODO Auto-generated method stub
		for(Observer observer: observers) {
			observer.update();
		}
	}

	@Override
	public Object getUpdate(Observer observer) {
		// TODO Auto-generated method stub
		return this.message;
	}
	
	public void postMessage(String msg) {
		System.out.println("Message posted to my topic : " + msg);
		this.message = msg;
		notifyObserver();
	}
}

```

> EmailTopicSubscriber.java 

```java

package _2019.Observer.ex_01.models;

import _2019.Observer.ex_01.controllers.Observer;
import _2019.Observer.ex_01.controllers.Subject;

public class EmailTopicSubscriber implements Observer {

	String name;
	
	// Reference to our Subject class
	private Subject topic;
	
	public EmailTopicSubscriber(String name) {
		this.name = name;
	}
	
	@Override
	public void update() {
		// TODO Auto-generated method stub
		String message = (String) topic.getUpdate(this);
		if(message == null) {
			System.out.println(name + " No new message on this topic");
		}else {
			System.out.println(name + " Retrieving message: " + message);
		}
	}

	@Override
	public void setSubject(Subject subject) {
		// TODO Auto-generated method stub
		this.topic = subject;
	}

}

```

> Main.java


```java

package _2019.Observer;

import _2019.Observer.ex_01.controllers.Observer;
import _2019.Observer.ex_01.models.EmailTopic;
import _2019.Observer.ex_01.models.EmailTopicSubscriber;

public class Main {

	public static void main(String[] args) {
		// System.out.println("The Observer Pattern");
		
		EmailTopic topic = new EmailTopic();
		
		// Create observer
		Observer firstObserver = new EmailTopicSubscriber("First Observer");
		Observer secondObserver = new EmailTopicSubscriber("Second Observer");
		Observer thirdObserver = new EmailTopicSubscriber("Third Observer");
		
		// Register them...
		topic.register(firstObserver);
		topic.register(secondObserver);
		topic.register(thirdObserver);
		
		// Attaching observer to subject
		firstObserver.setSubject(topic);
		secondObserver.setSubject(topic);
		thirdObserver.setSubject(topic);
		
		// Check for updates
		firstObserver.update();
		thirdObserver.update();
		
		// Provider Subject ( broadcaster )
		topic.postMessage("new user registered");
		topic.postMessage("Gaurav Kill Raghav");
		
		// Unregister
		topic.unregister(firstObserver);
		topic.postMessage("Gaurav Knock Down Seemran");
		
	}

}


```

# Builder Design Pattern

// BUILDER DESIGN PATTER
// When construction gets a little bit too complicated.

// Building piece by piece like StringBuilder Class does.
```java
	String[] words = {"Hello", "World"};
	StringBuilder sb = new StringBuilder();
	sb.append("<ul>\n");
	for(String word: words)
		sb.append(String.format("\t<li>%s</li>\n", word));
	sb.append("</ul>");
	System.out.println(sb.toString());
```


// - Some objects are simple and can be created in a single constuctor call
// - Other objects require a lot of ceremoney to create
// - Having an object with 10 constructor arguments is not productive
// - Instead, opt for piecewise construction
// - It helps to provide good API.

// When piecewise object construction is complicated, provice an API for doing it succinctly


> HtmlElement.java

```java

class HtmlElement {
	public String name, text;
	public ArrayList<HtmlElement> elements = new ArrayList<>();
	private final int indentSize = 2;
	private final String newLine = System.lineSeparator();
	
	public HtmlElement() {
	}
	
	public HtmlElement(String name, String text) {
		this.name = name;
		this.text = text;
	}

	private String toStringImpl(int indent)
	{
		StringBuilder sb = new StringBuilder();
		String i = String.join("", Collections.nCopies(indent * indentSize, " "));
		sb.append(String.format("%s<%s>%s", i, name, newLine));
		if(text != null && !text.isEmpty()) {
			sb.append(String.join("", Collections.nCopies(indentSize * (indent+1), " ")))
			.append(text)
			.append(newLine);
		}
		for (HtmlElement e : elements)
			sb.append(e.toStringImpl(indent + 1));
		
		sb.append(String.format("%s</%s>%s", i, name, newLine));
		return sb.toString();
	}
	
	@Override
	public String toString() {
		return toStringImpl(0);
	}
	
}

```

> HtmlBuilder.java

```java

class HtmlBuilder
{
	private String rootName;
	private HtmlElement root = new HtmlElement();
	
	public HtmlBuilder(String rootName) {
		this.rootName = rootName;
		root.name = rootName;
	}
	
	/* public void addChild(String childName, String childText)
	{
		HtmlElement e = new HtmlElement(childName, childText);
		root.elements.add(e);
	} */
	
	public HtmlBuilder addChild(String childName, String childText)
	{
		HtmlElement e = new HtmlElement(childName, childText);
		root.elements.add(e);
		return this;
	}
	
	public void clear() 
	{
		root = new HtmlElement();
		root.name = rootName;
	}

	@Override
	public String toString() {
		return root.toString();
	}
	
}


```



> Main.java
```java

public class Main {

	public static void main(String[] args) {
		HtmlBuilder builder = new HtmlBuilder("ul");
		builder.addChild("li", "Hello");
		builder.addChild("li", "World");
		System.out.println(builder);	
		
		// Fluent Builder i.e method chaining
		HtmlBuilder builder2 = new HtmlBuilder("ul");
		builder2.addChild("li", "Hello").addChild("li", "World");
		System.out.println(builder2);
	}

}


```

## Fluent Builder Inheritance with Recursive Generics

> Person.java

```java

class Person
{
	public String name;
	public String position;
	
	
	@Override
	public String toString() {
		return "Person [name=" + name + ", position=" + position + "]";
	}
	
}

```
> PersonBuilder.java

```java

class PersonBuilder<SELF extends PersonBuilder<SELF>>
{
	protected Person person = new Person();
	
	public SELF withName(String name)
	{
		person.name = name;
		return self();
	}

	public Person build()
	{
		return person;
	}
	
	@SuppressWarnings("unchecked")
	protected SELF self() {
		return (SELF) this;
	}
	
}

```

> EmployeeBuilder.java

```java

class EmployeeBuilder extends PersonBuilder<EmployeeBuilder>
{
	public EmployeeBuilder worksAt(String position)
	{
		person.position = position;
		return self();
	}

	@Override
	protected EmployeeBuilder self() {
		// TODO Auto-generated method stub
		return this;
	}
	
}


```



> Main.java
```java

public class Main {

	public static void main(String[] args) {
		// recursive generics: to preserve to fluent interface
		EmployeeBuilder eb = new EmployeeBuilder();
		Person p = eb.withName("Gaurav").worksAt("Developer").build();
		System.out.println(p);		
	}

}


```

## Build Complicated Bbject Via Multiple Builders

> Person.java

```java

class Person {
	// address
	public String streetAddress, postCode, city;

	// emplyement
	public String companyName, position;
	public int annualIncode;

	@Override
	public String toString() {
		return "Person [streetAddress=" + streetAddress + ", postCode=" + postCode + ", city=" + city + ", companyName="
				+ companyName + ", position=" + position + ", annualIncode=" + annualIncode + "]";
	}
}

```

> PersonBuilder.java  **Working as builder facade**

```java

class PersonBuilder {
	protected Person person = new Person();

	public PersonAddressBuilder lives() {
		return new PersonAddressBuilder(person);
	}

	public PersonJobBuilder works() {
		return new PersonJobBuilder(person);
	}

	public Person build() {
		return person;
	}
}

```

> PersonAddressBuilder.java

```java

class PersonAddressBuilder extends PersonBuilder {

	public PersonAddressBuilder(Person person) {
		this.person = person;
	}

	public PersonAddressBuilder at(String streetAddress) {
		person.streetAddress = streetAddress;
		return this;
	}

	public PersonAddressBuilder withPostCode(String postCode) {
		person.postCode = postCode;
		return this;
	}

	public PersonAddressBuilder in(String city) {
		person.city = city;
		return this;
	}

}

```

> PersonJobBuilder.java

```java

class PersonJobBuilder extends PersonBuilder {
	public PersonJobBuilder(Person person) {
		this.person = person;
	}

	public PersonJobBuilder at(String companyName) {
		this.person.companyName = companyName;
		return this;
	}

	public PersonJobBuilder asA(String position) {
		person.position = position;
		return this;
	}

	public PersonJobBuilder earning(int annualIncome) {
		person.annualIncode = annualIncome;
		return this;
	}

}

```

> Main.java

```java

public class Main() {
	public static void main(String[] args) {
	
		// jumps from one subBuilder to another subBuilder
		PersonBuilder pb = new PersonBuilder();
		Person person = pb
				.lives()
					.at("Korol Bhag, Street 6")
					.in("Delhi")
					.withPostCode("1234")
				.works()
					.at("MatchBox Inc")
					.asA("CEO")
					.earning(150000)
				.build();
		System.out.println(person);	
	
	}
}
```

# The Facade Pattern

Provides a unified interface to a set of interfaces in a subsystem. Facade defines a higer-level interface that makes the subsystem easier to use. ***(When you have less depandency, use facade i.e. black box)***

> Below image is not Object Oriented Design

![NotOODesignImage](https://res.cloudinary.com/dcalvdelc/image/upload/v1588565119/3.1_Facade_Pattern.png.png)


> CPU.java

```java

class CPU {
	public void freeze()
	{
		System.out.println("Computer freezing\u2026 ");
	}
	
	public void jump(long position)
	{
		System.out.println("Jumping to\u2026 " + position);
	}
	
	public void execute()
	{
		System.out.println("Computer executing commands\u2026 ");
	}
}

```

> HardDrive.java

```java


class HardDrive {
	public byte[] read(long Iba, int size)
	{
		return new byte[] {'f', 'z'};
	}
}

```

> Memory.java

```java

class Memory {
	public void load(long position, byte[] data)
	{
		System.out.println("Added item to memory\u2026 " + position);
	}
}

```

> ComputerFacade.java

```java

class ComputerFacade {
	private CPU processor;
	private Memory ram;
	private HardDrive hd;
	
	public ComputerFacade(CPU processor, Memory ram, HardDrive hd) {
		super();
		this.processor = processor;
		this.ram = ram;
		this.hd = hd;
	}
	
	public void start()
	{
		processor.freeze();
		ram.load(0xbc, hd.read(3456, 89));
		processor.jump(0xbc);
		processor.execute();
	}
	
}

```

> Main.java

```java

public class Main {
	
	public static void main(String[] args) {
		CPU cpu = new CPU();
		Memory memory = new Memory();
		HardDrive hd = new HardDrive();
		ComputerFacade cf = new ComputerFacade(cpu, memory, hd);
		cf.start();
	}
}

```

# The Template Method Design Pattern

```
Very simple yet very important,
whenever we have common methods, algorithm, strategy which can be abstract out
so that the responsibly is under sub classes
```

> This is the problem with repeatation of method
![image1](https://res.cloudinary.com/dcalvdelc/image/upload/v1588571251/4.2_TemplateDesignPattern2.png.png)

> We can solve this with `The Template Method Design Pattern`
![image2](https://res.cloudinary.com/dcalvdelc/image/upload/v1588571251/4.1_TemplateDesignPattern1.png.png)

##### Example 1 : Little More Complex...

> GameController.java

```java

interface GameController {
	void initGame();
	void playGame();
	void endGame();
	void start() throws InterruptedException;
	void printClassInformation();
	void destroy();
}

```

> Game.java

```java

abstract class Game<SELF extends Game<SELF>> implements GameController {
	protected String className;

	public Game() {
		className = getSelf().getClass().getSimpleName();
	}

	public void start() throws InterruptedException {
		initGame();
		TimeUnit.SECONDS.sleep(5);
		playGame();
		TimeUnit.SECONDS.sleep(5);
		if (playerConnected())openLobby();
		TimeUnit.SECONDS.sleep(5);
		endGame();
	}

	@SuppressWarnings("unchecked")
	protected SELF getSelf() {
		return (SELF) this;
	}

	// The Hooked-On Template Method
	public abstract void printClassInformation();
	public abstract void openLobby();

	boolean playerConnected() {
		return true;
	}

	/* gives a default implementation */
	public void destroy() {
		System.out.println(className + " is distroyed");
	}
	
}

```

> EndlessRunnerGame.java

```java

class EndlessRunnerGame extends Game<EndlessRunnerGame> {

	@Override
	public void initGame() {
		print("initialize");
	}

	@Override
	public void playGame() {
		print("playing");
	}

	@Override
	public void endGame() {
		print("ending");
	}

	@Override
	public void start() throws InterruptedException {
		print("starting");
		super.start();
	}

	@Override
	public void printClassInformation() {
		// parent has no body, so here we will have our own logic...
		System.out.println(className + " is a child of Game abstract class\u2026");
	}
	
	@Override
	public void openLobby() {
		System.out.println("Opening lobby for hosting\u2026");
	}

	// adding more code to default methods
	@Override
	public void destroy() {
		// this code will run before the parent code
		System.out.println(className + " controller has been removed successfully,");
		super.destroy(); // this is parent code
	}
	
	// Add more methods...
	private void print(String message) {
		System.out.println(String.format("%s, %s\u2026", className, message));
	}

}

```

> Main.java

```java

public class Main {

	public static void main(String[] args) throws InterruptedException {
		GameController gc = new EndlessRunnerGame();
		gc.start();
		System.out.println("===================");
		gc.printClassInformation();
		gc.destroy();
	}

}

```


##### Example 2 : Simple...

```java

abstract class Game {

	abstract void initGame();
	abstract void plagGame();
	abstract void endGame();
	
	public void start() {
		initGame();
		playGame();
		endGame();
	}

	// The Hooked-On Template Method
	public abstract void printClassInformation();
	public abstract void openLobby();

	/* gives a default implementation */
	public void destroy() {
		System.out.println("Game is distroyed");
	}
	
}


```

> EndlessRunnerGame.java

```java

class EndlessRunnerGame extends Game {

	@Override
	public void initGame() {
		print("initialize");
	}

	@Override
	public void playGame() {
		print("playing");
	}

	@Override
	public void endGame() {
		print("ending");
	}

	@Override
	public void start() throws InterruptedException {
		print("starting game...");
		super.start();
	}

	@Override
	public void printClassInformation() {
		// parent has no body, so here we will have our own logic...
		System.out.println("EndlessRunner is a child of Game abstract class\u2026");
	}
	
	@Override
	public void openLobby() {
		System.out.println("Opening lobby for hosting\u2026");
	}

	// adding more code to default methods
	@Override
	public void destroy() {
		// this code will run before the parent code
		System.out.println("EndlessRunner controller has been removed successfully,");
		super.destroy(); // this is parent code
	}
	
	// Add more methods...
	private void print(String message) {
		System.out.println(String.format("%s, %s\u2026", "EndlessRunner", message));
	}

}

```

> Main.java

```java

public class Main {

	public static void main(String[] args) {
		GameController gc = new EndlessRunnerGame();
		gc.start();
		gc.printClassInformation();
		gc.destroy();
	}
	
}	


```

# Iterator Design Pattern

![image](https://res.cloudinary.com/dcalvdelc/image/upload/v1588590405/Iterator_Design_Pattern.png)

> Iterator.java

```java

interface Iterator {
	boolean hasNext();
	Object next();
}

```

> Product.java

```java

class Product {
	private String name;
	private String description;
	private double price;

	public Product(String name, String description, double price) {
		super();
		this.name = name;
		this.description = description;
		this.price = price;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}

	public double getPrice() {
		return price;
	}

	public void setPrice(double price) {
		this.price = price;
	}

}

```

> GeekyStoreIterator.java

```java
class GeekyStoreIterator implements Iterator {

	ArrayList<Product> catalog;
	int position = 0;

	public GeekyStoreIterator(ArrayList<Product> catalog) {
		this.catalog = catalog;
	}

	@Override
	public boolean hasNext() {
		if (position >= catalog.size() || catalog.get(position) == null)
			return false;
		return true;
	}

	@Override
	public Object next() {
		Product product = catalog.get(position);
		position += 1;
		return product;
	}

}
```

> DevStoreIterator.java

```java

class DevStoreIterator implements Iterator {

	Product[] catalog;
	int position = 0;

	public DevStoreIterator(Product[] catalog) {
		this.catalog = catalog;
	}

	@Override
	public boolean hasNext() {
		if (position >= catalog.length || catalog[position] == null)
			return false;
		return true;
	}

	@Override
	public Object next() {
		Product product = catalog[position];
		position += 1;
		return product;
	}

}

```

> CatalogCreator.java

```java

interface CatalogCreator {
	Iterator createIterator();
}

```

> DevStoreCatalog.java

```java

class DevStoreCatalog implements CatalogCreator {

	private static final int MAX_ITEM = 4;
	private int numberOfProducts = 0;
	private Product[] catalog;

	public DevStoreCatalog() {
		catalog = new Product[MAX_ITEM];

		addItem("C++ is NOT dead. Yet!", "T-Shirt", 30.99);
		addItem("Java Rocks. Yes", "Silky mouse-pad", 10.99);
		addItem("Java Design Patterns", "Book - A must!", 130.99);
		addItem("Web development cookbook", "The best web development cookbook  - 2018", 18.99);
	}

	public void addItem(String name, String description, double price) {
		Product product = new Product(name, description, price);
		if (numberOfProducts >= MAX_ITEM) {
			System.out.println("Catalog is Full - can't add products.");
		} else {
			catalog[numberOfProducts] = product;
			numberOfProducts += 1;
		}
	}

	public Product[] getCatalog() {
		return catalog;
	}

	@Override
	public Iterator createIterator() {
		return new DevStoreIterator(catalog);
	}

}

```

> GeekyStoreCatalog.java


```java
class GeekyStoreCatalog implements CatalogCreator {
	private ArrayList<Product> catalog;

	public GeekyStoreCatalog() {
		catalog = new ArrayList<>();

		addItem("Superman Comic", "The best book in town", 12.99);
		addItem("Batman Comic", "Ok, But still good", 11.99);
		addItem("Star Wars", "Can't lice without it", 39.99);
		addItem("Jedi T-Shirt", "Gotta Have it", 29.99);
	}

	public void addItem(String name, String description, double price) {
		Product product = new Product(name, description, price);
		catalog.add(product);
	}

	public ArrayList<Product> getCatalog() {
		return catalog;
	}

	@Override
	public Iterator createIterator() {
		return new GeekyStoreIterator(getCatalog());
	}
}
```

> Seller.java

```java

class Seller {

	CatalogCreator catalogCreator;

	public Seller(CatalogCreator catalogCreator) {
		this.catalogCreator = catalogCreator;
	}

	public void printCatalog() {
		Iterator itr = catalogCreator.createIterator();
		System.out.println("Printing Catalog: ");
		printCatalog(itr);
	}

	private void printCatalog(Iterator itr) {
		while (itr.hasNext()) {
			Product product = (Product) itr.next();
			System.out.print(product.getName() + ", ");
			System.out.print(product.getDescription() + " ");
			System.out.println(product.getPrice());
		}
	}

}

```

> Main.java

```java

public class Main {

	public static void main(String[] args) {
	
		
		DevStoreCatalog devStoreCatalog = new DevStoreCatalog();
		GeekyStoreCatalog geekyStoreCatalog = new GeekyStoreCatalog();

		Seller seller = new Seller(geekyStoreCatalog);
		seller.printCatalog();

		System.out.println("__________");

		seller = new Seller(devStoreCatalog);
		seller.printCatalog();

	}

}

```


# State Design Design Pattern

``allows an object to alter it's behavior when it's internal state changes. The object will appear to change it's class.``

- Image 1
![Image1](https://res.cloudinary.com/dcalvdelc/image/upload/v1588593730/7.2_statePattern.png.png)

- Image 2
![Image2](https://res.cloudinary.com/dcalvdelc/image/upload/v1588593729/7.1_General-StatePatternDiagram.png.png)
 
**First, Obvious Solution For State Design Pattern (Pain in the ass)**

> SodaMachine.java

```java

class SodaMachine {
	
	// STATES
	final static int SOLD_OUT = 0;
	final static int NO_MONEY = 1;
	final static int HAS_MONEY = 2;
	final static int SOLD = 3;

	int state = SOLD_OUT;
	int count = 0;

	public SodaMachine(int count) {
		super();
		this.count = count;
		if (count > 0)
			state = NO_MONEY;

		System.out.println("Welcome to our soda vending machine");
		System.out.println("Inventory is " + count + " Sodas");
		System.out.println("Insert a $ bill to get started...");
		System.out.println("___________________________________________");
	}

	// Actions
	public void insertMoney() {
		if (state == HAS_MONEY) {
			System.out.println("You can't insert another $ bill");
		} else if (state == NO_MONEY) {
			state = HAS_MONEY;
			System.out.println("You inserted a $");
		} else if (state == SOLD_OUT) {
			System.out.println("The machine is sold out.");
		} else if (state == SOLD) {
			System.out.println("Please wait! We are aleardy giving you a soda!");
		}
	}

	public void ejectMoney() {
		if (state == HAS_MONEY) {
			System.out.println("Returning $ bill");
			state = NO_MONEY;
		} else if (state == NO_MONEY) {
			System.out.println("You haven't inserted a $ bill");
		} else if (state == SOLD) {
			System.out.println("Aleardy selected soda!");
		} else if (state == SOLD_OUT) {
			System.out.println("Machine sold out!");
		}
	}

	public void selectSoda() {
		if (state == SOLD) {
			System.out.println("Dispanse your soda as we speak... Enjoy");
		} else if (state == NO_MONEY) {
			System.out.println("You select soda, but money first, buddy!");
		} else if (state == SOLD_OUT) {
			System.out.println("You're outta luck - No sodas left :(");
		} else if (state == HAS_MONEY) {
			System.out.println("You selected a soda...");
			state = SOLD;
			dispence();
		}
	}

	public void dispence() {
		if (state == SOLD) {
			System.out.println("Dispanse your soda as we speak... Enjoy");
			count -= 1;
			if (count == 0) {
				System.out.println("sorry, out of sodas!");
				state = SOLD_OUT;
			} else
				state = NO_MONEY;
		} else if (state == NO_MONEY) {
			System.out.println("Please inserted a $ bill");
		} else if (state == SOLD_OUT) {
			System.out.println("Machine sold out!");
		} else if (state == HAS_MONEY) {
			System.out.println("No soda dispanse");
		}
	}

}

```

> Main.java

```java

public class Main {

	public static void main(String[] args) {

		SodaMachine sodaMachine = new SodaMachine(10);
		sodaMachine.insertMoney();
		sodaMachine.selectSoda();

		System.out.println("Inventory has: " + sodaMachine.count + " sodas");
		System.out.println("___________________");

		sodaMachine.insertMoney();
		sodaMachine.ejectMoney();
		sodaMachine.selectSoda();

		System.out.println("Inventory has: " + sodaMachine.count + " sodas");
		System.out.println("___________________");

		sodaMachine.insertMoney();
		sodaMachine.selectSoda();
		sodaMachine.insertMoney();
		sodaMachine.selectSoda();
		sodaMachine.ejectMoney();

		System.out.println("Inventory has: " + sodaMachine.count + " sodas");
		System.out.println("___________________");
		
	}
	
}

// OUTPUT:-

/*

Welcome to our soda vending machine
Inventory is 10 Sodas
Insert a $ bill to get started...
___________________________________________
You inserted a $
You selected a soda...
Dispanse your soda as we speak... Enjoy
Inventory has: 9 sodas
___________________
You inserted a $
Returning $ bill
You select soda, but money first, buddy!
Inventory has: 9 sodas
___________________
You inserted a $
You selected a soda...
Dispanse your soda as we speak... Enjoy
You inserted a $
You selected a soda...
Dispanse your soda as we speak... Enjoy
You haven't inserted a $ bill
Inventory has: 7 sodas
___________________

*/

```

**Second, Solution With State Design Pattern**

> State.java

```java

interface State
{
	void insertMoney();
	void ejectMoney();
	void select();
	void dispense();
}

```

> SodaVendingMachine.java

```java

class SodaVendingMachine
{
	State soldOutState;
	State noMoneyState;
	State hasMoneyState;
	State soldState;
	
	State state = soldOutState;
	int count = 0;
	
	public SodaVendingMachine(int numberOfSodas) {
		soldOutState = new SoldOutState(this);
		noMoneyState = new NoMoneyState(this);
		hasMoneyState = new HasMoneyState(this);
		soldState = new SoldState(this);
		count = numberOfSodas;
		
		if(numberOfSodas > 0)
		{
			state = noMoneyState; // if there are more than 0 sodas we set the state to the noMoneyState
		}
	}
	
	// Actions
	public void insertMoney() {
		state.insertMoney();
	}

	public void ejectMoney() {
		state.ejectMoney();
	}

	public void selectSoda() {
		state.select();
	}

	public void dispence() {
		state.dispense();
	}
	
	int getCount() {
		return count;
	}

	public State getSoldOutState() {
		return soldOutState;
	}

	public void setSoldOutState(State soldOutState) {
		this.soldOutState = soldOutState;
	}

	public State getNoMoneyState() {
		return noMoneyState;
	}

	public void setNoMoneyState(State noMoneyState) {
		this.noMoneyState = noMoneyState;
	}

	public State getHasMoneyState() {
		return hasMoneyState;
	}

	public void setHasMoneyState(State hasMoneyState) {
		this.hasMoneyState = hasMoneyState;
	}

	public State getSoldState() {
		return soldState;
	}

	public void setSoldState(State soldState) {
		this.soldState = soldState;
	}

	public State getState() {
		return state;
	}

	public void setState(State state) {
		this.state = state;
	}
	
	@Override
	public String toString() {
		StringBuffer result = new StringBuffer();
		result
		.append("\nThe Soda Machine, Co")
		.append("\nInventory:" + count + " soda");
		
		if(count != 1)
		{
			result.append("\'s");
		}
		result.append("\n");
		result.append("Soda Vending Machine is " + state + "\n");
		return result.toString();
	}
	
}


```

> SoldState.java

```java

class SoldState implements State 
{
	
	SodaVendingMachine sodaVendingMachine;
	
	public SoldState(SodaVendingMachine sodaVendingMachine)
	{
		this.sodaVendingMachine = sodaVendingMachine;
	}

	@Override
	public void insertMoney() {
		System.out.println("Give a second... soda coming right up!");
	}

	@Override
	public void ejectMoney() {
		System.out.println("Sorry... soda is coming...");
	}

	@Override
	public void select() {
		System.out.println("Nope... you can't eject the money if you already have the soad");
	}

	@Override
	public void dispense() {
		System.out.println("Stop trying to get second soda for free!");
		if(sodaVendingMachine.getCount() > 0)
		{
			sodaVendingMachine.setState(sodaVendingMachine.noMoneyState);
		}
		else
		{
			System.out.println("Sorry, out of sodas");
			sodaVendingMachine.setState(sodaVendingMachine.soldOutState);
		}
	}
	
	@Override
	public String toString() {
		return "Dispensing soda";
	}
}


```

> SoldOutState.java

```java

class SoldOutState implements State
{
	
	SodaVendingMachine sodaVendingMachine;
	
	public SoldOutState(SodaVendingMachine sodaVendingMachine) {
		this.sodaVendingMachine = sodaVendingMachine;
	}

	@Override
	public void insertMoney() {
		System.out.println("Machine sold out!");
	}

	@Override
	public void ejectMoney() {
		System.out.println("Insert money first before ejecting");
	}

	@Override
	public void select() {
		System.out.println("No soda available");
	}

	@Override
	public void dispense() {
		System.out.println("sold out");
	}
	
	@Override
	public String toString() {
		return "Sold out!";
	}
	
}


```

> NoMoneyState.java

```java

class NoMoneyState implements State
{
	
	SodaVendingMachine sodaVendingMachine;
	
	public NoMoneyState(SodaVendingMachine sodaVendingMachine) {
		this.sodaVendingMachine = sodaVendingMachine;
	}

	@Override
	public void insertMoney() {
		System.out.println("You inserted a doller");
		sodaVendingMachine.setState(sodaVendingMachine.getHasMoneyState());
	}

	@Override
	public void ejectMoney() {
		System.out.println("You haven't inserted a doller");
	}

	@Override
	public void select() {
		System.out.println("You selected, but there's no money!");
	}

	@Override
	public void dispense() {
		System.out.println("Pay me first!");
	}

	@Override
	public String toString() {
		return "waiting for a doller...";
	}
	
}


```

> HasMoneyState.java


```java

class HasMoneyState implements State
{

	SodaVendingMachine sodaVendingMachine;
	
	public HasMoneyState(SodaVendingMachine sodaVendingMachine) {
		this.sodaVendingMachine = sodaVendingMachine;
	}
	
	@Override
	public void insertMoney() {
		System.out.println("No need to insert another doller bill");
	}

	@Override
	public void ejectMoney() {
		System.out.println("Returning your doller");
		sodaVendingMachine.setState(sodaVendingMachine.getNoMoneyState());
	}
	
	@Override
	public void select() {
		System.out.println("Selected item...");
		sodaVendingMachine.setState(sodaVendingMachine.getSoldState());
	}

	@Override
	public void dispense() {
		System.out.println("No soda dispansed");
	}

	@Override
	public String toString() {
		return "waiting for a new selection...";
	}
	
}


```

> Main.java

```java

public class Main {

	public static void main(String[] args) {

		SodaVendingMachine sodaVendingMachine = new SodaVendingMachine(10);
		
		System.out.println(sodaVendingMachine);
		
		sodaVendingMachine.insertMoney();
		sodaVendingMachine.selectSoda();

		System.out.println(sodaVendingMachine);
		
		sodaVendingMachine.insertMoney();
		sodaVendingMachine.selectSoda();
		
		System.out.println(sodaVendingMachine);
		
		sodaVendingMachine.insertMoney();
		sodaVendingMachine.selectSoda();
		
		System.out.println(sodaVendingMachine);
		
	}

}

```
# The Remote Proxy Design Pattern

![ProxyPattern](https://res.cloudinary.com/dcalvdelc/image/upload/v1588757942/3.2_ProxyPattern.png.png)

![BankProxyPatternExample](https://res.cloudinary.com/dcalvdelc/image/upload/v1588757941/3.1_BankProxy.png.png)

``
Proxy pattern is used when we need to create a wrapper to cover the main object's complexity from the client. 
Remote proxy: They are responsible for representing the object located remotely. 
Talking to the real object might involve marshalling and unmarshalling of data and talking to the remote object.
``

> Bank.java

```java

interface Bank {
	void withdrawMoney(String clientName) throws Exception;
}

```

> RealBank.java

```java

class RealBank implements Bank {

	@Override
	public void withdrawMoney(String clientName) throws Exception {
		System.out.println(clientName + " is withdrawing form the ATM\u2026");
	}

}

```

> ProxyBank.java

```java

class ProxyBank implements Bank {

	private RealBank bank = new RealBank();
	private static List<String> bannedClients;

	static {
		bannedClients = new ArrayList<>();
		bannedClients.add("xXcs");
		bannedClients.add("me@me");
		bannedClients.add("@xml.com");
	}

	@Override
	public void withdrawMoney(String clientName) throws Exception {
		if (bannedClients.contains(clientName.toLowerCase())) {
			throw new Exception(clientName + ", Access Denied! You are not who you say you are!");
		}
		bank.withdrawMoney(clientName);
	}

}

```

> Main.java

```java

public class Main {

	public static void main(String[] args){
		
	Bank bank = new ProxyBank();
	try{
		bank.withdrawMoney("Gaurav");
		bank.withdrawMoney("me@me");
	}catch(Exception e){
		System.out.println(e.getMessage());
	}
	
}

```

##### EXAMPLE 2

> INotificationInterceptor.java

```java

interface INotificationInterceptor
{
	void sendMessage(String username, String message);
}

```

> NotificationInterceptor.java

```java

class NotificationInterceptor implements INotificationInterceptor
{
	
	@Override
	public void sendMessage(String username, String message) {
		StringBuilder builder = new StringBuilder();
		
		builder
			.append("sending message to:\n")
			.append("<p>")	
				.append("<b>@")
					.append(username.toLowerCase())
				.append("</b>")
					.append(message)
			.append("</p>");
		
		String result = builder.toString();
		
		// you can parse the HTML later...
		System.out.println(result);
	}
	
}

```

> NotificationSender.java 

```java

class NotificationSender implements INotificationInterceptor
{

	NotificationInterceptor notificationInterceptor = new NotificationInterceptor();
	static ArrayList<String> alreadySentTo;
	
	static {
		alreadySentTo = new ArrayList<>();
		alreadySentTo.add("saurav");
		alreadySentTo.add("mahesh");
	}
	
	@Override
	public void sendMessage(String username, String message) {
		if(!alreadySentTo.contains(username))
		{
			notificationInterceptor.sendMessage(username, message);
			alreadySentTo.add(username);
			return;
		}
		System.out.println("You have already sent notification to, " + username);
	}
}

```

```java

public class Main {

	public static void main(String[] args){
		INotificationInterceptor notificationInterceptor = new NotificationSender();
		notificationInterceptor.sendMessage("gaurav", "Welcomes you to the application.");
		notificationInterceptor.sendMessage("gaurav", "Welcomes you to the application.");
	}

}

```

# The MVC (Model View Controller) Design Pattern

> model/Employee.java

```java

class Employee
{
	private String firstName;
	private String lastName;
	private String ssNumber;
	private String salary;
	
	public String getSalary() {
		return salary;
	}
	public void setSalary(String salary) {
		this.salary = salary;
	}
	public String getFirstName() {
		return firstName;
	}
	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getSsNumber() {
		return ssNumber;
	}
	public void setSsNumber(String ssNumber) {
		this.ssNumber = ssNumber;
	}
	
}

```

> view/EmployeeDashboardView.java

```java

class EmployeeDashboardView
{
	public void printEmployeeInformation(Employee e)
	{
		System.out.println("Name: " + e.getFirstName() + " " + e.getLastName());
		System.out.println("SocialSecurtyNumber: " + e.getSsNumber());
		System.out.println("Salary: " + e.getSalary());
	}
}

```

> controller/EmployeeController.java

```java

class EmployeeController
{
	private Employee employeeModel;
	private EmployeeDashboardView employeeDashboardView;
	
	public EmployeeController(Employee employeeModel, EmployeeDashboardView employeeDashboardView) {
		this.employeeModel = employeeModel;
		this.employeeDashboardView = employeeDashboardView;
	}
	
	public void setEmployee(String firstName, String lastName)
	{
		employeeModel.setFirstName(firstName);
		employeeModel.setLastName(lastName);
	}
	
	public String getEmployeeName()
	{
		return employeeModel.getFirstName() + " " + employeeModel.getLastName();
	}
	
	public void setSSN(String ssn)
	{
		employeeModel.setFirstName(ssn);
	}
	
	public String getSSN()
	{
		return employeeModel.getSsNumber();
	}
	
	// Update our view
	
	public void updateDashboardView()
	{
		employeeDashboardView.printEmployeeInformation(employeeModel);
	}
	
}

```
> Main.java

```java

public class Main {

	public static Employee retrieveEmployeeFromServer()
	{
		Employee employee = new Employee();
		employee.setSsNumber("3222458");
		employee.setFirstName("Gourav");
		employee.setLastName("Gupta");
		employee.setSalary("150000");		
		return employee;
	}
	
	public static void main(String[] args) throws InterruptedException {
		
		Employee employee = retrieveEmployeeFromServer();
		
		// Creating our view to which we'll write our employee information into
		EmployeeDashboardView view = new EmployeeDashboardView();
		
		// Creating controller
		EmployeeController controller = new EmployeeController(employee, view);
		controller.updateDashboardView();
		
	}
	
}

```

# Prototype Design Pattern

``Used when creating an instance of a given class is either expensive or complicated.`` [Truth](https://softwareengineering.stackexchange.com/a/149569)

> Animal.java

```java

interface Animal extends Cloneable {
	Animal clone();
}

```

> Dog.java

```java

class Dog implements Animal {

	private String name;
	private int age;

	public Dog(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	@Override
	public Animal clone() {
		Dog doggy = null;
		try {
			doggy = (Dog) super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return doggy;
	}

	@Override
	public String toString() {
		return "A Dog [name=" + name + ", age=" + age + "]";
	}

}


```

> Main.java

```java

public class Main {

	public static void main(String[] args) throws InterruptedException {

		Dog joy = new Dog("Joy", 60);
		System.out.println(joy);

		Dog sam = (Dog) joy.clone();
		sam.setName("sam");
		System.out.println(sam);
		
		System.out.println(System.identityHashCode(sam));
		System.out.println(System.identityHashCode(joy));

	}
	
}

```

# Mediator Design Pattern
``Centralizes complex communications and control between related objects. it solves Subsystem (Actors || Calleagues) Communication Problem i.e. Centralized Communication``

![image](https://res.cloudinary.com/dcalvdelc/image/upload/v1589345913/4.1_Mediator-JDP.png.png)
[read difference between observer design pattern and the mediator design pattern](https://stackoverflow.com/a/9226745/8703198)

> ATCMediator.java

```java

interface ATCMediator {
	void sendMessage(String msg, AirCraft airCraft);

	void addAirCraft(AirCraft airCraft);
}

```

> AirCraft.java

```java

abstract class AirCraft {
	protected ATCMediator mediator;
	protected String name;

	public AirCraft(ATCMediator mediator, String name) {
		this.mediator = mediator;
		this.name = name;
	}

	public abstract void send(String message);

	public abstract void receive(String message);

}

```

> ATCMediatorImpl.java

```java

class ATCMediatorImpl implements ATCMediator {

	private List<AirCraft> airCrafts;

	public ATCMediatorImpl() {
		this.airCrafts = new ArrayList<>();
	}

	@Override
	public void sendMessage(String msg, AirCraft airCraft) {
		for (AirCraft a : airCrafts) {
			// message should not be receive by the aircraft sending the message...
			if (a != airCraft) {
				a.receive(msg);
			}
		}
	}

	@Override
	public void addAirCraft(AirCraft airCraft) {
		this.airCrafts.add(airCraft);
	}

}

```

> AirCraftImpl.java

```java

class AirCraftImpl extends AirCraft {

	public AirCraftImpl(ATCMediator mediator, String name) {
		super(mediator, name);
	}

	@Override
	public void send(String message) {
		System.out.println(this.name + " : Sending message = " + message);
		mediator.sendMessage(message, this);
	}

	@Override
	public void receive(String message) {
		System.out.println(this.name + " : Received message = " + message);
	}
	
}

```

> Main.java

```java


public class Main {

	public static void main(String[] args) throws InterruptedException {
	
		// Air Traffic Controller
		ATCMediator mediator = new ATCMediatorImpl();
		
		// Create a few airCrafts
		AirCraft boing1 = new AirCraftImpl(mediator, "Boing 1");
		AirCraft helicopter = new AirCraftImpl(mediator, "Helicopter");
		AirCraft boing707 = new AirCraftImpl(mediator, "Boing 707");
		
		// Adding airCrafts to the mediator
		mediator.addAirCraft(boing1);
		mediator.addAirCraft(helicopter);
		mediator.addAirCraft(boing707);
		
		boing1.send("Boing 1 Killed Helicopter");
	
	}
	
}

```


```
***********************************************************************************************
***********************************************************************************************
> PLAYING WITH DESIGN, MAKING MISTAKES, CHECKING HOW THINGS WORK
***********************************************************************************************
***********************************************************************************************

```
> Things we should know

- Whenever we use same method signature than pull that method and store in an interface, that woul be nice
- we have observe that any design to work we must wrap that with other solid classes and than use that solid class to wrap other class or little bit more like composition and all.

```java


interface IProxyChain {
	void bypassCall(int hex, boolean safe); // hex = 0xbc, safe = true;
}

interface IMessenger<MESESAGE, TO> {
	void createMessage(MESESAGE message, TO to);

	void createMMS();
}

interface ILogDumper {
	void printTrace();
}

interface IInternet<D, C, DialUp> {

	void init() throws InterruptedException;  // call this interface methods in sequence

	IProxyChain openConnection();

	boolean dialUp(DialUp number); // return true after get connected to the internet

	IMessenger<D, C> createMessage();

	void checkMessage(); // probably print simple message `no new message`

	ILogDumper closeConnection();
}

class MessangerUI<D extends String, C extends String> implements IMessenger<D, C> {

	@Override
	public void createMessage(D message, C to) {
		System.out.println(String.format("to: %s, message: %s", to, message));
	}

	@Override
	public void createMMS() {
		System.out.println("Creating mms for you\u2026 waiting");
	}

}

class ByPasser implements IProxyChain {
	@Override
	public void bypassCall(int hex, boolean safe) {
		System.out.println("Bypassing the connection using vpn, with route: " + hex + " SSL: " + safe);
	}
}

class DumpLogger implements ILogDumper {
	// more fields;
	
	@Override
	public void printTrace() {
		System.out.println("trace clear\u2026");
	}
	
	// more methods to manupulate the message
}

class Web implements IInternet<String, String, Integer> {

	private int itr = 0;
	private int maxItr = 5;
	
	private boolean isConnected = false;
	private ByPasser bp = new ByPasser();
	private DumpLogger dl = new DumpLogger();

	@Override
	public IProxyChain openConnection() {
		return bp;
	}

	@Override
	public ILogDumper closeConnection() {
		return dl;
	}

	@Override
	public boolean dialUp(Integer number) {
		return itr >= maxItr;
	}

	@Override
	public void init() throws InterruptedException {
		openConnection().bypassCall(0xbc, true);;
		while (!isConnected) {
			System.out.println("\u2026Waiting for connection");
			isConnected = dialUp(199);
			TimeUnit.SECONDS.sleep(2);
			itr++;
		}
		if(isConnected) System.out.println("Connection opened successfully");
		System.out.println("Creating a messge for ping testing\u2026");
		TimeUnit.SECONDS.sleep(2);
		createMessage().createMessage("hi, from user", "raisehand.io");;
		TimeUnit.SECONDS.sleep(3);
		System.out.println("Ping test passed\u2026");
		System.out.println("Checking new messages in inbox\u2026");
		TimeUnit.SECONDS.sleep(2);
		checkMessage();
		closeConnection();
		System.out.println("All done...");
	}

	@Override
	public void checkMessage() {
		System.out.println("No new messages!!!");
	}

	@Override
	public IMessenger<String, String> createMessage() {
		return new MessangerUI<String, String>();
	}

}

public class Main {

	public static void main(String[] args)throws InterruptedException {
		Web g = new Web();
		g.init();
	}
	
}

```
> output

```
Bypassing the connection using vpn, with route: 188 SSL: true
…Waiting for connection
…Waiting for connection
…Waiting for connection
…Waiting for connection
…Waiting for connection
…Waiting for connection
Connection opened successfully
Creating a messge for ping testing…
to: raisehand.io, message: hi, from user
Ping test passed…
Checking new messages in inbox…
No new messages!!!
All done...
```

##### Example 2 (Proxy Design Pattern)

> ISecureNetworkProxy.java

```java

interface ISecureNetworkProxy {
	void connect(String url) throws InterruptedException;
}

```

> SimpleNetwork.java

```java

class SimpleNetwork implements ISecureNetworkProxy {
	private boolean isConnected;
	private long port = 80;

	void disconnect() {
		if (isConnected)
			isConnected = false;
		System.out.println("Disconnected\u2026");
	}
	
	public void setPort(long port) {
		this.port = port;
	}

	@Override
	public void connect(String url) throws InterruptedException {
		isConnected = true;
		System.out.println("Please wait while we are connecting to " + url + " via\u2026 port:" + port);
		TimeUnit.SECONDS.sleep(1);
		if(port == 80)
		{
			disconnect();
			return;
		}
		System.out.println("You are connected...");
	}

}

```

> ProxyNetwork.java

```java

class ProxyNetwork implements ISecureNetworkProxy {

	SimpleNetwork network = new SimpleNetwork();
	// Ports ports;
	// NetworkHelpers helpers
	// More class instances here as composition
	static List<Long> availablePorts;

	static {
		availablePorts = new ArrayList<>();
		availablePorts.add(443L);
		availablePorts.add(5778L);
	}

	@Override
	public void connect(String url) throws InterruptedException {
		// it will help to find secure port within availble ports, 
		// all though end user can call the methods manually but they have to perform these operation on their own...
		if (availablePorts.contains(443L)) {
			network.setPort(443L);
			network.connect(url);
		} else
			System.out.println("No secure port available as of now\u2026");
	}

}

```

> Main.java

```java

public class Main {

	public static void main(String[] args) throws InterruptedException {

		ISecureNetworkProxy newtorkIO = new SimpleNetwork();
		newtorkIO.connect("www.google.com");
		
		ISecureNetworkProxy networkProxy = new ProxyNetwork();
		networkProxy.connect("www.google.com");
		
	}
	
}

```
