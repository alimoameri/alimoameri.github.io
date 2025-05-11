+++
date = '2025-03-31T17:25:37+03:30'
draft = false
title = 'bc Is the Only Calculator You Need in Unix'
tags = ["unix", "linux", "calculator", "math"]
summary = "`bc`, for basic calculator, is an arbitrary-precision calculator language with syntax similar to the C programming language. In this post, we will explore the features of bc and how to use it."
+++

<figcaption>
    <img width="80%" alt="Example of `bc` usage" title="Example of `bc` usage" src="/images/bc-example.webp"></img>
    <caption>
        <strong>Example of <code>bc</code> usage</strong>
    </caption>
</figcaption>

`bc`, for *basic calculator*, is an arbitrary-precision calculator language with syntax similar to the C programming language. In this post, we will explore the features of `bc` and how to use it.

## What is bc?

`bc` is an arbitrary precision calculator language that supports both integer and floating-point arithmetic. It can handle complex mathematical operations, including trigonometric functions, logarithms, and more. `bc` is often used in shell scripts for calculations that require high precision.

## Installing bc

Most Unix-like systems come with `bc` pre-installed. You can check if it is available on your system by running:

```bash
> bc --version
```

If it is not installed, you can install it using your package manager. For example on Debian/Ubuntu:

```bash
> sudo apt-get install bc
```

## Basic Usage

To start using `bc`, simply type `bc` in your terminal:

```bash
> bc
```

You will enter the `bc` interactive mode, where you can start typing your calculations. For example:

```bash
> 5+3
```

Press Enter, and you will see the result:

```bash
8
```

To exit `bc`, type quit or press `Ctrl + D`

You can also "pipe" echo output to `bc`:

```bash
> echo '2*6' | bc
12
```

## Working with Variables

`bc` is programming language by it self, So you can define variables too. Enter `bc` interactive mode and define your variables in each line:

```bash
> a=4
> b=2
> a*b
8
```

## Functions in bc

You can define your own functions in `bc`. Here's an example of how to define and use a function:

```bash
> bc
> define double(x) {return x*2}
> double(5)
10
```

## Using bc with Files

You can also use `bc` to read expressions from a file. For example create a file named `calc.txt` with the following content:

```txt
5+3
10/2
4*7
```

You can then run `bc` with this file as input:

```bash
> bc calc.txt
8
5
28
```

## More advanced calculations

If basic calculation is not enough for you, a standard math library is also available by the command line option `-l` or `--mathlib`. If requested, the math library is defined before processing any files. By typing executing `bc -l` command, you can enter interactive mode with standard math library defined as well:

```bash
> bc -l
```

Now you can calculate a sine (`s`), cosine (`c`), arctangent (`a`), natural logarithm (`l`), exponential (`e`) function as well using mathlib:

```bash
> bc -l

> s(90)
.89399666360055789051

> c(90)
-.44807361612917015236

> a(90)
1.55968567289728914662

> l(2.71)
.99694863489160953206

> e(1)
2.71828182845904523536
```

## Setting output precision

You can set the output precision by changing `scale` variable. When running `bc` with no command line option, By default it's set to `0`. If running with `-l` option (define mathlib) it's set to `20` by deafult. we can change it to what ever we want:

```bash
> bc -l

> scale
20

> scale=2

> s(90)
.88

> c(90)
-.44

> a(90)
1.55

> l(2.71)
.99

> e(1)
2.71
```

## Conclusion

The `bc` command is a versatile tool for performing calculations in Unix. With its support for arbitrary precision, variables, functions, and file input, it can handle a wide range of mathematical tasks. Whether you are a developer, a data analyst, or just someone who needs to perform calculations, `bc` is a valuable addition to your toolkit.

## References

[1] https://en.wikipedia.org/wiki/Bc_(programming_language)

[2] https://www.gnu.org/software/bc/manual/html_mono/bc.html