---
title: "Tips, Tricks & Agony Aunt"
---

## Tips & Tricks

### The initial direction is important
Let's call it \\(\delta \theta \\). If model sensitivity to \\(\\delta \theta \\) is high, then …
- if you're lucky the curve will change direction quickly onto a better ((more insensitive) direction  
- if you're unlucky it won't find a better direction, and will evolve in slow, jaggedly meandering manner that is disruptive of model behaviour.  
- if it involves lots of parameters, it might be hard to interpret  

Note that sensitivity is quantified by:

$$
\\delta \\theta^{T} \\nabla^{2} C[\\theta^{*}] \\delta \\theta
$$

The bigger this is, the more sensitive the direction. If you don't have intuitively motivated initial direction, you could **follow these steps** (which are explicitly done in the NFKB example script):

{{< box type="section" >}}
1. Fix parameters you're not interested in (if any) using `FixedParamsTransform()`.  
2. Generate an approximation of the Hessian \\(\\nabla^{2} C(\\theta^{*})\\) (finite difference, automatic differentiation, or otherwise).  
3. Use the built-in `sparse_init_dir` or `sparse_eigenbasis` functions to solve the [QCQP](https://en.wikipedia.org/wiki/):

$$
\min_{\delta \theta} \delta \theta^{T} \nabla^{2} C[\theta^{\*}] \delta \theta + \lambda \| \delta \theta \|_{1}
$$

$$
\\quad \\text{subject to} \\quad \\| \\delta \\theta \\|_{2}^{2} = 1.
$$  

- *The value of \\(\\lambda >0 \\) is proportional to your desire to involve only a small number of parameters in the initial direction: it is a sparsity‑encouraging (\\(\\mathcal{L}_{1}\\)) regularisation term.*  
- *If \\(\\lambda = 0\\) then the solution is just the eigenvector with the smallest eigenvalue.*  
- Otherwise, this is non‑convex, and will usually have multiple local solutions, depending on the initial condition of the optimisation problem. Each of these local solutions is a great candidate for an MD curve! (again, see the NFKB or STG neuron examples for an actual implementation 
- `sparse_init_dir` automatically sets extremely small (e.g. \\(<1e-5\\)) components of the initial direction vector to zero. This potentially saves the curve generator some trouble at the beginning.  
{{< /box >}}

{{< box type="info" >}}
**By the way**  
I will add better functionality for easily synthesising good initial directions at some point. Check the examples for now.  
{{< /box >}}

### The momentum usually isn't important.

If it's too low, the curve will terminate early (as soon as cost > momentum). It will also slow the evolution of the curve numerically. **But** it can be more accurate (providing a more minimally disruptive curve).


### Exploit parameter biasing and the `LogAbsTransform()` transform

{{< box type="section" >}}
**Questions**  
1. *What counts as a ‘big’ change in a parameter?*  
2. *Is a big change the same across different model parameters?*  

**Answers**  
1. Very thorny issue. Depends on the (questions being asked of the) model 
2. No.  
{{< /box >}}

MinimallyDisruptiveCurves.jl is using the L2 metric on parameter space to *get as far away as possible from the initial parameters while minimising disruption to model behaviour*.

But maybe a small (absolute) \\(L2\\) change in parameter 1 would be very significant for you, and a big (absolute) \\(L2\\) change in parameter 2 would not be.  

- If your parameters don't cross zero (i.e. they are only positive or negative) then use the `LogAbsTransform()` to consider relative, rather than absolute \\(L2\\) changes in parameters.  IE \\(0.01 \\to 0.02\\) is the same as \\(100 \\to 200\\).  

- Otherwise, use `ScaleTransform()` to weight the importance of absolute changes in different parameters.  The more uniform this is, **the faster the MD curve will evolve**, heuristically.  

Using these, and other transformation structures, are demonstrated in the Example scripts.  

### Run curves twice

…if you want to maximise numerical accuracy.

Check which parameters are significantly changing over the curve, and fix all the other parameters using `FixedParamsTransform()`, for the second run.

Of course…your curve might rely on apparently small changes in ‘‘unimportant’ parameters on the first curve. But that's useful information!

## Agony aunt

### Curve doesn't get started when evolving

In other words, when you set the Verbose callback to tell you how far along the curve you are, the distance from the origin grows very slowly / slowly / stalls close to zero.

- Use a fixed‑timestep method when calling `MDCSolve`:

```julia
MDCSolve(sys; span=MDCSpan(-10.0, 10.0), mode=:fixed, dt=0.05)
```
or  

```julia
MDCSolve(sys; span=MDCSpan(-10.0, 10.0), mode=:fast, dt=0.05)
```
where the `dt` keyword argument is optional and `0.05` is fairly arbitraryarbitrary.

{{< box type="info" >}}
**By the way**  
- Usually the cause of this problem is stiffness in the differential equation that generates the curve, when close to the origin.  
- For some discussion of this, see the fourth question on [Q&A with a sceptic](/sceptic/).  
- Heuristically, adaptive‑timestep solvers sometimes get worried about stiffness, and start proposing iteratively smaller step sizes to control numerical error at the origin. If they bludgeon through the first tiny section of curve, this disappears.  
{{< /box >}}

### Curve is sawtoothing

Default settings in the `MDCSolve` function (which generates MD curves) are to have `momentum_tol = 1e-3` via the `mdc_momentum_readjustment` callback. What is this?  

- Functionally, it helps with accuracy of the curve evolution.  
- Mathematically (not something to worry about in this context) it uses algebraic identity on what the costate (i.e. momentum) should be to occasionally reset the costate.  

There is a bug whereby if the MD curve cannot find any new good **and** it resets the costate, it can double back on itself and violate the monotonic increase in distance from initial parameter values. You can stop this by simply omitting the callback:

```julia
MDCSolve(sys; span=MDCSpan(-10.0, 10.0)) 
```
I'll fix this at some point. Pragmatically it means the curve can't find direction that gets further away from the origin, at the point of the sawtooth.

### Curve is slow to generate

- Make sure you have optimised the gradient calculation of your cost function. A gnarly curve might take e.g. 5000 evaluations of this, which is the dominant source of compute time.  
- Increase the momentum by two orders of magnitude.

### Curve generation hangs

- Does the span of your curve cross zero? IE is it of the form \\((-a, b)\\) where \\(a,b > 0\\)? Sometimes on Pluto, this hangs for unknown reasons probably to do with multithreading. Try generating two curves: \\((-a, 0)\\) and \\((0, b)\\) and sticking them together.  

More specifically, two‑sided curves as above are evolved as two separate curves running on two separate threads on MinimallyDisruptiveCurves.jl. Sometimes, this doesn't seem to play nicely on Pluto.jl notebooks.  
