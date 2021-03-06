# R judge for Dodona

> Note: this judge makes extensive use of R environments to separate student code from test code. If you are not familiar with environments, it might be useful to read [this](http://adv-r.had.co.nz/Environments.html).

## A basic exercise

The test file for a basic exercise can look like this:

```r
context({
  testcase('The correct method was used', {
    testEqual("test$alternative", function(env) { env$test$alternative }, 'two.sided')
    testEqual("test$method", function(env) { env$test$method }, ' Two Sample t-test')
  })
  testcase('p value is correct', {
    testEqual("test$p.value", function(env) { env$test$p.value }, 0.175)
  })
}, preExec = {
  set.seed(20190322)
})

context({
  testcase('x has the correct length', {
    testEqual("length(x)", function(env) { length(env$x) }, 100)
  })
})
```

Let's unpack what happens here.

### Tabs

First of all, something you can't see in the example code above. Dodona groups contexts in tabs. These are represented in the R judge by the files containing the test code. The name of the file (without the `.R`) extension is used to name the tab. A file should contain one or more calls to `context`. Tabs are ordered lexicographically by their filename. To make sure that tabs can be in a logical order, leading digits followed by a dash (`-`) are also stripped from the filename.

### Contexts

A context represents one execution of the student code. It is generally desirable to have as many contexts as possible, since students can filter by incorrect contexts. The `context` function does a few things:

 1. It creates a clean environment based on the global environment. Students have access to the global environment, but don't have access to the testing code or variables used in the testing code (the testing code is executed in a similar environment that is never bound).
 2. It executes the code passed through the `preExec` argument in this clean environment. This can be used for setting the seed (as in this example), but also to set variables or define functions that students can then use.
 3. It executes the student's code in the clean environment. If the code errors out or generates warnings/messages these are caught and handled. An error will interrupt the execution and set the `runtime error` state for the the submission. It also adds a message to the context containing the error message generated by R. Warnings and messages do not interrupt execution, but are also added as messages to the context.
 4. It executes the first argument (a code block containing testcases) in the test environment.

Note that the student code is executed once for each call to `context`. Technically, this allows the student to store intermediate results in the global environment. The use of this is limited, so we don't see this as a problem.

An extra `contextWithImage` function also exists. This function takes the same arguments, but adds an image to the output if it was generated by the student while their code is executed. By default, this function will make the output wrong if the expected image wasn't generated. This behaviour can be changed by setting the optional `failIfAbsent` parameter to `FALSE`.

For introductory exercises students often use R as a calculator and do not store the result of an expression as a variable in their script. For such scripts the eval function that executes the parsed script of the student does not store this result as a variable in the test environment. However, it simply returns the value to the parent environment. This value is now catched and assigned to the variable .Last.value in the test environment and can be accessed using env$.Last.value. This enables us to evaluate scripts that are expected to output a result to the console. 


### Testcases

Testcases group a number of related tests. The first argument of the `testcase` function is a description of that related group. The second argument is a code block (containing tests) which will be executed by the `testcase` function. There is little functionality in the `testcase` function. It is mostly used as a wrapper for the Dodona concept.

### Tests

A test is an actual evaluation of correctness. Multiple `test*` functions are available and are explained in more detail below. The only constant thing for tests are the first three arguments:

 1. A description of the test. Preferably, this is something the student can copy-paste into their local R environment (e.g. `length(x)`, `test$p.value`, etc.).
 2. A function extracting the value to be tested from the student's environment. This function should take one argument (`env`) and return a value.
 3. The expected value. This expected value is compared to the value extracted by the second argument.

#### `testEqual`

The `testEqual` function uses the `base::all.equal` function internally to determine whether the two values are equal. Any parameters that can be passed to `all.equal` can be passed to `testEqual` (but the first three arguments need to be as described above). In addition, one can pass a `comparator` argument to `testEqual`. This `comparator` should be a function that takes two arguments (`generated` and `expected`, in that order) and returns `TRUE` or `FALSE`. If this argument is passed, the comparator is used instead of `all.equal`. Any named arguments passed to `test_equal` that are not known by `testEqual` are passed to `all.equal` or `comparator` depending on what is used.

#### `testIdentical`

The `testIdentical` function use the `base::identical` function internally to determine whether the two values are equal. Any parameters that can be passed to `identical` can be passed to `testIdentical` (but the first three arguments need to be as described above).

#### `testImage`

The `testImage` function is a special case, since it won't actually add a test to the output. Instead, it only expects one argument: a function taking the environment, that will generate an image when called. By default, this function will make the output wrong if the expected image wasn't generated. This behaviour can be changed by setting the optional `failIfAbsent` parameter to `FALSE`.

