# Piet Programming Language

## 1. About

Interpreter for [*Piet Programming Language*](https://www.dangermouse.net/esoteric/piet.html), which is fully in Rust.

> *Piet is a programming language in which programs look like abstract paintings. The language is named after Piet Mondrian, who pioneered the field of geometric abstract art.*

| ![](https://retas.de/thomas/computer/programs/useless/piet/hw1-11.gif) |
|:-:|
| A famous `Hello, world!` program by Thomas Schoch ([link](https://retas.de/thomas/computer/programs/useless/piet/explain.html)) |

## 2. Usage

### 2.1 Installation

#### From GitHub

```bash
$ cargo install --locked --git 'https://github.com/your-diary/piet_programming_language'
```

#### From Source

```bash
$ git clone 'https://github.com/your-diary/piet_programming_language'
$ cd piet_programming_language/
$ cargo install --locked --path .
```

### 2.2 Run

```bash
$ piet_programming_language <image file>
```

### 2.3 Show Help

```bash
$ piet_programming_language --help

Interpreter for Piet Programming Language

Usage: piet_programming_language [OPTIONS] <IMAGE_FILE>

Arguments:
  <IMAGE_FILE>

Options:
  -c, --codel-size <CODEL_SIZE>  Specifies the codel size (default: auto detect)
  -v, --verbose                  Enables debug output (path trace etc.)
  -h, --help                     Print help
  -V, --version                  Print version
```

## 3. Specification

[The official specification](https://www.dangermouse.net/esoteric/piet.html) doesn't define Piet Programming Language very strictly: some behaviors are *implementation-defined*.

In this section, we explain how such behaviors are implemented.

### 3.1 Colors

> *Additional colours (such as orange, brown) may be used, though their effect is implementation-dependent. In the simplest case, non-standard colours are treated by the language interpreter as the same as white, so may be used freely wherever white is used. (Another possibility is that they are treated the same as black.)*

Our implementation marks any unknown color as an error, immediately terminating the interpreter before your program starts.

### 3.2 Codels

> *Individual pixels of colour are significant in the language, so it is common for programs to be enlarged for viewing so that the details are easily visible. In such enlarged programs, the term "codel" is used to mean a block of colour equivalent to a single pixel of code, to avoid confusion with the actual pixels of the enlarged graphic, of which many may make up one codel.*

The codel size is automatically detected or can be specified via `--codel-size` option. Note that, generally speaking, the codel size cannot be uniquely determined. If a positive integer `n` is valid as a codel size, then any divisors of `n` are also valid. It is even known there is a program whose behavior changes as the codel size changes (see [*Multi-Codel Size*](https://www.dangermouse.net/esoteric/piet/samples.html)). When automatic detection is performed, the maximum valid `n` is adopted.

### 3.3 Stack

> *The stack is notionally infinitely deep, but implementations may elect to provide a finite maximum stack size. If a finite stack overflows, it should be treated as a runtime error, and handling this will be implementation dependent.*

Our implementation doesn't explicitly set the limit of a stack size. The actual size depends on your computer. What happens when a stack overflows is *undefined*.

### 3.4 Integers

> *The maximum size of integers is notionally infinite, though implementations may implement a finite maximum integer size. An integer overflow is a runtime error, and handling this will be implementation dependent.*

We use Rust's [`isize`](https://doc.rust-lang.org/std/primitive.isize.html) type to handle integers. The number of bits of an `isize` is normally `32` or `64`. What happens when an overflow occurs is *undefined* (we naively use Rust's builtin integer operations).

### 3.5 `divide` command

> *If a divide by zero occurs, it is handled as an implementation-dependent error, though simply ignoring the command is recommended.*

Zero-division immediately terminates your program.

### 3.6 `mod` command

> *If the top value is zero, this is a divide by zero error, which is handled as an implementation-dependent error, though simply ignoring the command is recommended.*

Zero-division immediately terminates your program.

### 3.7 `roll` command

> *If a roll is greater than an implementation-dependent maximum stack depth, it is handled as an implementation-dependent error, though simply ignoring the command is recommended.*

As noted earlier, there is no explicit *implementation-dependent maximum stack depth*.

Practically, when the depth (i.e. the second top entry of a stack) is larger than `len(stack) - 2` (i.e. the whole stack minus the two popped entries), the command is simply ignored according to

> *Any operations which cannot be performed (such as popping values when not enough are on the stack) are simply ignored, and processing continues with the next command.*

By the way, the complexity of `roll` command only depends on the depth (i.e. `O(depth)`). Even if the number of rolls (i.e. the top entry of a stack) is large, the command runs quickly. The similar applies to `pointer` command and `switch` command, both of which take `O(1)`.

### 3.8 `out(char)` command

When the top entry of a stack exceeds the range `[0, char::MAX]` (i.e. when it isn't a valid Unicode character), the command is simply ignored according to

> *Any operations which cannot be performed (such as popping values when not enough are on the stack) are simply ignored, and processing continues with the next command.*

## 4. Tests

### 4.1 Unit Tests

Many unit tests are written.

### 4.2 Integration Tests

As integration tests, almost all of the samples exhibited in [*Piet Program Gallery*](https://www.dangermouse.net/esoteric/piet/samples.html) are tested.

Some tests are set `#[ignore]` (i.e. skipped) because they fail. As far as we investigated, we suspect the reason is not because our implementation is wrong but because some samples are not standard-compliant (anymore). Especially, how white blocks shall be handled was not clarified in the first version of the spec, and it was afterwards clarified as seen in the latest spec. Our implementation conforms to [*the spec as of 2023/03/29*](https://web.archive.org/web/20230327044126/https://www.dangermouse.net/esoteric/piet.html).

## 5. References

- [*Language Specification*](https://www.dangermouse.net/esoteric/piet.html)

- [*trace output*](http://www.bertnase.de/npiet/hi-npiet-trace.html)

- [./readme_assets/spec.png](./readme_assets/spec.png)

<!-- vim: set spell: -->
