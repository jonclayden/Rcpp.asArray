[![CRAN version](https://www.r-pkg.org/badges/version/RcppArray)](https://cran.r-project.org/package=RcppArray) [![CI](https://github.com/jonclayden/RcppArray/actions/workflows/ci.yaml/badge.svg)](https://github.com/jonclayden/RcppArray/actions/workflows/ci.yaml) [![Dependencies](https://tinyverse.netlify.app/badge/RcppArray)](https://tinyverse.netlify.app)


# `Rcpp` interoperability with `std::array` (et al.)

[`Rcpp`](https://www.rcpp.org) is a mature and very widely used package providing seamless interoperability between C++ and [the R language](https://www.r-project.org), which has a native C API. This package facilitates conversion between R data structures and the templated [`std::array` class](https://en.cppreference.com/w/cpp/container/array) introduced in C++11, by using the interface provided in `Rcpp`. It originated with [this Stack Overflow question](https://stackoverflow.com/questions/74887786/specialising-rcppas-for-stdarray).

Support for [`std::tuple`](https://en.cppreference.com/w/cpp/utility/tuple) was added in package version 0.3.0. This data structure, also introduced in C++11, combines a fixed set of elements of potentially different types.

`std::array` is a templated container type with a fixed number of elements, an object-orientated analogue of a C-style array type like `int[3]`. Client packages can interface this type with R if they add `RcppArray` to `LinkingTo` and include the header. (A simple [example package](https://github.com/jonclayden/RcppArray/tree/main/clients/array.test) is provided.)

```c++
// No need to include "Rcpp.h" as well
#include "RcppArray.h"

// [[Rcpp::export]]
RObject test() {
  Rcpp::NumericVector vec = Rcpp::NumericVector::create(1,2,3);
  std::array<double,3> arr = Rcpp::as<std::array<double,3>>(vec);
  // Do something with the array
  
  std::array<unsigned int,3> result = { 1, 2, 3 };
  return result;    // Implicitly Rcpp::wrap(result)
}
```

In either direction the element type of the array can be any atomic type that `Rcpp` knows how to convert, and isn't limited to the types that R uses internally. In the example above, `unsigned int` is used even though it doesn't directly correspond to an R vector mode: the `wrap()` function will convert it to an R `numeric` vector, i.e., `double`.

A tuple, declared in C++ via a variadic template, as in `std::tuple<int, double, std::string>`, can be converted similarly using `Rcpp::as()` and `Rcpp::wrap()`. In this case the R analogue is a list rather than an atomic vector, since the element types can vary.

## std::span

There is also experimental support for [`std::span`](https://en.cppreference.com/w/cpp/container/span), introduced in C++20. This is a typed container of fixed or dynamic length which provides a "view" onto a contiguous block of data owned by another object. Once again, there is a simple [example client package](https://github.com/jonclayden/RcppArray/tree/main/clients/span.test) showing usage of this facility.

In this case there are some notable caveats. Client packages must obviously request C++20 support, and [a `configure` script](https://github.com/jonclayden/RcppArray/blob/main/clients/span.test/configure.ac) may well be needed to check for `span` availability. Moreover, since a `span` does not own the memory it points to, `as<span<T,D>>()` will only compile where the requested type `T` matches a type that R uses internally (viz. `int`, `double`, etc.). The latter limitation does not apply to `wrap`, however, because a new vector is created and the data converted.
