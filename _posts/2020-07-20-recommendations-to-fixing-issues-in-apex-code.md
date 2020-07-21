---
title: Recommendations to fixing issues in Apex code
layout: blog
category: [Salesforce, Best Practices]
excerpt: This blog explains the best practices & its recommended fixes that every apex code should follow. By following these recommendations, you will be able to write clean code everytime and will help make you a excellent programmer.
comments: true
---

## Best Practices
##### Apex Unit Test Class Should Have Asserts
- Ensure that every test method in a test class follows the **Arrange-Act-Assert** methodology
- **Arrange** refers to preparing your test data, **Act** refers to running the code to test with the arranged test data, **Assert** refers to checking the actual values with the expected values
- Every test method should have a purpose (business expectation to be checked). Writing test code to simply increase code coverage without focus on covering business expectations is not desired.
- Test Class should follow the following naming convention. **ClassNameTests**
- Test Methods should follow the following naming conventions. These are my personal favorites.
	- **MethodName_StateUnderTest_ExpectedBehavior** (For example: withdrawMoney_InvalidAccount_ExceptionThrown)
	- **Should_ExpectedBehavior_When_StateUnderTest** (For example: Should_FailToWithdrawMoney_ForInvalidAccount)

##### Apex Unit Test Should Not Use SeeAllData=True
- Every test class/method should be self-sufficient. This means that the test class/method should prepare its own test data in the Arrange phase and use it for Act and Assert.
- Never refer any data in the org (assuming that this data will be available). SeeAllData=True can cause the test to fail if the org in which the test class is run doesn't have that data for any reason.
- Only use SeeAllData=True if you are 200% sure that the data being referred in the test class/method will be available in the org.

##### Avoid Logic In Trigger
- Apex Triggers do not allow you to write methods. Therefore, do not stuff your logic in the trigger. Move it to an apex class (like Trigger Handler).
- This helps better encapsulate the business logic. Ensure to do the encapsulation correctly by following SRP (Single Reponsibility Principle). 

##### Unused Local Variable
- Do not declare/assign variables which is never used in the logic. Unused variables simply consume memory for no purpose and can affect performance.

##### Avoid Global Modifier
- Global classes can never be deleted or changes in its signature cannot be made (especially in managed packages). Therefore, do not use Global modifier unless you need it to be exposed as a web service. Use **public** instead.

##### Apex Assertions Should Include Message
- An assert statement in a test method reporting a **False** means that the test method failed. Getting False in the log is not useful in terms of troubleshooting.
- It is recommended to use the 3rd override of the System.assert which takes in **expected**, **actual**, **message**.
- By using this override, the message will be available in logs for troubleshooting and it adds meaningful content into logs.
- For example:
    ```java
    System.assertEquals('abc', opportunity.StageName, 'Opportunity stageName is wrong. abc is expected.');
    ```

##### Debugs Should Use Logging Level
- The 2 parameter override of System.debug is recommended to be used. The first parameter is LoggingLevel and second parameter is the debug string.
- For example:
    ```java
    System.debug(LoggingLevel.WARN, 'my debug string');
    ```
- Valid log levels are (listed from lowest to highest):
	- NONE
	- ERROR
	- WARN
	- INFO
	- DEBUG
	- FINE
	- FINER
	- FINEST

##### Apex Unit Test Method Should Have IsTest Annotation
- Apex test methods should have **@isTest** annotation. As testMethod keyword is deprecated, Salesforce advices to use @isTest annotation for test class/methods.

## Code Smell

##### Avoid Hardcoding Id
- Record Ids change between environments. Therefore, it is recommended not to hardcode any record Id's (either 15 or 18 digit).
- Always derive the record Id (either by SOQL or by DML) in code and use it.

##### Method With Same Name As Enclosing Class
- Non-constructor methods should not have the same name as the enclosing class. Only constructor method is supposed to be named with the class name.

##### Avoid Direct Access Trigger Map
- Never access any records directly from Trigger.old or Trigger.new (For example: Trigger.new[0] is not recommended).
- Iterate through the Trigger Map and then handle records separately.

##### Empty If Statement
- An empty If statement refers to an existence of If statement that checks a condition but does nothing.
- Remove such If statements as it adds no value to the code.

##### Empty Statement Block
- Any code written inside an enclosing brace is a block. Methods, If-Else, Switch-Case, Try-Catch etc are all blocks.
- A block with no code serves no purpose and should be removed.

##### Empty Try Or Finally Block
- A try-catch block is used if an exception causing condition needs to be handled gracefully.
- An empty TRY or an empty CATCH indicates either that no exception is handled in TRY or an exception is caught in TRY but not handled in CATCH at all. This kind of exception handling completely defeats the purpose of why TRY-CATCH blocks were invented.

## Code Styling

##### If Statements Must Use Braces
- Avoid using if statements without using braces to surround the code block. If the code formatting or indentation is lost then it becomes difficult to separate the code being controlled from the rest.
- Recommended to put braces in if blocks even if there is only one line of code.

##### If-Else Statements Must Use Braces
- Avoid using if-else statements without using surrounding braces. If the code formatting or indentation is lost then it becomes difficult to separate the code being controlled from the rest.
- Recommended to put braces in if-else blocks even if there is only one line of code.

##### Field Declarations Should Be At Start
- Field declarations should appear before the method declarations within a class rather declaring it at random lines of code.
- It ensures that code is readable & organized.

