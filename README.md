# EVM Gas optimization tricks

## Optimization process

### Security is #1 concern !!!

1. Make the code correct
2. Prove code correctness with unit tests
3. Measure the performance
4. Pick the change that will have the most impact
5. Make the code changes
6. Go to 1

## Main Gas optimization areas in solidity

- [Storage](#storage)
- [Variables](#variables)
- [Functions](#functions)
- [Loops](#loops)

## Storage

- Saving one variable in storage costs 20,000 gas (<a href="#some-more-resources">Check gas used by EVM opcodes</a>)
- 5,000 gas when we rewrite the variable
- reading from the slot take 200 gas
- But storage variable declaration doesn't cost anything, as there's no initialization

### Tips

- always save a storage variable in memory in a function
- if you wanna update a storage variable then first calculate everything in the memory variables
- organize & try to pack two or more storage variables into one it's much cheaper
- also while using structs, try to pack them

### Refunds

- Free storage slot by zeroing corresponding variables as soon as you don't need them anymore. This will refund 15,000 of gas.
- Removing a contract by using Selfdestruct opcodes refunds 24,000 gas. But a refund must not surpass half the gas that the ongoing contract call uses.

### Data types and packing

- Use bytes32 whenever possible, because it is the most optimized storage type.
- If the length of bytes can be limited, use the lowest amount possible from bytes1 to bytes32.
- Using bytes32 is cheaper than using string
- Variable packing only occurs in storage — memory and call data does not get packed. 
- You will not save space trying to pack function arguments or local variables

#### Doubt -

Q. So modifying a uint8 is cheaper than uint256?

Ans. No, storing a small number in a uint8 variable is not cheaper than storing it in uint256 coz the number in uint8 is padded with numbers to fill 32 bytes.

### Inheritance

- when we extend a contract, the variables in the child can be packed with the variables in the parent.

- The order of variables is determined by [C3 linearization](https://en.wikipedia.org/wiki/C3_linearization), all you need to know is that child variables come after parent variables.

### Memory vs Storage

- copying between the memory and storage will cost some gas, so don't copy arrays from storage to memory, use a [storage pointer](https://blog.b9lab.com/storage-pointers-in-solidity-7dcfaa536089).

- the cost of memory is complicated. you "buy" it in chunks, the cost of which will go up quadratically after a while

- Try adjusting the location of your variables by playing with the keywords "storage" and "memory". Depending on the size and number of copying operations between Storage and memory, switching to memory may or may not give improvements. All this is coz of varying memory costs. So optimizing here is not that obvious and every case has to be considered individually.

## Variables

- Avoid public variables
- Use global variables efficiently
- it is good to use global variables with private visibility as it saves gas
- Use events rather than storing data
- Use memory arrays efficiently
- Use return values efficiently
- A simple optimization in Solidity consists of naming the return value of a function. It is not needed to create a local variable then.

### Mapping vs Array

- Use mapping whenever possible, it's cheap instead of the array
- But an array could be a good choice if you have a small array
 
### Fixed vs Dynamic

- Fixed size variables are always cheaper than dynamic ones.
- It's good to use memory arrays if the size of the array is known, fixed-size memory arrays can be used to save gas.
- If we know how long an array should be, we specify a fixed size
- This same rule applies to strings. A string or bytes variable is dynamically sized; we should use a byte32 if our string is short enough to fit.
- If we absolutely need a dynamic array, it is best to structure our functions to be additive instead of subractive. Extending an array costs constant gas whereas truncating an array costs linear gas.

## Functions

- use external most of the time whenever possible
- Each position will have an extra 22 gas, so
  - Reduce public varibles
  - Put often-called functions earlier
- reduce the parameters if possible (Bigger input data increases gas because more things will be stored in memory)
- payable function saves some gas as compared to non-payable functions (as the compiler won't have to check)

### View Functions

- You are not paying for view functions but this doesn't mean they aren't consuming gas, they do.
- it cost gas when calling in a tx

## Loops

- use memory variables in loops
- try to avoid unbounded loops
- write uint256 index; instead of writing uint256 index = 0; as being a uint256, it will be 0 by default so you can save some gas by avoiding initialization.
- if you put `++` before `i` it costs less gas

## Other Optimizations

- Remove the dead code
- Use different solidity versions and try
- EXTCODESIZE is quite expensive, this is used for calls between contracts, the only option we see to optimize the code in this regard is minimizing the number of calls to other contracts and libraries.

### Libraries

When a public function of a library is called, the bytecode of that function is not made part of a client contract. Thus, complex logic should be put in libraries (but there is also the cost of calling the library function)

### Errors

- Use "require" for all runtime conditions validations that can't be prevalidated on the compile time. And "assert" should be used only for static validation that normally fails never fail on a properly functioning code.
- string size in require statements can be shortened to reduce gas.
- A failing "assert" consumer all the gas available to the call, while "require" doesn't consume any.

### Hash functions

- keccak256: 30 gas + 6 gas for each word of input data
- sha256: 60 gas + 12 gas for each word of input data
- ripemd160: 600 gas + 120 gas for each word of input data
- So if you don't have any specific reasons to select another hash function, just use keccak256

### Order

- Order cheap functions before
  - f(x) is cheap
  - g(y) is expensive
  - ordering should be
  - f(x) || g(y)
  - f(x) && g(y)

### Use Short-Circuiting rules to your advantage

When using logical disjunction (||), logical conjunction (&&), make sure to order your functions correctly for optimal gas usage. In logical disjunction (OR), if the first function resolves to true, the second one won’t be executed and hence save you gas. In logical disjunction (AND), if the first function evaluates to false, the next function won’t be evaluated. Therefore, you should order your functions accordingly in your solidity code to reduce the probability of needing to evaluate the second function.

### Use ERC1167 To Deploy the same Contract many times

EIP1167 minimal proxy contract is a standardized, gas-efficient way to deploy a bunch of contract clones from a factory.EIP1167 not only minimizes length, but it is also literally a “minimal” proxy that does nothing but proxying. It minimizes trust. Unlike other upgradable proxy contracts that rely on the honesty of their administrator (who can change the implementation), the address in EIP1167 is hardcoded in bytecode and remain unchangeable


## Merkle proof

- A Merkle tree can be used to prove the validity of a large amount of data using a small amount of data.

## Tools for estimating gas

- Remix
- Truffle
- Eth Gas reporter
