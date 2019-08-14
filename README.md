# _designPattern_with_example
NOTE: this is not a bible. In this repo each design pattern has different example. so that we can better understand the design and best use cases.

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

