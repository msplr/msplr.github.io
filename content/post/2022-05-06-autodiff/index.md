+++
author = "Michael Spieler"
title = "Exploring Eigen AutoDiff"
slug = "eigen-autodiff"
date = "2022-05-06"
tags = [
    "C++",
    "Math"
]
image = "derivative.svg"
+++

<!-- ![Derivative Plot](derivative.png) -->

## Automatic Differentiation with Eigen

A probably rather unknown feature of the excellent Eigen library is the (unsupported) `AutoDiff` module.
It implements simple forward mode [Automatic Differentiation](https://en.wikipedia.org/wiki/Automatic_differentiation) through a special [AutoDiffScalar](https://eigen.tuxfamily.org/dox/unsupported/classEigen_1_1AutoDiffScalar.html) type that replaces `float` or `double`.
An `AutoDiffScalar` keeps track of the scalar value as well as the derivatives with respect to the input variables.

Here is a very simple example

```cpp
#include <iostream>
#include <Eigen/Dense>
#include <unsupported/Eigen/AutoDiff>

using Derivative = Eigen::Vector2d;
using ADScalar = Eigen::AutoDiffScalar<Derivative>;

template <typename Scalar>
Scalar f(const Scalar &x, const Scalar &y)
{
    return x*sin(x) + exp(y);
}

int main()
{
    ADScalar x(3.141, Derivative::RowsAtCompileTime, 0);
    ADScalar y(1.0, Derivative::RowsAtCompileTime, 1);

    ADScalar res = f(x, y);

    double value = res.value();
    Derivative derivatives = res.derivatives();

    std::cout << "f(x, y) = " << value << '\n'
              << "df/dx = " << derivatives(0) << '\n'
              << "df/dy = " << derivatives(1) << '\n';
}
```

This outputs the following:

```
f(x, y) = 2.72014
df/dx = -3.14041
df/dy = 2.71828
```

The derivatives of `f(x, y)` are `df/dx = sin(x) + x * cos(x)` and
`df/dy = exp(y)`.
Evaluating them at `x=3.141` and `y=1.0` gives the same result.

## Conclusion

I think this is very useful for prototyping without introducing any additional dependencies.
Of course it's not as nicely integrated as pytorch and keep in mind that forward mode AD performs poorly with many input variables.

If you're interested in the how this works underneath, there is excellent documentation of the [Ceres Solver on Automatic Derivatives](http://ceres-solver.org/automatic_derivatives.html).
Ceres ships with its own implementation of AutoDiff, similar to the one from Eigen.
