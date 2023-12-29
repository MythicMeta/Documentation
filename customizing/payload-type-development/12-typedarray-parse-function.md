# TypedArray Parse Function



## What is it?

This is an optional function to supply when you have a command parameter of type `TypedArray`. This doesn't apply to BuildParameters or C2Profile Parameters because you can only supply those values through the GUI or through scripting. When issuing tasks though, you generally have the option of using a modal popup window _or_ freeform text. This `typedarray_parse_function` is to help with parsing freeform text when issuing tasks.



## Where is it?

This is an optional function you can supply as part of your command parameter definitions.



## What does it do?

A `TypedArray` provides two things from the operator to the agent - an array of values and a type for each value. This makes its way to the payload type containers during tasking and building as an array of arrays (ex: `[ ["int", "5"], ["string", "hello"] ]` ). However, nobody types like that on the command line when issuing tasks. That's where this function comes into play.

Let's say you have a command, `my_bof`, with a TypedArray parameter called `bof_args`. `bof_args` has type options of `int`, `wstring`, and `char*`. When issuing this command on the command line, you'd want the operator to be able to issue something a bit easier than multiple nested arrays. Something like:

```
my_bof -bof_args int:5 char*:testing wstring:"this is my string"
my_bof -bof_args int/5 char*/testing wstring/"this is my string"
my_bof -bof_args int(5) char*(testing) wstring(this is my string)
...
```

The list of options can go on and on and on. There's no ideal way to do it, and everybody's preferences for doing something like this is a bit different. This is where the TypedArray parsing function comes into play. Your payload type can define or explain in the command description how arguments should be formatted when using freeform text, and then your parsing function can parse that data back into the proper array of arrays.
