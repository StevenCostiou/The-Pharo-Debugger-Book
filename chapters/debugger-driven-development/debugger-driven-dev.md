# Debugger driven development

## Semi-automatic code generation in the debugger 

When writing tests, you often want to write a test about the result of a method but sometimes you do not know what exactly the result should be. 
For instance, if the method call performs complex computations, it is complicated to know in advance what the result should be.
Because of that, it can be difficult to write assertions.

For these reasons, we introduced a system to generate assertions on the fly based on the result of an expression.

To use it, first, you need to write a test with the assertion `see:`.
This assertion takes as argument the expression whose result you're interested in:

<img src="./graphics/see-assertion-before-rewriting.png" alt="Test with `see:` assertion, used to generate assertions" width="250" height="auto">

Then, when you run the test, the assertion will raise an exception.
This will open a debugger that will handle the exception to rewrite the assertion into an `assert:equals:` with the result of the tested expression as second argument (1 in the figure below):

<img src="./graphics/see-assertion-after-rewriting.png" alt="Test generated in the debugger with `see:` assertion, when running the test" width="700" height="auto">

You can then judge by yourself if the actual value is correct or not.

According to your judgement, you can either: 
- click on **Gen&Proceed** in the debugger action bar (2 in the figure above) to confirm the code rewriting and proceed your program.
- close the debugger, which will cancel the code rewriting. You can then fix the test (or the tested code) until you are satisfied with the generated code.

For now, the only exception allowing to automatically rewrite code allows to generate `assert:equals:` with literals.
However, the system has been designed to allow exceptions to generate any type of code, not solely tests.