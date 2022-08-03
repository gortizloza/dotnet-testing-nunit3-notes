# dotnet-testing-nunit3-notes
A brief recap of NUnit 3 and general testing concepts
# NUNIT

## Start from scratch:
1. Add a Class Library Project to the Solution
2. Call it \<SolutionName>.Tests
3. Delete default Class1.cs file
4. Add references to the projects in the solution that are going to be tested. If manually, add references in the .csproj file, e.g.:
``` xml
    <ItemGroup>
      <ProjectReference Include="..\TestMe\TestMe.csproj" />
    </ItemGroup>
```
5. Add NuGet Packages:
	* NUnit
	* Nunit3TestAdapter
	* Microsoft.NET.Test.Sdk

### General
* Tests in NUnit are represented as... &rarr; Methods that get executed.
* Run tests from the command line &rarr; `dotnet test`.
* See available tests in the command line &rarr; `dotnet test --list-tests`.
* Regression defect &rarr; Something that once was working is no longer working.
* Two main parts of the NUnit Test Framework &rarr; NUnit library and Test runner.
* Contains Attributes, assertions, extensibility/customization &rarr; NUnit Library.
* Recognises attributes, execute test methods, report test results, explore tests, dotnet test &rarr; Test runner.
* During test execution NUnit will create only one instance of a Test class regardless of how many methods are inside the class, unlike XUnit that will create a new instance for every test method.
* The logical AAA test phases &rarr; Arrange, Act, Assert.
* Arrange &rarr; Set up test object(s), initialise test data, etc.
* Act &rarr; call method, set property.
* Assert &rarr; compare return value with expected.
* Good unit tests should be &rarr; Fast, Repeatable, Isolated, Trustworthy, Valuable.

### Some attributes
`[TestFixture]` &rarr; Attribute to indicate to NUnit that a class is a test class and contains test methods (is not required from NUnit 3)<br>
`[Test]` &rarr; Mark method as a test<br>
`[Category]` &rarr; Organize tests into categories<br>
`[TestCase]` &rarr;  Serves the dual purpose of marking a method with parameters as a test method and providing inline data to be used when invoking that method<br>
`[Values]` &rarr; Data driven test parameters<br>
`[Sequential]` &rarr; How to combine test data<br>
`[SetUp]` &rarr; Run code before each test<br>
`[OneTimeSetUp]` &rarr; Run code before first test in class<br>

### NUnit Assertions
* **Constraint model** (*newer*) &rarr;<br> Assert.That(test results, constraint instance), e.g. `Assert.That(sut.Years, Is.EqualTo(1))`
	same as:
	`Assert.That(sut.Years, new EqualConstraint(1))`
* **Classic model** (*older*) &rarr;<br> Assert.AssertClassMethod(expected, actual), e.g. `Assert.AreEqual(1, sut.Years)`

### Recognizing testing scenarios
* Adds value, business logic, follow the money
* Excercise all code branches, code coverage: Does every line of code in the product applicaltion gets executed when our test execute?
* Bad data/input, check unexpected input

### Testing for equality
Value types:
```C#
int a = 2;
int b = 2;
Assert.That(a, Is.EqualTo(b));
// Will pass
```
Classes are reference types:
```C#
var a = new LoanTerm(2);
var b = new LoanTerm(2);
Assert.That(a, Is.EqualTo(b)); 
// Will fail unless Equals method is overriden with custom logic.
``` 

Points to the same object in memory (Reference equality):
``` C#
var a = LoanTerm(2);
var b = a;
Assert.That(a, Is.SameAs(b)); 
// Will pass
``` 

###  Custom failure message
* To add a custom failure message to an assertion... &rarr; Add a string as 3rd parameter to the `That()` method, e.g. 
```C#
Assert.That(a, Is.EqualTo(b), "Custom error message");
``` 
*Use with caution, test methods name should provide enough explanation.*

### Floating point values
To add some tolerance when asserting floating point values use &rarr; `Within()` e.g.
``` C#
double a = 1.0 / 3.0;
Assert.That(a, Is.EqualTo(0.33).Within(0.004));
// or use Percent modifier
Assert.That(a, Is.EqualTo(0.33).Within(10).Percent);
```

