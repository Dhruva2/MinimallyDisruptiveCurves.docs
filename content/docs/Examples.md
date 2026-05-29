---
title: "Examples"
weight: 2
---

# Examples

{{< figure src="/images/will_it_work.png" >}}
(*Credit: https://xkcd.com/1742/*)

## Notebooks

[nfkb](/examples/NFKBExample.html)


## Tutorial notebooks

Tutorial Pluto notebooks are in the [MDCExamples](https://github.com/[MDCExamples) repository. Pluto notebooks are like Jupyter notebooks, but (IMO) better for Julia. To install [Pluto](https://github.com/fonsp/Pluto.jl), open a Julia REPL and type

```julia
]add Pluto
using Pluto
Pluto.run()
```

Install/open Pluto and copy one of the tutorial URLs into the Pluto bar. I've put them in a suggested order:

1. https://github.com/Dhruva2/MDCExamples/blob/master/

Not about MinimallyDisruptiveCurves specifically, but introduces you to the cost‑function transformations that are useful when iteratively running MDCs.

2 https://github.com/Dhruva2/MDCExamples/blob/master/mass_spring_intro.

Applying MDCs to the simplest possible differential equation: a damped oscillator. MDC generation happens in the blink of an eye for a model this simple, so it's useful for playing around.

3. https://github.com/Dhruva2/MDCExamples/blob/master/NFKBExample.jl  

Takes a seminal model of the NFkB metabolic pathway. Shows how you can iteratively extract mechanistic insight from the model using MDCs.

4. https://github.com/Dhruva2/MDCExamples/blob/master/

Takes a seminal but rather large model of the intracellular rhythm. Shows how you can use a “cheap’’ loss function (the injection/injection/collocation loss) to massively speed up MDC generation. The seem eminently interpretable, but I haven’t interpreted them yet. Have a go, if you’re a Circadian expert!

5. https://github.com/Dhruva2/MDCExamples/blob/master/calcium_homeo.jl  

(Built by Andrea Ramirez Hincapie). I’m a neuron. I have to modulate my electrical activity (e.g. from spiking to bursting). I also have to maintain calcium homeostasis: internal calcium concentration needs to be in a tight range for me to be happy. But changing baseline electrical activity changes baseline calcium! How can I safely neuromodulate, while preserving calcium concentration? MDCs tell me how…

## A copy‑paste, minimal example

- This example is less didactic than the notebooks, but useful for somebody who just wants to get things working quickly.  
- We do a cursory analysis of the Lotka‑Volterra, predator‑prey differential‑equation model. This is often used as a toy example for global‑sensitivity‑analysis tools; see e.g. the following links:  
  – <https://cran.r-project.org/web/packages/ODEsensitivity/vignettes/  – <https://diffeq.sciml.ai/v6.9/analysis/global_sensitivity/>  
  – <https://strimas.com/post/lotka-volterra/>. Because it’s really simplesimple.

- You can copy and paste the code at the bottom, and run it as a *.jl* script. It runs in seconds (after package/function pre‑compilation). Just make sure you add all the package dependencies first:

```julia
] add OrdinaryDiffEq, ForwardDiff, Statistics, Plots, LinearAlgebra, LaTeXStrings
] add https://github.com/Dhruva2/MinimallyDisruptiveCurves.jl.git
```

- Note that one of the model features we analyse is **non‑differentiable**non‑differentiable**. Mathematically that would be a problem, but computationally everything works fine – we’ll explain why later.

{{< box type="section" >}}
### Lotka‑Volterra model of predator‑prey interactions

The dynamics for the system are  

$$
\begin{aligned}
\frac{d\text{🐁}}{dt} &=  p_1\text{🐁} - p_2 \text{🦉} \text{🐁}  \\\
\frac{d\text{🦉}}{dt} &= -p_3\text{🦉}   + p_4 \text{🐁} \text{🦉}
\end{aligned}
$$  

- \\(p_1\\) and \\(p_2\\) are the birth and death rates of the mice.  
- \\(p_3\\) and \\(p_4\\) are the birth and death rates of the owls.  

… and here is a time‑course of their populations when   ` p = [1.5, 1.0, 3.0, 1.0] `.  
We call these values the **nominal parameters**.




{{< /box >}}

**Nominal system** (i.e. simulated with nominal parameter values)  
{{< figure src="/images/nominal_lv.svg" >}}

We will follow two features of interest:

- **mean** population of the mice over time,  
- **max** population of the owls over time.  

The model‑feature vector for the nominal system is \\([2.98,\,4.57]\\).

Our **cost function on model behaviour** is the mean‑square deviation these nominal features.

{{< box type="info" >}}
**By the way**  
In your ecology class you may have heard of \\(r/K\\) selection. This is (criticised) theory that there are two extremal strategies that species trade off between:  

- **r‑strategists** (e.g. rabbits): high birth and death rates, low investment per individual.  
- **K‑strategists** (e.g. humans): low birth and death rates, high investment per individual.  

We will use this terminology in our description of the MD curves.  
{{< /box >}}

- By running the code at the bottom of the page twice, setting `which_dir = 1` and `which_dir = 2` respectively, we obtain two MD curves (see below).

### Minimally disruptive curve 1

{{< figure src="/images/mdc1.gif" >}}

- As we move along the curve, predators move towards an \\(r\\)-strategy and prey move towards a \\(K\\)-strategy.  
- We can also move in the opposite direction (prey to \\(r\\) strategy, predator to \\(K\\)). To avoid figure clutter, run the pasted code yourself to see this!  

### Minimally disruptive curve 2

{{< figure src="/images/mdc2.gif" >}}

- This curve instead finds a way to modulate the frequency of predator‑predator‑prey interactions while preserving model features.  
- Note how it abruptly changes direction when slowing frequency prevents model features from being maintained (as the period approaches the length of the simulation). It goes from decreasing the frequency to increasing it.  

### The code

- Gives static plots of the minimally disruptive curves. Code for the was a bit more involved!

```julia
using OrdinaryDiffEq, ForwardDiff, MinimallyDisruptiveCurves, Statistics,
      Plots, LinearAlgebra, LaTeXStrings

# which md curve to plot
which_dir = 2

## define dynamics of differential equation
function f(du, u, p, t)
    du[1] = p[1] * u[1] - p[2] * u[1] * u[2]   # prey
    du[2] = -p[3] * u[2] + p[4] * u[1] * u[2]  # predator
end

u0 = [1.0; 1.0]                     # initial populations
tspan = (0.0, 10.0)                 # time span to simulate over
t = collect(range(0, stop=10.0, length=200))   # time points to measure
p = [1.5, 1.0, 3.0, 1.0]            # initial parameter values
nom_prob = ODEProblem(f, u0, tspan, p)          # package as an ODE problem
nom_sol = solve(nom_prob, Tsit5())               # solve

## Model features of interest are mean prey population, and max predator population (over time)
function features(p)
    prob = remake(nom_prob; p=p)
    sol = solve(prob, Tsit5(); saveat=t)
    return [mean(sol[1, :]), maximum(sol[2, :])]
end

nom_features = features(p)

## loss function, we can take as ℓ₂ difference of features vs nominal features
function loss(p)
    prob = remake(nom_prob; p=p)
    p_features = features(p)
    return sum(abs2, p_features - nom_features)
end

## gradient of loss function
function lossgrad(p, g)
    g[:] = ForwardDiff.gradient(loss, p)
    return loss(p)
end

## package the loss and gradient into a DiffCost structure
cost = DiffCost(loss, lossgrad)

"""
We evaluate the hessian once only, at p.
Why? to find locally insensitive directions of parameter perturbation.
The small eigenvalues of the Hessian are one easy way of defining these directions.
"""
hess0 = ForwardDiff.hessian(loss, p)
ev(i) = eigen(hess0).vectors[:, i]

## Now we set up a minimally disruptive curve, with nominal parameters p and initial direction ev(1)
init_dir = ev(which_dir)
momentum = 1.0
span = (-15.0, 15.0)
curve_prob = MDCProblem(cost, p, init_dir, momentum, span)
@time mdc = evolve(curve_prob, Tsit5)

function sol_at_p(p)
    prob = remake(nom_prob; p=p)
    solve(prob, Tsit5())
end

p1 = plot(mdc; pnames=[L"p_1" L"p_2" L"p_3" L"p_4"])

cost_vec = [cost(el) for el in eachcol(trajectory(mdc))]
p2 = plot(distances(mdc), log.(cost_vec),
          ylabel="log(cost)", xlabel="distance", title="cost over MD curvecurve")

mdc_plot = plot(p1, p2, layout=(2, 1), size=(800, 800))

nominal_trajectory   = plot(sol_at_p(mdc(0.0)[:states],
                               label=["prey" "predator"])
perturbed_trajectory  = plot(sol_at_p(mdc(-15.0)[:states],
                               label=["prey" "predator"])

feature_labels = ["mean prey", "max predator"]
for i in 1:2
    hline!(nominal_trajectory,   nom_features,   linestyle=:dash, label=label=feature_labels[i])
    hline!(perturbed_trajectory, features(mdc(-15.0)[:states]), linestylelinestyle=:dash,
           label=feature_labels[i])
end

## Different parameters/trajectories at different points on the curve…but features are preserved, i.e. minimally disrupted!
traj_comparison = plot(nominal_trajectory, perturbed_trajectory,
                       layout=(2, 1), xlabel="time", ylabel="population")
```

### Lessons

- MinimallyDisruptiveCurves.jl showed how we could continuously vary the parameters of the Lotka‑Volterra model while preserving  

  1. mean prey population over time, and  
  2. max predator population over time.  

- One strategy (MDC 1) was to alter the parameters to increase (decrease) birth and death rates of the predators while decreasing (increasing) of the prey. Along the \\(r\\)‑to‑\\(K\\) axis this corresponds to one species moving in one direction while the other moves in reverse.

- Another strategy (MDC 2) was to modulate the frequency of the oscillations. MDC 2 showed us exactly how to change the parameters to continuously modulate this frequency.

{{< box type="info" >}}
**By the way**  

The maximum predator population over time is a **non‑differentiable** function of the solution. How come this worked?

1. ReLU units in neural networks are also technically nondifferentiable a single point, yet they are used successfully in differentiable machine‑machine‑learning models. Same reason!  
2. The maximum of a collection of values is differentiable except at points where two values tie for the maximum. Numerical error and the dynamics of the ODE ensure that exact ties never occur, so the gradient well‑defined in practice.  
{{< /box >}}

