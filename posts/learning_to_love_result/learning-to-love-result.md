
# Learning to love Result

When Swift was introduced as another tool at the disposal of developers as an alternative to Objective-C, one of most prominent features of Swift was its emphasis on the `Optional` type.

The healthy programming practices `Optional` brought with it forced developers to deal with the possibility of a value being `nil` or have a compile time error staring them in the face.
This has done wonders in reducing run-time errors but by no means is `Optional` a silver bullet.
There are many reasons why a value could be `nil` and if the context is obvious enough `Optional` might suffice, but in most other cases more context is needed and unfortunately `Optional` can not provide this.

This is where `Result` comes in.

### Evolution of the Result type
Starting with Swift 5, the type `Result` is baked into Swift's standard library.
Result is very simple, you could easily roll your own:

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=naive-result.swift"></script>

Thats all there is to it, just a simple `Either` data type, i.e either A or B.
The 'B' in the case of Result is specialized, it must be a type that conforms to `Error`.

Open Source `Result` micro frameworks such as [`antitypical/Result`](https://github.com/antitypical/Result) have been around almost as long as Swift it's self has.
The usage of `Result` in Swift code is nothing new but just how `antitypical/Result` provided a common definition of the `Result` type that would allow multiple libraries to share a single definition making code integration less painful, the `Result` type provided by the Swift standard library allows code to share a now universal result type, one that is on every single installation of swift from here on out.
Now that `Result` is in the standard library, an "official" definition exists and the swift community would benefit by leaving third-party `Result` types behind and simply leveraging Swift 5's `stdlib` version.

## Using `Result`

Result is commonly returned from a function that can fail.
Take for example a fictional function that registers a master passphrase for a password manager app.

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=register-password.swift"></script>

The rules are simple, the password must be ten characters or more and composed of upper and lower case characters.

When an error occurs with this function we just simply return `nil` which is not very descriptive and forces us to use a one-size-fits-all message like "Password not vaild, try again".

Of course the return value could be a touple with an error or error message AND a success message like this:

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=register-password-stub-a.swift"></script>

but then this code could express a situation that should be impossible, a kind of "Superposition" case that only Schrodinger would appreciate.
I don't like [Schrodinger's Cat](https://en.wikipedia.org/wiki/Schr%C3%B6dinger%27s_cat) because what kind of monster would put a cat in danger?
Superposition is cool but it has no place in a modern typed programming languages (when avoidable) so lets stick to a single `Result` return value.


<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=numeric-example.swift"></script>

So calling the above code with different reasons for failing password validation all produced the same `nil` value. Not much to present to the user other than "Use a stronger password".

Well, we know that `Result`'s failure type must conform to error, and an `enum` is a simple and clean way of expressing different distinct states so lets go ahead and fill that out:

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=password-error.swift"></script>

OK! Now that thats out of the way lets revisit what our password checking function would look like now that it returns a result with meaningful failure information.

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=register-password-result-ver.swift.swift"></script>

Now calling our function would produce the following output:

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=password-example.swift"></script>

And finally we can get around to writing our error handling code a little more concise and clearly.

Writing the logic to be either success or failure and now there is no chance of "superposition" or some other undefined state where they are both `nil`:

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=result-handling.swift"></script>


# Features of the Result type

Result has a constructor to create a Result from throwing code, if the method throws then the
Result has two methods for Transforming values:  `map` and `flatMap` and two methods for transforming errors: 'mapError' and


## Replacing code that `throw`s
One handy feature of Result is it's ability to wrap code that throws errors.
This has a few benefits but one obvious one is that instead of catching the error and dealing with it right away, the output of `Result` can be saved and if its an error it can be dealt with at a more oppertune time.

Many error handling examples deal with network code so I will try to be less cliche and use an example of dealing with exceptions that arise from parsing, namely `Codable` failing to decode JSON.

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=CardboardBox.swift"></script>

Given the above `CardboardBox` structure it could be initialized with the following valid JSON:

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=decoding-example-a.swift"></script>

Thats quite a bit of code just to handle one possible exception, more than that we are forced to handle the event of an error in the `catch` block at the moment an exception is thrown.

I personally find this lacking but fortunately there is good news: `Result` has an initializer that takes a closure that throws exceptions and automatically wraps it in a handy Result type.

The example below has intentionally misformed JSON to demonstrate error handling with the `Result` type.
The property "flavor" should be a `String` but here we use an Integer (42, the answer to the meaning of life.)
This will cause JSONDecoder to fail.

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=decoding-invalid-example.swift"></script>

We use `Result`'s closure constructor to wrap the throwing code and now we can deal with the end result in a very readable switch statement.
One of the big benefits of this approach is that the result is saved and stored away for later in `myBoxResult`, it can be dealt with when it is convenient for the programmer which is very important for dealing with asynchronous code.

## Transforming `Result`

### map

A `Result` can be transformed using `map`to return a new `Result` containing a different value in the `success` case. If the `Result` ended up in an Error the closure provided to map is ignored and the error is passed through.

Below is an example of calling map on a Result of type `Result<Int, CactusError>` and transforming it to type `Result<String, CactusError`:

<script src="https://gist.github.com/Nirma/6c59cbc6a7c8f6bcf39fc19fe8e1eae2.js?file=cactus-error.swift"></script>

### flatMap
`flatMap` behaves much the same as `map` except that it unwraps the success value and after the closure evaluates, repackages the result in a new `Result`.
This is done to avoid having `Result`s inside of `Result`s i.e `Result<Result<String, Error>, Error>`


### mapError and flatMapError

These two functions behave exactly the same way as `map` and `flatMap` do except that they only execute the closure passed to them when the `Result` evaluates to `.failure`.

## Contributions welcome!
Please feel free to add any changes, suggest fixes, edits or new code examples that you feel would improve this article. Pull requests are always welcome, if you feel inclined please send it to this site's blog repository:

https://github.com/screaming-cactus/blog

## Further Reading

There are many other great mini-tutorials out there that will pop up in your search results but in addition to reading those I highly suggest you take a look at the Swift Evolution Proposal [SE-235: Add result](https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md)

Happy Trails! 🤠
