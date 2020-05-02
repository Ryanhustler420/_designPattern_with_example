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
