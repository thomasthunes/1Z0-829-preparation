# Beyond Classes

## Interface
```java
public abstract interface Run { //abstract implicit modifiers

  public abstract void run();  //public abstract implicit modifiers
}
```
Concise form:

```java
public interface Run {
  void run();
}
```
An interface cannot be marked as final
```java
//does not compile
public final interface Walk {}
```
Interface variables are implicitly **public, static, final.**
```java
public interface Weight {
  int max_weight = 5;
  public static final int max_height = 50;  
  String color;  //does not compile as it's not initialized
}  

```

### Interfaces with the same default name
Class which implements two interfaces having the same(signature) default method:
```java
@Override
public void go() {
    Run.super.go();
    //or
    Walk.super.go();
}
```
[Full Example](../src/main/java/org/enricogiurin/ocp17/book/ch7/interfaces/defaultmethods/InheritTwoDefaultMethods.java)
#### Case abstract class
[Abstract class default methods](../src/main/java/org/enricogiurin/ocp17/book/ch7/interfaces/defaultmethods/AbstractClassDefaultMethods.java)

### Defualt methods in an interface
1. A default method may be declared only within an interface.
2. A default method must be marked with the default keyword and include a method body.
3. A default method is implicitly public (cannot be marked private or any other access modifiers.).
4. A default method cannot be marked abstract, final, or static.
5. A default method may be overridden by a class that implements the interface.
6. If a class inherits two or more default methods with the same method signature, then the class must override the method.

#### default conflict abstract method
```java
interface House {
  default void work(){}  //default method
}

interface Office {
  void work(); //abstract method
}

class HomeOffice implements Office, House {
  //I am forced to implement work()!
  @Override
  public void work() {}
}
```
[default vs abstract conflict](../src/main/java/org/enricogiurin/ocp17/book/ch7/interfaces/defaultmethods/DefaultVsAbstractConflict.java)
### private methods
A private interface method **cannot** be called in a method outside the interface declaration.

```java
interface InterfaceWithAPrivateMethod {
  private void message();
  void hello();
}

class InterfaceWithAPrivateMethodImpl implements InterfaceWithAPrivateMethod {
  @Override
  public void hello(){
    //does not compile!
    //message();
  }
}
```

### Override
`@Override` annotation informs the compiler that the element is meant to override an element declared
in a superclass and/or interface. While **it is not required** to use this annotation when overriding a method, it helps
to prevent errors.

### Default Methods
First, if a class or interface inherits two interfaces containing default methods with the same
signature, it must override the method with its own implementation.

[Inherit two default methods with the same signature](../src/main/java/org/enricogiurin/ocp17/book/ch7/interfaces/defaultmethods/InheritTwoDefaultMethods.java)

#### Methods accessible by default method
A default method can invoke any other type of method within the interface:
- public static methods
- private static methods
- public instance methods
- private static methods
- other default methods

[DefaultMethodCallingOtherMethods](../src/main/java/org/enricogiurin/ocp17/book/ch7/interfaces/defaultmethods/DefaultMethodCallingOtherMethods.java)
### static methods

```java
interface Certifications {
  //public by default
  public static void ocp17() {}
}

class MyCertifications implements Certifications {
  public static void main(String[] args) {
    Certifications.ocp17();  //valid
    MyCertifications myCertifications = new MyCertifications();
    //I cannot invoke a static method of the interface from an instance of the class which implements the interface!
    myCertifications.ocp17();  //DOES NOT COMPILE!
  }
}
```

[interface with static methods](../src/main/java/org/enricogiurin/ocp17/book/ch7/interfaces/InterfaceWithStaticMethods.java)
## Sealed
A sealed class requires at least one subclass to extend.   

### Rules
 - A sealed class needs to be declared in the same package or named module as their direct subclasses.
 - Direct subclasses of a sealed classes must be either `final` or `sealed` or `non-sealed`.
 - The permits class is optional if the classes (sealed and subclasses) are declared in the same file.

#### Same file
The `permits` clause is optional if the subclass is nested or declared in the same file.

```java
//same file
public sealed class Snake {}
final class Cobra extends Snake {}
```
### Sealed interfaces
An interface can be declared sealed. The `permits` list can apply to a class that implements the interface 
or an interface that extend the interface.   
Then a sealed interface can be extended only by:
- `sealed` interfaces
- `non-sealed` interfaces.

```java
// Sealed interface
public sealed interface Pet permits Cat, Dog, Rabbit {}

// Classes permitted to implement sealed interface Pet
public final class Cat implements Pet {}
public final class Dog implements Pet {}

// Interface permitted to extend sealed interface pet
public non-sealed interface Rabbit extends Pet {}
```
[Example of sealed interface](../src/main/java/org/enricogiurin/ocp17/book/ch7/sealed/interfaces/Pet.java)

### Different files
```java
public sealed class Snake permits Cobra, Viper {}
//separated files, but same pacakge
final class Cobra extends Snake {}
non-sealed class Viper extends Snake {}
```
```java
//same file
//no permits needed 
sealed class HumanBeing {}

sealed class Male extends HumanBeing {}
non-sealed class EuropeanMale extends HumanBeing {}
final class AsianMale extends HumanBeing {}
```
A subclass (Male) of a sealed class (HumanBeing) must be marked either `final` or `sealed` or `non-sealed`.
### Modules
_Named Module_: which allow sealed classes and their direct subclasses in different packages, 
provided they are in the same named module.

### sealed nested
```java
  sealed class Pet {
    public final class Dog extends Pet {}
  }
```
[sealed nested](../src/main/java/org/enricogiurin/ocp17/book/ch7/sealed/SealedNested.java)

