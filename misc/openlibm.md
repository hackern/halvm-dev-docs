# OpenLibM

The famous Mirage project is ported to the ARM Platform based on the mini-OS and openlibm backend. So it seems that HaLVM also wants to use openlibm. But there is nothing under the directory. So I will mainly cover some general information about openlibm.

[Website](http://openlibm.org)

In general, OpenLibm is an effort to have a high quality, portable, standalone C mathematical library (`libm`). It is maintain under the famous Julia Language project.

OK, but what's the difference between `libgmp` and `libm`?

### GMP (The GNU Multiple Precision arithmetic library)

> GMP is a free library for arbitrary precision arithmetic, operating on signed integers, rational numbers, and floating-point numbers. There is no practical limit to the precision except the ones implied by the available memory in the machine GMP runs on. GMP has a rich set of functions, and the functions have a regular interface.

> The main target applications for GMP are cryptography applications and research, Internet security applications, algebra systems, computational algebra research, etc.

### libM
According to a [Sun implementation version's specification](http://docs.oracle.com/cd/E19957-01/806-3568/ncg_lib.html), the content involves:


* Algebraic functions: `cbrt, hypot, sqrt`
* Elementary transcendental functions: `asin, acos, atan, atan2, asinh, acosh, atanh, exp, expm1, pow, log, log1p, log10, sin, cos, tan, sinh, cosh, tanh`
* Higher transcendental functions: `j0, j1, jn, y0, y1, yn, erf, erfc, gamma, lgamma, gamma_r, lgamma_r`
* Integral rounding functions: `ceil, floor, rint`
* IEEE standard recommended functions: `copysign, fmod, ilogb, nextafter, remainder, scalbn, fabs`
* IEEE classification functions: `isnan`
* Old style floating-point functions: `logb, scalb, significand`
* Error handling routine (user-defined): `matherr`