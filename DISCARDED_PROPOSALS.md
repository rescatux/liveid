# live id - Proposal 2 - DISCARDED

## UUID generation
[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) should be generated at build time and it has to identify a given ISO.

Basically at build time, instead of generating a random UUID, you concatenate a series of build variables that make unique your CD.
Then you hash them so that you have something that changes a lot when a byte from your build variables have changed.
Finally you can shrink the result to a 8.3 file.

Thomas Schmitt says:

> Even in a conservative 8.3 name it would be possible to expose 56 bit
> with a birthday paradox threshold of 256 million. A plain hex encoding
> could expose 44 bit of random id (~ 4 million children in the class).

This is not as good enough as an UUID we can do it better.

# live id - Proposal 3 - DISCARDED

## UUID generation
[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) should be generated at build time and it has to identify a given ISO.

Basically at build time, instead of generating a random UUID, you concatenate a series of build variables that make unique your CD.
Then you hash them so that you have something that changes a lot when a byte from your build variables have changed.

An UUID like:
```
123e4567-e89b-12d3-a456-426655440000
```
means 32 characters. That might be encoded in four files such as:

```
/liveid/123e4567.001
/liveid/e89b12d3.002
/liveid/a4564266.003
/liveid/55440000.004
```

This is too complex for searching for grub.