## Final
```java
final class AFinal {
  public static void main(String[] args) {
    //I can't override a final class
    //new AFinal() {}; //does not compile
  }
}
```
## Record
1. Records are implicitly final.
2. Record instances are immutable.
3. A record may have any number of overloaded constructors.
4. A record may not extend other classes, but can implement any number of interfaces.

### Variables in Records
- A record can not define instance fields
    - No instance variables
    - No instance initializers
```java
public record Truck(int maxTonnage) implements Donut, Donut2 {
    // Static variables are allowed
    static int age = 10;
    static {age = 11;}
    
    // Does not compile
    int id = 3;
    {this.maxTonnage = 20;}
```

### Compact Constructor
```java
public record Person(String firstName, String lastName) {

  //this is a compact constructor
  public Person {  //no parenthesis!!!
    if (firstName.isEmpty() || lastName.isEmpty()) {
      throw new IllegalArgumentException("invalid");
    }
  }
}
```
A compact constructor cannot set an instance variable through `this` but just with the normal assignment operator.
```java
public record Name(String name) {
  public Name {
    name = "Enrico"; //this works
    //this.name = "Enrico"; //this does not compile
  }
}
```
A compact constructor must have the same access modifiers as the record itself.

```java
//does not compile
public record Name(String name) {
  Name {
    name="John";
  }
}
```

### Overloaded constructor
```java
public record Person(String firstName, String lastName) {

  //this is an overloaded constructor 
  public Person() {  
    //The first line must be a call to another constructor,
    this("Enrico", "Giurin");
  }
}
```
### Record with instance variables
```java
record Game() {
  //does not compile
   final int score = 10;
}

```
It does not compile because records cannot include instance variables not listed in the declaration of the record, 
as it could break immutability.

### Record with instance initializers
Instance intializers are not allowed, while static initializers are allowed.
```java
record Sheap(){
    // Does not compile
    {int i = 0;} 

    // Compiles successfully
    static {int i = 0;}
}
```

## enum
1. An enum can be defined inside a constructor or method, including main().
2. If defined in a method or constructor, the enum can not have access modifiers.
```java
public class EnumContainer {
    EnumContainer() {
        enum Switch{ON, OF}
        //does not compile
        private enum Switch1{ON, OF}
    }
    void m1(){
        enum Weather{SUN, CLOUDY, RAINY, SNOWY}      
        //does not compile
        public enum Weather1{SUN, CLOUDY, RAINY, SNOWY}
    }
    public static void main(String[] args) {
        enum Dirs{NORTH, EAST, SOUTH, WEST}
        //does not compile
        protected enum Dirs{NORTH, EAST, SOUTH, WEST}
    }
}
```

### Abstract method
If an enum contains an abstract method, then every enum value must include an override of this abstract method.
```java
public enum Gender {
  MALE {
    @Override
    public String description() {
      return "male";
    }
  }, FEMALE {
    @Override
    public String description() {
      return "female";
    }
  };
  public abstract String description();
}
```
### Constructors
enum constructors are implicitly private!
```java
  public SeasonWithValues(String description) {  //DOES NOT COMPILE!
    this.description = description;
  }
```

### Access to static field not final
It is illegal to access (not final) static member from enum constructor or instance initializer.
```java
  enum Api {
    UPDATE("update the data"), READ("read the data");
    private String description;
    private static String version = "2.0";
    private static final String firstVersion = "1.0";

    Api(String description) {
      this.description = description;
      System.out.println(firstVersion);  //this is fine
      //It is illegal to access (not final) static member 'version' from enum constructor or instance initializer
      //( JLS 8.9.2 )
      //System.out.println(version);  //does not compile
    }
  }
```
[Enum With Fields](../src/main/java/org/enricogiurin/ocp17/book/ch7/useofenum/SeasonWithValues.java)

### Comparable
all the enum implements `Comparable`.
```java
    int compare = CardinalPoints.NORTH.compareTo(CardinalPoints.EAST);

    //so they can be used in a TreeSet
    TreeSet<CardinalPoints> set = new TreeSet<>();
    set.add(CardinalPoints.SOUTH);
    set.add(CardinalPoints.NORTH);
    set.add(CardinalPoints.WEST);
    System.out.println(set); //[NORTH, WEST, SOUTH]
```
### Clone
- The clone method can not be overridden in an enum because java.lang.Enum (which every enum extends) defines clone as final.
```java
public enum EnumA{
  A, AA, AAA;
  public Object clone(){ // Does not compile
    return B
  }
}
```

## Nested Classes
### Inner Class
Reference: `Outer.this.field`.
```java
class University {
  int id;
  class Department {
    int id;
    class Professor {
      int id;
      void printID() {
        System.out.println("University id: " + University.this.id);
        //System.out.println("University id: "+this.id); //equivalent
        System.out.println("Department id: " + University.Department.this.id);
        // System.out.println("Department id: "+Department.this.id); //equivalent
        System.out.println("professor id: " + University.Department.Professor.this.id);
        //System.out.println("professor id: "+Professor.this.id);//equivalent
      }
    }
  }
}
```
[University](../src/main/java/org/enricogiurin/ocp17/book/ch7/nested/inner/University.java)
### Nested Local Class
A local class can access only final and effectively final local variables.
```java
  public void sortArray() {
    int size = 0;
    class NestedLocal {
      public void sort() {
        //size is not effectively fianl, as it's changed in the last line
        int c = size;  //does not compile!
      }
    }
    //here I modify size, so I make it not effectively final
    size = 0;
  }
```
[Nested Local Class](../src/main/java/org/enricogiurin/ocp17/book/ch7/nested/local/ArraySorter.java)