##### Method Naming Conventions
- It is recommended to follow "Camel Case" conventions while naming a method.
- For example: `public void instanceMethod() { }`

##### Local Variable Naming Conventions
- It is recommended to follow "Camel Case" conventions while naming local variables.
- For example: ` Integer localVariable;`

##### Formal Parameter Naming Conventions
- It is recommended to follow "Camel Case" conventions while specifying parameters in a method.
- For example: `public bar(Integer methodParameter) { }`

##### Property Naming Conventions
- It is recommended to follow "Camel Case" conventions while declaring properties.
- For example: `public Integer instanceProperty { get; set; }`
- If your org follows a different convention, you can skip this recommendation.

##### Field Naming Conventions
- It is recommended to follow "Camel Case" conventions while declaring fields.
- For example: `Integer instanceField;`
- Static and Enum fields can have underscores.

##### Class Naming Conventions
- It is recommended to follow "Pascal Case" conventions while naming a class.
- For example: `public class FooClass { }`

## Design

##### Cyclomatic Complexity, Standard Cyclomatic Complexity, Cognitive Complexity
- Writing too much decisional logic in a single method makes its behaviour hard to read and change. This in turn increases the Cyclomatic Complexity.
- Reported methods should be broken down into several smaller methods. Reported classes should probably be broken down into subcomponents.
- With scientific techniques, following are the ranges of complexities.

| Range | Meaning |
|-------|---------|
|5     | Change absolutely required. Behavior is critically broken/buggy|
|4      | Change highly recommended. Behavior is quite likely to be broken/buggy|
|3      | Change recommended. Behavior is confusing, perhaps buggy, and/or against standards/best practices|
|2      | Change optional. Behavior is not likely to be buggy, but more just flies in the face of standards/style/good taste|
|1      | Change highly-optional. Nice to have|

##### Avoid Deeply Nested If Statements
- Deeply nested if-then statements are harder to read and maintain.
- Recommendation is to optimize the decision statements or break down deeply nested logic into smaller methods.

##### Excessive Public Count
- Classes with large numbers of public methods and attributes require disproportionate testing efforts since combinational side effects grow rapidly and increase risk. 
- Refactoring these classes into smaller ones not only increases testability and reliability but also allows new variations to be developed easily.

##### Excessive Parameter List
- Methods with numerous parameters are a challenge to maintain, especially if most of them share the same datatype. 
- These situations usually denote the need for new objects to wrap the numerous parameters into a Request object/Wrapper object.

##### Non Commenting Source Statements Method Count (NCSS Method Count)
- NCSS (Non-Commenting Source Statements) determines the number of lines of code for a given method. NCSS ignores comments, and counts actual statements. 
- Using NCSS, lines of code that are split with spaces are counted as one.
- With NCSS method count, you can evaluate if the method is long enough or not. Usually 40.0 is the threshold.

##### Non Commenting Source Statements Type Count (NCSS Type Count)
- NCSS (Non-Commenting Source Statements) determines the number of lines of code for a given type/sub class. NCSS ignores comments, and counts actual statements. 
- Using NCSS, lines of code that are split are counted as one.
- With NCSS type count, you can evaluate if the sub-class is long enough or not. Usually 500 is the threshold.

##### Excessive Class Length
- Excessive class file lengths are usually indications that the class may be burdened with excessive responsibilities that could be provided by external classes or functions. 
- In breaking these methods apart the code becomes more managable and ripe for reuse. Better not to have class with more than 1 KLOC.

##### Too Many Fields
- Classes that have too many fields can become unwieldy and could be redesigned to have fewer fields, possibly through grouping related fields in new objects.
- For example, a class with individual city/state/zip fields be could parked within a single Address field.

## Performance

##### Avoid Dml Statements In Loops
- Avoid DML statements inside loops to avoid hitting the DML governor limit. 
- Instead, try to batch up the data into a list and invoke your DML once on that list of data outside the loop.

##### Avoid Soql In Loops
- Avoid executing SOQL queries inside loops. This can consume your SOQL governor limits faster.
- Instead, do queries outside the loops and modify the data of the objects in collections.

## Security

##### Apex Sharing Violations
- Classes declared without explicit sharing mode will bypass sharing restrictions when DML statements are executed. This causes the classes to ignore sharing rules specified by administrators.
- Recommended to have developers check DML permissions before the DML is initiated and comply with sharing rules. 

##### Apex CRUD Violation
- Avoid executing SOQL/SOSL/DML assuming that the user context in which the class executes has permissions to do this operation.
- Always check for access permissions before executing CRUD.

##### Apex Open Redirect
- Any redirects to user-controlled locations are potentially dangerous. 
- For example: getting a url path from query string & redirecting the user to that path is an Apex Open Redirect issue. This can cause attackers to send different paths in query string and our code blindly redirects users to phishing sites.

##### Apex XSS From URL Param
- Reading data from URL parameters and using it directly in code is dangerous.
- Always escape/sanitize the data read from URL parameters to avoid XSS attacks.

##### Apex SOQL Injection
- When reading values as arguments and supplying it to generate dynamic SOQL can be dangerous.
- Always escape the data from arguments/variables before generating dynamic SOQL/DML statements.
- For example: `Database.query('SELECT Id from Account'+ whereCondition);`. If the attacker supplies a different value for whereCondition, they can obtain data from the system.

