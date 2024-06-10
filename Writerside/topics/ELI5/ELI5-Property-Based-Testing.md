# Property Based Testing ELI5

Assume you know about regular unit testing, possibly you've heard about the common test. There is another way to check
your system, different from ordinary testing methods. It is called *property based testing*.

To make a test one should define types and ranges for the input data, which the system will take under the test.
Definition of input data looks like a tuple with type and range argument descriptions. They are *data*, and you build
this data before the test.

Then the test engine will call your code with random combinations of defined input data. Test engine tries to observe if
the *system under test* still shows the defined properties.

If at some moment the required property is not observed, test will stop and engine will try to shrink the input. This is
an attempt to find the simplest input data which still breaks the test. Test engine will try to reduce complexity for as
many parameters as possible.

> See Also: Libraries which offer property based testing: [PropEr](https://github.com/manopapad/proper)
> and [Quviq QuickCheck (commercial)](http://www.quviq.com/products/erlang-quickcheck/) (note that QuickCheck also
> exists for other languages!)

## Example

Imagine that your interviewer has asked you to write a `fizz_buzz` function. From the documentation it is clear, that a
good `fizz_buzz` takes an integer argument between 1 and 100. Also it will return `fizz` for arguments that are
divisible by 3, `buzz` for arguments that are divisible by 5, `fizbuzz` for arguments divisible by both 3 and 5.

So the input range: it is an integer between 1 and 100.

And 3 properties we want to observe:

1. When `X rem 15 =:= 0` we want to see `fizzbuzz` as return value.
2. When `X rem 3 =:= 0` we want to see `fizz`.
3. When `X rem 5 =:= 0` we want to see `buzz`.

> Please consult actual documentation for the selected test library. The syntax of the library calls differs.

Test engine will apply random combinations of values from the defined input range.
Test engine will ensure that all 3 properties are observed during the run.

> You can test any black box system this way, without knowing its internals. The *system under test* can be a C program
> or even a real piece of hardware!
>
> It is also possible to run parallel tests (ensure that a parallel system doesn't break when things happen in
> parallel) and state machine tests (ensure that a state machine doesn't break under the random input).
