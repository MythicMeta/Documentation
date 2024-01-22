# 12 TypedArray Parse Function

## What is it?

This is an optional function to supply when you have a command parameter of type `TypedArray`. This doesn't apply to BuildParameters or C2Profile Parameters because you can only supply those values through the GUI or through scripting. When issuing tasks though, you generally have the option of using a modal popup window _or_ freeform text. This `typedarray_parse_function` is to help with parsing freeform text when issuing tasks.

{% hint style="info" %}
When creating a `TypedArray` parameter, the `choices` attribute is the set of options presented to the user in the dropdowns for the modal and the `default_value` attribute is the option that's selected by default when the user adds a new array entry.
{% endhint %}

## Where is it?

This is an optional function you can supply as part of your command parameter definitions.

## What does it do?

A `TypedArray` provides two things from the operator to the agent - an array of values and a type for each value. This makes its way to the payload type containers during tasking and building as an array of arrays (ex: `[ ["int", "5"], ["string", "hello"] ]` ). However, nobody types like that on the command line when issuing tasks. That's where this function comes into play.

Let's say you have a command, `my_bof`, with a TypedArray parameter called `bof_args`. `bof_args` has type options (i.e. the `choices` of the parameter) of `int`, `wstring`, and `char*`. When issuing this command on the command line, you'd want the operator to be able to issue something a bit easier than multiple nested arrays. Something like:

```
my_bof -bof_args int:5 char*:testing wstring:"this is my string"
my_bof -bof_args int/5 char*/testing wstring/"this is my string"
my_bof -bof_args int(5) char*(testing) wstring(this is my string)
...
```

The list of options can go on and on and on. There's no ideal way to do it, and everybody's preferences for doing something like this is a bit different. This is where the TypedArray parsing function comes into play. Your payload type can define or explain in the command description how arguments should be formatted when using freeform text, and then your parsing function can parse that data back into the proper array of arrays.

This function takes in a list of strings formatted how the user presented it on the command line, such as:

```
[ "int:5", "char*:testing", "wstring:this is my string" ]
```

The parse function should take that and split it back out into our array of arrays:

```
[ ["int": "5"], ["char*", "testing"], ["wstring", "this is my string"] ]
```

## When does it get called?

This function gets called in a few different scenarios:

* The user types out `my_bof -bof_args int:5 char*:testing` on the command line and hits enter
  * If the modal is used then this function is _not_ called because we already can create the proper array of arrays from the UI
* The user uses Mythic scripting or browser scripts to submit the task

### What do I do for parse\_arguments or parse\_dictionary functions then?

These functions are essentially the first line of processing that can happen on the parameters the user provides (the first optional parsing being done in the browser). For your typed array parameter, `bof_args` in this case, you just need to make sure that one of the following is true after you're done parsing with either of these functions:

* the typed array parameter's value is set to an array of strings, ex: `["int:5", "char*:testing"]`
* the typed array parameter's value is set to an array of arrays where the first entry is empty and the second entry is the value, ex: `[ ["", "int:5"], ["", "char*:testing"] ]`

After your `parse_arguments` or `parse_dictionary` function is called, the mythic\_container code will check for any typed\_array parameters and check their value. If the value is one of the above two instances, it'll make it match what's expected for the typedarray parse function, call your function, then automatically update the value.

### How can I check that it parsed correctly in the UI?

If you're using the Mythic UI and want to make sure your parsed array happened correctly, it's pretty easy to see. Expand your task, click the blue plus sign, then go to the keyboard icon. It'll open up a dialog box showing you all the various stages of parameter parsing that's happened. If you just typed out the information on the command line (no modal), then you'll likely see an array of arrays where the first element is always `""` displayed to the user, but the `agents parameters` section will show you the final value after all the parsing happens. That's where you'll see the result of your parsing.
