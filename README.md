# Root finding functions for Julia

This package contains simple routines for finding roots of
continuous scalar functions of a single real variable. 


The following functions are provided:

* `find_zero`: A robust and pretty efficient bracketing method guaranteed
  to find a root in the interval [a,b] provided the interval brackets a
  root due to Alefeld, Potra, and Shi.

* `newton`: Newton's method for iteratively finding a root using f and
  its derivative f'. If the derivative is unknown, it is computed for
  simple functions through forward automatic differentiation.

* `halley`: Halley's third-order improvement to Newton's method using f,
  f', and f''. If the derivatives are unknown, they are computed for simple
  functions through forward automatic differentiation.

* `thukral`: A derivative free, eighth-order iterative method due to
  Thukral, ["New Eighth-Order Derivative-Free Methods for Solving
  Nonlinear Equations", Intl. Jrnl. of Mathematics and Mathematical
  Sciences
  (2012)](http://www.hindawi.com/journals/ijmms/2012/493456/). There
  is also an order 16 method due to Thukral and an order 5 method of
  Li, Mu, Ma, Hou.

* `multroot`: An improvement on the `roots` function of the
  `Polynomial` package when multiple roots are present. Follows
  algorithms due to Zeng, ["Computing multiple roots of inexact
  polynomials", Math. Comp. 74 (2005),
  869-903](http://www.ams.org/journals/mcom/2005-74-250/S0025-5718-04-01692-8/home.html).

## fzero

The main interface is through the function `fzero` which dispatches to an appropriate algorithm  based on its arguments:

* `fzero(p::Poly)` calls `multroot`

* `fzero(p::Union(Function, Poly), a::Real, b::Real)` calls `find_zero` algorithm for bracket `[a,b]`.

* `fzero(p::Union(Function, Poly), bracket::Vector)` calls `find_zero` algorithm.

* `fzero(p::Union(Function, Poly), x0::Real)` calls `thukral` algorithm with initial guess `x0`

* `fzero(p::Union(Function, Poly), x0::Real, bracket::Vector)` calls
  `thukral` algorithm with initial guess `x0` with steps constrained
  to remain in the specified bracket.

## Usage examples

### Finding roots with `fzero`: Polynomials

```julia
using Roots, Polynomial

p = Poly([1, -4, 5, -2])	## (x-1)^2 * (x-2)
fzero(p)			## roots with multiplicities
fzero(p, 1.70)			## use iterative algorithm from x0 = 1.7
fzero(p, 1.60)			## converges to 1.0, not 2.0
fzero(p, 1.60, [0.5,3])		## bracket ensures convergence to 2.0
```


## Finding roots of functions

For functions one must either bracket or provide an initial
guess (or both). As there is no general root solving algorithm equivalent of `multroot` for functions.




```julia
using Roots

f(x) = sin(x)
fzero(f, 3)
fzero(f, 3) |> f
fzero(f, 3, [2, 4])
```

The convergence can be controlled by the parameters `tol` to specify
the desired tolerance between `f(xn1)` and `f(xn)`; `delta` for the
tolerance between `xn1` and `xn`, and `max_iter` to limit the number
of iterations:

```julia
using Roots

f(x) = 2x*exp(-20) - 2*exp(-20x) + 1

root = fzero(f, [0.0, 1.0]; tol=1e-10, max_iter=100) ## 0.03465735902085387
f(root)						     ## 3.3306690738754696e-16
```

### Using the other functions directly

There may be problems where the direct use of the underlying
algorithms is desired. For example, if you know the derivative of your
function, you can use the Newton method with a single initial guess:

```julia
using Roots

f(x) = exp(x) - cos(x)
fp(x) = exp(x) + sin(x)

root = newton(f, fp, 3.0)	## 1.549117546845831e-17
f(root)		     		## 0.0
```

If you are lucky enough to know the second derivative, you can also use
Halley's method, which has cubic convergence (rather than quadratic for Newton):

```julia
using Roots

f(x) = exp(x) - cos(x)
fp(x) = exp(x) + sin(x)
fpp(x) = exp(x) + cos(x)

root = halley(f, fp, fpp, 3.0)	## 2.868142812191158e-17
f(root)		 		## 0.0
```

### Derivative free computations

There is s ome simple forward automatic differentiation code to
automatically compute the derivatives for `newton` and `halley`, so
one need not know the derivative:

```julia
using Roots
f(x) = exp(x) - cos(x)

root = newton(f, 3.0)		## 1.549117546845831e-17
f(root)		 		## 0.0
```


The `thukral` function implements an alternative algorithm which is
derivative free. In addition, the `thukral` method converges with a
higher order. It is a viable alternative to `newton` or `halley`.  For
example, as automatic differentiation works only for simple functions
-- and not those returned by an automatic differentiation, `thukral`
can be used to find critical points, as follows:

```julia
using Roots

f(x) = 1/x^2 + x^3
root = thukral(D(f), 1)		## 0.9221079114817278, also just fzero(D(f), 1)
D2(f)(root)			## positive, so a minimum
```

The operator `D(f::Function,k::Integer)` returns the function `x ->
f^(k)(x)`, with specializations `D(f)` giving the first derivative and
`D2(f)` the second.

### Polynomials

The `Polynomial` package provides the `roots` function. It works well
for polynomials of moderate degree with simple, well-separated
roots. The `multroot` function is an implementation of work due to
Zeng that handles polynomials with multiplicities in their roots. (For
polynomials without identified multiplicities, it does some scratch work then
basically returns a call to `roots`.)

```julia
using Roots
using Polynomial

## (x-1)^2*(x-2)^2*(x-3)^4
p = Poly([1,-1])^2 * Poly([1, -2])^2 * Poly([1, -3])^4
roots(p) ## not terrible, but multroot does a bit better
z,l = fzero(p) ## ([1.0000000000000004,1.999999999999997,3.0000000000000013],[2,2,4])
```
	
Some reasonably large polynomials can be factored:

```julia
using Roots
using Polynomial

p =Poly([1, -1])^20 * Poly([1,0])^20 * Poly([1,1])^20
fzero(p)   ## "fails" with: ([-8.235908404184379e-13,-0.9999999999992275,1.000000000000883],[20,20,20])
```

Compare to 

```{execute=false}
using Winston
r = roots(p)
plot(real(r), imag(r))
```
