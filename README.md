
# **Qwick Engineering Coding Guide**

# **Purpose**

This is a living document that is meant to showcase how the engineering team at Qwick thinks about writing code, why we think the way we do, what benefits and drawbacks our approach yields, and what are appropriate exceptions for deviating from the standard. This document is meant to be a guide and a heuristic for writing code and understanding the thought process behind why and how decisions were made. It is not meant to be infallible and immutable and will change over time but most of the guiding principles will most likely stay the same.

# **Philosophies and Principles**

## Falling into the Pit of Success

The system as a whole should make the correct decision obvious to the developer and wrong ones difficult to make. This will force the developer to consistently make good decisions despite their best efforts to the contrary. Things such as smart defaults, CI scripts preventing bad behavior, well typed interfaces (no strings\!), APIs that strongly encourage the correct path, etc… are some of the tools that we use to help guard the code repo against the fallibility of humans. 

## Simple Over Easy

Most people think that something being simple is the same thing as being easy (and vice versa). We believe that it is not. The difference between simple and easy is that simple is an absolute metric, and easy is relative to each person. [Rich Hickey has an excellent talk](https://www.infoq.com/presentations/Simple-Made-Easy/) describing the difference.

We define “simple” by counting the number of moving parts in something. A thing is simpler than another thing if it has fewer moving parts. In code, this could be the number of arguments accepted by a function, whether or not the function is referentially transparent, the complexity of the calculation done inside of the function, etc… In this situation, we are able to quantify our domain and since we can measure it, we can track it and improve it over time.

This is opposed to something being “easy”. When somebody says that something is “easy” they mean that it’s not difficult for them to do. However, easy doesn’t translate well to another person. For example, it’s easy for a native english speaker to read, write, and speak english. However, for somebody that is not familiar with the English language, it would certainly not be easy for them to understand it. So for something to be easy, it depends on many things that change from person to person and does not translate into a very quantifiable domain. Thus, it is a difficult thing to measure and improve upon over time.

An example argument often used for making things “easy” is code readability. Code readability is highly subjective depending on the background of the developer and is a very difficult thing to optimize for and get right. Everybody has their own opinions and in a growing team, you won’t be able to satisfy everyone. Instead of tasking ourselves with an impossible problem, we instead will optimize for making our code simple over easy.

When we are faced with making a solution that has to be either simple or easy, we will lean towards choosing simple over easy. This gives us a way to make things progressively better in the future and helps prevent us from over-optimizing for the current skill-set of the team.

## Pure By Default

In software development, there is a concept of “Referential Transparency” (a.k.a. “Purity”) which means that a piece of code could be replaced with its result and the behavior of the software would be the same. The leverage of this concept results in code that requires less effort to reason about, debug, maintain, and extend. This concept, done properly, will ultimately result in a codebase whose complexity scales linearly with the number of features instead of exponentially. 

We say pure by default because there are a few cases where the impure solution will make more sense than the pure solution. For example, a section of code that is not properly configured for a pure solution and a hotfix is required. It would make more sense to implement the impure solution to get a fix out and then come back later and upgrade that part of the code to be optimized for the pure solution. As long as all parties are aware of the situation and (this is critical) are also ok with coming back later and improving that part of the codebase to be pure, this is an acceptable solution.

An example of a situation where an impure solution would last forever would be a section of code that would require an impure solution to meet performance requirements. Before reaching for an impure solution, make sure you have confirmed **with data** that this is a section of code that would require such a thing and that there is no practical way of implementing a pure solution. Make sure to socialize the problem, associated data, and potential solutions before moving forward.

Another example of a situation where an impure solution would last forever would be IO boundaries such as a button’s onClick event. In these cases we should formalize the boundary using the type system and accept a type that would allow us to execute the impure code in a controlled manner.

Being pure by default, ties back into our principle of choosing the simple solution over the easy solution and helps developers fall into the pit of success.

## Explicit Over Implicit

As a software engineer, the majority of your time will be spent reading and understanding code that was written by other people. As such, you should write your code such that things never “work by coincidence” or do superfluous things. Code should be direct, to-the-point, and with purpose and intent. No developer that is familiar with the system should be surprised by the behavior of a block of code. If something happens that is surprising, then the code is not explicit enough and should be updated.

An example of this would be leveraging the type system to restrict something to its valid use-cases. Such as a primary key for a table shouldn’t be passed around as an integer value. Even though a primary key is represented by an integer at a low level, the primary key of a table only makes sense in a subset of the domain of things you can do with an integer. It doesn’t make sense to multiply a primary key, add 2 primary keys together, or take the square root of one. Thus, we can be explicit with the usage of such a value by putting it in its own type and then making functions that use that type in the ways that it makes sense (such as looking up the row associated with that primary key). This makes the existence and usage of such a thing explicit and obvious.

Another example would be destructuring values out of function arguments instead of referencing them indirectly later in code. By pulling out all of the values used in a block of code in the function signature, you are explicitly showing how many values are actually used in a given block of code and will allow the reader to understand the level of complexity more easily by just reading the function signature. If something is not needed, then it can be removed from the destructuring and thus would allow the function signature to explicitly show that the code block it contains has become less complex. If such values were not referenced until they were used, you would have to read and understand the entire code block to gain insight into how complex it is.

Being explicit instead of implicit is an example of simple over easy. It can be easy to have your code “work by coincidence” but it’s much simpler to make it explicit.

## Rule of Least Power

This is simply to lean towards using the least powerful thing that gets the job done. When you are writing code, always be thinking about using something that has less features, is a higher level of abstraction, or is less powerful than what you are currently using. In other words, the code isn’t done when there is nothing left to add, the code is done when there is nothing left to take away.

For example, If you are writing something using a for loop and that can be expressed using a forEach then the forEach should be used. If that forEach could be replaced with a flatMap then use that. If that flatMap could be replaced with a map then that’s the better choice. Each step in our example is a step that allows the user less flexibility in what they can do. This seems counterintuitive but the reasoning behind it is that not only are you explicitly showing what you are intending to do, but you are also explicitly showing your intent to NOT do something.

What we can derive from code is inversely proportional to what that code is able to do. If that code can do anything and everything that it wants, we can derive nothing. If it’s very limited in scope, we are able to anticipate more about the developers intent and we can do it much faster. The quicker we can reason about a block of code the faster we move. The deeper our reasoning about a block of code, the less mistakes we make. The less mistakes we make, the faster we move. 

Simplify your code, and explicitly show your intent and what you mean to do by using things that make you use less.

## Don’t prematurely optimize

A trap that a lot of people fall into is thinking that something needs to be fast when it really doesn’t. This mentality is a hold-over from the late 20th Century when computing power was much less affordable than it is now. The problem with this assumption is that when you optimize code for performance, you are reducing its ability to be maintained and extended in the future. In practice, code is more often in a state of needing to be maintained and extended than it is in a state of having to be as fast as possible.

The complexity of a codebase where everything is optimized for performance scale very poorly and will dramatically slow down the team. Our solution is to confirm, with data, the paths that require performance tuning and to limit our optimizations to those paths. 

Our main exception to this is code that is running on the database. The usage of a relational database will result in that part of the system being the main bottleneck as you scale. There is already evidence proving this to be the case so we don’t need to provide more data. As such, code that runs in the DB should be constantly analyzed and optimized to provide the best UX across the platform.

# **Coding Standards**

This section is meant to be a collection of standards that we should apply in our codebase. For each subsection we will talk about the why, how, when, and known exceptions for each standard. Examples will be given when appropriate but the list of examples will not be exhaustive.

## Side Effects Should be Contained in a LogIO Monad

### Why?

Containing side-effects inside of a LogIO monad reduces the complexity of our codebase. It lifts side-effects into an environment where we can compose them with other side-effects and control their execution more precisely. It also formalizes their existence in the type system and allows us to leverage the compiler to help with knowing which paths of code contain a side-effect.

When doing this you often remove the existence of race conditions inside of your code, timing issues and bugs related to time are also often removed. In addition, it will maintain a pure FP mindset since side-effects that are contained in an IO are pure and we can control when they are being used.

LogIO’s are also lazy, which means that we can code proactively and not reactively. This means that we are not reacting to what the system has done, but instead we can write code that will anticipate things that might happen and cover those cases. For example, when using a JavaScript Promise (which is a non-lazy and very close to a monad), you can only write code that deals with the result of something, you can’t write code that takes a promise and re-executes it. This makes something like a “retry on network error” system very difficult to implement in a non-lazy environment.

When using LogIO, you are also able to write to a logging system that executes out-of-band with the IO which allows us to add logging to performance “hot-spots” with no overhead. Writing logs helps the engineering team understand and gain insights into how the system is running in production. Logging is part of the API of the monad and the complexity of the system is abstracted away from the developer. 

### How?

#### LogIO.pure

Use `LogIO.pure` to lift a value into a side effect. 

```reasonml
let intInIO = 123 |> LogIO.pure;
```

#### LogIO.map

Use `LogIO.map` to change a value from a side effect. 

```reasonml
let stringInIO = 123 |> LogIO.pure |> LogIO.map(Int.toString);
```

#### LogIO.flatMap

Use LogIO.flatMap to run another side effect parameterized by the value from a previous side effect.

```reasonml
"user@qwick.com" |> updateEmailField |> LogIO.flatMap(validateEmail);
```

#### LogIO.suspend

Use LogIO.suspend to lift side-effecting functions into the pattern. 

Don’t do:

```reasonml
let date = Js.Date.make();
```

Do:

```reasonml
let dateInIO = () => LogIO.suspend(Js.Date.make);
```

#### Wrapped components

We’ve wrapped base level components of the system (like \<button\>) to provide an interface that uses LogIO instead of functions that return a unit type. This allows us to provide a standardized terminating boundary for the execution of IOs. The wrapped components are a TitleCased version of the original base component.

Don’t:

```reasonml
<button onClick={const(onCancel) >> IOUtils.unsafeRunHandledAsync}>
  <S> "Cancel" </S>
</button>
```

Do:

```reasonml
<Button  onClick={const(onCancel)}>
  <S> "Cancel" </S>
</Button>
```

### When?

When determining what things might be a side-effect, one thing to look for are things that either begin or end with a unit type. Examples:

- Js.Date.make   
- CommonTypes_Guid.make   
- Events such as onClick 

Anything that changes the state of something, is also a side effect. Things like:

- Writing to the database  
- Dispatching a Redux action  
- Updating the state of a React.useState or a React.useReducer hook

Things that go over the network are also side-effects:

- Making a GraphQL request  
- Querying data from a database

This list is not exhaustive, it’s only meant to show common areas that are side-effects that need to be contained in our LogIO pattern.

### When not?

There should be no production code that contains side effects that are not wrapped in a LogIO. Anything that does should be considered legacy and should be updated the next time we are in that area making modifications.

## Point free style

### Why?

By writing functions in a point free style we are explicitly showing that the final argument is only used to pipe into the first part of the code block. Since you only have access to the arg via composing, this dramatically reduces the things you can do with it which makes the usage of the argument both simple and explicit. 

This forces the function to be as simple as possible and proves that the main argument is only used for composition. 

### How?

#### Composition pipeline

Don’t:

```reasonml
let foo = str => 
  str 
  |> String.toNonWhitespace
  |> Option.flatMap(Int.fromString);
```

Do:

```reasonml
let foo = 
  String.toNonWhitespace >> Option.flatMap(Int.fromString);
```

#### Pattern matching

When using pattern matching, make sure to use the fun keyword over the switch keyword to show that the function is only pattern matching the value, and not doing anything more complicated.

Don't:

```reasonml
let noPointFree = foo => 
  switch (foo) {
    | Bar => "bar"
    | Baz => "baz"
  };
```

Do:

```reasonml
let pointFree =
  fun
  | Bar => "bar"
  | Baz => "baz";
```

### When?

If you only use an argument in a series of function compositions and do not reference that argument anywhere outside of that usage, that argument should be the last arg and it should be point free.

### When not?

If you must reference an argument in multiple places in a block of code, you cannot write something as point free. This will explicitly show to the developer that this block of code is more complicated than a simple composition pipeline and if it’s being changed will require a bit more time to understand what the code is doing.

## Data last

[Article describing the difference between pipe first/last, also known as data first/last](https://www.javierchavarri.com/data-first-and-data-last-a-comparison/)

### Why?

The above article does a great job of show-casing the pro/cons of data first/last. We’ve decided to emphasize composition which makes leveraging data last make more sense.

### How?

When writing an function, placing the “data” as the last arg allows us to us |\> in our codebase. “Data” is whatever data we’re working with. It could be a list of strings or the client connection to a database.

### When?

If you are working with data, put it as the last arg in a function signature.

### When not?

No currently known exceptions.

## Reverse application operator (pipe last operator) |>

### Why?

By using the pipe operator, we can express our code in a series of clearly defined steps where the starting point of them is some data, and each section of the pipe is expressing a transformation of that data. This allows us to encapsulate the logic of each step into an isolated context while restricting the complexity.

By following this, we end up with simple, composable, reusable, and extendable code. It’s very difficult to write spaghetti code with this pattern and it makes for code that is much easier to reason about, especially if you are unfamiliar with the intention behind it.

### How?

```reasonml
let result =
  someData
  |> someFn
  |> someOtherFn;
```

### When?

Anytime where a calculation can be described as a series of steps or you are passing the result of a function into the input of another function. This is a surprisingly common occurrence and most of your code will use this.

### When not?

We currently don’t use piping when writing CSS. Another exception is when you’re just using a single function, just putting the data into the parentheses is ok in this case but some people still prefer to pipe in the data even when there is only 1 function call.

## No blocks in functions if possible

### Why?

Limiting let statements in blocks reduces the need for arbitrarily naming variables. Composition often eliminates the need for declaring the variable in the first place.

By removing blocks and superfluous let statements, you force yourself to write simpler code which leads into code that feels like it has less moving parts and is more predictable.

### How?

Don’t

```reasonml
let add = (a, b) => {
    a + b;
  };
```

Do:

```reasonml
let add = (a, b) => a + b;
```

Don’t

```
let formatUserName = userName => {
    let trimmedUserName = String.trim(userName);
    String.toNonWhitespace(trimmedUserName);
  };
```

Do

```reasonml
let formatUserName = String.trim >> String.toNonWhitespace;
```

### When?

Always try to express your code without a block.

### When not?

If a calculation cannot be expressed without a block. Outside of React hooks, this is rare.

## Just Destructuring in function definition

### Why?

By destructuring all data that is used in a function block, we can get a very nice summary of how complex of a block of code we can expect. There is a strong correlation between how much destructuring is done and how complex a piece of code is. This allows us to see at-a-glance the complexity of a block of code which allows us to make better decisions and estimates.

### How?

Don’t

```reasonml
let getFullName = user => user.firstName + " " + user.lastName;
```

Do:

```reasonml
let getFullName = ({firstName, lastName}) => firstName + " " + lastName;
```

### When?

When you are referencing values inside of a record/object.

### When not?

No currently known exceptions.

## Use higher level features instead of lower level features

### Why?

Lower level features will allow you to write code that does more things. Higher level features restrict the usage of data and explicitly expresses intent when writing code. This seems counterintuitive but when reading lower level code, you have to have a deeper understanding of things before you can really know what is going on with a block of code. When reading higher level code using proper abstractions, you know very quickly what the code does, and also very importantly, what it **CANNOT** do. This allows you to form better assumptions when reading through a block of code.

For example, when using map we know that we can only take in a value, and output a transformation of that value. We know we cannot change the context, or fire off something to make a network call, or modify the global state. By limiting us to this subset of functionality, we can derive a lot of intention from what the previous developer’s code.

To showcase the opposite, if the developer instead used flatMap, then we would assume that the code changes the monad context of the value as well as the value itself and if the code does not do that, then we are basing our decisions and future changes on wrong assumptions. In this case, since we aren’t using the more powerful abstraction, we should rewrite the code to use the less powerful, and higher level abstraction to express our intentions more clearly.

### How?

This depends on the structures being used. It’s impractical to include an exhaustive list of how to accomplish this.

Instead, we’ll think about what tool to use when you are writing code. When doing so, you can follow this simple order:

```
map > flatMap > foldByValue > fold > Pattern matching > If/else block
```

With each step increasing the functionality of the code that is written but decreasing the amount that we can leverage the type system and what we can derive from the code. Note that some things might not have all of the available functions listed above.

### When?

This can be considered a universal principle in our codebase.

### When not?

No currently known exceptions

## Write smaller function that are easily composed, and avoid large functions that do more

### Why?

“There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies, and the other way is to make it so complicated that there are no obvious deficiencies. The first method is far more difficult.”

― **C. A. R. Hoare**

When solving complex problems, a common strategy is to break apart the problem into very small pieces and solve those, then use those solutions to solve a bigger and more complex problem.

We mirror this philosophy by creating smaller functions and composing them together. When coupled with referential transparency, this allows the developer to evaluate the correctness of the smaller functions in isolation and can help make things have “obviously no deficiencies”.

### How?

There’s no real complete solution for this. Heuristics for this include:

- A function should solve 1 and only 1 problem  
- More than 3 levels of nesting  
- Excessively long function names trying to explain what is being done

### When?

As much as possible.

### When not?

A lot of this is really a judgment call. Some React components will have more than 3 levels of nesting but doesn’t quite make sense to break up. Don’t be afraid to ask other people to get different opinions to help you land on a good solution.

## Avoid base-level types such as int, float, string, bool, and char \- instead use types that are more restrictive

### Why?

Using primitive types leads to “[stringly typed](https://wiki.c2.com/?StringlyTyped)” interfaces. These types of function interfaces do a poor job of describing what the function does and leads to bugs when parameters are accidentally flipped. Primitive types should be lifted into higher level types that help describe the domain and provide more context and protections for the developer.

This also applies to errors.

A fantastic talk explaining this in depth is [Domain Modeling Made Functional](https://www.youtube.com/watch?v=PLFl95c-IiU).

### How?

Most often, we call out to our internal TypeUtils module to create an “opaque type” which is a fancy way to say that the end user can’t see inside and the module has to provide an API to use the internal type.

```
module URL = TypeUtils.MakeOpaqueStringType({});
```

Something like the above would allow us to know that a function isn’t just looking for any old string, but is actually looking for a string that is a valid URL. This not only makes function definitions much more explicit about what they are doing, it makes our code more resilient and maintainable.

### When?

If you are doing something with values that easily fit into a primary type but doesn’t make sense in the primary type’s domain, you should be looking into making a new type that restricts the primary type’s domain into valid use cases for the type.

For example, a user’s ID might fit into an int type, but it doesn’t make sense to take a square root of a user’s ID, or add 2 user IDs together. So a user ID fits in nicely to an int but we do not want to do all of the things we can do with an int to a user ID.

### When not?

If you are working with a value that does fit in nicely with a primitive type *and* it also makes sense in its domain it might make sense to keep it a primitive type. Usually though it makes for a cleaner API to lift those into a higher level type

## The string type should be reserved for 2 cases: showing text to the user, or taking input from the user

### Why?

When working with strings, we have an unlimited number of possibilities. This leads to code that has to handle an unlimited number of cases which makes things very fragile and much more difficult to maintain and extend then it needs to be.

Therefore, the usage of strings in the codebase should be reserved as a boundary type. In other words, it’s either at the very end of code that is meant to show something to an end user. *Or* it is used to parse input from the user into something more strongly typed.

### How?

When we are taking input from the user, take the raw string and put it into functions that parse it into the next type we need. For example, if the user is typing in an email, we’ll take the string and put it into Email.fromString which will return an option of the email. This will also facilitate any form validation since we know that the string the user entered isn’t a valid type for the thing we need.

And if you need to display that email to the user, you would use Email.toString to get the raw string that is needed to display to the user.

### When?

All of the time. 

### When not?

No currently known exceptions.

## Don’t use hooks directly exposed from React - instead use hooks from ReactIO and ReactUtils

### Why?

The React.use\* are zero cost bindings but are not very ergonomic and it violates our “Falling into the Pit of Success” philosophy (in addition to others). We’ve instead wrapped those hooks with our own version that leads the developer into using things in a standard fashion.

### How?

Don’t:

```reasonml
React.useEffect1(() => {
    selectedCities
    |> updateSelectedCities
    |> dispatch
    |> IO.mapError(ignore)
    |> IOUtils.unsafeRunHandledAsync;
    None;
  },
  [|selectedCities|],
);
```

Do:

```reasonml
ReactIO.useEffect1(
    updateSelectedCities >> dispatch >> IO.mapError(ignore),
    selectedCities,
  );
```

### When?

Pretty much every time.

### When not?

No currently known exceptions. Any current usage should be considered deprecated and removed if found.  
