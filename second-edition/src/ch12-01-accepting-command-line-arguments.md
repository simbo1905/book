## Accepting Command Line Arguments

Our first task is to make `greprs` able to accept its two command line
arguments: the filename and a string to search for. That is, we want to be able
to run our program with `cargo run`, a string to search for, and a path to a
file to search in, like so:

```text
$ cargo run searchstring example-filename.txt
```

Right now, the program generated by `cargo new` ignores any arguments we give
it. There are some existing libraries on crates.io that can help us accept
command line arguments, but since we're learning, let's implement this
ourselves.

<!--Below -- I'm not clear what we need the args function for, yet, can you set
it out more concretely? Otherwise, will it make more sense in context of the
code later? Is this function needed to allow our function to accept arguments,
is that was "args" is for? -->
<!-- We mentioned in the intro to this chapter that grep takes as arguments a
filename and a string. I've added an example of how we want to run our
resulting tool and what we want the behavior to be, please let me know if this
doesn't clear it up. /Carol-->

### Reading the Argument Values

In order to be able to get the values of command line arguments passed to our
program, we'll need to call a function provided in Rust's standard library:
`std::env::args`. This function returns an *iterator* of the command line
arguments that were given to our program. We haven't discussed iterators yet,
and we'll cover them fully in Chapter 13, but for our purposes now we only need
to know two things about iterators:

1. Iterators produce a series of values.
2. We can call the `collect` function on an iterator to turn it into a vector
   containing all of the elements the iterator produces.

Let's give it a try: use the code in Listing 12-1 to read any command line
arguments passed to our `greprs` program and collect them into a vector.

<!-- Give what a try, here, what are we making? Can you lay that out? I've
tried above but I'm not sure it's complete -->
<!-- We're not creating anything, we're just reading. I'm not sure if I've made
this clearer. /Carol -->

<span class="filename">Filename: src/main.rs</span>

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

Listing 12-1: Collect the command line arguments into a vector and print them
out

<!-- Will add wingdings in libreoffice /Carol -->

First, we bring the `std::env` module into scope with a `use` statement so that
we can use its `args` function. Notice the `std::env::args` function is nested
in two levels of modules. As we talked about in Chapter 7, in cases where the
desired function is nested in more than one module, it's conventional to bring
the parent module into scope, rather than the function itself. This lets us
easily use other functions from `std::env`. It's also less ambiguous than
adding `use std::env::args;` then calling the function with just `args`; that
might look like a function that's defined in the current module.

<!-- We realized that we need to add the following caveat to fully specify
the behavior of `std::env::args` /Carol -->

<!-- PROD: START BOX -->

> Note: `std::env::args` will panic if any argument contains invalid Unicode.
> If you need to accept arguments containing invalid Unicode, use
> `std::env::args_os` instead. That function returns `OsString` values instead
> of `String` values. We've chosen to use `std::env::args` here for simplicity
> because `OsString` values differ per-platform and are more complex to work
> with than `String` values.

<!-- PROD: END BOX -->

<!--what is it we're making into a vector here, the arguments we pass?-->
<!-- The iterator of the arguments. /Carol -->

On the first line of `main`, we call `env::args`, and immediately use `collect`
to turn the iterator into a vector containing all of the iterator's values. The
`collect` function can be used to create many kinds of collections, so we
explicitly annotate the type of `args` to specify that we want a vector of
strings. Though we very rarely need to annotate types in Rust, `collect` is one
function you do often need to annotate because Rust isn't able to infer what
kind of collection you want.

Finally, we print out the vector with the debug formatter, `:?`. Let's try
running our code with no arguments, and then with two arguments:

```text
$ cargo run
["target/debug/greprs"]

$ cargo run needle haystack
...snip...
["target/debug/greprs", "needle", "haystack"]
```

<!--Below --- This initially confused me, do you mean that the argument at
index 0 is taken up by the name of the binary, so we start arguments at 1 when
setting them? It seems like it's something like that, reading on, and I've
edited as such, can you check? -->
<!-- Mentioning the indexes here seemed repetitive with the text after Listing
12-2. We're not "setting" arguments here, we're saving the value in variables.
I've hopefully cleared this up without needing to introduce repetition.
/Carol-->

You may notice that the first value in the vector is "target/debug/greprs",
which is the name of our binary. The reasons for this are out of the scope of
this chapter, but we'll need to remember this as we save the two arguments we
need.

### Saving the Argument Values in Variables

Printing out the value of the vector of arguments just illustrated that we're
able to access the values specified as command line arguments from our program.
That's not what we actually want to do, though, we want to save the values of
the two arguments in variables so that we can use the values in our program.
Let's do that as shown in Listing 12-2:

<!-- By 'find the ones we care about' did you mean set particular arguments so
the user knows what to enter? I'm a little confused about what we are doing,
I've tried to clarify above -->
<!-- We're incrementally adding features and adding some code that helps the
reader be able to see and experience what the code is doing rather than just
taking our word for it. I've hopefully clarified below. /Carol -->

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", filename);
}
```

Listing 12-2: Create variables to hold the query argument and filename argument

<!-- Will add ghosting and wingdings in libreoffice /Carol -->

As we saw when we printed out the vector, the program's name takes up the first
value in the vector at `args[0]`, so we're starting at index `1`. The first
argument `greprs` takes is the string we're searching for, so we put a
reference to the first argument in the variable `query`. The second argument
will be the filename, so we put a reference to the second argument in the
variable `filename`.

We're temporarily printing out the values of these variables, again to prove to
ourselves that our code is working as we intend. Let's try running this program
again with the arguments `test` and `sample.txt`:

```text
$ cargo run test sample.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/greprs test sample.txt`
Searching for test
In file sample.txt
```

Great, it's working! We're saving the values of the arguments that we need into
the right variables. Later we'll add some error handling to deal with
situations such as when the user provides no arguments, but for now we'll
ignore that and work on adding file reading capabilities instead.
