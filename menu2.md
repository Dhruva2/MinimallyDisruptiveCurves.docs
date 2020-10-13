@def title = "Examples"
@def hascode = true


<!-- @def tags = ["syntax", "code", "image"] -->

# Examples


\toc

~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/will_it_work.png">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
  (*Credit: https://xkcd.com/1742/*)

## Pulling notebooks from Github



Tutorial notebooks are in the MDCExamples repository. 
1. At a terminal, in the directory of your choice, type the following commands:
```julia
 git clone https://github.com/Dhruva2/MDCExamples.git
cd MDCExamples
julia ```
This clones the github repository with the examples, changes location to the cloned repository, and starts a julia session.

2. In your new julia session, activate the environment:
```julia
] activate .   
```
`]` switches to the package manager. `activate .` activates the predetermined package environment in which the notebooks can be run.

3. (optional): Precompile the dependencies to save a few minutes later
```julia
] precompile
```

5. Open jupyter notebooks:
```julia
using IJulia
notebook(dir=pwd())
```
and then open/run the individual notebooks in the notebook environment. 

**Suggested order to go through notebooks**

1. `Transforming_cost_functions.ipynb`
2. `MassSpringExample.ipynb`
3. `NFKB_example.ipynb`
4. `stg_neuron_prelim_collocation.ipynb`
5. `CircadianOscillator.ipynb`

- No. 1 just shows you how to play around with cost functions
- No. 2 is an example on an extremely simple model. Running curves takes fractions of a second here, so it's a good playground for playing with curve settings.
- No. 3 is fairly detailed. It's useful if you want to see how scientific insight can be gained from looking at Minimally Disruptive Curves. Each curve on this notebook takes 2-10 mins to generate on my laptop (2017 macbook pro).
- Nos 4. and 5. show you how to use the (much quicker to evaluate) collocation cost. I've generated, but haven't really had time to properly analyse, the minimally disruptive curves.

## A copy-paste, minimal example


- This example is less didactic than the notebooks, but useful for somebody who just wants to get things working quickly.

- We do a cursory analysis of the Lotka-Volterra, predator-prey differential equation model. This is often used as a toy example for global sensitivity analysis tools, see e.g. the following web links: [a](https://cran.r-project.org/web/packages/ODEsensitivity/vignettes/ODEsensitivity.html), [b](https://diffeq.sciml.ai/v6.9/analysis/global_sensitivity/), [c](https://strimas.com/post/lotka-volterra/). Because it's really simple.


- You can copy and paste the code at the bottom, and run it as a .jl script. It runs in seconds (after package/function pre-compilation). Just make sure you add all the package dependencies first:
```julia
] add OrdinaryDiffEq, ForwardDiff, Statistics, Plots, LinearAlgebra, LaTeXStrings
] add https://github.com/Dhruva2/MinimallyDisruptiveCurves.jl.git
```
- Note that one of the model features we analyse is **non-differentiable**. Mathematically. Computationally everything works fine, as we'll explain.

@@unobtrusivebox
### Lotka Volterra model of predator-prey interactions

The dynamics for the system are:
$$ \frac{d
🐁}{dt} =  p_1 🐁 - p_2🐁 🦉 \\
\frac{d
🦉}{dt} = -p_3🦉   + p_4 🐁 🦉
$$  
- $p_1$ and $p_2$ are the birth and death rates of 🐁 
- $p_3$ and $p_4$ are the birth and death rates of 🦉
... and here is a time-course of their populations when
`p = [1.5,1.0,3.0,1.0]`. We call these values the **nominal parameters**.
\\ \\
**Nominal system** (i.e. simulated with nominal parameter values)
~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/nominal_lv.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
@@

- We will follow [b](https://diffeq.sciml.ai/v6.9/analysis/global_sensitivity/) in having **two** features of interest: 
  - **mean** population of 🐁  over time
  - **max** population of 🦉 over time
- The model features of the nominal system are $[2.98, 4.57]$
- Our **cost function on model behaviour** is the mean square deviation from these nominal model features. 

@@infobox
**By the way**
In your ecology class, you may have heard of $r/K$ selection. This is a (criticised) theory of how there are two extremal strategies that individual species trade off between:  
- r-Strategists (e.g. rabbits): **high** birth and death rate. **low** investment in individual organisms
- K-strategists (e.g. humans): **low** birth and death rate. **high** investment in individual organisms
We will use this terminology in our description of the MD curves.
@@

- By running the code at the bottom of the page twice, setting `whichdir = 1` and `whichdir=2` respectively, we get two MD curves (see below).


@@unobtrusivebox
### Minimally disruptive curve 1
~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/mdc1.gif">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
- As we move along the curve, predators move towards an $r$-strategy, and prey moves towards a $K$-strategy.
- We can also move in the opposite direction (prey to $r$ strategy, predator to $K$). To avoid figure clutter, run the pasted code yourself to see this!
@@

@@unobtrusivebox
### Minimally disruptive curve 2
- This curve instead finds a way to modulate the frequency of predator prey interactions, while preserving model features
- Note how it abruptly changes direction when slowing frequency eventually prevents model features being maintained (as the period approaches the length of simulation). It goes from decreasing the frequency to increasing it.
~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/mdc2.gif">
    <div style="clear: both"></div>      
  </div>
</div>
~~~

@@



### The code
- Gives static plots of the minimally disruptive curves. Code for the gifs was a bit more involved!
```julia
using OrdinaryDiffEq, ForwardDiff, MinimallyDisruptiveCurves, Statistics, Plots, LinearAlgebra, LaTeXStrings

#which md curve to plot
which_dir = 2

## define dynamics of differential equation
function f(du,u,p,t)
  du[1] = p[1]*u[1] - p[2]*u[1]*u[2] #prey
  du[2] = -p[3]*u[2] + p[4]*u[1]*u[2] #predator
end

u0 = [1.0;1.0] # initial populations
tspan = (0.0,10.0) #time span to simulate over
t = collect(range(0, stop=10., length=200)) # time points to measure
p = [1.5,1.0,3.0,1.0] # initial parameter values
nom_prob = ODEProblem(f,u0,tspan,p) # package as an ODE problem
nom_sol = solve(nom_prob, Tsit5()) # solve 


## Model features of interest are mean prey population, and max predator population (over time)
function features(p)
  prob = remake(nom_prob; p=p)
  sol = solve(prob, Tsit5(); saveat = t)
  return [mean(sol[1,:]), maximum(sol[2,:])]
end

nom_features = features(p)

## loss function, we can take as l2 difference of features vs nominal features
function loss(p)
  prob = remake(nom_prob; p=p)
  p_features = features(p)
  loss = sum(abs2, p_features - nom_features)
  return loss
end

## gradient of loss function
function lossgrad(p,g)
  g[:] = ForwardDiff.gradient(p) do p
    loss(p)
  end
  return loss(p)
end

## package the loss and gradient into a DiffCost structure
cost = DiffCost(loss, lossgrad)

"""
We evaluate the hessian once only, at p.
Why? to find locally insensitive directions of parameter perturbation
The small eigenvalues of the Hessian are one easy way of defining these directions 
"""
hess0 = ForwardDiff.hessian(loss,p)
ev(i) = eigen(hess0).vectors[:,i]

## Now we set up a minimally disruptive curve, with nominal parameters p and initial direction ev(1) 
init_dir = ev(which_dir); momentum = 1.; span = (-15.,15.)
curve_prob = curveProblem(cost, p, init_dir, momentum, span)
@time mdc = evolve(curve_prob, Tsit5)

function sol_at_p(p)
  prob = remake(nom_prob; p=p)
  sol = solve(prob, Tsit5())
end

p1 = plot(mdc; pnames = [L"p_1" L"p_2" L"p_3" L"p_4"])

cost_vec = [mdc.cost(el) for el in eachcol(trajectory(mdc))]
p2 = plot(distances(mdc), log.(cost_vec), ylabel = "log(cost)", xlabel = "distance", title = "cost over MD curve");

mdc_plot = plot(p1,p2, layout=(2,1), size = (800,800))

nominal_trajectory = plot(sol_at_p(mdc(0.)[:states]), label = ["prey" "predator"])
perturbed_trajectory = plot(sol_at_p(mdc(-15.)[:states]), label = ["prey" "predator"])
traj_comparison = plot(nominal_trajectory, perturbed_trajectory, layout = (2,1), xlabel = "time", ylabel = "population")
```
### Lessons

- MinimallyDisruptiveCurves.jl showed how we could continuously vary the parameters of the Lotka-Volterra model, while preserving
  1. mean prey population over time
  2. max predator population over time 

- One strategy (MDC1) was to alter the parameters to increase (decrease) birth and death rates of the predators while decreasing (increasing) those of the prey. Along the axis of $r$-strategies to $K$-strategies, this corresponds to one species moving in one direction, with the other going in reverse.

- Another strategy (MDC2) was to modulate the frequency of the oscillations. MDC2 showed us exactly how to change the parameters to continuously modulate this frequency.
  
@@infobox
**By the way**

The maximum predator population over time is a **non differentiable** function of the solution. How come this worked?

*Well, the maximum of a collection of elements (predator measurements over time), is differentiable except at crossing points, where two elements share a maximal value. Different values of predator(t) will never have **exactly** the same value, due to numerical error as well as equation dynamics. So it works out! Indeed, notice the timepoints at which maximal values are attained changes with the model parameters.
@@


