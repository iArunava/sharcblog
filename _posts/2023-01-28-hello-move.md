---
title: "First move with Move!"
subtitle: An Introduction
categories: web3 blockchain token defi finance decentralized
image: 
  path: /assets/images/banners/move_header.png
layout: post
---

In this article, we are going to take our first step with the Move Programming Language. Learn about some intricate details of how it works. Learn a bit about how to write Move and **understand its syntax and publish our first Move Module** on the Aptos Chain. We will setup our local CLI to achieve all this.

Move is a programming language for Web3 that emphasizes on **Scarcity and Access control**. **Structs cannot be accidentally duplicated or dropped** unless explicitly defined, this by default enforces scarcity. Access Control comes from the notion of accounts as well as access privileges that have been enforced on the modules. A module **unless it has a public constructor or public setters and getters, it cant be accessed from elsewhere**.

Here we will write a Move program that will **set a message on the Aptos Chain**, thus altering the state of the chain and then **read it back**. Alright, lets jump in now!

---

In this article, we will see:
- [Setting up the env](#setting-up-the-env)
- [What is a Module?](#what-is-a-module)
- [Use Statements](#use-statements)
- [Resources and Events](#resources-and-events)
- [Drop and Store](#drop-and-store)
- [Constants](#constants)
- [Functions](#functions)
- [Mutable Functions](#mutable-functions)
- [Test Functions](#test-functions)
- [Unit Tests](#unit-tests)
- [Compile, Test and Publish](#compile-test-and-publish)
- [Interact with the Module](#interact-with-the-module)
- [References](#references)

---

## Setting up the env
Let's start by cloning the codebase that we are going to work with first.
Please note that this repository was created for the purpose of this tutorial with some modifications from [1].

{% highlight bash %}
git clone https://github.com/iArunava/hellomove.git
{% endhighlight %}

Now, your directory structure should look something like this.
```
.
├── Move.toml
└── sources
    ├── hello_blockchain.move
    └── hello_blockchain_test.move
```

You would notice an extra directory called `framework`. That directory just contains all the dependencies that are required for the project.

Before moving forward we need aptos cli installed on our system. You can head over to the installation site for all the different ways to install it, which can be found [here](https://aptos.dev/tools/aptos-cli/install-cli/). If you are on linux/WSL the following should work fine. Only prerequisite is that you should have python3 installed in your system. Which comes by default in most OS.
```
curl -fsSL "https://aptos.dev/scripts/install_cli.py" | python3

```
That should be it. After that you should be able to do this.
```
$ aptos -V
aptos 2.0.2
```
My version is `2.0.2`. Earlier versions upto 1.4 or above should work fine for this tutorial.

## What is a Module?
Now, we will understand what each file here is doing. Let's start with `hello_move.move` file. This is the source file which contains the logic that how this program will behave once it has been published on the chain. Lets take a look into this file. 

```
module hello_move::message {
```
The first line of the file looks somewhat like this. Here we are defining a `module` in Move. Lets start with what is a module in Move. 

> Modules are libraries that define struct types along with functions that operate on these types. Struct types define the schema of Move's global storage, and module functions define the rules for updating storage. Modules themselves are also stored in global storage.

A `module` in Move has the following syntax.
```
module <address>::<identifier> {
    (<use> | <friend> | <type> | <function> | <constant>)*
}
```
`address` here can be both named or literal address. Here comparing the syntax and code we find that `hello_move` in the code is a named address. And `message` is an identifier. 

This named address, is defined in the `Move.toml` file, which we will explore later. Alright, so this module `message` will be published under the named address `hello_move`.

## Use Statements
Next we see some `use` statements.
```
use std::error;
use std::signer;
use std::string;
use aptos_framework::account;
use aptos_framework::event;
```
`use` has the following syntax
```
use <address>::<module name>;
use <address>::<module name> as <module alias name>;
```
These are declaring some public modules that one plans to use through out that can help us get things done faster. Looking closer, we are importing the `error`, `signer` and `string` libraries from the `std` account and `account` and `event` from the `aptos_framework`. `use std::error` would introduce an alias `error`, and that would mean anywhere we could use `error` instead of using `std::error`. Similarly, to importing libraries in other languages, we can use an `as E` statement to change the alias from `error` to `E`.


## Resources and Events
```
struct MessageHolder has key {
 message: string::String,
 message_change_events: event::EventHandle<MessageChangeEvent>,
}
```
This block of code defines a `resource`. Lets understand a bit more of what that means. Here we have defined a `struct` that has the `key` ability. The `key` ability allows a `struct` to be used as a storage identifier. Its stored at the top-level and acts as storage. When a `struct` has the `key` ability it turns it into a `resource`.

> `Resource` is stored under the account and it exists only when assigned to an account and can be accessed only through that account.

Here, we are defining a `resource` called `MessageHolder`, which contains 2 objects:
- `message` of type `string::String`: which would store the message, and 
- `message_change_events` of type `event::EventHandle<MessageChangeEvent>`: which stores the `event` when the resource is updated.

In short, `event`s are emitted during the execution of a transaction. Modules can define its own `event` and choose when to emit it upon the execution of the module. Here, we are going to emit an `event` every time the resource is updated. And `message_change_events` will store such events.


## Drop and store
```
struct MessageChangeEvent has drop, store {
        from_message: string::String,
        to_message: string::String,
}
```
Here we are defining a `struct` with the abilities `drop` and `store`.
- `store`: A `struct` needs a `store` ability as its been stored inside another `struct`. If you see, the `MessageHolder` struct, it contains `MessageChangeEvent`.
- `drop`: value of this `struct` can be dropped by the end of the execution of the module.

And looking into the struct, we have 2 variables
- `from_message` of type `string::String`, and
- `to_message` of type `string::String`

So when an event is emitted, it would contain the old message "from" which it is being updated and the new message "to" which it is being updated.


## Constants
```
const ENO_MESSAGE: u64 = 0;
```

That line is self explanatory. We are defining a constant using the keyword `const` named `ENO_MESSAGE` of type `u64` and it has a value of `0`.

## Functions
```
#[view]
public fun get_message(addr: address): string::String acquires MessageHolder {
		assert!(exists<MessageHolder>(addr), error::not_found(ENO_MESSAGE));
		borrow_global<MessageHolder>(addr).message
}
```
The first line here `#[view]` declares this as a view function. Such functions do not modify the state of the blockchain and can be called using an API. A mutable function can also be declared as a view function. But please mark a mutable function as private so that such functions cannot be maliciously called during runtime.

In the next line, we define the function `get_message()`. The keywords used here are `public` which indicates that this function would be publicly accessible and `fun` which just says that this is a function.

Next, we see the parameters that are passed to the function `(addr: address)`, the name of the parameter is `addr` and it is of type `address`. It has a return type of `string::String` and it `acquires MessageHolder`. `acquires` is required here as the function accesses the `MessageHolder` resource and we dont want other parties to modify the value of the `resource` when we are reading it. Also, if we have a function that calls a function that uses `acquires` then we also need to mark that child function with `acquires`.

Next, we see an `assert!` statement. `assert` has the following syntax,
```
assert!(condition: bool, code: u64)
```
So we see that, `exists<MessageHolder>(addr)` is the condition, which would return a boolean value True or False depending on if `MessageHolder` resource is present for the address that is being passed. If its not present we throw an `error` with the const code we declared earlier `ENO_MESSAGE`. Incase, it is present. we return a immutable reference of the message that is present in the `key` `MessageHolder` using the `borrow_global` function.

`borrow_global` has the following syntax
```
borrow_global<T>(address): &T
```
It looks for a type `T` in the `address`. If its present it returns an immutable reference to that object. in the code we can see that once it is returned we are reading the `.message` parameter from the object and returning it.

Alright, now that we have the `get_message` function defined that would read the `message` that is set in the chain, lets start defining the `set_message` function that would set the message to what we want on the chain.

## Mutable Functions
```
public entry fun set_message(account: signer, message: string::String)
    acquires MessageHolder {
        ...
}
```
We see the function `set_message` declared here like before. It has an extra keyword called `entry`. An `entry` modifier to a function allows to invoke these functions directly. And this allows to specify which function to begin the execution with. Thus, they can be considered as the "main" function in the module.

Next
```
let account_addr = signer::address_of(&account);
```
we get the `address` of the `signer` using the `address_of` function which takes in a reference to the `account`. Next we see.

```
if (!exists<MessageHolder>(account_addr)) {
	move_to(&account, MessageHolder {
		message,
		message_change_events: account::new_event_handle<MessageChangeEvent>(&account),
	})
}
```
We check if the `resource` `MessageHolder` exists in the given account address. Incase it does not we create a `MessageHolder` and move that resource under the account using the `move_to` function.

Incase, it does exist
```
else {
	let old_message_holder = borrow_global_mut<MessageHolder>(account_addr);
	let from_message = old_message_holder.message;
	event::emit_event(&mut old_message_holder.message_change_events, MessageChangeEvent {
			from_message,
			to_message: copy message,
	});
	old_message_holder.message = message;
}
```
We use `borrow_global_mut` to get a mutable reference of the `MessageHolder` resource in the `account_addr` and get the message stored in it. We emit an event using `event::emit_event`. `emit_event` accepts a reference to the event handle and a message. Here in our case we are passing a reference of the `message_change_events` in the `MessageHolder` resource and a new `MessageChangeEvent` that contains the old and the new message. After that we set the message in the `MessageHolder` resource to the new `message` that we have received.

Alright, that defines our `set_message` function.

## Test Functions
Now that we have our `get_message` and `set_message` function declared lets write some tests to verify that everything works as expected. So moving down the file next we see a test function defined. 
```
#[test(account = @0x1)]
```
The first line here declares that we are going to write a test function. And it mentions an account `account = @0x1` which is the account through which this module is going to be tested. Here we set the account to a random account address that is created using a built-in function for testing purposes. Whenever, a module or a module member is marked for `test` then it will not be included in the bytecode when compiled, unless compiled for testing.

```
public entry fun sender_can_set_message(account: signer) acquires MessageHolder {
    ...
}
```
Alright, next we just write a function as before. Please note that this is a test function. And thus when compiled normally (ie not for testing) this function wont be included in the bytecode. Such a test function, doesnt accept any parameters other than the type `signer`, if passed any parameters other than the type `signer` it will result in an error when run.

Also, the arg name passed in testing annotation `account = @0x1` should match the parameter name that is being passed in the function `sender_can_set_message(account: signer)`.

And then we get the account address of the `signer` and store it in `addr`. 
```
let addr = signer::address_of(&account);
```
We use the `create_account_for_test()` and pass it the `addr` to create an account for the test purposes. 
```
aptos_framework::account::create_account_for_test(addr);
```
We declare a variable called `stra` and store the string `"Hello, Move"` in that.
```
let stra = string::utf8(b"Hello, Move");
```
Then we use the `set_message` function that we have already defined above to set a message in this test account.
```
set_message(account,  stra);
```
We then use the `get_message` function defined above, to retrieve the message we have set in the above line.
```
let rmsg = get_message(addr);
```
And finally, the check to see that message that we defined initially `stra` matches with the message that was returned using `get_message` that is stored in `rmsg`. Incase it doesnt match, we throw the error code `ENO_MESSAGE`.

```
assert!(
 rmsg == stra,
 ENO_MESSAGE
);
``` 

That defines the test function that was written. And additionally that explains the entire file `hello_move.move`.

## Unit Tests
One can also write unit tests in seperate files and aptos will pick that up based on the testing annotations. And test all the functions. Like for an example we have another file in the `sources/` directory named `hello_move_test.move`. This file has a test only module declared using `#[test_only]` and thus this module will be included only when compiled for testing.

The file name can be anything, it doesnt necessarily need to have a `_test.move` suffix. As long as there are test modules and functions declared using the test annotations, the compiler will pick it up while testing.

Looking into the `hello_move_test.move` file we will see very similar things as before. It starts with
```
#[test_only]
```
Which declares this entire module as test only.

Next, it declares the `message_tests` module.
```
module hello_move::message_tests {
```
After which it declares some of the aliases that we will use. Note that we also declared the `hello_move::message` that is the module we just wrote in the other file.
```
use std::signer;
use std::unit_test;
use std::vector;
use std::string;
use hello_move::message;
```
Moving from there, we declare a function named `get_account()` to get the account we will use for testing purposes. We use the `create_signers_for_testing()` to create `1` `signer` for testing. It returns a `vector` and we then use the `pop_back` method to return that one `signer`.

```
fun get_account(): signer {
   vector::pop_back(&mut unit_test::create_signers_for_testing(1))
}
```
Next, we see the `sender_can_set_message` function which is exactly similar to the one we wrote in the other file except here we get the test account address using the `get_account()` function that we just defined.

```
let account = get_account();
let addr = signer::address_of(&account);
```

Please note again, that this is a test only module, and the compiler will run this test and include in the bytecode, only when compiled for testing. Alright, that should explain the workings of the `hello_move_test.move` file.

## Move.toml
Now, lets take a look at the `Move.toml` file that is defined at the root of the package.

Each move package has a manifest file at the root with the `.toml` extension and it contains the metadata for the package such as name, version, and dependencies. Taking a look at the `Move.toml` file.
```
[package]
name = "HelloMove"
version = "0.0.0"
```
Here we are declaring the `name` and the `version` of the package.

Next, we are declaring the named address `hello_move` but we are leaving it undefined using the `_` so that would mean that we need to pass the named address when we are compiling.
```
[addresses]
hello_move = "_"
```
And then we declare a dependency on the `aptos-framework` as we have already seen that we use a few functions from the `aptos-framework`.
```
[dependencies]
AptosFramework = { local = "./framework/aptos-framework" }
```

And that should define all the contents of the `Move.toml` file. And thus all the files that are there in the package.

## Compile, Test and Publish
Now that we know what all the files are about and what they contain. We can go ahead and compile the package. But first lets start with initializing aptos.
```
aptos init --network devnet
```
Tap Enter and follow through. The output should look something like this.
```
Configuring for profile default
Configuring for network Devnet
Enter your private key as a hex literal (0x...) [Current: Redacted | No input: Generate new key (or keep one if present)]

No key given, keeping existing key...
Account c1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76 has been already found onchain

---
Aptos CLI is now set up for account c1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76 as profile default!  Run `aptos --help` for more information about commands
{
  "Result": "Success"
}

```
Now lets fund this account using a faucet.
```
aptos account fund-with-faucet --account default
```
We should see an output similar to this
```
{
  "Result": "Added 100000000 Octas to account c1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76"
}
```

Now lets compile the module using
```
aptos move compile --named-addresses hello_move=default
```
Remember we kept the named address in the `Move.toml` file empty so we need to pass the named address as `default` one when we are compiling. Compilation output should look similar to this
```
Compiling, may take a little while to download git dependencies...
INCLUDING DEPENDENCY AptosFramework
INCLUDING DEPENDENCY AptosStdlib
INCLUDING DEPENDENCY MoveStdlib
BUILDING HelloMove
{
  "Result": [
    "c1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76::message"
  ]
}
```
This would mean that the file was successfully compiled.

Next, lets test the modules
```
aptos move test --named-addresses hello_move=default
```
We should be able to see both the tests passing
```
INCLUDING DEPENDENCY AptosFramework
INCLUDING DEPENDENCY AptosStdlib
INCLUDING DEPENDENCY MoveStdlib
BUILDING HelloMove
Running Move unit tests
[ PASS    ] 0xc1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76::message::sender_can_set_message
[ PASS    ] 0xc1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76::message_tests::sender_can_set_message
Test result: OK. Total tests: 2; passed: 2; failed: 0
{
  "Result": "Success"
}
```
And finally lets publish the module using
```
aptos move publish --named-addresses hello_move=default
```
The output you will see is similar to this.
```
Compiling, may take a little while to download git dependencies...
INCLUDING DEPENDENCY AptosFramework
INCLUDING DEPENDENCY AptosStdlib
INCLUDING DEPENDENCY MoveStdlib
BUILDING HelloMove
package size 1765 bytes
Do you want to submit a transaction for a range of [140300 - 210400] Octas at a gas unit price of 100 Octas? [yes/no] >
yes
{
  "Result": {
    "transaction_hash": "0x22328486cd87414663d4461182b5f2de6447b29f9c32c682b152f6c127276568",
    "gas_used": 1403,
    "gas_unit_price": 100,
    "sender": "c1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76",
    "sequence_number": 0,
    "success": true,
    "timestamp_us": 1690433681245985,
    "version": 40008336,
    "vm_status": "Executed successfully"
  }
}
```

Now we have successfully published our module. We can go to the aptos explorer and check the transaction hash. 

{% include image.html 
           title=""
           img="/assets/images/firstmovetransaction.png"
           caption="Move Transaction"
%}

Here if we notice we can see the payload, ie the message that we want to be stored in the chain being passed.

## Interact with the Module
Now that we have our module published on the chain, lets start interacting with the module. 

To interact with our published module lets first trigger the `set_message` method and update the `message` stored in the resource `MessageHolder`
```
aptos move run --function-id 'default::message::set_message' --args 'string:first move with move'
```
Here we are setting the message to 'first move with move'. The output should be similar to the following.
```
Do you want to submit a transaction for a range of [50300 - 75400] Octas at a gas unit price of 100 Octas? [yes/no] >
yes
{
  "Result": {
    "transaction_hash": "0x4e7d9e1ba498f3367f714d7e425f184f8d64d7d5d68ad962f30273384cea6464",
    "gas_used": 504,
    "gas_unit_price": 100,
    "sender": "c1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76",
    "sequence_number": 1,
    "success": true,
    "timestamp_us": 1690434192512775,
    "version": 40055278,
    "vm_status": "Executed successfully"
  }
}
```

And now that the message is set, we can get the message using the view function that we wrote `get_message`
```
curl https://fullnode.devnet.aptoslabs.com/v1/accounts/c1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76/resource/0xc1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76::message::MessageHolder | json_pp
```
Here substitute the url with your account address. 

Recall, its a view function, so we can call it direcly like an API. So we can actually just directly go to this link in the browser and see our `message` that is stored in the `MessageHolder` resource.

The output for `curl` should give a result similar to this.
```
{
   "data" : {
      "message" : "first move with move",
      "message_change_events" : {
         "counter" : "0",
         "guid" : {
            "id" : {
               "addr" : "0xc1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76",
               "creation_num" : "4"
            }
         }
      }
   },
   "type" : "0xc1afaaf97eaa61128b5b1439ee9732ef581b70f0e56f740f1bcf964e5427cf76::message::MessageHolder"
}
```
Here in the `message` key you can see the message that we have just set.!

You can continue with updating the message as many times as required as you want using the `set_message` function and then reading it. Do note that `set_message` initiates a transaction that is it updates the state of the chain and thus you do need to pay gas everytime you make a transaction. If you see above, you can see the amount of gas used and the price per gas. This is why initially the account needs to be funded so that we can pay the gas required. The current account can be funded using a faucet as it is on the devnet, if on mainnet you need to have APT in your account to pay for this gas. 

Alright, Congratulations! You now understand Move language, how to compile, test and publish a Move module and interact with that Move module!
As a next step, you can head over to the aptos.dev docs to learn more about Move and the Aptos Chain.

### References
[1] [Move Tutorial from aptos-core](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/move-tutorial)

[2] [Aptos Docs](https://aptos.dev)