### Asserting on Collections
Collections contains exactly `n` items &rarr; 
```C#
Assert.That(collection, Has.Exactly(n).Items);
```
Collections contains not duplicate items &rarr; 
```C#
Assert.That(collection, Is.Unique);
```
Collections contains an specific `x` item&rarr; 
```C#
Assert.That(collection, Does.Contain(x));
```
Collections contains an partially known `x` item&rarr; 
```C#
Assert.That(collection, Has.Exactly(1)
	  .Property("PropertyName").EqualTo("x")
	  .And
	  .Property("OtherPropertyName").GreaterThan(0);
```
In a type safe way:
```C#
Assert.That(collection, Has.Exactly(1)
	.Matches<TypeOfItemsInCollection>(
		item => item.PropertyOne == "x" &&
		item.PropertyTwo > 0));
```
### Asserting that Exceptions are thrown
***Scenario:*** <br>Test that the constructor `new Foo(int x)` throws an exception if `x <= 0`.

*Assert.That(Action, Throws.TypeOf<ExpectedExceptionType>())*
```C#
Assert.That(() => new Foo(0), Throws.typeOf<ArgumentOutOfRangeException>());
```
Specifying message
```C#
Assert.That(() => new Foo(0), Throws.typeOf<ArgumentOutOfRangeException>())
		.With
		.Message
		.EqualTo("The expected exception message"));
```
Type safe using the `Matches<T>()` method
```C#
Assert.That(() => new Foo(0), Throws.typeOf<ArgumentOutOfRangeException>())
		.With
		.Matches<ArgumentOutOfRangeException>(
			ex => ex.ParamName == "x"));
```

### Asserting with Date ranges
```C#
DateTime d1 = new DateTime(2000, 2, 20);
DateTime d2 = new DateTime(2000, 2, 25);

Assert.That(d1, Is.EqualTo(d2)); // fail
Assert.That(d1, Is.equalTo(d2).Within(5).Days); // pass
```

