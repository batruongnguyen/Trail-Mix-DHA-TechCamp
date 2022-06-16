# **[Write Negative Tests](https://trailhead.salesforce.com/content/learn/modules/unit-testing-on-the-lightning-platform/negative-tests)**

## Write positive and negative unit tests

Everyone loves a good calculator. In your Trailhead Playground is the calculator.cls. Write positive and negative unit tests for the methods found inside.
- Create a new class named Calculator_Tests
- Write positive and negative tests for the Calculator class
- Be sure to use @testSetup methods where appropriate
- Run your unit tests and confirm that the code coverage for calculator.cls is 100%

**Class**

*Calculator_Tests.apxc*
```java
@isTest
   public class Calculator_Tests {

@isTest
 public static void addition() {
    Calculator.addition(1, 0);
   }
@isTest
  public static void subtraction() {
    Calculator.subtraction(1, 0);
   }

@isTest
 public static void divide_throws_exception_for_division_by_zero() {
 Boolean caught = false;
 try {
    Calculator.divide(1, 0);
  } catch (Calculator.CalculatorException e) {
    System.assertEquals('you still can\'t divide by zero', e.getMessage(), 
  'caught the right exception');
    caught = true;
   }
   System.assert(caught, 'threw expected exception');
   }

  @isTest
 public static void divide_throws_exception_for_division_by_two() {
 Boolean caught = true;
 try {
    Calculator.divide(1, 2);
 } catch (Calculator.CalculatorException e) {
    System.assertEquals('you still can\'t divide by zero', e.getMessage(), 
  'caught the right exception');
    caught = true;
   }
   System.assert(caught, 'threw expected exception');
 }


@isTest
public static void multiply_by_one() {
  Boolean caught = false;
  try {
    Calculator.multiply(1, 0);
    } catch (Calculator.CalculatorException e) {
    System.assertEquals('It doesn\'t make sense to multiply by zero', 
    e.getMessage(), 'caught the right exception');
     caught = true;
    }
    System.assert(caught, 'threw expected exception');
  }

@isTest
 public static void multiply_by_two() {
  Boolean caught = true;
  try {
     Calculator.multiply(1, 2);
   } catch (Calculator.CalculatorException e) {
    System.assertEquals('It doesn\'t make sense to multiply by zero', 
  e.getMessage(), 'caught the right exception');
    caught = true;
   }
   System.assert(caught, 'threw expected exception');
}   
       
       @isTest
    public static void divide_throws_exception_for_negative_number() {
        Boolean caught = true;
        try {
            Calculator.divide(-1, 2);
        } catch (Calculator.CalculatorException e) {
            System.assertEquals('negative value(s) not allowed.',e.getMessage());
            caught = true;
        }
        System.assert(caught, 'threw expected exception');
    }
}
```

*Calculator.apxc*
```java
public class Calculator {
public class CalculatorException extends Exception{}

public static Integer addition(Integer a, Integer b){
    return a + b;
    }

public static Integer subtraction(Integer a, Integer b){
    return a - b;
    }

public static Integer multiply(Integer a, Integer b){
    if(b==0 || a==0){
        throw new CalculatorException('It doesn\'t make sense to multiply by zero');
    }
        return a * b;
    }

public static Decimal divide(Integer numerator, Integer denominator){
    if(denominator == 0){
        throw new CalculatorException('you still can\'t divide by zero');
    }
    if(numerator < 0 || denominator < 0)
        throw new CalculatorException('negative value(s) not allowed.');

    Decimal returnValue = numerator / denominator;

    return returnValue;
    }
}
```