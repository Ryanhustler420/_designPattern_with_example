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