### More constraints at:
[NUnit Docs Constraints](https://docs.nunit.org/articles/nunit/writing-tests/constraints/Constraints.html)<br>
[NUnit Constraint Model](https://docs.nunit.org/articles/nunit/writing-tests/assertions/assertion-models/constraint.html)

## Controlling test execution
### Ignoring tests
Use the `Ignore` attribute:<br>
`[Ignore("Custom message: why ignoring")]` &rarr; Ignores a test method or a whole test class. It will show a warning and the message in the text explorer.
### Adding categories to tests
Use `Category` attribute:<br>
* `[Category("Category name")]` &rarr; Adds a category to a test or test class.<br>
* Tests can be group by *Category* in the Test explorer &rarr; *Group by* &rarr; select *Traits*.<br>
* Can tests belong to more than one category &rarr; True
* Using the command line: `dotnet test --filter "TestCategory=Category name"`
### Setting up and cleaning
* `[SetUp]` &rarr; Marks a method to be executed before each test.
* `[TearDown]` &rarr; Executes the decorated method after each test.
* `[OneTimeSetUp]` &rarr; Marks a method to be executed a single time before the first test of the class.
* `[OneTimeTearDown]` &rarr; Run after the last test of the test class.
## Data Driven Tests
#### Providing method level test data
`TestCase` &rarr; To execute the same test with different data.
``` C#
[Test]
[TestCase("test@xmail.com", true)]
[TestCase("__)test@xmail.com", false)]
[TestCase("testxmail.com", false)]
[TestCase("test@xmailcom", false)]
public void ValidateAnEmail(string email, bool expectToBe)
{
    bool isValidEmail = EmailValidationChallenge.ValidateEmail(email, out string _);
    Assert.That(isValidEmail, Is.EqualTo(expectToBe));
}
```
Can be simplified by **removing the assertion**, **changing the return type**, **adding parameter** `ExpectedResult` to the test method and **returning the result**.<br>
 Example:
``` C#
[Test]
[TestCase("test@xmail.com", ExpectedResult = true)]
[TestCase("__)test@xmail.com", ExpectedResult = false)]
[TestCase("testxmail.com", ExpectedResult = false)]
[TestCase("test@xmailcom", ExpectedResult = false)]
public bool ValidateAnEmail_Simplified(string email)
{
    return EmailValidationChallenge.ValidateEmail(email, out string _);
}
```

#### Retrieving data from a centralized location
Add a new class to yield the required test data:
```C#
public class EmailValidationTestData
{
  public static IEnumerable TestCases
  {
    get
    {
	yield return new TestCaseData("test@xmail.com", true);
	yield return new TestCaseData("__)test@xmail.com", false);
	yield return new TestCaseData("testxmail.com", false);
	yield return new TestCaseData("test@xmailcom", false);
    }
  }
}
```
Then the test data class can be use in a test with the `TestCaseSource` attribute:
```C#
[Test]
[TestCaseSource(typeof(EmailValidationTestData), nameof(EmailValidationTestData.TestCases))]
public void ValidateAnEmail_Centralized(string email, bool expectToBe)
{
    bool isValidEmail = EmailValidationChallenge.ValidateEmail(email, out string _);
    Assert.That(isValidEmail, Is.EqualTo(expectToBe));
}
```
As above, the test can be simplified by including the expected returned value in the `TestCases` property of the `EmailValidationTestData` class:
``` C#
public static IEnumerable TestCases
{
    get
    {
        yield return new TestCaseData("test@xmail.com").Returns(true);
        yield return new TestCaseData("__)test@xmail.com").Returns(false);
        yield return new TestCaseData("testxmail.com").Returns(false);
        yield return new TestCaseData("test@xmailcom").Returns(false);
    }
}
``` 
...and in the test: 
```C#
[Test]
[TestCaseSource(typeof(EmailValidationTestData), nameof(EmailValidationTestData.TestCases))]
public bool ValidateAnEmail_Centralized_Simplified(string email)
{
    return EmailValidationChallenge.ValidateEmail(email, out string _);
}
```
#### Reading test data from external sources
Example using a `Data.csv` file:

```
test@xmail.com,true
__^test@xmail.com,false
testxmail.com,false
test@xmailco.m,false
```
Return data as `IEnumerable` (or a proper NuGet package for handling more complex csv files):
```C#
public class EmailValidationCsvData
{
    public static IEnumerable GetTestCases(string csvFileName)
    {
        var csvLines = File.ReadAllLines(csvFileName);
        var testCases = new List<TestCaseData>();

        foreach (var line in csvLines)
        {
            string[] values = line.Split(',');

            string email = values[0];
            bool expectedResult = bool.Parse(values[1]);

            testCases.Add(new TestCaseData(email) { ExpectedResult = expectedResult});
        }
        return testCases;
    }
}
```
And for the test to consume this data:
```C#
[Test]
[TestCaseSource(typeof(EmailValidationCsvData), nameof(EmailValidationCsvData.GetTestCases), new object[] { "Data.csv" })]
public bool ValidateAnEmail_Csv(string email)
{
    return EmailValidationChallenge.ValidateEmail(email, out string _);
}
```
#### Generating test data
* Combinatorial:<br>
The `[Values]` attribute is applied to individual parameters in the test method.
No assertion nor expected result because by default NUnit will create all the possible combinations for the provided values:
``` C#
[Test]
public void CorrectlyCalculateSomeComplexStuff(
	[Values(109_980, 2780_890, 678_000)] decimal firstValue,
	[Values(6.5, 10, 45)] decimal secondValue,
	[Values(10, 20, 30)] int thirdValue)
{
	var sut = new SystemUnderTest();
	
	var result = sut.CalculateSomeComplexStuff(firstValue, secondValue, thirdValue)
```
For this example NUnit will create 27 test cases because there are 27 combinations, so it is not possible to know what the result will be, therefore no assertion is possible.<br>
Use  &rarr; provide a wide range of values to the tested thing to ensure it doesn't throw an exception.<br> 
Override this behavior with:
* Sequential: <br>
This will be equivalent to a test method with 3 TestCase attributes
``` C#
[Test]
[Sequential]
public void CorrectlyCalculateSomeComplexStuff(
[Values(109_980, 2780_890, 678_000)] decimal firstValue,
[Values(6.5, 10, 45)] decimal secondValue,
[Values(10, 20, 30)] int thirdValue)
{
    var sut = new SystemUnderTest();

    var result = sut.CalculateSomeComplexStuff(firstValue, secondValue, thirdValue)
    // sut.CalculateSomeComplexStuff(109980m, 6.5m, 10)
    // sut.CalculateSomeComplexStuff(2780890m, 10m, 20)
    // sut.CalculateSomeComplexStuff(678000m, 45m, 30)
```
