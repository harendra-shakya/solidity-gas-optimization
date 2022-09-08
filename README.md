# Gas optimization

## Optimization process

### Security is #1 concern !!!

1. Make the code correct
2. Prove code correctness with unit tests
3. Measure the performance
4. Pick the change that will have the most impact
5. Make the code changes
6. Go to 1

## Main Gas optimization areas in solidity

- Storage
- Variables
- Functions
- Loops

## Storage

- Saving one variable in storage costs 20,000 gas
- 5,000 gas when we rewrite the variable
- reading from the slot take 200 gas
- But storage variable declaration doesn't cost anything, as there's no initialization

### Tips

- always save a storage variable in memory in a function
- if you wanna update a storage variable then first calculate everything in the memory variables
- organize & try to pack two or more storage variables into one it's much cheaper
- also while using structs, try to pack them

### Refunds

- Free storage slot as by zeroing corresponding variables as soon as you don't need them anymore. This will refund 15,000 of gas.
- Removing a contract by using Selfdestruct opcodes refunds 24,000 gas. But a refund must not surpass half the gas that the ongoing contract call uses.

### Data types and packing

- Use bytes32 whenever possible, because it is the most optimized storage type.
- If the length of bytes can be limited, use the lowest amount possible from bytes1 to bytes32.
- Using bytes32 is cheaper than using string

#### Doubt -

Q. So modifying a uint8 is cheaper than uint256?

Ans. No, storing a small number in a uint8 variable is not cheaper than storing it in uint256 coz the number in uint8 is padded with numbers to fill 32 bytes.

### Inheritance

- when we extend a contract, the variables in the child can be packed with the variables in the parent.

- The order of variables is determined by [C3 linearization](https://en.wikipedia.org/wiki/C3_linearization), all you need to know is that child variables come after parent variables.

### Memory vs Storage

- copying between the memory and storage will cost some gas, so don't copy arrays from storage to memory, use a [storage pointer](https://blog.b9lab.com/storage-pointers-in-solidity-7dcfaa536089).

- the cost of memory is complicated. you "buy" it in chunks, the cost of which will go up quadratically after a while

- Try adjusting the location of your variables by playing with the keywords "storage" and "memory". Depending on the size and number of copying operations between Storage and memory, switching to memory may or may not give improvements. All this is coz of varying memory costs. So optimizing here is not that obvious and ebery case has to be considerd individually.

## Variables

- Avoid public variables
- Use events rather than storing data
- Use memory arrays efficiently
- it's good to use memory arrays if the size of the array is known, fixed size memory arrays can be used to save gas.
- Use global variables efficiently
- Use return values efficiently
- A simple optimization in Solidity consists of naming the return value of a function. It is not needed to create a local variable then.

### Mapping vs Array

- Use mapping whenever possible, it's cheap instead of array
- But array could be a good choice if you have small array

## Functions

- use external most of the time whenever possible
- Each position will have an extra 22 gas, so
  - Reduce public varibles
  - Put often called functions earlier
- reduce the parameters if possible

### View Functions

- You are not paying for view functions but this doesn't mean they aren't consuming gas, they do.
- it cost gas when calling in a tx

## Loops

- use memory variables in loops
- try to avoid unbounded loops
- if you put `++` before `i` it costs less gas

## Other Optimizations

- Remove the dead code
- Use different solidity versions and try
- EXTCODESIZE is quite expensive, this is used for calls between contracts, the only option we see to optimize the code in this regard is minimizing the numbers of calls to other contracts and libraries.

### Libraries

When a public function of a library is called, the bytecode of that function is not made part of a client contract. Thus, complex logic should be put in libraries (but there is also cost of calling the library function)

### Errors

- Use "require" for all runtime conditions validations that can't be prevalidated on the compile time. And "assert" should be used only for static validation that normally fail never fail on a properly functioning code.
- string size in require statements can be shortened to reduce gas.
- A failing "assert" consumer all the gas available to the call, while "require" doesn't consume any.

### Hash functions

- keccak256: 30 gas + 6 gas for each word of input data
- sha256: 60 gas + 12 gas for each word of input data
- ripemd160 - 600 gas + 120 gas for each word of input data
- So if you don't have any specific reasons to select another hash function, just use keccak256

### Layout

Layout contract elements in the following order:

1. Pragma statements
2. Import statements
3. Interfaces
4. Libraries
5. Errors
6. Contracts

Inside each contract, library or interface, use the following order:

1. Type declarations
2. State variables
3. Events
4. Modifiers
5. Functions

- Function order
  - constructor
  - receive (if exists)
  - fallback (if exists)
  - external
  - public
  - internal
  - private
  - view / pure

### Order

- Order cheap functions before
  - f(x) is cheap
  - g(y) is expensive
  - ordering should be
  - f(x) || g(y)
  - f(x) && g(y)

## Markle proof

- A markle tree can be used to prove the validity of a large amount of data using a small amount of data.

## Tools for estimating gas

- Remix
- Truffle
- Eth Gas reporter

#

I will teach you everything that I am learning and I will keep updating this repo if I find something new. You can also make PR if you wanna update something.

If you have any doubts or find any mistakes, please feel free to reach out to me and I’d try to reply AFAP. Consider me as a friend and [Contact Me](https://linktr.ee/harendra_shakya).
