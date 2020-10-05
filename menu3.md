
# Tips, Tricks & Agony Aunt
\toc
## Tips & Tricks
### The initial direction is important
Let's call it $\delta \theta$. If model sensitivity to $\delta \theta$ is high, then ...
- if you're lucky the curve will change direction quickly onto a better (more insensitive) direction
- if you're unlucky it won't find a better direction, and will evolve in a slow, jaggedly meandering manner that is disruptive of model behaviour.
- if it involves lots of parameters, it might be hard to interpret

Note that sensitivity is quantified by:

$$ \delta \theta^T \nabla^2 C[\theta^*] \delta \theta $$

The bigger this is, the more sensitive the direction. If you don't have an intuitively motivated initial direction, you could **follow these steps** (which are explicitly done in the NFKB example notebook):

@@unobtrusivebox
1. Fix parameters you're not interested in (if any).
2. Generate an approximation of the Hessian $\nabla^2 C[\theta^*]$ (finite difference, automatic differentiation, `l2_hessian`, or otherwise).  
3. Solve the [QCQP](https://juliacomputing.com/industries/optimization.html):
$$ \min_{\delta \theta} \delta \theta^T \nabla^2 C[\theta^*] \delta \theta + \lambda \| \delta \theta \|_1 \quad \text{subject to} \quad \| \delta \theta \|_2^2 = 1. $$ 

- *The value of $\lambda >0 $ is proportional to your desire to involve only a small number of parameters in the initial direction: it is a sparsity-encouraging ($\mathcal{L}_1$) regularisation term.*
- *If $\lambda = 0$ then the solution is just the eigenvector with the smallest eigenavalue.*  
- Otherwise, this is non-convex, and will usually have multiple local solutions, depending on the initial condition of the optimisation problem. Each of these local solutions is a great candidate for an MD curve! (again, see the NFKB or STG neuron examples for an actual implementation.)
- Set extremely small (e.g. $<1e-5$) components of the initial direction vector to zero. This potentially saves the curve generator some trouble at the beginning
@@
\\ \\
@@infobox
**By the way**
\\
I will add better functionality for easily synthesising good initial directions at some point. Check the examples for now.
@@

### The momentum usually isn't important.

If it's too low, the curve will terminate early (as soon as cost > momentum). It will also slow the evolution of the curve numerically. **But** it can be more accurate (providing a more minimally disruptive curve).

### Exploit parameter biasing and the logabs transform

@@unobtrusivebox
**Questions**
1. *What counts as a 'big' change in a parameter?* 
2. *Is a big change the same across different model parameters?*

**Answers**
1. Very thorny issue. Depends on the (questions being asked of the) model.
2. No.
@@

MinimallyDisruptiveCurves.jl is using the L2 metric on parameter space to *get as far away as possible from the initial parameters while minimising disruption to model behaviour*.

But maybe a small (absolute) $L2$ change in parameter $1$ would be very significant for you, and a big $L2$ change in parameter $2$ would not be.  

- If your parameters don't cross zero (i.e. they are only positive or negative) then use the `logabs_transform` to consider relative, rather than absolute $L2$ changes in parameters. IE $0.01 \to 0.02$ is the same as $100 \to 200$. 

- Otherwise, use `bias_transform` to weight the importance of absolute changes in different parameters.

Using these, and other transformation structures, are demonstrated in the Examples notebook: transforming_costs.ipynb .

### Run curves twice

...if you want to maximise numerical accuracy.

Check which parameters are significantly changing over the curve, and fix all the other parameters using `only_free_parametes`, for the second run.

Of course....your curve might rely on apparently small changes in 'unimportant' parameters on the first curve. But that's useful information!

## The MinimallyDisruptiveCurves.jl agony aunt

@@unobtrusivebox
**You promised that the distance from initial parameters would be monotonically increasing. But my curve is sawtoothing!**
@@

Default settings in the `evolve` function (which generates MD curves) are to have `momentum_tol = 1e-3`. What is this? 
- Functionally, it helps with accuracy of the curve evolution.
- Mathematically (not something to worry about in this context) it uses an algebraic identity on what the costate (i.e. momentum) should be to occasionally reset the costate. 

There is a bug whereby if the MD curve cannot find any new good directions **and** it resets the costate, it can double back on itself and violate the monotonic increase in distance from initial parameter values. You can stop this by setting the keyword argument:
```julia
evolve(...; momentum_tol = NaN)
``` 
I'll fix this at some point.
\\ 
@@unobtrusivebox
**My curve looks like the simulation of a Rodeo rider!**
@@

- Again, try playing around with `; momentum_tol = NaN`, or increasing momentum_tol to a higher value. 

- **Increasing** the momentum can also help. 




