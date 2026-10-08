<!---
{
  "id": "ce9d3ddc-a17a-4f76-b03a-3c85c14a154f",
  "teaches": "Cpp: Operator Overloading on a Complex Number Type",
  "depends_on": [
    "46f90e5e-6416-4bf5-8f5c-9255015fd23a",
    "0aecd541-54e5-4f8b-8360-aeea231d05e9"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2026-10-08",
  "keywords": [
    "C++",
    "operator overloading",
    "operator+",
    "operator==",
    "operator<<",
    "complex",
    "const correctness",
    "friend"
  ]
}
--->

# Cpp: Operator Overloading on a Complex Number Type

> In this exercise you write a class for complex numbers and teach it `+`, `-`, `*`, `==`, `+=` and output with `<<`, so that `Complex c = a * b + 1.0;` reads like mathematics. You will see which operators are methods, which are free functions, and what the compiler generates behind the symbols. No `sudo` is required.

## Introduction

C only knows the built-in meaning of `+`. For your own types you write `complex_add(a, b)`, and a longer formula becomes unreadable. C++ lets a class define what an operator means when one of its operands is an object of that class. An overloaded operator is an ordinary function with a special name: `a + b` is a call to `operator+(a, b)` or `a.operator+(b)`, nothing more.

The lecture lists the operators that can be overloaded: arithmetic, comparison, logical, assignment, memory and a few special ones. This exercise uses the ones you will meet most, on the lecture's own example, a complex number.

### 1.1) Further Readings and Other Sources

* [cppreference: operator overloading](https://en.cppreference.com/w/cpp/language/operators)
* [C++ Core Guidelines: C.160 to C.168 (overloading)](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-overload)
* [Moodle page of this course: Vergleich der Implementierung einer komplexen Zahl](https://moodle.thga.de/course/view.php?id=2448&section=24)

## Tasks

### Task 1: The C way, for comparison

```bash
mkdir -p ~/oop-6 && cd ~/oop-6
```

```c
// file: complex.c
#include <stdio.h>
struct complex { double re, im; };
struct complex cadd(struct complex a, struct complex b) { return (struct complex){ a.re + b.re, a.im + b.im }; }
struct complex cmul(struct complex a, struct complex b) { return (struct complex){ a.re*b.re - a.im*b.im, a.re*b.im + a.im*b.re }; }
int main(void) {
    struct complex a = {1, 2}, b = {3, -1};
    struct complex c = cadd(cmul(a, b), a);     /* (a*b) + a */
    printf("%.1f%+.1fi\n", c.re, c.im);
    return 0;
}
```

```bash
gcc -Wall -Wextra complex.c -o complex_c && ./complex_c
```

### Task 2: `operator+` and `operator*` as methods

```c++
// file: complex.cpp
#include <cstdio>
class Complex {
public:
    Complex(double r = 0, double i = 0) : re(r), im(i) {}
    Complex operator+(const Complex &o) const { return { re + o.re, im + o.im }; }
    Complex operator*(const Complex &o) const { return { re*o.re - im*o.im, re*o.im + im*o.re }; }
    double real() const { return re; }
    double imag() const { return im; }
private:
    double re, im;
};
int main() {
    Complex a(1, 2), b(3, -1);
    Complex c = a * b + a;
    std::printf("%.1f%+.1fi\n", c.real(), c.imag());
}
```

```bash
g++ -Wall -Wextra complex.cpp -o complex && ./complex
```

Same result, readable formula. Then `nm -C complex | grep operator`: the operators are functions named `Complex::operator+(Complex const&) const`.

### Task 3: Precedence is not yours to change

Verify that `a * b + a` multiplies first: print `(a * b) + a` and `a * (b + a)`. Operator precedence and associativity come from the language, overloading only supplies the meaning.

### Task 4: Mixed operands need free functions

`Complex d = a + 1.0;` works, because `1.0` converts to `Complex(1.0)` through the constructor. `Complex e = 1.0 + a;` does **not** compile as a method: the left operand is a `double`, which has no `operator+` taking a `Complex`. Add a free function:

```c++
Complex operator+(double d, const Complex &c) { return Complex(d) + c; }
```

Then try `1.0 * a` and explain what is needed. This is why many libraries define binary operators as free functions for both orders.

### Task 5: Comparison and compound assignment

```c++
    bool operator==(const Complex &o) const { return re == o.re && im == o.im; }
    bool operator!=(const Complex &o) const { return !(*this == o); }
    Complex& operator+=(const Complex &o) { re += o.re; im += o.im; return *this; }
```

Use `+=` in a loop that sums ten values, and note that `+=` returns a reference so that `a += b += c` works. Then implement `operator+` *in terms of* `operator+=`, the way the Core Guidelines recommend:

```c++
Complex operator+(Complex a, const Complex &b) { return a += b; }
```

### Task 6: Output with `<<`

```c++
#include <iostream>
std::ostream& operator<<(std::ostream &os, const Complex &c) {
    return os << c.real() << (c.imag() < 0 ? "" : "+") << c.imag() << "i";
}
std::cout << a * b + a << "\n";
```

`<<` must be a free function because the left operand is the stream, not your class. If the operator needs private members, declare it a `friend` inside the class and compare both variants.

### Task 7: Operators that should not be overloaded

Overload `operator&&` for `Complex` to return `true` if both are non-zero. Then write

```c++
Complex zero;
if (zero && expensive())   // expensive() prints something
```

and observe that `expensive()` is called although `zero` is zero: an overloaded `&&` loses short-circuit evaluation. Delete the overload. Look up in the Core Guidelines which other operators (`,` and `&`) are on the same list.

## Questions

- What is `a + b` for a class with `operator+`? Write the call the compiler generates, for the method and for the free-function form.
- Why can `1.0 + a` not be a method of `Complex`?
- Why does `operator+=` return `Complex&` and `operator+` return `Complex` by value?
- Why must `operator<<` be a free function, and when does it need to be a `friend`?
- Which operators cannot be overloaded at all, and which can but should not? Give the reason for `&&`.
- Compare `cadd(cmul(a,b),a)` with `a * b + a`. Which is easier to get wrong, and why does that matter for a numeric library?

## Advice

Overloaded operators are syntactic sugar over functions you could have written anyway. Use them only where the meaning is obvious to a reader who knows the domain: `+` for vectors and complex numbers, `==` for anything comparable, `<<` for output. If you find yourself explaining what your `operator%` does, write a named method instead.

The pattern "compound assignment as a method, binary operator as a free function built on it" scales to every arithmetic type you will ever write.
