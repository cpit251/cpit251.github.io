---
title: "Unit Testing and Code Coverage"
date: 2023-01-05T22:34:22+03:00
lastmod: 2023-01-07T09:30:51+03:00
weight: 1
draft: false
description: "This lecture note presents an introduction to unit testing in Java using the JUnit framework and generating test coverage report. We’ll go through an example of how to use a CI service (circleCI) with a version control service (GitHub) to automate the process of building, testing your software, and generating coverage report."
---

![https://xkcd.com/1700/](https://imgs.xkcd.com/comics/new_bug.png)


![](https://img.shields.io/badge/coverage-90%25-green)
![](https://img.shields.io/badge/coverage-60%25-orange)
![](https://img.shields.io/badge/coverage-50%25-red)



Unit testing is a form of software testing where individual units of source code (methods and classes) are tested. JUnit is a unit testing framework for the Java programming language with several features that make writing unit test easy. It integrates seamlessly with build tools such as Maven and gradle making it easy to automate running tests. JUnit features a set of annotations to execute a method before running a test (`@Before`), after running a test (`@After`), as well as marking a method as a unit testing method (`@Test`). These annotations are used to facilitate initialization, cleaning up resources, and executing unit tests respectively. JUnit also provides a set of assertions to compare expected and actual output allowing easy check for pass or failure of tests. Example assertions include `assertEquals(expected, actual)`, `assertNotNull(object)`, `assertNotEquals(unexpected, actual)`, `assertTrue(condition)`, `assertDoesNotThrow(executable)`, and [many other powerful assertions](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Assertions.html).

Code coverage is the percentage of the source code lines that gets executed when running a test. For example, suppose you wrote a program with _40_ lines of code and had a set of unit test methods that executed a total of _30_ lines code, then your test has _75%_ coverage.
A program with high test coverage has more of its source code executed during testing. There are several development environment and tools that support generating coverage report for your project. This code coverage report could then be used for assessing code quality of your project.

## JUnit Example App

We start with a very simple calculator example. Start by creating a new Java Maven project in your IDE. For example in Intellij IDEA:
  - Go to **File** -> **New**  -> **Project**
  - Give the project a name (e.g., calculatorApp)
  - Create a Java project with maven using the archetype generator `maven-archetype-quickstart`, which generates a sample Maven project:
  ![create a Java project with Maven](/images/notes/ci/idea-new-maven.png)
 
Create a class named `Calculator`. We will write a method inside this class that returns the sum of two integers.

{{< highlight java "linenos=table, linenostart=1" >}}
public class Calculator {
   public int add(int number1, int number2) {
      return number1 + number2;
   }
}
{{< / highlight >}}

We'll need to add junit 5 as a test dependency to our `pom.xml` file. Open the `pom.xml` file and add the following dependency:

```xml
 <dependencies>
   <dependency>
     <groupId>org.junit.jupiter</groupId>
     <artifactId>junit-jupiter-engine</artifactId>
     <version>5.4.0</version>
     <scope>test</scope>
   </dependency>
 </dependencies>
```
### Writing your first unit test case

You will add a new class `CalculatorTest` for unit testing under `src/test/<your-group-name>/` (replace \<your-group-name\> with the group name you chose when creating a maven project).

In the `CalculatorTest` class, import the Junit `Test` interface and a static assertion method `assertEquals`. We will use the `Test` interface to annotate or mark a method as a unit testing method, and we will use the `assertEquals` to test equality.

{{< highlight java "linenos=table, linenostart=1, hl_lines=1-2 7-10" >}}
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
public class CalculatorTest
{
    private final Calculator calc = new Calculator();
    
    @Test
    void shouldAddTwoNumbers(){
        assertEquals(4, calc.add(2,2));
    }
}
{{< / highlight >}}


## Running the test
To run the test in Intellij IDEA, right click on the test file and select run as shown below:

![running junit test in IDEA](/images/notes/unit-testing/idea-run-unit-test.png)

You can also run the maven test cycle in Intellij IDEA, open the Maven window as shown below:

![running maven test cycle in IDEA](/images/notes/unit-testing/idea-run-maven-test-cycle.png)


## Adding more unit tests
Let's add another method to our **Calculator** class. This method will return the grade letter for a given grade value.

{{< highlight java "linenos=table, linenostart=1" >}}
public String getGradeLetter(int grade) {
    if(grade < 0 || grade >100){
        throw new IllegalArgumentException("Grades can not be below 0 and above 100.");
    }
    else if (grade >=0 && grade < 60){
        return "F";
    }
    else if (grade >=60 && grade <65) {
        return "D";
    }
    else if (grade >=65 && grade <70) {
        return "D+";
    }
    else if (grade >=70 && grade <75) {
        return "C";
    }
    else if (grade >=75 && grade <80) {
        return "C+";
    }
    else if (grade >=80 && grade <85) {
        return "B";
    }
    else if (grade >=85 && grade <90) {
        return "B+";
    }
    else if (grade >=90 && grade <95) {
        return "A";
    }
    return "A+";
}

{{< / highlight >}}


Let's add a unit test method to test this method, the `getGradeLetter(int grade)` method. We are going to use the [`assertEquals(Object expected, Object actual)`](https://junit.org/junit5/docs/5.4.0/api/org/junit/jupiter/api/Assertions.html#assertEquals(java.lang.Object,java.lang.Object)), which asserts that expected object and actual object are equal.

{{< highlight java "linenos=table, linenostart=1" >}}
@Test
void shouldReturnGradeLetters(){
    assertEquals("F", calc.getGradeLetter(0));
}

{{< / highlight >}}

We will also use [`assertThrows`](https://junit.org/junit5/docs/5.4.0/api/org/junit/jupiter/api/Assertions.html#assertThrows(java.lang.Class,org.junit.jupiter.api.function.Executable)) to ensure that execution of a grade below 0 or above 100 always throws an exception of the expectedType (IllegalArgumentException).

To test invalid inputs, we will create a unit test method as follows:

{{< highlight java "linenos=table, linenostart=1" >}}
@Test
void shouldTestWrongGradeValues() {
    assertThrows(IllegalArgumentException.class, new Executable() {
        @Override
        public void execute() throws Throwable {
            calc.getGradeLetter(-10);
            calc.getGradeLetter(101);
        }
    });
}
{{< / highlight >}}


We can also refactor this method using [lambda expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html), which uses the arrow token `->`. This feature was introduced in Java 8.

{{< highlight java "linenos=table, linenostart=1" >}}
@Test
void shouldTestWrongGradeValues() {
    assertThrows(IllegalArgumentException.class, () -> {
        calc.getGradeLetter(-10);
    });
    assertThrows(IllegalArgumentException.class, () -> {
        calc.getGradeLetter(101);
    });
}
{{< / highlight >}}

If you run the test, you should see the test is passing with green check marks.

## Test Coverage Report

Recall that test coverage refers to the percentage of the source code lines that gets executed when running a test.

To run the test with coverage in Intellij IDEA, right click on the unit test file **->** More Run/Debug **->** Run with Coverage.

![run test with coverage in IDEA](/images/notes/unit-testing/idea-run-test-with-coverage.png)

To view the coverage report, open the coverage window as shown below:

![view coverage report in Intellij IDEA](/images/notes/unit-testing/idea-view-coverage-report.png)

If you open the calculator class, you should see the lines that have been executed during the test are highlighted in green and the ones that have not been executed are highlighted in red.

![view coverage report in Intellij IDEA](/images/notes/unit-testing/idea-view-coverage-lines.png)

You can also create a run configuration to run tests with coverage. For more information on how to do that, refer to the [Intellij IDEA documentation](https://www.jetbrains.com/help/idea/running-test-with-coverage.html)


Now, we see that our test coverage is **low (28% of lines)**, as shown in the coverage report. We need to add more test scenarios to the unit test method `shouldReturnGradeLetters`, so our test touches and executes these lines:


{{< highlight java "linenos=table, linenostart=1" >}}
@Test
void shouldReturnGradeLetters(){
  assertEquals("F", calc.getGradeLetter(0));
  assertEquals("D", calc.getGradeLetter(61));
  assertEquals("D+", calc.getGradeLetter(66));
  assertEquals("C", calc.getGradeLetter(71));
  assertEquals("C+", calc.getGradeLetter(76));
  assertEquals("B", calc.getGradeLetter(80));
  assertEquals("B+", calc.getGradeLetter(88));
  assertEquals("A", calc.getGradeLetter(92));
  assertEquals("A+", calc.getGradeLetter(97));
}
{{< / highlight >}}


Re-run the test with coverage, and you should see that the test is coverage is now at 100% for the `Calculator` class.


![Intellij IDEA test coverage result](/images/notes/unit-testing/idea-test-coverage-result.png)



## Test and Coverage Automation
We can automate running unit testing and coverage report. Test automation can be part of continous integration (CI) pipeline. There are many CI tools out there that facilitate creating testing and coverage pipeline. Popluar (CI) services include [circleci]() and [Travis CI](https://www.travis-ci.com/)

There are also a number of services that can read coverage reports, calculate the coverage percentage, and generate a badge that you can place in your README file of your project. Populare code coverage services include [coveralls.io](https://coveralls.io/) and [codecov](https://about.codecov.io/). These tools can show you which parts of your code aren’t covered by your test suite.

For more information, refer to the [continuous integration (CI) lecture note.](/notes/continuous-integration/).


#### Questions
- The current project has 100% code coverage for the `Calculator` class. Does this mean it's free of bugs? Why?
- Why is it important to choose a diverse set of test scenarios even if 100% code coverage is achieved?
  - **Hint:** Change the add method to return the result of multiplying two numbers instead of adding them.
  {{< highlight java "linenos=table, hl_lines=3, linenostart=1" >}}
public class Calculator {
   public int add(int number1, int number2) {
      return number1 * number2;
   }
}
{{< / highlight >}}

  - The test below is still passing:
  {{< highlight java "linenos=table, linenostart=1, hl_lines=9" >}}
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
public class CalculatorTest
{
    private final Calculator calc = new Calculator();
    
    @Test
    void shouldAddTwoNumbers(){
        assertEquals(4, calc.add(2,2));
    }
}
{{< / highlight >}}

-----
The source code for this example is available on [GitHub](https://github.com/cpit251/unit-testing-coverage-demo).