<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Java 8](#java-8)
  - [Default Methods for Interfaces](#default-methods-for-interfaces)
    - [What?](#what)
    - [How?](#how)
    - [Why?](#why)
  - [Lambda Expressions](#lambda-expressions)
    - [What?](#what-1)
    - [How?](#how-1)
      - [Using invokedynamic](#using-invokedynamic)
    - [Why?](#why-1)
  - [Streams](#streams)
    - [What?](#what-2)
      - [Streams vs Collections](#streams-vs-collections)
      - [Parallel Streams](#parallel-streams)
    - [How?](#how-2)
      - [Internal representation of a Stream](#internal-representation-of-a-stream)
      - [Internal representation of a Stream pipeline](#internal-representation-of-a-stream-pipeline)
      - [Internals of a Stream execution](#internals-of-a-stream-execution)
    - [Why?](#why-2)
      - [Collectors "utility" class](#collectors-utility-class)
      - [Collector Abstraction](#collector-abstraction)
      - [Collector interface](#collector-interface)
      - [Custom collector](#custom-collector)
      - [Parallelism](#parallelism)
      - [Using Parallelism carefully](#using-parallelism-carefully)
      - [Amdahl's law](#amdahls-law)
      - [NQ Model](#nq-model)
    - [Common Usecases](#common-usecases)
      - [1. Converting List Objects to Maps](#1-converting-list-objects-to-maps)
      - [2. Handling of Exceptions](#2-handling-of-exceptions)
      - [3. Converting Optional to a Stream](#3-converting-optional-to-a-stream)
      - [4. Serializing Lambda](#4-serializing-lambda)
      - [5. Split Stream](#5-split-stream)
      - [6. Performance impact of lambdas & streams](#6-performance-impact-of-lambdas--streams)
      - [7. Stream operations on Lists](#7-stream-operations-on-lists)
      - [8. Modify a non-Final variable in lambda](#8-modify-a-non-final-variable-in-lambda)
      -	[9. flatMap vs Map](#9-flatmap-vs-map)
  - [Java Type Annotations](#java-type-annotations)
    - [Overview of Built-in Annotations in JDK < 1.7](#overview-of-built-in-annotations-in-jdk--17)
    - [What?](#what-3)
    - [How?](#how-3)
    - [Why?](#why-3)
  - [Generalized Target Type Inference](#generalized-target-type-inference)
    - [What?](#what-4)
    - [How?](#how-4)
    - [Why?](#why-4)
  - [Optional and Value Types](#optional-and-value-types)
    - [About Objects Identity and Values](#about-objects-identity-and-values)
      - [Why think about identity?](#why-think-about-identity)
    - [Value Types and Value Based Classes](#value-types-and-value-based-classes)
  - [Optional](#optional)
    - [What?](#what-5)
    - [Why?](#why-5)
    - [How?](#how-5)
      - [Short story on Monads](#short-story-on-monads)
      - [Back to Optional](#back-to-optional)
      - [Closures](#closures)
  - [Compact Profiles](#compact-profiles)
  - [Date and Time API](#date-and-time-api)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Java 8



## Default Methods for Interfaces

### What?

Java 8 enables us to add non-abstract method implementations to interfaces by utilizing the `default` keyword. This feature is also known as [virtual extension methods](http://stackoverflow.com/a/24102730).

### How?

A default method is a method declaration and implementation, and it is denoted with the default keyword preceding the declaration.
Any class that extends an interface containing a default method can choose to implement and override the default implementation or simply use the default implementation.

### Why?

Why not simply use abstract classes, rather than default methods? Default methods were added in order to provide a way to change interfaces without breaking backward compatibility. Many interfaces needed to be changed to accommodate Java 8 API updates, and in doing so, these interfaces would possibly break functionality of code that was written for prior releases of Java. 

By adding default methods to interfaces, engineers were able to add new functionality to existing interfaces without breaking the code that relied on those interfaces.

For e.g. In Map Interface there are quite some default methods added like 

> V java.util.Map.putIfAbsent(K key, V value)
>
> If the specified key is not already associated with a value (or is mapped to null) associates it with the given value and returns null, else returns the current value.



**Static method** in interface:

- You can call it directly (InterfacetA.staticMethod())

- Sub-class will not be able to override.
- Sub-class may have method with same name as staticMethod
- static method can be invoked within other *static* and *default* methods.

**Default method** in interface:

- You can not call it directly.
- Sub-class will be able to override it

Advantage:

- **static Method:** You don't need to create separate class for utility method.
- **default Method:**Provide the common funtionality in default method.



Notes:

- If 2 or more interfaces are implemented that contains default methods of same signature then the code will not compile. Solutions are
  - override in the base class and provide your own implementation
  - override and call a specific super implementation

- It's legal to call the following from a default method

  - other default method
  - other abstract methods
  - other static methods

- Implementation precedence is in the following order when there is a method name collision: 

  - instance methods > abstract class method > default methods 


One approach would be to define default public methods that utilize private static utility methods in the interface.

## Lambda Expressions

### What?

Lambda expressions provide a clear and concise way of implementing a single-method interface using an
expression. They allow you to reduce the amount of code you have to create and maintain. While similar to
anonymous classes, they have no type information by themselves. Type inference needs to happen.

### How?

Lambdas can only operate on a functional interface, which is an interface with just one abstract method. **Functional interfaces** can have any number of default or static methods. (For this reason, they are sometimes referred to as *Single Abstract Method Interfaces*, or *SAM Interfaces*).

When declaring a functional interface the `@FunctionalInterface` annotation can be added. This has no special effect, but a compiler error will be generated if this annotation is applied to an interface which is not functional, thus acting as a reminder that the interface should not be changed.

```java
@FunctionalInterface 
interface Foo5 {
	void bar(); 
}
@FunctionalInterface
interface BlankFoo1 extends Foo5 { 
    // inherits abstract method from Foo5 
}
@FunctionalInterface 
interface Foo6 {
	void bar();
	boolean equals(Object obj); 
    // overrides one of Object's method so not counted 
}
```



Structure of Lambda expression is: 

​			(Method signature) -> Method Implementation

But does that mean we can define and invoke any piece of code as we please

```java
public static void main(String[] args){
    methodA = () -> System.out.println("hello"); //invalid
    methodA();
}
```

definitely yes but not like this.

All we need to do is define a generic funtional interface and use it.

```java
@FunctionalInterface
interface SingleLiner {
	void doSomething();
}

public static void main(String[] args) {
		
		SingleLiner singleLineCode = () -> System.out.println("hello");
		singleLineCode.doSomething();
}
```

`singleLineCode` will then hold a singleton instance of class, similar to an anonymous class but not the same.

Notes:

- The lambda is only "mostly equivalent" to the anonymous class because in a lambda, 

  - the meaning of `this`, `super` or `toString()` reference the class within which the assignment takes place, not the newly object.
  - there are no anonymous class code generated

- You cannot specify the name of the method in a lambda; and you need not since these are SAM interfaces.

- Types of method arguments are infered 

- Lambda's could be created from inherited interfaces but as long as the child interfaces retain the SAM principle.

  ```java
  interface ComplexLiner extends SingleLiner{
  
      void doSomething(); //this line is ambiguous
      default void doMagic() {
          System.out.println("Magic works");
      }
      //void doSomethingElse();//wrong
  }
  
  ComplexLiner complexLine = () -> System.out.println("hello");
  
  complexLine.doSomething();
  complexLine.doMagic();
  ```

- If the code placed inside a lambda is a java expression rather than a statement then the return keyword can be omitted and it implicitely means that it returns a value.

```java
@FunctionalInterface
interface SingleLiner {
	int doSomething(int n);
}
SingleLiner singleLineCode = (x) -> (x+1); //same as { return (x+1); } 
singleLineCode.doSomething(3);
```

- follow the same rules for accessing local variables in the enclosing scope; the variables must be treated as final (implicitly final) and not modified inside the lambda.


*Are Java 8 lambda expressions just the syntactic sugar over anonymous inner classes, or how do functional interfaces otherwise get translated to bytecode?*

The short answer is **NO**. Java 8 does not use anonymous inner classes mainly for two reasons:

1. **Performance impact**: If lambda expressions were implemented using anonymous classes, then each lambda expression would result in a class file on disk. If these classes were loaded by the JVM at startup, then the startup time of the JVM would increase, as all the classes would need to be first loaded and verified before use.
2. **Possibility to change in future**: If Java 8 designers would have used anonymous classes from the start, then it would have limited the scope of future lambda implementation changes.

#### Using invokedynamic

Java 8 designers decided to use the `invokedynamic` instruction, added in Java 7, to defer the translation strategy at runtime. When `javac` compiles the code, it captures the lambda expression and generates an `invokedynamic` call site (called lambda factory). The `invokedynamic` call site, when invoked, returns an instance of the functional interface to which the lambda is being converted. 

I have only glossed over this topic. You can read about the internals at <http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html>.

### Why?

The following are the advantages of lambdas which are explained using examples

1. Already existing interfaces became smarter

   ```java
   Runnable r1 = new Runnable() {
       @Override
       public void run() {
           System.out.println("My Runnable");
       }
   };
   
   Runnable r2 = () -> System.out.println("My Runnable");
   ```


2. Reduced Lines of code

3. Predefined Functional Interfaces from the JDK for most of the common use cases.

   Remember I had to write an interface called SingleLiner above just to get that piece of lambda code working.

   JDK includes the following classes for most of the common scenarios:

   ```java
   Consumer<T> 
   Desc: (T) -> void
   Methods: 	
   void accept(T t)
   default Consumer<T>	andThen(Consumer<? super T> after)
   
   BiConsumer<T,U>
   Desc: (T, U) -> void
   Methods:
   void accept(T t, U u)
   default BiConsumer<T,U>	andThen(BiConsumer<? super T,? super U> after)
   
   Supplier<T>
   Desc: () -> T
   Methods: 
   T get()
   
   Function<T,R>
   Desc: (T) -> R
   Methods: 
   R apply(T t)
   default <V> Function<V,R>	compose(Function<? super V,? extends T> before)
   default <V> Function<T,V>	andThen(Function<? super R,? extends V> after)
   static <T> Function<T,T>	identity()
   
   BiFunction<T, U, R>
   Desc: (T, U) -> R
   Methods: 
   R apply(T t, U u)
   default <V> BiFunction<T,U,V>	andThen(Function<? super R,? extends V> after)
   
   Predicate<T>
   Desc: (T) -> boolean
   Methods:
   boolean	test(T t)
   default Predicate<T>	and(Predicate<? super T> other)
   default Predicate<T>	or(Predicate<? super T> other)
   static <T> Predicate<T>	isEqual(Object targetRef)
   default Predicate<T>	negate()
   
   BiPredicate<T, U>
   Desc: (T, U) -> boolean
   Methods:
   boolean	test(T t, U u)
   default BiPredicate<T,U>	and(BiPredicate<? super T,? super U> other)
   default BiPredicate<T,U>	or(BiPredicate<? super T,? super U> other)
   default BiPredicate<T,U>	negate()
   
   UnaryOperator<T> extends Function<T,T>
   Desc: (T) -> T
   Methods:
   static <T> UnaryOperator<T>	identity()
   
   BinaryOperator<T> extends BiFunction<T,T,T>
   Method: (T, T) -> T
   static <T> BinaryOperator<T>	maxBy(Comparator<? super T> comparator)
   static <T> BinaryOperator<T>	minBy(Comparator<? super T> comparator)
   ```

   There are many more. Check out the intefaces in package: https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html

   Thus this ready to use interfaces helps, for eg in the previous example of doing x+1 computation can be done without a interface definition from the developer.

   ```java
   Function<Integer, Integer> lambda = (x) ->  (x+1);
   System.out.println(lambda.apply(3));
   ```

4. Advanced loops - true internal interators

   ```java
   List<Integer> numbers = Arrays.asList(2,3,4,5);
   		
   for(int i=0; i<numbers.size(); i++){
       System.out.println(numbers.get(i));
   }
   		
   	
   //External Iterators (or Active Iterators)
   //responsibility of iterating over elements lies with the programmer
   
   //In java 1.4 and below we have Enumerations and Iterators
   
   Iterator<Integer> iterator = numbers.iterator();
   while(iterator.hasNext()) {
       System.out.println(iterator.next());
   }
   
   //In java 1.5 added
   //enhanced for-loop (behind the scenes its external iterator)
   for(Integer number: numbers) {
       System.out.println(number);
   }
   
   //Internal Iterators(or Passive Iterators)  
   //Internal Iterators manage the iterations in the background. 
   //This leaves the programmer to just declaratively code what is meant to be done with the elements of the Collection,
   //Advantage of Internal Iterators is that there are method calls and hence it gives way to polymorphism
   
   
   
   /*
   	Collection interface has Iterable as its super interface
   	and it added a new API:
   			void forEach(Consumer<? super T> action)
   	Simply put, the Javadoc of forEach stats that it “performs the given action for each element of the Iterable until all elements have been processed or the action throws an exception.” 
   
   you specify that action by providing a class that implements Consumer interface
   
   		@FunctionalInterface
   		public interface Consumer {
   		    void accept(T t);
   		}
   
   		*/
   
   numbers.forEach(new Consumer<Integer>() {
       public void accept(Integer number) {
           System.out.println(number);
       }
   });
   
   //using lambda expressions
   
   numbers.forEach((number) -> System.out.println(number));
   
   //more syntactic sugar
   
   numbers.forEach(System.out::println);
   
   //using streams for more control (will see later)
   
   numbers.stream().forEach(System.out::println);
   ```

   

   Using streams/parallel streams are not necessarily faster always: https://jaxenter.com/java-performance-tutorial-how-fast-are-the-java-8-streams-118830.html

   Note: Structural modification of list is still not allowed

   Remember structural modification is any operation that adds or deletes one or more elements,  or explicitly resizes the backing array; merely setting the value of an element is not a structural modification.

   ```java
   for(int i=0; i<numbers.size(); i++){
   	Integer x = numbers.get(i)-1;
   	numbers.set(i, x);
   	//numbers.remove(i); //Unsupported Operation Error
   }
   System.out.println(numbers);
   ```


   using enhanced for-loop i.e, for-Each construct still doesn't allow you to modify the contents of the list because Integer is immutable so modifying it creates a new one which needs to be assigned back into the list which is not possible here.

   So is the case with lambdas too.

```java
for(Integer number: numbers){
	number = number-1;	//these changes are lost
}

//lamdbas provide you with immutables and hence the changes are lost
numbers.forEach(number-> { number = number.intValue()-1; });
```

​	The only approach that works is through Iterators. 

```java
Iterator<Integer> it = numbers.iterator();
	while (it.hasNext()) {
		Integer integer = it.next();
		if (integer < 3) {
		    it.remove();
		 }
	}
```

​	The only down-side of this approach is that you need to switch your for-each to a while. However, this approach is the most efficient one, especially for LinkedList where it is O(n) (it's O(n2) for ArrayList because it has to copy array data on each remove(index) call). 

Ref: https://stackoverflow.com/questions/6101789/why-does-javas-arraylists-remove-function-seem-to-cost-so-little

5. Passing behaviours into methods

Let's consider a scenario of a client writing a program to do some complex mathematical operation on a set of Integers but based on some condition.

Let's say that the client doesn't have the time nor the skill to invest in doing the operation so he get's it from a Vendor.

For simplicity let's assume that the mathematical operation is addition.



  - Level 1

  Client code -> creates a list of integers, filters it based on a condition, sends them to Vendor for calculation

  ```java
  //client creates list of Ints
  List<Integer> listOfInts = new ArrayList<Integer>();
  listOfInts.add(2);
  listOfInts.add(3);
  listOfInts.add(4);
  
  //client does some logic
  //let's say he wants to filter out even numbers
  Iterator<Integer> itr = listOfInts.iterator();
  while (itr.hasNext()) {
    Integer i = itr.next();
  	if(i%2==0)
  	  itr.remove();
  }
  
  //Client calls Vendor provided function
  int result = MathUtility.sum(listOfInts);
  ```

  Vendor code -> does the mathematical calculation for the given list of integers and sends the result back

  ```java
  //In the provided Vendor's MathUtility class
  class MathUtility {
  	public static int sum(List<Integer> list){
  		//ability to add
  		int sum=0;
  		for(Integer i: list){
  			sum=sum+i;
  		}
  		return sum;
  }
  ```



  - Level 2

  A better way would be if they collaborate and agree on a contract. In the Java world, contract is through interfaces.

  Common code -> the interface

  ```java
  interface ConditionCheck {
  	public boolean check(Integer i);
  }
  
  ```

  Vendor code

  ```java
  class MathUtility {
  	public static int sum(List<Integer> list, ConditionCheck condition){
  		//ability to add is only here
  		int sum=0;
  		for(Integer i: list){
  			if(condition.check(i)){
  				sum=sum+i;
  			}
  		}
  		return sum;
  }
  ```

  Client code

  ```java
  //client creates reusuable objects that will do the filtering 
  ConditionCheck isEven = new ConditionCheck<Integer>(){
  	@Override
  	public boolean check(Integer i) {
  		return i%2==0;
  	}	
  };
  
  //client calls Vendor provided function
  int result = MathUtility.sum(listOfInts, isEven);
  
  //client feels like creating another filter and he can do it easily
  final List<Integer> special = Arrays.asList(new Integer[]{2,3});
  ConditionCheck isSpecial = new ConditionCheck(){
  	@Override
  	public boolean check(Integer i) {
  		return special.contains(i);
  	}
  };
  
  int result = MathUtility.sum(listOfInts, isSpecial);
  ```



  - Level 3

  Let's say the Vendor now get's requests to handle different types of numerics.

  Vendor code

  ```java
//Generics into the mix :)  
class MathUtility<T extends Number> {
  	public static <T extends Number> Number sum(List<T> list, ConditionCheck<T> condition){
  		//ability to add is only here
  		
  		Number sum = 0;
  		for(T i: list){
  			if(condition.check(i)){
  				if (i instanceof Integer) {
  					sum=sum.intValue()+i.intValue();
  				}
  				if(i instanceof Double) {
  					sum=sum.doubleValue()+i.doubleValue();
  				}
  				//so on
  			}
  		}
  		return sum;
  	}
  }
  ```

  Client code

  ```java
  ConditionCheck isEven = new ConditionCheck<Integer>(){
  	@Override
  	public boolean check(Integer i) {
  		return i%2==0;
  	}	
  };
  
  int result = MathUtility.sum(listOfInts, isBig))
  ```



  - Level 4

  Final improvments using lambdas.

  Client code (no changes to Vendor code)

  ```java
  //in client just this line of code ;)
  
  int result = MathUtility.sum(listOfInts, (Integer i)->{return (i<3);});
  
  //still shorter way of doing it
  
  int result = MathUtility.sum(listOfInts, i ->i<3);
  ```



6. Sorting becomes cleaner

Consider another example.

Pre java 8 style

```java
List<Person> people = ...
Collections.sort(
    people,
    new Comparator<Person>() {
        public int compare(Person p1, Person p2){
            return p1.getFirstName().compareTo(p2.getFirstName());
        } 
  	}
);
```

Starting with Java 8, the anonymous class can be replaced with a lambda expression.

```java
Collections.sort(
    people,
    (p1, p2) -> p1.getFirstName().compareTo(p2.getFirstName())
);
```

Further simplified using Comparator.comparing.

```java
Collections.sort(
    people,
    Comparator.comparing(Person::getFirstName)
);
```



7. Method references

Instance method reference (to an arbitrary instance) 

```java
people.stream().map(Person::getName) 
//The equivalent lambda: 
people.stream().map(person -> person.getName())
```

 Instance method reference (to a specific instance) 

```java
people.forEach(System.out::println);
//The equivalent lambda: 
people.forEach(person -> System.out.println(person)); 
```

Static method reference

```java
numbers.stream().map(String::valueOf)
//The equivalent lambda:
numbers.stream().map(num -> String.valueOf(num))
```

Reference to constructor

```java
strings.stream().map(Integer::new)
//The equivalent lambda:
strings.stream().map(s -> new Integer(s));
```



8. Type of Lambda Expression

A lambda expression, by itself, does not have a specific type. The lambda receives a type when it is assigned to a functional interface. 

To illustrate, consider the lambda expression` o -> o.isEmpty()`. The same lambda expression can be assigned to many different functional types:

```java
Predicate<String> javaStringPred = o -> o.isEmpty();
Function<String, Boolean> javaFunc = o -> o.isEmpty();
Predicate<List> javaListPred = o -> o.isEmpty();
Consumer<String> javaStringConsumer = o -> o.isEmpty(); //return value is ignored
com.google.common.base.Predicate<String> guavaPredicate = o -> o.isEmpty();
```



9. Implementing multiple interfaces

Sometimes you may want to have a lambda expression implementing more than one interface.

For example, you want to create a TreeSet with a custom Comparator and then serialize it and send it over the network. The trivial approach: 

`TreeSet<Long> ts = new TreeSet<>((x, y) -> Long.compare(y, x));`
 doesn't work since the lambda for the comparator does not implement Serializable. You can fix this by using intersection types and explicitly specifying that this lambda needs to be serializable: 

`TreeSet<Long> ts = new TreeSet<>((Comparator<Long> & Serializable) (x, y) -> Long.compare(y, x));`

If you haven't heard of intersection types in java: http://iteratrlearning.com/java/generics/2016/05/12/intersection-types-java-generics.html

10. Lamdba Closures

A lambda closure is created when a lambda expression references the variables of an enclosing scope (global or local). The rules for doing this are the same as those for inline methods and anonymous classes.

Local variables from an enclosing scope that are used within a lambda have to be final or effectively final.

```java
int n=0; 
Runnable r3 = () -> {
	//n = n+1; //compile error
	//Local variable n defined in an enclosing scope must be final/effectively final
	System.out.println("My Runnable"+ n);
};

r3.run();
```

What happens if I use Integer instead of int?

The behaviour will be same as above as reassigning the original reference (to Integer) is not allowed.

The below is also not feasible.

```java
Integer n=0; 
Runnable r3 = () -> {
	int i = n.intValue();
	i++;
	//n.setValue(i); //not feasible; wrapper classes are immutable
	System.out.println("My Runnable"+ n);
};

r3.run();
```

One solution is to use a copy variable.

```java
int n=0; 
Runnable r3 = () -> {
	int i =n;
	i = i+1;
	System.out.println("My Runnable"+ i);
};

r3.run();
```

Another solution is to use a state variable.

```java
//legal but not safe
Obj obj = new Obj();
obj.n=0;
Runnable r3 = () -> {
	Obj temp = obj;
	temp.n = temp.n+1;
	System.out.println("My Runnable"+ temp.n);
};

r3.run();
```

The above works as the object reference is not touched within the lambda scope.

```java
// Same as above --> legal, but not safe
List<Integer> evenLengths = new ArrayList<>();
evenLengths.stream()
    .filter(s -> s.length() % 2 == 0)
    .forEach(evenLengths::add);
System.out.println("Using add: " + evenLengths);
```

Another solution is if we are using instance or static variables - it works.

```java
// Does not compile ...
public IntUnaryOperator createAccumulator() { 
    int value = 0;
	IntUnaryOperator accumulate = (x) -> { value += x; return value; };
	return accumulate;
}
```

But,

```java
// Compiles, but is incorrect ...
public class AccumulatorGenerator { 
    private int value = 0;
	public IntUnaryOperator createAccumulator() {
	IntUnaryOperator accumulate = (x) -> { value += x; return value; }; return accumulate;
	} 
}
```



Why does it work and why isn't it not a good practice?

To understand, we need to see why the language doesn't allow non-final local variables.

The local variable is **copied ** when JVM creates a lambda instance. Since it's a copy that the lambda is working on (and there might be multiple threads running the same lambda code), there is no way to tell that lambda expression body is not working on a stale copy of the variable unless it is kept final or effectively final.

Now, in case of instance fields, when you access an instance field inside the lambda expression then compiler will append a `this` to that variable access (if you have not done it explicitly) and since `this` is effectively final so compiler is sure that lambda expression body will always have the latest copy of the variable (please note that multi-threading is out of scope right now for this discussion). 

Note that the case of `this` and also `obj`, the JVM is creating a copy just like in local variable case but it's just that the copy is the object reference.

If the *synchronization* required in **instance mutation**, you can use directly the [stream reduction methods](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) or if there is dependency issue in instance mutation, you still can use `thenApply` or `thenCompose` in [Function](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html) while `mapping` or methods similar.

More: https://stackoverflow.com/questions/25055392/lambdas-local-variables-need-final-instance-variables-dont

Better approach would be to use regular classes when playing with mutable states.

```java
// Correct ...
public class Accumulator { 
    private int value = 0;
	public int accumulate(int x) { value += x;
		return value; 
    }
}
```



11. Lambdas and memory utilization

    Be aware that some lambdas are stateless eg. `Consumer c = (s)->s.length()` where some are stateful

    eg. `Function f = (t) -> t+1;`

    Another example of stateful.

```java
for(int i=0; i<1,000,000,000; i++) {
	int j=i;
	Supplier<Integer> comsumer = () -> j;
}
```

Although it might not be immediately obvious since the new keyword doesn't appear anywhere in the snippet, this code is liable to create 1,000,000,000 separate objects to represent the instances of the `() 
-> j` lambda expression. 



## Streams

### What?

A `java.util.Stream` represents a sequence of elements on which one or more operations can be performed. 

All stream computations share a common structure: They have a 

1. stream source,
2. zero or more intermediate operations which are lazy,
3. and a single terminal operation. 

While terminal operations return a result of a certain type, intermediate operations return the stream itself so you can chain multiple method calls in a row. Streams are created on a source, e.g. a `java.util.Collection` like lists or sets (maps are not supported). 

Stream operations can either be executed sequentially or parallely.



Sample Stream sources are: 

| Method                | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `Collection.stream()` | Create a stream from the elements of a collection.           |
| `Stream.of(T...)`     | Create a stream from the arguments passed to the factory method. |
| `Stream.of(T[])`      | Create a stream from the elements of an array.               |
| `Stream.empty()`      | Create an empty stream.                                      |



Sample Intermediate stream operations:

| Operation                        | Contents                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| `filter(Predicate<T>)`           | The elements of the stream matching the predicate            |
| `map(Function<T, U>)`            | The result of applying the provided function to the elements of the stream |
| `flatMap(Function<T, Stream<U>>` | The elements of the streams resulting from applying the provided stream-bearing function to the elements of the stream |
| `distinct()`                     | The elements of the stream, with duplicates removed          |
| `sorted()`                       | The elements of the stream, sorted in natural order          |
| `Sorted(Comparator<T>)`          | The elements of the stream, sorted by the provided comparator |
| `limit(long)`                    | The elements of the stream, truncated to the provided length |
| `skip(long)`                     | The elements of the stream, discarding the first N elements  |
| `takeWhile(Predicate<T>)`        | (Java 9 only) The elements of the stream, truncated at the first element for which the provided predicate is not `true` |
| `dropWhile(Predicate<T>)`        | (Java 9 only) The elements of the stream, discarding the initial segment of elements for which the provided predicate is `true` |



Sample Terminal stream operations:

| Operation                           | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `forEach(Consumer<T> action)`       | Apply the provided action to each element of the stream.     |
| `toArray()`                         | Create an array from the elements of the stream.             |
| `reduce(...)`                       | Aggregate the elements of the stream into a summary value.   |
| `collect(...)`                      | Aggregate the elements of the stream into a summary result container. |
| `min(Comparator<T>)`                | Return the minimal element of the stream according to the comparator. |
| `max(Comparator<T>)`                | Return the maximal element of the stream according to the comparator. |
| `count()`                           | Return the size of the stream.                               |
| `{any,all,none}match(Predicate<T>)` | Return whether any/all/none of the elements of the stream match the provided predicate. |
| `findFirst()`                       | Return the first element of the stream, if present.          |
| `findAny()`                         | Return any element of the stream, if present.                |



#### Streams vs Collections

 A collection is a data structure; its main concern is the organization of data in memory, and a collection persists over a period of time. A collection might often be used as the source or target for a stream pipeline, but a stream's focus is on **computation**, not data.

Operations on collections are eager and mutative; For streams, only the terminal operation is **eager**; the others are **lazy**.

**Processing Order**

If the stream is sequential (not parallel) then the ordering is based on the ordering of the source. For eg. In case of SortedMap or ArrayList but in case of HashMap.keySet() the ordering is not garanteed.

#### Parallel Streams

If the stream is parallel then it utilizes multiple threads on multiple cores and hence the order is never guaranteed.



```java
List<Integer> integerList = Arrays.asList(0, 1, 2, 3, 42);

// sequential

long howManyOddNumbers 
	= integerList.stream().filter(e -> (e % 2) == 1).count(); 

System.out.println(howManyOddNumbers); // Output: 2

// parallel 

long howManyOddNumbersParallel 
	= integerList.parallelStream() .filter(e -> (e % 2) == 1).count(); 

System.out.println(howManyOddNumbersParallel); // Output: 2 
```

Note that if you are using a reduction then parallel can be effectively utilized else if your printing or re-utilizing the stream then keep in mind that the ordering will vary.



### How?

Streams are neither a collection of OBJECTS nor a sequence of OBJECTS.
It is an abstraction that represents 0 or more VALUES.

Any given Stream can potentially have an unlimited amount of data flowing through it. As a result, data received from a Stream is processed individually as it arrives, as opposed to performing batch processing on the data altogether. When combined with lambda expressions they provide a concise way to perform operations on sequences of data using a functional approach.

```java
Stream<String> fruitStream = Stream.of("apple", "banana", "pear", "kiwi", "orange");
fruitStream.filter(s -> s.contains("a"))
           .map(String::toUpperCase)
.sorted() .forEach(System.out::println);
//APPLE BANANA ORANGE PEAR
```



A *stream pipeline* is composed of a *stream source*, zero or more *intermediate operations*, and a *terminal operation*. Stream sources can be collections, arrays, generator functions, or any other data source that can suitably provide access to its elements.



#### Internal representation of a Stream

A stream source is described by an abstraction called `Spliterator`. As its name suggests, `Spliterator` combines two behaviors: accessing the elements of the source (iterating), and possibly decomposing the input source for parallel execution (splitting).

Although `Spliterator` includes the same basic behaviors as `Iterator`, it doesn't extend `Iterator`, instead taking a different approach to element access. An `Iterator` has two methods, `hasNext()` and `next()`; accessing the next element can involve (but doesn't require) calling both of these methods. As a result, coding an `Iterator` correctly requires a certain amount of defensive and duplicative coding. (What if the client doesn't call `hasNext()` before `next()`? What if it calls `hasNext()` twice?) Additionally, the two-method protocol generally requires a fair amount of statefulness, such as peeking ahead one element (and keeping track of whether you've already peeked ahead). Together, these requirements add up to a fair degree of per-element access overhead.

Having lambdas in the language enables `Spliterator` to take an approach to element access that's generally more efficient — and easier to code correctly. `Spliterator` has two methods for accessing elements:

```java
boolean tryAdvance(Consumer<? super T> action); 
void forEachRemaining(Consumer<? super T> action);
```

The `tryAdvance()` method tries to process a single element. If no elements remain, `tryAdvance()` merely returns `false`; otherwise, it advances the cursor and passes the current element to the provided handler and returns `true`. The`forEachRemaining()` method processes all the remaining elements, passing them one at a time to the provided handler.

Even ignoring the possibility of parallel decomposition, the `Spliterator` abstraction is already a "better iterator" — simpler to write, simpler to use, and generally having lower per-element access overhead. But the `Spliterator` abstraction also extends to parallel decomposition. 

It comes with one more method,

```java
Spliterator<T> trySplit();
```

The behavior of `trySplit()` is to try to split the remaining elements into two sections, ideally of similar size. 

For sources whose encounter orders significant (for example, arrays, `List`, or `SortedSet`), `trySplit()` must preserve this order; it must split off the initial portion of the remaining elements into the new `Spliterator`, and the current spliterator must describe the remaining elements in an order consistent with the original ordering.



#### Internal representation of a Stream pipeline

A stream pipeline is built by constructing a linked-list representation of the stream source and its intermediate operations. In the internal representation, each stage of the pipeline is described by a bitmap of *stream flags* that describe what's known about the elements at this stage of the stream pipeline.

| Stream flag  | Interpretation                                               |
| ------------ | ------------------------------------------------------------ |
| `SIZED`      | The size of the stream is known.                             |
| `DISTINCT`   | The elements of the stream are distinct, according to `Object.equals()` for object streams, or according to `==`for primitive streams. |
| `SORTED`     | The elements of the stream are sorted in the natural order.  |
| `ORDERED`    | The stream has a meaningful encounter order                  |
| `SUBSIZED`   | we split the instance using a *trySplit()* method and obtain Spliterators that are *SIZED* as well |
| `CONCURRENT` | source can be safely modified concurrently                   |
| `IMMUTABLE`  | if elements held by source can’t be structurally modified    |
| `NONNULL`    | if source holds nulls or not                                 |

The stream flags for the source stage are derived from the `characteristics` bitmap of the spliterator (spliterators support a larger set of flags than do streams). A high-quality spliterator implementation not only provides efficient element access and splitting but also describes the characteristics of the elements. (For example, the spliterator for a `HashSet` reports the `DISTINCT` characteristic, since the elements of a `Set` are known to be distinct.)

Each intermediate operation has a known effect on the stream flags; an operation can set, clear, or preserve the setting for each flag. For example, the `filter()` operation preserves the `SORTED` and `DISTINCT` flags but clears the `SIZED` flag; the `map()` operation clears the `SORTED` and `DISTINCT` flags but preserves the `SIZED` flag; and the `sorted()` operation preserves the `SIZED` and `DISTINCT` flags and injects the `SORTED` flag. As the linked-list representation of stages is constructed, the flags for the previous stage are combined with the behavior of the current stage to arrive at a new set of flags for the current stage.

In some cases, the flags make it possible to omit an operation entirely, as in the stream pipeline below

```java
TreeSet<String> ts = ...
String[] sortedAWords = ts.stream()
                          .filter(s -> s.startsWith("a"))
                          .sorted()
                          .toArray();
```



The stream flags for the source stage include `SORTED`, because the source is a `TreeSet`. The `filter()` method preserves the `SORTED` flag, so the stream flags for the filtering stage also include the `SORTED` flag. Normally, the result of the `sorted()`method would be to construct a new pipeline stage, add it to the end of the pipeline, and return the new stage. However, because it's known that the elements are already sorted in natural order, the `sorted()` method is a no-op — it just returns the previous stage (the filtering stage), since sorting would be redundant. 



#### Internals of a Stream execution

Intermediate operations are lazy and terminal operations are eager.

When the terminal operation is initiated, the stream implementation picks an execution plan. Intermediate operations are divided into 

- *stateless* (`filter()`, `map()`, `flatMap()`) operations 

- *stateful* (`sorted()`, `limit()`, `distinct()`) operations

A stateless operation is one that can be performed on an element without knowledge of any of the other elements. For example, a filtering operation only needs to examine the current element to determine whether to include or eliminate it, but a sorting operation must see all the elements before it knows which element to emit first.

- *single pass*

  If the pipeline is executing sequentially, or is executing in parallel but consists of all stateless operations, it can be computed in a single pass. 

- *multiple passes*

  Otherwise, the pipeline is divided into sections (at stateful operation boundaries) and is computed in multiple passes.

Terminal operations are either 

- *short-circuiting* (`allMatch()`, `findFirst()`) 

- *non–short-circuiting* (`reduce()`, `collect()`, `forEach()`)

If the terminal operation is non–short-circuiting, the data can be processed in bulk (using the`forEachRemaining()` method of the source spliterator, further reducing the overhead of accessing each element); if it's short-circuiting, it must be processed one element at a time (using `tryAdvance()`).

For sequential execution, Streams constructs a "machine" — a chain of `Consumer` objects whose structure matches that of the pipeline structure. Each of these `Consumer` objects knows about the next stage; when it receives an element (or is notified that there are no more elements), it sends zero or more elements to the next stage in the chain. For example, the `Consumer` associated with a `filter()` stage applies the filter predicate to the input element and either does or doesn't send it on to the next stage; the `Consumer` associated with a `map()` stage applies the mapping function to the input element and sends the result to the next stage. The `Consumer` associated with a stateful operation such as `sorted()` buffers elements until it sees the end of the input, and then it sends the sorted data to the next stage. The final stage in the machine implements the terminal operation. If this operation produces a result, such as `reduce()` or `toArray()`, this stage acts as accumulator for the result.


**Encounter order**

Another subtle consideration that influences the library's ability to optimize is *encounter order*. Encounter order refers to whether or not the order in which a source dispenses elements is significant to the computation. Some sources (such as hash-based sets and maps) have no meaningful encounter order. A stream flag, `ORDERED`, describes whether the stream has a meaningful encounter order or not. The spliterators for the JDK collections set this flag based on the specification of the collection; some intermediate operations might inject `ORDERED` (`sorted()`) or clear it (`unordered()`).

If the stream does have an encounter order, most stream operations must respect that order. For sequential executions, preserving encounter order is essentially free, because elements are naturally processed in the order in which they're encountered. Even in parallel, for many operations (stateless intermediate operations and certain terminal operations such as `reduce()`), respecting the encounter order doesn't impose any real costs. But for others (stateful intermediate operations, and terminal operations whose semantics are tied to encounter order, such as `findFirst()` or `forEachOrdered()`), the obligation to respect the encounter order in a parallel execution can be significant. If the stream has a defined encounter order, but that order isn't significant to the result, it might be possible to speed up parallel execution of pipelines containing order-sensitive operations by removing the `ORDERED` flag with the `unordered()` operation.

As an example of an operation that's sensitive to encounter order, consider `limit()`, which truncates a stream at a specified size. Implementing `limit()` in a sequential execution is trivial: Keep a counter of how many elements have been seen, and discard any elements after that. But in a parallel execution, implementing `limit()` is much more complicated; you have to keep the **first** `N` elements.

If the stream has no encounter order, the `limit()` operation is free to choose **any** `N` elements, which admits a much more efficient execution. Elements can be sent downstream as soon as they're known, without any buffering, and the only coordination needed between threads is a semaphore to ensure that the target stream length isn't exceeded.

A similar story exists for `distinct()`: If the stream has an encounter order, then for multiple equal input elements, `distinct()` must emit the **first** of them, whereas for an unordered stream, it can emit any of them — which again admits a much more efficient parallel implementation.

A similar situation arises when you aggregate with `collect()`. If you execute a `collect(groupingBy())` operation on an ordered stream, the elements corresponding to any key must be presented to the downstream collector in the order in which they appear in the input. Often, this order isn't significant to the application, and any order would do. In these cases, it might be preferable to select a *concurrent* collector (such as `groupingByConcurrent())`, which is allowed to ignore encounter order and let all threads collect directly into a shared concurrent data structure (such as `ConcurrentHashMap`) rather than having each thread collecting into its own intermediate map, and then merging the intermediate maps (which can be expensive).


### Why?


1. Operations defined on the Stream are performed because of the terminal operation.
  Without a terminal operation, the stream is not processed. 
2. Streams can not be reused. Once a terminal operation is called, the Stream object becomes unusable.
3. Stream generally does not have to be closed. It is only required to close streams that operate on IO channels. Most Stream types don't operate on resources and therefore don't require closing.
4. You can consider Streams to be the first library to take advantage of the power of lambda expressions in Java, but there's nothing magical about it (though it is tightly integrated into the core JDK libraries). Streams isn't part of the language — it's a carefully designed library that takes advantage of some newer language features.
5. Because the Streams library is orchestrating the computation, but performing the computation involves callbacks to lambdas provided by the client, what those lambda expressions can do is subject to certain constraints. 
  - Most stream operations require that the lambdas passed to them be *non-interfering* and *stateless*. Non-interfering means that they won't modify the stream source; stateless means that they won't access (read or write) any state that might change during the lifetime of the stream operation. 
  - These requirements stem in part from the fact that the stream library might, if the pipeline executes in parallel, access the data source or invoke these lambdas concurrently from multiple threads. The restrictions are needed to ensure that the computation remains correct. 
  - Certain reduction operations needs to satisfy associativity principle.

6. Accumulator antipattern (this is the 6th version; 1 to 5 are discussed above)

  - We usually write this code in loops to accumulate values into a single variable. 

  - Why is this bad? First, this style of code is difficult to parallelize. Secondly, accumulator models the computation at too low a level — at the level of individual elements, rather than on the data set as a whole.

  - Instead use Reduction or Collection.

    -  Reduction is simple, flexible, and parallelizable, and operates at a higher level of abstraction than imperative accumulation.

    - ```java
      int sum = Stream.of(ints).reduce(0, (x,y) ‑> x+y);
      ```

    - Collection is used if you want to organize the results into a data structure like a `List` or `Map`, or reduce it to more than one summary value

      ```java
      <R> collect(Supplier<R> resultSupplier,
                  BiConsumer<R, T> accumulator, 
                  BiConsumer<R, R> combiner) 
      ```

7. Use collect instead of reduce operations gives better performance in certain scenarios.

  ```java
  String concatenated = strings.stream().reduce("", String::concat);
  ```

  produces the correct result, but — because strings are immutable in Java and concatenation entails copying the whole string — it will have *O(n 2)* runtime (some strings will be copied many times). 

  ```java
  StringBuilder concat = strings.stream()
                                .collect(() ‑> new StringBuilder(),
                                 (sb, s) ‑> sb.append(s),
                                 (sb, sb2) ‑> sb.append(sb2));
  ```

  This approach uses a `StringBuilder` as the result container. The three functions passed to `collect()` use the default constructor to create an empty container, the `append(String)` method to add an element to the container, and the `append(StringBuilder)` method to merge one container into another. This code is probably better expressed using method references rather than lambdas:

  ```java
  StringBuilder concat = strings.stream()
                                .collect(StringBuilder::new,
                                         StringBuilder::append,
                                         StringBuilder::append);
  ```



- This approach can be applied to anything like,

  ```java
  Set<String> uniqueStrings = strings.stream()
                                     .collect(HashSet::new,
                                              HashSet::add,
                                              HashSet::addAll);
  ```

 
- The relationship among the three functions passed to `collect()`— creating, populating, and merging result containers — is important enough to be given its own abstraction, `Collector`, along with a corresponding simplified version of `collect()`. 

 

#### Collectors "utility" class

 In general, collectors can be divided into three broader categories:

1. Reducing and summarizing stream elements to a single value. Eg: `counting()`,`maxBy()`, ..

2. Grouping stream elements. Eg: `groupingBy()`

3. Partitioning stream elements. Eg: `partitioningBy()`

| Collector                                               |                           Behavior                           |
| :------------------------------------------------------ | :----------------------------------------------------------: |
| `toList()`                                              |              Collect the elements to a `List`.               |
| `toSet()`                                               |               Collect the elements to a `Set`.               |
| toCollection(Supplier(less-thanCollection>)             |   Collect the elements to a specific kind of `Collection`.   |
| toMap(Function(less-thanT, K>, Function(less-thanT, V>) | Collect the elements to a `Map`, transforming the elements into keys and values according to the provided mapping functions. |
| summingInt(ToIntFunction(less-thanT>)                   | Compute the sum of applying the provided `int`-valued mapping function to each element (also versions for `long` and `double`). |
| summarizingInt(ToIntFunction(less-thanT>)               | Compute the `sum`, `min`, `max`, `count`, and `average` of the results of applying the provided `int`-valued mapping function to each element (also versions for `long` and `double`). |
| `reducing()`                                            | Apply a reduction to the elements (usually used as a downstream collector, such as with `groupingBy`) (various versions). |
| partitioningBy(Predicate(less-thanT>)                   | Divide the elements into two groups: those for which the supplied predicate (function which returns a boolean) holds and those for which it doesn’t. |
| partitioningBy(Predicate(less-thanT>, Collector)        | Partition the elements, and process each partition with the specified downstream collector. |
| groupingBy(Function(less-thanT,U>)                      | Group elements into a `Map` whose keys are the provided function applied to the elements of the stream, and whose values are lists of elements that share that key. |
| groupingBy(Function(less-thanT,U>, Collector)           | Group the elements, and process the values associated with each group with the specified downstream collector. |
| minBy(BinaryOperator(less-thanT>)                       | Compute the minimal value of the elements (also `maxBy()`).  |
| mapping(Function(less-thanT,U>, Collector)              | Apply the provided mapping function to each element, and process with the specified downstream collector (usually used as a downstream collector itself, such as with `groupingBy`). |
| `joining()`                                             | Assuming elements of type `String`, join the elements into a string, possibly with a delimiter, prefix, and suffix. |
| `counting()`                                            | Compute the count of elements. (Usually used as a downstream collector |

  Grouping the collector functions together into the `Collector` abstraction is syntactically simpler, but the real benefit comes from when you start to compose collectors together — such as when you want to create complex summaries such as those created by the `groupingBy()` collector, which collects elements into a `Map` according to a key derived from the element. For example, to create a `Map` of transactions over $1,000, keyed by seller:

  ```java
  Map<Seller, List<Txn>> bigTxnsBySeller =
      txns.stream()
          .filter(t ‑> t.getAmount() > 1000)
          .collect(groupingBy(Txn::getSeller));
  ```

 Another example:

  ```java
List<String> listOfStr = new ArrayList<>();
listOfStr.add("apple");
listOfStr.add("orange");
listOfStr.add("orange");
		
Map<String, Long> wordCount = listOfStr.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

wordCount
	.entrySet()
    .forEach(System.out::println);
/*
orange=2
apple=1
*/
  ```

 The string-concatenation example can be rewritten as:

```java
String concat = strings.stream().collect(Collectors.joining());
```

The above is also a very common example of converting Lists to Maps using lambdas.



#### Collector Abstraction

We have already seen how effectively we can convert list of string to another collection using streams.

```java
Set<String> uniqueStrings = listOfStr.stream()
	.collect(HashSet::new,
			HashSet::add,
			HashSet::addAll);
```

and we also have seen that there is the `Collectors` Uility class which can be used instead of the above code.

```java
Set<String> uniqueStrings = listOfStr.stream().collect(Collectors.toSet());
```

If you check the type of uniqueStrings it will be a HashSet. 

But how to determine this -

- one way is to print the class `uniqueStrings.getClass().getName()` 

- another way is to check the source code

  ```java
  public static <T>
      Collector<T, ?, Set<T>> toSet() {
          return new CollectorImpl<>(
          	(Supplier<Set<T>>) HashSet::new, 
          	Set::add,
              (left, right) -> { left.addAll(right); return left; },
              CH_UNORDERED_ID);
      }
  ```


If you check the documentation it says nothing about the data type instead it says -

Returns a `Collector` that accumulates the input elements into a new `Set`. There are no guarantees on the type, mutability, serializability, or thread-safety of the `Set` returned; if more control over the returned `Set` is required, use [`toCollection(Supplier)`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toCollection-java.util.function.Supplier-).

```java
Supplier<Set<String>> supplier = () -> new LinkedHashSet<String>();
Set<String> uniqueStrings = listOfStr.stream().collect(Collectors.toCollection(supplier));
```

You will notice that Collectors class and in various JDK classes, it will be littered with tons of implementations of Collector interface. 



#### Collector interface

```java
public interface Collector<T, A, R> {
    /*
    T - the type of input elements to the reduction operation
	A - the mutable accumulation type of the reduction operation (often hidden as an 			implementation detail)
	R - the result type of the reduction operation
	*/
    
    // Return a function that creates a new empty result container 
    Supplier<A> supplier();

    // Return a function that incorporates an element into a container 
    BiConsumer<A, T> accumulator();

    // Return a function that merges two result containers
    BinaryOperator<A> combiner();

    //Return a function that converts the intermediate result container into the final representation
    Function<A, R> finisher();

    // Special characteristics of this collector
    Set<Characteristics> characteristics();
}
```



#### Custom collector

Let's try to build a custom collector for word count.

```java
public static void main(String[] args) {
	HashMap<String, Integer> wordCount = Stream.of("apple", "apple", "oranges").collect(new WordCountCollector());
	System.out.println(wordCount);
}

private static class WordCountCollector implements Collector<String, HashMap<String, Integer>, HashMap<String, Integer>> {

	@Override
	public Supplier<HashMap<String, Integer>> supplier() {
		return () -> {
			System.out.println("supplier call");
			return new HashMap<String, Integer>();
		};
	}

	@Override
	public BiConsumer<HashMap<String, Integer>, String> accumulator() {
		return (map, s) -> {
			System.out.println(
					"accumulator call," + " accumulator container: " + System.identityHashCode(map)
							+ " thread: " + Thread.currentThread().getName() + ", processing: " + s);
			Integer currentVal = map.get(s)==null?0:map.get(s);
			map.put(s, currentVal+1);
		};
	}

	@Override
	public BinaryOperator<HashMap<String, Integer>> combiner() {
		return (map1, map2) -> {
			System.out.println("combiner call");
			HashMap<String, Integer> map3 = new HashMap<String, Integer>();
			map3.putAll(map1);
			map3.putAll(map2);
			return map3;
		};
	}

	@Override
	public Function<HashMap<String, Integer>, HashMap<String, Integer>> finisher() {
		System.out.println("finisher call");
		return null;
	}

	@Override
	public Set<Characteristics> characteristics() {
		return EnumSet.of(Characteristics.IDENTITY_FINISH, Characteristics.UNORDERED);
	}
}
```

```pseudocode
supplier call
accumulator call, accumulator container: 1528637575 thread: main, processing: apple
accumulator call, accumulator container: 1528637575 thread: main, processing: apple
accumulator call, accumulator container: 1528637575 thread: main, processing: oranges
{apple=2, oranges=1}
```

Using `Characteristics.IDENTITY_FINISH` which directly returns the instance created in the `supplier()`instead of using the `finisher()` function for a conversion.

Let's try to modify it to build the most used word.

```java
public static void main(String[] args) {
	String mostRepeatedWord = Stream.of("apple", "apple", "oranges").collect(new TopWordCollector());
	System.out.println(mostRepeatedWord);
}

private static class TopWordCollector implements Collector<String, HashMap<String, Integer>, String> {

	@Override
	public Supplier<HashMap<String, Integer>> supplier() {
		return () -> {
			System.out.println("supplier call");
			return new HashMap<String, Integer>();
		};
	}

	@Override
	public BiConsumer<HashMap<String, Integer>, String> accumulator() {
		return (map, s) -> {
			System.out.println(
					"accumulator call," + " accumulator container: " + System.identityHashCode(map)
							+ " thread: " + Thread.currentThread().getName() + ", processing: " + s);
			Integer currentVal = map.get(s)==null?0:map.get(s);
			map.put(s, currentVal+1);
		};
	}

	@Override
	public BinaryOperator<HashMap<String, Integer>> combiner() {
		return (map1, map2) -> {
			System.out.println("combiner call");
			HashMap<String, Integer> map3 = new HashMap<String, Integer>();
			map3.putAll(map1);
			map3.putAll(map2);
			return map3;
		};
	}

	@Override
	public Function<HashMap<String, Integer>, String> finisher() {
		System.out.println("finisher call");
		return (map) -> 
				map
				.entrySet()
				.stream() //cannot apply stream on map only on list and set interfaces
				.peek(System.out::println)
				.sorted((set1, set2)->Double.compare(set2.getValue(), set1.getValue())) 				//reverse sorting
				.limit(1)	//limiting to one
				.map(Map.Entry::getKey) //getting the key
				.findFirst() //to reduce stream to optional
				.get(); //to get actual value

	}

	@Override
	public Set<Characteristics> characteristics() {
		return Collections.emptySet();
	}
}
```


#### Parallelism

*concurrency* and *parallelism* don't have standard definitions, and they are often (erroneously) used interchangeably. Historically, *concurrency* described a property of a **program**— the degree to which a program is structured as the interaction of cooperating computational activities — whereas *parallelism* was a property of a program's **execution**, describing the degree to which things actually happen simultaneously. (Under this definition, concurrency is the **potential **for parallelism.) This distinction was useful when true concurrent execution was mostly a theoretical concern, but it has become less useful over time.

More modern curricula describe *concurrency* as being about correctly and efficiently controlling access to shared resources, whereas *parallelism* is about using more resources to solve a problem faster. Constructing thread-safe data structures is the domain of concurrency, as enabled by primitives such as locks, events, semaphores, coroutines, or software transactional memory (STM). On the other hand, parallelism uses techniques like partitioning or sharding to enable multiple activities to make progress on the task **without** coordination.

Why is this distinction important? After all, concurrency and parallelism have the common goal of getting multiple things done simultaneously. But there's a big difference in how easy the two are to get right. Making concurrent code correct with coordination primitives such as locks is difficult, error prone, and unnatural. Making parallel code correct by arranging that each worker has its own portion of the problem to work on is comparatively simpler, safer, and more natural.

This is why java 8 parallel streams gives you easy parallism but assumes you to write thread safe code.



#### Using Parallelism carefully

The measure of parallel effectiveness, called ***speedup***, is simply the ratio of parallel runtime to sequential runtime. Choosing parallelism (assuming it delivers a speedup) is a deliberate choice to value time over CPU and power utilization. A parallel execution always does more work than a sequential one, since — in addition to solving the problem — it also must decompose the problem, create and manage tasks to describe the subproblems, dispatch and wait for those tasks, and merge their results. So the parallel execution always starts out "behind" the sequential one and hopes to make up for the initial deficit through economy of scale.

For parallelism to be the better choice, several things must come together. We need a problem that admits a parallel solution in the first place — which not all problems do. Then, we need an implementation of the solution that exploits the inherent parallelism. We need to ensure that the techniques used to implement parallelism don't come with so much overhead that we squander the cpu cycles we throw at the problem. And we need **enough data** so that we can achieve the economy of scale needed to get a speedup.



Consider the problem of computing the function `h(n)`:

```mathematica
h(0) = f(0)
h(n) = f(n) + h(n-1)
```

![par1Figure3](https://github.com/john77eipe/Java-8-Handbook/blob/master/par1Figure3.png)



It might appear that there is a recursive dependency and hence it's not possible to exploit parallesim.

However, if we rewrite `h(n)` in a different way, we can immediately see how this problem has exploitable parallelism. We can compute each of the terms independently and then add them up (which also admits parallelism):

```mathematica
h(n) = f(0) + f(1) + .. + f(n)
```

The result is a dataflow dependency graph like the one shown in Figure 4, in which each `h(n)` can be computed independently.

![par1Figure4](https://github.com/john77eipe/Java-8-Handbook/blob/master/par1Figure4.png)

 These examples show us two things: first, that similar-looking problems can have vastly different degrees of exploitable parallelism; and second, that the "obvious" implementation of a solution to a problem with exploitable parallelism might not necessarily **exploit** that parallelism. To have any chance of getting a speedup, we need both.



#### Amdahl's law

[Amdahl's law](https://en.wikipedia.org/wiki/Amdahl's_law) describes how the sequentiala portion of a computation limits the possible parallel speedup. Most problems have some amount of work that cannot be parallelized; this is called the *serial fraction*. For example, if you are going to copy the data from one array to another, the copying might be parallelizable, but the allocation of the target array — which is inherently sequential — must happen before any copying can happen.

![Par1Figure5](https://github.com/john77eipe/Java-8-Handbook/blob/master/par1Figure5.png)



The various curves illustrate the best possible speedup allowed by Amdahl's Law as a function of the number of processors available, for varying parallel fractions (the complement of the serial fraction.) If, for example, the parallel fraction is .5 (half the problem must be executed sequentially) and an infinite number of processors are available, Amdahl's law tells us that the best speedup we can hope for is 2x. 



> The Free Lunch Is Over: A Fundamental Turn Toward Concurrency in Software
>
> http://www.gotw.ca/publications/concurrency-ddj.htm



Several factors that might cause a parallel execution to lose efficiency:

- The source is expensive to split, or splits unevenly.

  We have seen that with parallel streams, we use the `Spliterator` method `trySplit()` to split a segment of the source data in two. Each node in the computation tree corresponds to a binary split, forming a binary tree. Ideally, this tree would be balanced — with each leaf node representing exactly the same amount of work — and the cost of splitting would be zero.

  This ideal is not achievable in practice, but some sources come much closer than others. Arrays are the best case. On the other hand, a linked list splits terribly. The cost of splitting is poor — to find the midpoint, we must traverse half the list, one node at a time. 

  That said, it's not **impossible** to get parallelism out of a linked list — if the operations being performed for each node are sufficiently expensive.

  Binary trees (such as `TreeMap`) and hash-based collections (such as `HashSet`) split better than linked lists, but not as well as arrays. Binary trees are fairly cheap to split in two, and if the tree is relatively balanced, the resulting computation tree will be as well. 

- Merging partial results is expensive.

  Every time we split the source, we accrue an obligation to combine the intermediate results of that split. After the leaf nodes have completed the work on their segment of the input, we proceed back up the tree, combining results as we go.

  Some combining operations — such as reduction with addition — are cheap. But others — such as merging two sets — are much more expensive. The amount of time spent in the combination step is proportional to the depth of the computation tree; a balanced tree will have depth of *O(lg n)*, whereas a pathologically unbalanced tree (such as that we get from splitting a linked list or an iterative generating function) will have depth of *O(n)*.

  Another problem with expensive merges is that the last merge — in which two half-results are being merged — will be performed sequentially (because there is no other work left to do). Stream pipelines whose merge steps are *O(n)*— such as those using the `sorted()` or `collect(Collectors.joining())` terminal operations — might see their parallelism limited by this effect.

- The problem doesn't admit sufficient exploitable parallelism.

  For example, the `findFirst()` terminal operation yields the first element in the stream. (This operation is typically combined with filtering, so it usually ends up meaning "find the first element that satisfies some condition.") Implementing `findFirst()` sequentially is extremely cheap: Push data through the pipeline until some result is produced, and then stop. In parallel, we can easily parallelize the upstream operations, but when a result is produced by some subtask, we're not done. We still have to wait for all the subtasks that come earlier in the encounter order to finish. (At least we can cancel any subtasks that appear later in the encounter order.) 

  A parallel execution pays all the costs of decomposition and task management, but is less likely to reap the benefits. On the other hand, the terminal operation `findAny()` is much more likely to reap a parallel speedup, because it keeps all the cores busy searching for a match, and can terminate immediately when it finds one.

  Frequently, we can replace `findFirst()` with `findAny()` without any loss of correctness. Similarly, by making the stream unordered via the `unordered()` operation, we can often remove the encounter-order dependence inherent in `limit()`, `distinct()`, `sorted()`, and `collect()` with no loss of correctness.

- Utilizing the correct JDK API

  Two examples show two ways to generate a stream consisting of integers 0 to 99:

  ```java
  IntStream stream1 = IntStream.iterate(0, n -> n < 100, n -> n + 1);
  ```

  And this code uses `IntStream.range()`:

  ```java
  IntStream stream2 = IntStream.range(0, 100);
  ```

  `Stream.iterate()` takes an initial value and two functions — one to produce the next value, and one to determine whether to stop producing elements — just like a `for`-loop. It's intuitively clear that this generator is fundamentally sequential: It cannot produce element *n* until it has produced element *n-1*.

  On the other hand, the `range()` generator splits more like an array — it's easy and cheap to compute the midpoint of the range, without having computed the intervening elements.

- The layout of the data results in poor access locality.

- There's not enough data to overcome the startup costs of parallelism.



#### NQ Model

A simple but useful model for parallel performance is the *NQ* model, where *N* is the number of data elements, and *Q* is the amount of work performed per element. The larger the product *N\*Q*, the more likely we are to get a parallel speedup. For problems with trivially small *Q*, such as adding up numbers, you generally want to see *N* > 10,000 to get a speedup; as *Q* increases, the data size required to get a speedup decreases.

Many of the impediments to parallelism — such as splitting cost, combining cost, or encounter order sensitivity — are moderated by higher *Q* operations.



Going back to the usecase that we discussed in lambda's section 5.

The code can now utilise parallel streams.

```java
//Vendor makes this change to utilize java 8 features but he faces this issue
public static <T extends Number> Optional<T> sum(List<T> list, Predicate<T> condition){
	//ability to add is only here
		return list.parallelStream()
		.filter(condition)
		.map(i -> i)
        .reduce((a,b)->a+b); //Compile error: Operator + is undefined for type T, T
}

//client
int result = MathUtility.sum(listOfInts, i->i<4).get();
```

One solution is to move the addition logic back to the client.

```java
int result = MathUtility.sum(listOfInts, i->i<4, (a,b)->a+b).get();

//vendor code
public static <T extends Number> Optional<T> sum(List<T> list, Predicate<T> condition, BinaryOperator<T> operation){
	//ability to add is only here
		return list.parallelStream()
		.filter(condition)
		.map(i -> i)
        .reduce(operation);
}
```



But we don't want to do this!!! As our inital assumption is that client doesn't or don't want to do the mathematical operation. 

One of the solution to this problem would be to get the Vendor by the coller and threaten him to support different additions and at the same time utilize parallel streams.

Vendor code:

```java
class MathUtility<T extends Number> {

	static class IntegerAddition implements BinaryOperator<Integer> {

		static private IntegerAddition instance = new IntegerAddition();

		private IntegerAddition() {
		}

		@Override
		public Integer apply(Integer t, Integer u) {
			return t + u;
		}

		public static IntegerAddition instance() {
			return instance;
		}
	}

	public static <T extends Number> Optional<T> sum(List<T> list, Predicate<T> condition,
			BinaryOperator<T> operation) {
		return list.parallelStream().filter(condition).map(i -> i).reduce(operation);
	}
	
}
```

Client code:

```java
System.out.println(
		MathUtility.sum(listOfInts, i->i<4, MathUtility.IntegerAddition.instance()).get()
		);
//incase in future if the client wants to control the mathematical operaiton. That is also possible because of the way sum is defined
System.out.println(
		MathUtility.sum(listOfInts, i->i<4, (a,b)->100-a-b).get()
		);
```

References:

Most of the parallel discussion is taken from here:

https://www.ibm.com/developerworks/java/library/j-java-streams-4-brian-goetz/index.html?ca=drs-

http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html



### Common Usecases

#### 1. Converting List Objects to Maps

   A usuall usecase is the word count which we have already seen

   ```java
   List<String> listOfStr = new ArrayList<>();
   listOfStr.add("apple");
   listOfStr.add("orange");
   listOfStr.add("orange");
   		
   Map<String, Long> wordCount = listOfStr.stream()
       .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
   
   wordCount
   	.entrySet()
       .forEach(System.out::println);
   /*
   orange=2
   apple=1
   */
   ```

#### 2. Handling of Exceptions

   ```pseudocode
                                       ---> Throwable <--- 
                                       |    (checked)     |
                                       |                  |
                                       |                  |
                                       ---> Exception     Error
                                       |    (checked)     (unchecked)
                                       |
                                       RuntimeException
                                       (unchecked)
   ```


   **Runtime or Unchecked exceptions**

   Runtime or Unchecked exceptions are straight forward because the system shouldn't recover from these and technically these shouldn't occur as it's a bug and should be fixed asap.

   Consider an example of doing a math operation provided by Util.integerOps. 

   ```java
   //traditional loop
   private static void mathOperation() {
   	int total = 0;
   	for (int i = 1; i < LIMIT; i++) {
   		total = total + Util.integerOps(i, FACTOR);
   	}
   	System.out.println("Total: " + total);
   }
   //streams
   private static void mathOperationStreams() {
   	int total = IntStream.range(1, LIMIT).map(t -> Util.integerOps(t, FACTOR)).sum();
   	System.out.println("Total: " + total);
   }
   
   class Util {
   	static int integerOps(int i, int j) {
   		if (i == 1) {
   			System.out.println("Operation in " + Thread.currentThread().getName());
   		}
   		return i + 100 / j;
   	}
   }
   
   public class TempExceptions {
   	final static int FACTOR = 0;
   	final static int LIMIT = 10;
   
   	public static void main(String[] args) {
   		mathOperation();
   		mathOperationStreams();
   	}
   }    
   ```

   When we run the first method `mathOpertion()` we get the below exception as expected.

   ```pseudocode
   Operation in main
   Exception in thread "main" java.lang.ArithmeticException: / by zero
   	at com.test.Util.integerOps(TempExceptions.java:48)
   	at com.test.TempExceptions.mathOperation(TempExceptions.java:20)
   	at com.test.TempExceptions.main(TempExceptions.java:13)
   ```

   And for the second method we get the same but the trace differs slightly as seen below whether streams or for loops, in case or runtime exceptions it behaves the same.

   ```pseudocode
   Operation in main
   Exception in thread "main" java.lang.ArithmeticException: / by zero
   	at com.test.Util.integerOps(TempExceptions.java:48)
   	at com.test.TempExceptions.lambda$0(TempExceptions.java:26)
   	at java.base/java.util.stream.IntPipeline$4$1.accept(IntPipeline.java:246)
   	at java.base/java.util.stream.Streams$RangeIntSpliterator.forEachRemaining(Streams.java:104)
   ........
       at com.test.TempExceptions.mathOperationStreams(TempExceptions.java:26)
   	at com.test.TempExceptions.main(TempExceptions.java:14)
   ```

   We had to comment the second method call while running the first method because the runtime exceptions causes the program to exit.

   What if we didn't want the program to not exit and continue the execution even for runtime exceptions.

   

   One approach is to run them in a different thread.

   ```java
   Thread t = new Thread(new Runnable() {
   	@Override
   	public void run() {
   		mathOperation();
   	}
   });
   t.start();
   
   System.out.println("this line does execute");
   ```

   But how are these error messages shown or handled in real world applicaitons.

   GUI-Apps:  exception occurs in an event handler (e.g. in response to a mouse click), the exception stack trace will be printed as above, but the program won't quit.

   Servlets:  exception occurs in a request handling threads breaks but the system may take some other action but the program won't quit. (program shouldn't quit else the whole web application would crash the web app)

   Phone apps: same as above but can deside to quit or not.

   

   How it handles unchecked exceptions in the above cases is through an **uncaught exception handler**.

   When an uncaught exception occurs in a particular thread, Java looks for what is called an **uncaught exception handler**, actually an implementaiton of the interface `UncaughtExceptionHandler`. The latter interface has a method `handleException()`, which the implementer overrides to take appropriate action, such as printing the stack trace to the console. As we'll see in a moment, we can actually install our own instance of `UncaughtExceptionHandler` to handle uncaught exceptions of a particular thread, or even for the whole system.

   The specific procedure is as follows. When an uncaught exception occurs, the JVM does the following:

   - it calls a special private method, `dispatchUncaughtException()`, on the `Thread` class in which the exception occurs;
   - it then **terminates the thread** in which the exception occurred1.

   However, we can actually override this process for an individual thread, for a `ThreadGroup`, or for all threads, as follows:

| Which Thread's handler to set | How to set                                    |
| ----------------------------- | --------------------------------------------- |
| All                           | `Thread.setDefaultUncaughtExceptionHandler()` |
| All for a Thread Group        | Override `ThreadGroup.uncaughtException()`    |
| Individual Thread             | Thread.setUncaughtExceptionHandler()          |

   Let's try applying a uncaught exception handler.

   ```java
   Thread t = new Thread(new Runnable() {
   	@Override
   	public void run() {
   		mathOperation();
   	}
   });
   t.start();
   t.setUncaughtExceptionHandler(new ExceptionHandler());
   
   System.out.println("this line does execute");
   
   t = new Thread(new Runnable() {
   	@Override
   	public void run() {
   		mathOperationStreams();
   	}
   });
   t.start();
   t.setUncaughtExceptionHandler(new ExceptionHandler());
   
   class ExceptionHandler implements UncaughtExceptionHandler {
   	public void uncaughtException(Thread t, Throwable e) {
   		System.out.printf("An exception has been captured but ignored in Thread: %s Thread status is : %s\n",
   				t.getName(), t.getState());
   	}
   }
   ```

   Output:

   ```pseudocode
   this line does execute
   Operation in Thread-0
   An exception has been captured but ignored in Thread: Thread-0 Thread status is : RUNNABLE
   Operation in ForkJoinPool.commonPool-worker-7
   An exception has been captured but ignored in Thread: Thread-1 Thread status is : RUNNABLE
   ```

   

   Thus we are able to handle the exception and not affect the main process.

**Checked exceptions**

   Checked exception's are created for the sole purpose of knowing what happened and to recover from it.

   Let's change the example a bit to introduce checked exceptions

   ```java
   class Util {
   	static int integerOps(int i, int j) throws MyException {
   		if (i % 10 == 0) {
   			System.out.println("Operation in " + Thread.currentThread().getName());
   			throw new MyException("ERROR on " + i);
   		}
   		return i * j;
   	}
   }
   
   class MyException extends Exception {
   
   	private static final long serialVersionUID = 1L;
   	
   	String errorCode;
   	
   	public MyException(String errorCode) {
   		this.errorCode = errorCode;
   	}
   	
   	public String getErrorCode() {
   		return errorCode;
   	}
   
   	@Override
   	public String toString() {
   		return String.format("MyException [errorCode=%s]", errorCode);
   	}
   	
   }
   ```

   The invocation of those methods using the traditional approach is 

   ```java
   private static void mathOperation() {
   	int total = 0;
   	List<Exception> lisOfExceptions = new ArrayList<>();
   	for (int i = 1; i < LIMIT; i++) {
   		try {
   			total = total + Util.integerOps(i, FACTOR);
   		} catch (MyException e) {
   			lisOfExceptions.add(e);
   		}
   	}
   	System.out.println("Total: " + total);
   	System.out.println(lisOfExceptions);
   }
   ```

   How to handle it in streams?

   Approach 1: One of the approach is to handle it within the stream.

   ```java
   private static void mathOperationStreams() {
   	int total = 0;
   	List<Exception> lisOfExceptions = new ArrayList<>();
   	total = IntStream.range(1, LIMIT).map(t -> {
   		try {
   			return Util.integerOps(t, FACTOR);
   		} catch (MyException e) {
   			lisOfExceptions.add(e);
   		}
   		return 0;
   	}).sum();
   	System.out.println("Total: " + total);
   	System.out.println(lisOfExceptions);
   }
   ```

   In the above approach we not only get to handle but also track the exceptions in a array list.

   Not advised to modify a datastructure within a lambda expression but this is one exception ;-).

Approach 2:

Another approach is to extend an appropriate Funtional interface, in this case map on an IntStream requires IntUnaryOperator expression.



```java
@FunctionalInterface
interface IntUnaryOperatorWithException extends IntUnaryOperator {
	
	List<Exception> listOfExceptions = new ArrayList<>();

	@Override
	default int applyAsInt(int t) {
		int result = 0;
		try {
			result = acceptThrows(t);
		}catch (Exception e) {
			listOfExceptions.add(e);
		}
		return result;
	}
	int acceptThrows(int elem) throws Exception;
}	
```

And our invocation code is

```java
total = IntStream.range(1, LIMIT).map((IntUnaryOperatorWithException)(t->Util.integerOps(t, FACTOR))).sum();
System.out.println("Total: " + total);
System.out.println(IntUnaryOperatorWithException.listOfExceptions);
```



This is a favored approach because we now have a reusable functional interface and the exception handling code has been moved out.  The same approach can be used to extend any other type of functional interface lilke Function, Consumer, etc.

Approach 3:

Write your own Functional interface that allows exceptions then write a wrapper funciton in your code that handler exception.



```java
@FunctionalInterface
interface MyIntUnaryOperator<E extends Exception> {
    int apply(int t) throws E;
}

static List<Exception> listOfExceptions = new ArrayList<>();

private static <E extends Exception> IntUnaryOperator wrapper(MyIntUnaryOperator<E> fe) {
	return arg -> {
		int result = 0;
		try {
			result = fe.apply(arg);
		} catch (Exception e) {
			listOfExceptions.add(e);
		}
		return result;
	};
}
```



Go for this approach if you want to keep the exception handling code and also the exception list out of the functional interface.

```java
total = IntStream.range(1, LIMIT).map(wrapper(t->Util.integerOps(t, FACTOR))).sum();
System.out.println("Total: " + total);
System.out.println(listOfExceptions);
```


#### 3. Converting Optional to a Stream

Yes, this was a small hole in the API, in that it's somewhat  inconvenient to turn an Optional into a zero-or-one length Stream. You  could do this:

```java
Optional<Other> result =
    things.stream()
          .map(this::resolve)
          .flatMap(o -> o.isPresent() ? Stream.of(o.get()) : Stream.empty())
          .findFirst();
```

There is a solution for this in Java 9.

#### 4. Serializing Lambda

Java 8 introduces the possibility to [cast an object to an intersection of types by adding multiple bounds](http://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.16). In the case of serialization, it is therefore possible to write:

```java
Runnable r = (Runnable & Serializable)() -> System.out.println("Serializable!");
```


#### 5. Split Stream

It is not possible to split a stream into 2 separate stream but you can do split operations and continue into a single original stream.

Let's assume there is a Human class with attributes `name` and `age`.

```java
List<Human> humans = new ArrayList<>();
humans.add(new Human("john", 11));
humans.add(new Human("eipe", 22));

humans.stream().map(
		new PredicateSplitterFunction<>(
				human -> human.getName().equals("john"),
				human -> human.getAge()+10 , 
				human -> human.getAge()+30))
		.forEach(human -> System.out.println(human));
//Output: 
//21
//52

class PredicateSplitterFunction<T, R> implements Function<T, R> {
	private Predicate<T> predicate;
	private Function<T, R> positiveFunction;
	private Function<T, R> negativeFunction;

	public PredicateSplitterFunction(Predicate<T> predicate, Function<T, R> positive, Function<T, R> negative) {
		this.predicate = predicate;
		this.positiveFunction = positive;
		this.negativeFunction = negative;
	}

	@Override
	public R apply(T t) {
		if (predicate.test(t)) {
			return positiveFunction.apply(t);
		} else {
			return negativeFunction.apply(t);
		}
	}
}
```



An example using Consumers is [here](https://stackoverflow.com/questions/19940319/can-you-split-a-stream-into-two-streams)


#### 6. Performance impact of lambdas & streams

 Using stream and lambdas for small for loops 

- degrades performance

- some of the cases affects readability

- maintainability 

A detailed article about all 3 is [here](https://blog.jooq.org/2015/12/08/3-reasons-why-you-shouldnt-replace-your-for-loops-by-stream-foreach/)

I personally did a performance check using different approaches.

```java
private static void findingTotalOldSchool()  {
	long total = 0;
	long start = System.nanoTime();
	
	for (long i = 1; i < LIMIT; i++) {
		total = total + (i * FACTOR);
	}

	long duration = (System.nanoTime() - start) / 1_000_000;
	System.out.println("Duration: "+duration);
	System.out.println("Total: "+total);
}

public static Range range(int max)  {
    return new Range(max);
}

private static void findingTotalCustomIterator() {
	long total = 0;
	long start = System.nanoTime();
	
	for (long i : range(LIMIT)) {
		total = total + i * FACTOR;
	}

	long duration = (System.nanoTime() - start) / 1_000_000;
	System.out.println("Duration: "+duration);
	System.out.println("Total: "+total);
}

private static void findingTotalStream() {
	long start = System.nanoTime();	
	long total = 0;
	
	total = LongStream.range(1, LIMIT)
			.map(t -> t * FACTOR)
			.sum();
	
	long duration = (System.nanoTime() - start) / 1_000_000;
	System.out.println("Duration: "+duration);
	System.out.println("Total: "+total);
}

private static void findingTotalParallelStream() {
	long start = System.nanoTime();	
	long total = 0;
	
	total = LongStream.range(1, LIMIT)
			.parallel()
			.map(t -> t * FACTOR)
			.sum();
	
	long duration = (System.nanoTime() - start) / 1_000_000;
	System.out.println("Duration: "+duration);
	System.out.println("Total: "+total);
}

private static void findingTotalCFS() {
	 long start = System.nanoTime();
	 
	 List<CompletableFuture<Long>> futures = 
			 LongStream.range(1, LIMIT).boxed()
			 .map(t -> CompletableFuture.supplyAsync(() -> t * FACTOR ))
			 .collect(Collectors.toList());
	 //Code here --- could run ahead hence joining on futures
     long total = futures.stream().map(CompletableFuture::join).mapToLong(t->t).sum();
     
     long duration = (System.nanoTime() - start) / 1_000_000;
     System.out.println("Futures used: "+futures.size());
     System.out.println("Duration: "+duration);
	 System.out.println("Total: "+total);
}

private static void findingTotalCFSE() {
	long start = System.nanoTime();
	
	ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors() + 1);
	List<CompletableFuture<Long>> futures =
			 LongStream.range(1, LIMIT).boxed()
	         .map(t -> CompletableFuture.supplyAsync(() -> {
					return t * FACTOR;
			}, executor))
	         .collect(Collectors.toList());
	 
	 long total = futures.stream().map(CompletableFuture::join).mapToLong(t->t).sum();
	 executor.shutdownNow();
	 
	 long duration = (System.nanoTime() - start) / 1_000_000;
	 System.out.println("Futures used: "+futures.size());
     System.out.println("Duration: "+duration);
	 System.out.println("Total: "+total);
}

private static void findingTotalES() {
	long start = System.nanoTime();
	
	ExecutorService executorService = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors() + 1);
	long total  = LongStream.
		range(1, LIMIT)
		.boxed()
		.map((i)->executorService.submit(new Operation(i, FACTOR)))
		.map((Future<Long> future)-> {
			try {
	            return future.get();
	        } catch (InterruptedException e) {
	            Thread.currentThread().interrupt();
	        }catch (ExecutionException e) {
	            // Extract the actual exception from its wrapper
	            Throwable t = e.getCause();
	        } 
			return 0;
		})
		.mapToLong(t->t.longValue())
		.sum();
		
	executorService.shutdown();
	
	long duration = (System.nanoTime() - start) / 1_000_000;
	System.out.println("Duration: "+duration);
	System.out.println("Total: "+total);
}

class Operation implements Callable<Long> {
	
	long i; int j;
	Operation(long i, int j) { this.i = i; this.j = j; }
	
	@Override
	public Long call() {
		return i * j;
	}
}


class Range implements Iterable<Integer> {

    private int limit;

    public Range(int limit) {
        this.limit = limit;
    }

    @Override
    public Iterator<Integer> iterator() {
        final int max = limit;
        return new Iterator<Integer>() {

            private int current = 0;

            @Override
            public boolean hasNext() {
                return current < max;
            }

            @Override
            public Integer next() {
                if (hasNext()) {
                    return current++;   
                } else {
                    throw new NoSuchElementException("Range reached the end");
                }
            }

            @Override
            public void remove() {
                throw new UnsupportedOperationException("Can't remove values from a Range");
            }
        };
    }
}
```



We ran test runs with 2 sets of data. Each test should be run individually and not as part of a single whole run (as JVM optimizes and the result might vary).

```java

//first run
final static int FACTOR = 1;
final static int LIMIT = 10000;

//second run
final static int FACTOR = 9876;
final static int LIMIT = 1000000;


System.out.println("-----Traditional Loop-----");
findingTotalOldSchool();
// 0 ms
// 4 ms		

System.out.println("-----Custom Iterator----");
findingTotalCustomIterator();
// 1 ms
// 15 ms


System.out.println("-----Streams-----");
findingTotalStream();
// 38 ms
// 33 ms		


System.out.println("-----Parallel Streams-----");
findingTotalParallelStream();
// 29 ms
// 64 ms


System.out.println("-----Completable Futures with Streams-----");
findingTotalCFS();
// 77 ms
// 635 ms		


System.out.println("-----Executor Service with Streams-----");
findingTotalES();
// 323 ms
// 12632 ms

System.out.println("-----Completable Futures with Executor Service with Streams-----");
findingTotalCFSE();
// 77 ms
// 844 ms	
```

Observations: 
- Traditional loop is fast most of the cases.
- Use parallel streams when performance or IO operations are involved.
- For simple iterations (involving substitutions or simple numeric calculations) go for traditional loop.
- Completable Futures with Executor Service is flexible and a go to option when you need more control on the number of threads, etc.

#### 7. Stream operations on Lists

 Situation 1: Need to create a list populated with integers from 0 to 19.
 
 ```java
 List<Integer> numbers = IntStream.range(0, 20).boxed().collect(Collectors.toList());
 ```

 Situation 2: Need to remove from the list within stream pipeline.
 
 ```java
 numbers.forEach(number -> {numbers.remove(number); }); //java.lang.UnsupportedOperationException
 numbers.removeIf(number -> number>3);  //works
 ```
 Situation 3: We need to iterate from 0 to 19 and store the these values in a list and the doubled values in another list
 
 ```java
List<Integer> list1 = new ArrayList<>();
List<Integer> list2 =  
	IntStream.range(0, 20)
		.boxed()
		.peek(n -> list1.add(n))
		.map(n -> n*2)
		.collect(Collectors.toList());

System.out.println(list1);
System.out.println(list2);
//[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
//[0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36, 38]
 ```
Can we parallize this to achieve better throughput

```java
List<Integer> list1 = new ArrayList<>();
List<Integer> list2 = 
	IntStream.range(0, 20)
		.parallel()
		.boxed()
		.peek(n -> list1.add(n))
		.map(n -> n*2)
		.collect(Collectors.toList());

System.out.println(list1);
System.out.println(list2);
//[12, 11, 17, 19, 16, 6, 2, 8, 5, 9, 15, 4, 10, 3, 1, 0, 13, 14]
//[0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36, 38]
```
It's obvious that some of the values are missing from the first list but the second list is perfect. This is because collect takes care of the parallism and collecting (reducing) the values together. 

Any unsafe operation that is done within the parallel pipeline cannot be guaranteed to execute correctly. Which is exactly the above case.

One way to make this still work is to make list1 a Synchronized List.
```List<Integer> list1 = Collections.synchronizedList(new ArrayList<>());```

#### 8. Modify a non-Final variable in lambda

A most common scenario that pops up again and again is a requirement to manage an external state within a lambda.

Consider this example

```java
long sum = 0L;
final long LIMIT = 10000000L;
for(long i=1; i<=LIMIT; i++) {
	sum += i;
}
```

A lambda version would be 

```java
LongStream.rangeClosed(1, LIMIT).forEach(i -> sum += i);
```

But since sum cannot be final this fails at compilation. 

A Bad trick to achieve this is to use a container like an array or list. We have already seen examples of modifying lists within lambdas.

```java
long[] sumarr = { 0L }; //stays in heap because it's a array
final long LIMIT = 10000000L;
LongStream.rangeClosed(1, LIMIT).forEach(i -> sumarr[0] += i);
```
Note that you cannot parallize this operation. Parallel streams will cause the sumarr[0] reference to be over-written by multiple threads.

One solution is 

```java
AtomicLong sum = new AtomicLong(0); //doing frequent operations on atomic variables are expensive
LongStream.rangeClosed(1, LIMIT).parallel().forEach(i -> sum.addAndGet(i));
```

Better solution to Avoid summation by accumulation and instead use summation by reduction (but the operation has to be associative)

```java
long sumR = LongStream.rangeClosed(1, LIMIT)
	.parallel()
	.reduce(0, (i,j) -> i+j); 
```

But wait there is a method in JDK that does exactly this

```java
long sumRT = LongStream.rangeClosed(1, LIMIT)
	.parallel()
	.sum();	 //built in function appears to be faster than reduce!!!
```

That's the end of the story and the lesson learnt is to always look for existing APIs within the JDK.

#### 9. flatMap vs Map

Consider the following scenario

```java
public class Customer {
    private String name;
    private List<Order> orders = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public Customer addOrder(Order order) {
        orders.add(order);
        return this;
    }

    public List<Order> getOrders() {
        return orders;
    }

    @Override
    public String toString() {
        return name;
    }
}
public class Order {
    private int id;
}

Customer sheridan = new Customer("Sheridan");
Customer ivanova = new Customer("Ivanova");
Customer garibaldi = new Customer("Garibaldi");

sheridan.addOrder(new Order(1))
        .addOrder(new Order(2))
        .addOrder(new Order(3));
ivanova.addOrder(new Order(4))
        .addOrder(new Order(5));

List<Customer> customers = Arrays.asList(sheridan, ivanova, garibaldi);
```

Let's say that you need to print only the customers names:
```java
// map for 1-1 customer to name --> Stream<String>
customers.stream()
        .map(Customer::getName) // function<Customer,String>
        .forEach(System.out::println);
```

Let's say you want to print all the order numbers from the `customers`
```java
// map 1-many customer to orders --> Stream<List<Order>>
customers.stream()
	.map(Customer::getOrders) // function<Customer,List<Order>>
    .map(order -> order)
    .forEach(System.out::println);
```
But this prints it grouped by customer as follows
```
[Order{id=1}, Order{id=2}, Order{id=3}]
[Order{id=4}, Order{id=5}]
[]
```
Streaming the orders again

```java
customers.stream()
	.map(customer -> customer.getOrders().stream()) //returns Stream<Stream<Order>
    .forEach(System.out::println);
```

The above only prints Stream object ids.
```
java.util.stream.ReferencePipeline$Head@33e5ccce
java.util.stream.ReferencePipeline$Head@5a42bbf4
java.util.stream.ReferencePipeline$Head@270421f5
```

Only way to flatten it out is to use a flatMap.

```java
customers.stream()
    .flatMap(customer -> customer.getOrders().stream()) // function<Customer,Stream<Order>>
    .forEach(System.out::println);
```

Prints as expected

```
Order{id=1}
Order{id=2}
Order{id=3}
Order{id=4}
Order{id=5}
```

## Java Type Annotations

### Overview of Built-in Annotations in JDK < 1.7

Annotations can be easily recognized in code because the annotation name is prefaced with the `@` character. Annotations have no direct effect on code operation, but at processing time, they can cause an annotation processor to generate files or provide informational messages. 

### What?

In its simplest form, an annotation can be placed in Java source code to indicate that the compiler must perform specific “checking” on the annotated component to ensure that the code conforms to specified rules.

Java comes with a basic set of built-in annotations. The following Java annotations are available for use out of the box: 

- `@Deprecated`: Indicates that the marked element should no longer be used. Most often, another element has been created that encapsulates the marked element’s functionality, and the marked element will no longer be supported. This annotation will cause the compiler to generate a warning when the marked element is found in source code.
- `@Override`: Indicates that the marked method overrides another method that is defined in a superclass. The compiler will generate a warning if the marked method does not override a method in a superclass.
- `@SuppressWarnings`: Indicates that if the marked element generates warnings, the compiler should suppress those warnings.
- `@SafeVarargs`: Indicates that the marked element does not perform potentially unsafe operations via its `varargs` parameter. Causes the compiler to suppress unchecked warnings related to `varargs`.
- `@FunctionalInterface`: Indicates that the type declaration is intended to be a functional interface. 


But all the above annotations are for methods/classes. 

Type Annotations are annotations that can be placed anywhere you use a type. This includes the new operator, type casts, implements clauses and throws clauses. Type Annotations allow improved analysis of Java code and can ensure even stronger type checking.	

This means we get two new ElementTypes for annotations (in SDK):

ElementType.TYPE_USE: can be applied at any type use 

ElementType.TYPE_PARAMETER: allows an annotation to be applied at type variables (e.g. MyClass<T>)

```java
`@Target``({ElementType.TYPE_USE, ElementType.TYPE_PARAMETER})``public` `@interface` `Test {``}`
```

### How?

Please note that the annotations from the below examples will not work out of the box when Java 8 is released. Java 8 only provides the ability to define these types of annotations. It is then up to framework and tool developers to actually make use of it. 

Checker Framework is a third party library that has made extensive use of this SDK feature.

Check out: https://checkerframework.org/jsr308/specification/java-annotation-design.html

Writing your own custom annotations is a whole different discussion for later.

### Why?

We will see the uses of this feature by considering one of the libraries that uses these features extensively -  checker framework.

https://checkerframework.org/

## Generalized Target Type Inference

### What?

Java 8 supports inference using Target-Type in a method context.

 *When we invoke a generic method without explicit type arguments, the compiler can look at the method invocation and corresponding method declarations to determine the type argument (or arguments) that make the invocation applicable.*

### How?

Thee introduction of generics resulted in the necessity of writing boilerplate code due to the need to pass type parameters.

```java
Map<String, Map<String, String>> mapOfMaps = new HashMap<String, Map<String, String>>();
List<String> strList = Collections.emptyList();
List<Integer> intList = Collections.emptyList();
```

Type Inference was introduced which **is the process of automatically deducing unspecified data types of an expression based on the contextual information.**

Type inference makes line numbers 2 and 3 possible.

Java 7 expanded apon this and introduced the <> operator.

```java
Map<String, Map<String, String>> mapOfMaps = new HashMap<>();
```

Java 8 further expanded the scope of Type Inference. We refer to this expanded inference capability as **Generalized Target-Type Inference**. 

The Target-Type of an expression is the data type that the Java Compiler expects depending on where the expression appears.

In Generalized Target-Type Inference, JDK tries to infer the Target-Type in a method context using 2 techniques.

- invocation applicability inference analysis
- invocation type inference analysis

```java
public static void main(String[] args) {
	List<String> strListGeneralized = add(new ArrayList<>(), "abc", "def");
	List<Integer> intListGeneralized = add(new ArrayList<>(), 1, 2);
}
static <T> List<T> add(List<T> list, T a, T b) {
    list.add(a);
    list.add(b);
    return list;
}
```



In the code, *ArrayList<>* does not provide the type argument explicitly. So, the compiler needs to infer it. First, the compiler looks into the arguments of the add method. Then, it looks into the parameters passed at different invocations.

**It performs invocation applicability inference analysis to determine whether the method applies to these invocations**. If multiple methods are applicable due to overloading, the compiler would choose the most specific method.

Then, **the compiler performs invocation type inference analysis to determine the type arguments.** **The expected target types are also used in this analysis**. It deduces the arguments in the three instances as *ArrayList<String>*, *ArrayList<Integer>* and *ArrayList<Number>*.

Does this work in chanied calls?

`List<Integer> intListGeneralized = add(new ArrayList<>(), 1, 2).add(1);`

No. For this to work the the type inference needs to be delayed until the whole assignment expression has been evaluated.

It's not yet supported. Supported reading: https://blog.jooq.org/2013/11/25/a-lesser-known-java-8-feature-generalized-target-type-inference/

But it could be available in future JDKs as the original JEP metions it: https://openjdk.java.net/jeps/101



### Why?

This is used internally for lambda expressions.



## Optional and Value Types

### About Objects Identity and Values


The Java VM type system offers two ways to create aggregate data types: 

- heterogeneous aggregates with identity (using classes), and 
- homogeneous aggregates with identity (using arrays). 

The only types that do not have identity are the eight hardwired primitive types: byte, short, int, long, float, double, char, and boolean. 

Object identity has footprint and performance costs, which is a major reason Java, unlike other many object oriented languages, has primitives. 
These costs are most burdensome for small objects, which do not have many fields to amortize the extra costs.

For e.g. A double object takes 24 bytes, where as the double takes 8 bytes (64 bits). Reasons here: https://www.javamex.com/tutorials/memory/object_memory_usage.shtml


Note: In terms of footprint, objects with identity are heap-allocated, have object headers of one or two words, and (unless dead) have one or more pointers pointing at them. In terms of performance, each distinct aggregate value must be allocated on the heap and each access requires a dependent load (pointer traversal) to get to the “payload”, even if it is just a single field (as with java.lang.Integer). Also, the object header supports numerous operations including Object.getClass, Object.wait, and System.identityHashCode; these require elaborate support.

**Why we need these aggregates with identity (or Objects with Identity in Java)?**

Object identity serves only to support mutability, where an object’s state can be mutated but remains the same intrinsic object. 

**What makes up the identity of an object in an object-oriented system?** 

A common answer is "object identity" is defined by being in the same spot in memory, which is called "**reference equality**" and matches Java's == operator. A consequence of this situation is if two objects are identical, a change to one will always affect the other. This distinguishes identity from the notion of "being equal," or "**value equality**" (aka value identity).

**Is it always needed?**

But many programming idioms do not require identity, and would benefit from not paying the memory footprint, locality, and optimisation penalties of identity. Despite significant attempts, JVMs are still poor at figuring out whether an object’s identity is significant to the program, and so must pessimistically support identity for many objects that don’t need it.

Note: Even a nominally stateless object (with all final fields) must have its identity tracked at all times, even in highly optimized code, lest someone suddenly use it as a means of synchronization. This inherent “statefulness” impedes many optimizations.  

#### Why think about identity?

How does identity relate to developing software? Having a solid understanding of what makes two objects identical can be very important to getting an implementation to match the requirements; if two distinct objects in a system represent something the customer considers to be one, problems can arise. 

Even without looking at any requirements or customer's expectations you can construct a scenario where not thinking about identity properly can break your code. 

Consider Employee class.

```java
public class Employee {       
    private long id;   
    private String name;   
    private Date dateOfBirth;    
    private BigDecimal salary;   
    //Getter and Setters
    // toString
}
```

Say now a user creates 

```java
HashMap<Employee, String> employeeMap = new HashMap<Employee, String>();
Employee employee1 = new Employee();
employee1.setId(1);
employee1.setName("Sachin");
employee1.setDateOfBirth(new Date(1987, 2, 1));
employee1.setSalary(new BigDecimal(100000));
employeeMap.put(employee1, "India");
```

Another user tries to check if there is a value for the same employee somewhere else in the flow of logic

```java
Employee employee2 = new Employee();
employee2.setId(1);
employee2.setName("Sachin");
employee2.setDateOfBirth(new Date(1987, 2, 1));
employee2.setSalary(new BigDecimal(100000));
// Here we wanted to leave the map as if the employee is found 
//else assign him to Japan
if(employeeMap.get(employee2)==null);
	employeeMap.put(employee2, "Japan");

System.out.println(employeeMap);
// Output of this will be 2 as below
/*
{Employee{id=1, name='Sachin', dateOfBirth=Tue Mar 01 00:00:00 IST 3887, salary=100000}=Japan, 
Employee{id=1, name='Sachin', dateOfBirth=Tue Mar 01 00:00:00 IST 3887, salary=100000}=India}
 */
```

This is because the hashcode and equals of Object class ran and both the employees generates different hash codes.

So we now override hashcode and equals

```java
@Override
public boolean equals(Object o) {
	if (this == o)
		return true;
	if (o == null || getClass() != o.getClass())
		return false;
	Employee employee = (Employee) o;
	if (id != employee.id)
		return false;
	if (name != null ? !name.equals(employee.name) : employee.name != null)
		return false;
	if (dateOfBirth != null ? !dateOfBirth.equals(employee.dateOfBirth) : employee.dateOfBirth != null)
		return false;
	return salary != null ? salary.equals(employee.salary) : employee.salary == null;
}

@Override
public int hashCode() {
	int result = (int) (id ^ (id >>> 32));
	result = 31 * result + (name != null ? name.hashCode() : 0);
	result = 31 * result + (dateOfBirth != null ? dateOfBirth.hashCode() : 0);
	result = 31 * result + (salary != null ? salary.hashCode() : 0);
	return result;
}
```



Now another client comes in and writes the below code.

```java
HashMap<Employee,String> employeeMap = new HashMap<Employee,String>();  
Employee employee1 = new Employee();   
employee1.setId(1);   
employee1.setName("Sachin");   
employee1.setDateOfBirth(new Date(1987,2,1));   
employee1.setSalary(new BigDecimal(100000));

// Step 1
employeeMap.put(employee1,"India");   
for (Map.Entry<Employee, String> employeeStringEntry : employeeMap.entrySet()) {
    System.out.println(employeeStringEntry.getKey().hashCode());   
}
System.out.println(employeeMap.get(employee1)); //get "India" 

// Step 2
// Mutating the Employee Object
employee1.setName("Rahul");    
for (Map.Entry<Employee, String> employeeStringEntry : employeeMap.entrySet()) {
    System.out.println(employeeStringEntry.getKey().hashCode());    
}
// The HashMap key is mutated and in the wrong bucket for that hashcode. 

// Step 3
System.out.println(employeeMap.get(employee1));    
// Returns null
// Client is confused so creates a new employee with the same original values 

Employee employee2 = new Employee();   
employee2.setId(1);   
employee2.setName("Sachin");   
employee2.setDateOfBirth(new Date(1987,2,1));   
employee2.setSalary(new BigDecimal(100000));
System.out.println(employeeMap.get(employee2)); 
// Still returns null becuase the object in the map is the changed object and it's hashcode evaluation returns a different hashcode from now on.
```

What happened? Did I break HashMap?

I actually did break it. In the end, if you couldn't find what values were changed then that object will continue to remain there unuse-able.   

One of the solution is to use builder pattern to create immutable objects.

```java
public class Employee {

	private long id;
	private String name;
	private Date dateOfBirth;
	private BigDecimal salary;

	public Employee(EmployeeBuilder employeeBuilder) {
		this.id = employeeBuilder.id;
		this.name = employeeBuilder.name;
		this.dateOfBirth = employeeBuilder.dateOfBirth;
		this.salary = employeeBuilder.salary;
	}

	//getters and setters

	//hashcode and equals 

	public static final class EmployeeBuilder {
		private long id;
		private String name;
		private Date dateOfBirth;
		private BigDecimal salary;

		private EmployeeBuilder() {
		}

		public static EmployeeBuilder anEmployee() {
			return new EmployeeBuilder();
		}

		public static EmployeeBuilder anEmployee(Employee employee) {
			return anEmployee().withId(employee.getId()).withName(employee.getName())
					.withDateOfBirth(employee.getDateOfBirth()).withSalary(employee.getSalary());
		}

		public EmployeeBuilder withId(long id) {
			this.id = id;
			return this;
		}

		public EmployeeBuilder withName(String name) {
			this.name = name;
			return this;
		}

		public EmployeeBuilder withDateOfBirth(Date dateOfBirth) {
			this.dateOfBirth = dateOfBirth;
			return this;
		}

		public EmployeeBuilder withSalary(BigDecimal salary) {
			this.salary = salary;
			return this;
		}

		public Employee build() {
			return new Employee(this);
		}
	}
}

```

 But now the client has to re-write quite a lot.

```java
HashMap<Employee, String> employeeMap = new HashMap<Employee, String>();
Employee employee1 = 
    Employee.EmployeeBuilder.anEmployee()
    .withId(1)
    .withName("Sachin")
    .withDateOfBirth(new Date(1987, 2, 1))
    .withSalary(new BigDecimal(100000))
    .build();

employeeMap.put(employee1, "India");

Employee updatedEmployee1 =
    Employee.EmployeeBuilder.anEmployee(employee1)
    .withName("Rahul")
    .build();

System.out.println(employeeMap.get(updatedEmployee1));
// Returns null

Employee employee2 = 
    Employee.EmployeeBuilder.anEmployee()
    .withId(1)
    .withName("Sachin")
	.withDateOfBirth(new Date(1987, 2, 1))
    .withSalary(new BigDecimal(100000))
    .build();
		
System.out.println(employee2.hashCode());
System.out.println(employeeMap.get(employee2));		
// Now this works fine and it shall return the correct object from the	
// HashMap
```



All in all if you are not carefully designing builder pattern, you can run into the following problems:

- Hard-to-find bugs since asking if the object is in the map/set produces an unexpected result
- Memory leaks since remove() fails and you get dangling references
- Performance issues if you use a hash structure as cache for such objects
- Even if you have correctly created immutable references; the extra code is verbose and massive.



A lasting solution is to have value types.


### Value Types and Value Based Classes

Value Types are the solution for the above problem. 

The gross simplification of that idea is that the user can define a new kind of type, different from classes and interfaces. Their central characteristic is that they will not be handled by reference (like classes) but by value (like primitives) or, as Brian Goetz puts it in his introductory article State of the Values:

> Codes like a class, works like an int!

In Java 8 we don't have value types but are preceded by value-based classes. Their precise relation in the future is unclear but it could be similar to that of boxed and unboxed primitives (e.g. Integer and int).

**What Value-Based Classes Exist?**

These are all the classes I found in the JDK to be marked as value-based:

java.util
Optional, OptionalDouble, OptionalLong, OptionalInt

java.time
Duration, Instant, LocalDate, LocalDateTime, LocalTime, MonthDay, OffsetDateTime, OffsetTime, Period, Year, YearMonth, ZonedDateTime, ZoneId, ZoneOffset

java.time.chrono
HijrahDate, JapaneseDate, MinguaDate, ThaiBuddhistDate

I can not guarantee that this list is complete as I found no official source listing them all.

It is interesting to note that the existing boxing classes like Integer, Double and the like are not marked as being value-based. While it sounds desirable to do so – after all they are the prototypes for this kind of classes – this would break backwards compatibility because it would retroactively invalidate all uses which contravene the new limitations.

Still, they are very similar so let’s call them “value-ish”.

Instances of a value-based class:

- are final and immutable (though may contain references to mutable objects);
- have implementations of equals, hashCode, and toString which are computed solely from the instance's state and not from its identity or the state of any other object or variable;
- make no use of identity-sensitive operations such as reference equality (==) between instances, identity hash code of instances, or synchronization on an instances's intrinsic lock;
- are considered equal solely based on equals(), not based on reference equality (==);
- do not have accessible constructors, but are instead instantiated through factory methods which make no commitment as to the identity of returned instances;
- are freely substitutable when equal, meaning that interchanging any two instances x and y that are equal according to equals() in any computation or method invocation should produce no visible change in behavior.
- Use of identity-sensitive operations (like == or synchronisation) on instances of value-based classes may have unpredictable effects and should be avoided.


Let's look at few of the value based classes introduced

## Optional

### What?

The beauty of Optional is its name.

```java
java.util.Optional<T> //FailSafe using Optional
```

### Why?

```java
String name = "abc";
if(name != null){
    System.out.println(name.length);
}
```

changes to the more elegant

```java
Optional<String> name = Optional.of("abc"); 
name.ifPresent(n -> System.out.println(n.length()));
```



There are three ways to create an instance of Optional:

```java
// an empty 'Optional';
// before Java 8 you would simply use a null reference here
Optional<String> empty = Optional.empty();
 
// an 'Optional' where you know that it will not contain null;
// (if the parameter for 'of' is null, a 'NullPointerException' is thrown)
Optional<String> full = Optional.of("Some String");
 
// an 'Optional' where you don't know whether it will contain null or not
Optional<String> halfFull = Optional.ofNullable(someOtherString);

```



Something to note is that Optional does not have `orElse()` method. Hence it's not possible to write

```java
opt.ifPresent( x -> System.out.println("found " + x))
   .orElse( System.out.println("NOT FOUND"));
```

One solution is writing a [new Monad that wraps Optional](https://stackoverflow.com/questions/23773024/functional-style-of-java-8s-optional-ifpresent-and-if-not-present) But I prefer using simple if else.

### How?

Optional is solely described as a return type for queries to a collection, which was discussed in the context of streams. More precisely, it was needed for those terminal operations which can not return a value if the stream is empty. (Currently those are reduce, min, max, findFirst and findAny.)

https://blog.codefx.org/java/dev/design-optional/
http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html

Java 8 designers designed Optional as a monad.


#### Short story on Monads

- think of that as a design pattern for java developers but it's got a more of a mathematical background
- creating monads was possible since Java 5 but after version 8 with FP, it has become a lot more easier

In javascript we have asyc code like

```javascript
$("#Button").fadeIn("slow",
	function() {
		console.log("hello world");
});
```

In java we have java.util.concurrent.Future<V> class in java to represent asyc call backs

Future has the following methods: isDone(), get(), get(int timeout, TimeUnit timeUnit)	

Future almost get's our job done but it's possible to make it better.



Monads definition: 

> A Monad  is  a structure that puts a value in a computational context.



A monad is 

- a parameterized type `M<T>`

with 2 operations

- a “unit” ("return") operation: 
  ​	`T -> M<T>`
  ​	is the operation that takes a value and wraps it in a monadic context. More like a factory
- a “bind” operation: 			
  ​	`M<T> bind T -> M<U>` 
  ​	is the operation that binds a function that produces a monad to our monad 

- a "get" operation: `M<T> -> T`

and it follows 3 rules

- Left Identity: If we put a value in a monad and bind a function to it, its the same as just applying the function to a value: 
  ​	`unit(v).bind(f) ~ f.apply(v)`
- Right Identity: If we have a monad and bind that monad’s return method — it is the same as the original value:
  ​	`m.bind(m::unit) ~ m`
- Associativity: If we have a series of functions applied to a monad it doesn’t matter how they’re nested:
  ​	`m.bind(f).bind(g) ~ m.bind(v -> f.apply(v).apply(g))`



Think of monads as an object that wraps a value and allows us to apply a set of transformations on that value and get it back out with all the transformations applied.

Monad in the eyes of Java:

```java
public interface Monad<V> {
	Monad<V> unit(V v);
	<R> Monad<R> bind(Function<V, Monad<R>> f);
	V get();
}
```

These may seems over complicated, but it is really simple considering an example, such as the Optional monad:

```java
Parameterized type: Optional<T>
unit: Optional.of()
bind: Optional.flatMap()
get: Optional.get()

Left Identity: 
Function<Integer, Optional<Integer>> addOne = x -> Optional.of(x + 1);
Optional.of(5).flatMap(addOne).equals(addOne.apply(5))

Right Identity: 
Optional.of(5).flatMap(Optional::of).equals(Optional.of(5))

Associativity:
Function<Integer, Optional<Integer>> addOne = i -> Optional.of(i + 1);
Function<Integer, Optional<Integer>> addTwo = i -> Optional.of(i + 2);
Function<Integer, Optional<Integer>> addThree = i -> addOne.apply(i).flatMap(addTwo);
Optional.of(5).flatMap(addOne).flatMap(addTwo).equals(Optional.of(5).flatMap(addThree));
```

References:

https://stackoverflow.com/questions/2704652/monad-in-plain-english-for-the-oop-programmer-with-no-fp-background
https://dzone.com/articles/whats-wrong-java-8-part-iv
https://www.slideshare.net/mariofusco/monadic-java
https://www.youtube.com/watch?v=nkUafcNWiQE
https://afcastano.com/



#### Back to Optional 

Let's look back at Optional with the knowledge that it's a monad ;-)

> a case of NPE - NullpointerException

```java
Person person = personMap.get("Name");
process(person.getAddress().getCity());
```

which makes you step through the code line by line, trying to answer these questions:

- Where does the null reference come from?
- is null a legal state for that variable? (another eg is map.get(a) could return null because there is a pair <a, null> or it maybe because of a not being present at all)
- Is it maybe the return value of some method which behaves in a crazy way?



So we go for Defensive checking

```java
Person person = personMap.get("Name");
if (person != null) {
  Adress address = person.getAddress();
  if (address != null) {
    City city = address.getCity();
    if (city != null) {
      process(city)
    }
  }
}
```



Now that we know optional we write,

```java
Optional<Person> person = personMap.get("Name");
if (person.isPresent()) {
  Optional<Adress> address = person.getAddress();
  if (address.isPresent()) {
    Optiona<City> city = address.getCity();
    if (city.isPresent()) {
      process(city)
    }
  }
}
```

But this doesn't WORK. We first have to modify the Map, Person, and Address classes so that all their methods return an Optional.



But this is NOT how Optional is intended to be used and those changes are not needed. Optional is a monad and is intended to be use as such:

```java
Optional.ofNullable(person)
        .map(p -> p.getAddress())
        .map(a -> a.getCity())
        .orElse("");
```




On Functors:
https://dzone.com/articles/functor-and-monad-examples-in-plain-java

Monads without using Java 8
http://logicaltypes.blogspot.in/2011/09/monads-in-java.html

Dependency injection using Monads
https://medium.com/@johnmcclean/dependency-injection-using-the-reader-monad-in-java8-9056d9501c75



**Difference between map and flatmap**

They both take a function from the type of the optional to something.

Map applies the function "as is" on the optional you have:

```java
if (optional.isEmpty()) 
	return Optional.empty();
else 
	return Optional.of(f(optional.get()));
```



What happens if your function is a function from `T -> Optional<U>`? Your result is now an `Optional<Optional<U>>`!

That's what flatMap is about: if your function already returns an Optional, flatMap is a bit smarter and doesn't double wrap it, returning Optional<U>. 
It's the composition of two functional idioms: map and flatten.



#### Closures

Lambda expressions do not depend on anything external; instead, they are content relying on their own parameters and constants. Closures, on the other hand, depend on both parameters and constants, and variables in their lexical scope.

```java
//a case of functions as arguments
public static void call(Runnable runnable) {
    System.out.println("calling runnable");
                    
    //level 2 of stack
    runnable.run();
  } 
   
  public static void main(String[] args) { 
    int value = 4;  //level 1 of stack
    call(
      () -> System.out.println(value) //level 3 of stack
    );
  }
```

The closure in this code uses the variable `value` from its lexical scope. If the execution of `main` is in stack level 1, then the execution of the body of the `call` method will be in stack level 2. Since the `run` method of `Runnable` is called from within `call`, the body of the closure runs in level 3. 

Now consider another example:

```java
//a case of returning functions
public static Runnable create() {                   
    int value = 4;
    Runnable runnable = () -> System.out.println(value);
     
    System.out.println("exiting create");
    return runnable;
  } 
   
  public static void main(String[] args) { 
    Runnable runnable = create();
     
    System.out.println("In main");
    runnable.run();
  }
```



You might now be wondering how in the world an execution in one level of stack is able to reach for a variable in another, previous level of the stack—especially without context being passed through the calls. The short answer is that it can't.

Suppose my office is about 10 miles from my home (that's 16 KM for those using evolved units of measure), and I leave for work at 8 a.m. Right about noon I have a short time for lunch, but being health conscious I'd rather have a home-cooked meal. Given my short break, that's only possible if I carried lunch with me when I left home.

That is exactly what closures do: they carry their lunch (or state) with them.

To further clarify, let's look again at the lambda expression within `create`:

```java
Runnable runnable = () -> System.out.println(value);
```

The lambda we wrote does not take any arguments but needs its `value`. Compile the class `Sample` and run `javap -c -p Sample.class` to examine the bytecode. You will notice that the compiler has created a method for the closure, where that method takes an `int` parameter:

```pseudocode
private static void lambda$create$0(int);
    Code:
       0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: iload_0
       4: invokevirtual #9                  // Method java/io/PrintStream.println:(I)V
       7: return
}
```

Now look at the bytecode generated for the `create` method:

```pseudocode
0: iconst_4
1: istore_0
2: iload_0
3: invokedynamic #2,  0              // InvokeDynamic #0:run:(I)Ljava/lang/Runnable;

```

The value `4` is stored into a variable, which is then loaded and passed to the function created for the closure. The closure holds on to a copy of `value` in this case.

That is how closures carry state.



Java doesn't support closures fully but with limitations.

Essentially, you can't do this

```java
int FACTOR = 3;
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
//cannot modify MAX here
list.stream().filter(i ->i*FACTOR).forEach(i -> System.out.print(i+" "));
//not here nor anywhere within in it's scope
```

The reason why this can't be supported is already explained above.



But the following works

```java
private int FACTOR = 3;
public static void main(String[] args) {
	(new Main()).doSomething();				
}
	
public void doSomething() {
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    list.stream().map(comparison()).forEach(i -> System.out.print(i+" "));
    FACTOR = 10 //no problem
}
	
public Function<Integer, Integer> comparison() {
	return new Function<Integer, Integer>() {
		public Integer apply(Integer t) {
			return t*FACTOR;
		}
	};
}
```



FACTOR could also be static int.

But both are a bad practice, a better approach would be to use static final variables and final local variables.

The problem is that this breaks the design contract for Lambdas. Drop Lambdas and go for regular classes in cases you need mutability.



## Compact Profiles

Java SE Embedded 8 introduces a new concept called, Compact Profiles, which enable reduced memory footprint for applications that do not require the entire Java platform. The Java SE 8 `javac` compiler has a new `-profile` option, which allows the application to be compiled using one of the new supported profiles.

There are three supported profiles: `compact1`, `compact2` and `compact3`. These are additive layers, so that each Profile contains all of the APIs in the previous smaller Compact Profiles and adds appropriate APIs on top. 

This is a precursor to JDK 9 modularity.



## Date and Time API

Problems with the old API:

**1) It's not intuitive**

Look at the following example, here a user simply wants to create a Date object for 25th December 2017, 8.30 at night, do you think this is the right way to represent that date in Java?

`Date date = new Date(2017, 12, 25, 20, 30);`

Well, even though it is looking alright, it is not correct. The above code contains two bugs, which is not at all intuitive. If you are using the java.util.Date class then you must know that year starts from 1900 and month starts from zero i.e. January is the zeroth month, not the first.

Here is the right way to declare same date in Java:

`int year = 2017 - 1900;`
`int month = 12 - 1;`
`Date date = new Date(year, month, 25, 20, 30);`

**2) Timezones**

Prior to JDK 8, Java uses String to represent TimeZone, a very bad idea. Ideally, they should have defined a constant or Enum instead of allowing the user to pass String.

`TimeZone zone = TimeZone.getInstance("America/NewYork");`

**3) Calendar**

The Calendar class is also not very user-friendly or intuitive in Java. You just cannot apply common sense or predictive knowledge.

**4) Mutable**

They are mutable. As a result, any time you want to give a date back (say, as an instance structure) you need to return a clone of that date instead of the date object itself (since otherwise, people can mutate your structure).

**5) Not thread safe**


Extensive list of samples of new DateTime API is  here: https://www.baeldung.com/java-8-date-time-intro


Any missed out features in Java 8:

https://www.oracle.com/technetwork/cn/community/developer-day/2-55-new-features-java-se-8-2202551-zhs.pdf

