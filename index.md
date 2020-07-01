@def title = "MinimallyDisruptiveCurves.jl: Documentation"
@def tags = ["syntax", "code"]

# MinimallyDisruptiveCurves.jl

\tableofcontents <!-- you can use \toc as well -->
\\
\\
~~~
<p style="color:black;font-size:22px;"> Building a good model is hard</p>  
~~~

~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/physicists.png">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
\\ \\
~~~
<p style="color:black;font-size:28px;">But extracting useful insight from a model is harder</p>  
~~~
~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/curve_fitting.png">
    <div style="clear: both"></div>      
  </div>
</div>
~~~

\\
## Use Cases  
<!-- **The Context** -->

~~~
<p style="color:black;font-size:18px;"> You have a mathematical model (of any flavour), and you want to answer questions like: </p> 
~~~
<!-- **Next Questions**: -->
- Are some model interactions unnecessary? Which ones?
- What spaces of parameters are (approximately/exactly) consistent with the data?
- Could changes in parameter $x$ could be compensated for by changes in parameters $y$ and $z$? Or by $u$, $v$, and $w$? If so, what form of compensation?
- Is there a hidden approximation you could make in the model that wouldn't greatly change behaviour? 
\\
<!-- ~~~
<p style="color:black;font-size:18px;">MinimallyDisruptiveCurves.jl can help!</p> 
~~~

<!-- &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -->

<!-- <p style = "text-align:right">  ...(but nothing is a substitute for expert domain knowledge) </p> -->

## What it does (short form)

At its' core, only one thing: 

> Finds functional relationships between model parameters that best preserve model behaviour.

~~~
<p style = "text-align:right">  (...by solving a differential equation on the parameters) </p>
~~~
\\
Each functional relationship is termed a 'minimally disruptive curve'. It's basically a path in parameter space over which model behaviour changes minimally. A more rigorous definition is provided in 

- You need a cost function, which maps model parameters to 'how bad' the model behaviour is (whether the optimal behaviour is matching data, oscillating at 3Hz, or doing backflips).
- You need a locally optimal set of parameters for that cost function. (You get these from 'fitting' the model).
Then the workflow might look something like this 
~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/intro_workflow_schematic.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
- At each iteration, you choose which parameters the minimally disruptive curve can change (you can choose all of them!). 

A minimally disruptive curve might look something like this (in the easily-visualisable case of a model with three parameters):

~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/mdc_traj_schematic.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~

- Every point on the curve corresponds to a set of model parameters. 
- The starting point (which you provide, red dot) is a locally optimal set of parameters. That is, one that best matches 'optimal' model behaviour, where 'optimal' is defined in terms of your cost function. 
- The minimally disruptive curve generator tries to change the parameters *as much as possible* while best preserving model behaviour (i.e. keeping the cost low/minimal). So any parameter set on the curve (e.g. green dot) should induce model behaviour that is similar/identical to the red dot. 
-  *As much as possible* requires a notion of distance on parameter space (i.e. a metric). You can play with this metric. It could be quantified as relative changes in parameter values. And/or you could bias the curve so that small changes in a particular parameter (let's call it p7) correspond to large changes in the metric. So the curve will try to align with p7, and tell you how other parameters in the model can compensate for changes in p7, to preserve model behaviour.


## Workflow on toy example
#### Preamble


1. You've built a model. It depends on tunable parameters. You want the model to do **X**. 
   \\ \\ 
   $ \quad \quad \ \ \ \ \ \ \ \ \ X = \text{(matching data/oscillating at 3Hz/doing backflips/all of the above). } $ 
\\

2. So you build a **cost function**, that maps parameters, to "*how badly does the model do X*". A lower cost is better. The schematic might look like this:

~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/loss_schematic.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
(*Bracketed blocks are optional, but common*)
\\ \\

3. **Model fitting**  is the process of *finding the parameters that (locally/) minimise the cost function*. If we wrap up all parameters in a vector $\theta$, this amounts to
$$ \theta^* = \arg\min_{\theta} C(\theta).$$

Your model works! Now come to MinimallyDisruptiveCurves.jl. If you can, bring the following:
  - ☑ The cost function $C(\theta)$
  - ☑ Ideally, a gradient of the cost: $\nabla C(\theta)$ (i.e.  $\frac{d}{d\theta} C(\theta)$ ).
  - ☑ A locally optimal parameter vector, $\theta^*$.

\\

**If not, we provide examples, helper functions, and references** for creating cost functions and their gradients. These will focus on differential equation-based models. 

<!-- - It might be easier than you think to take the gradient of a cost function of [insert complicated differential-equation model here]. There are some extremely good tools for automatic differentiation of arbitrary code in Julia. -->
\\ \\ 
#### Basic workflow

1. From the preamble, you have a **differentiable cost function**, and a **locally optimal parameter vector**. The cost function should have two methods:

```julia
function cost(params)
  ... 
  return cost
end

function cost(params, gradient_holder)
  ...
  gradient_holder[:] = ∇C(params) # MUTATE gradient holder
  return cost
end
```
- A cost function of the above form is compatible with several julia optimisation packages (such as Optim.jl and NLOpt.jl) that you can use for the model fitting step. So you can switch straight from optimising a cost function (i.e. model-fitting) to generating MD curves based off of it. Moreover, you can use these packages to build your cost function for use here.

- Otherwise, Zygote.jl and ForwardDiff.jl can take the gradient of pretty much any differentiable Julia code between them. Other methods can be faster for models based on differential equations. If all else fails, MinimallyDisruptiveCurves.jl provides a function: make_fd_differentiable(cost), that spits out a function of the above form, with a finite-difference step to calculate the gradient.

Meanwhile, let us call the locally optimal parameter vector $\theta^*$. By local optimality, $\nabla C(\theta^*) = 0$. 
\\ \\

2. You generate a 


1. For your cost function 

#### Workflow on toy example

1. **Let's first figure out what this means**
2.  **Then we will go on to see how it can help answer model related questions**

\\ 

<!-- Once you've built a model, you tune the model parameters to recreate a desired behaviour / match data. This is called **model-fitting**. Mathematically, it looks something like this:
\\ \\
\begin{align}
 &\text{Minimise } & & C(p) \\
 &\text{subject to} 
 & & g_1(p) = 0; \quad g_2(p) \leq 0
 <!-- \end{align}
 \\ -->

 <!-- - $p$ is a vector containing all the parameters of the model.
- $C(p) \geq 0$ is some cost function that maps your parameters $p$ to **how badly** the model performs. Smaller is better. Zero is best.
- $g_1(p)$, $g_2(p)$ represent constraints on your parameters (if any). Maybe $p_1$ represents the mass of an object, in which case $p_1 \geq 0$ is a constraint.  --> -->



Once you done this, you have some 'optimal' parameter vector $p_0$, that best fits your data. Now



<!-- So we've recalled how model-fitting can be conceptualised as descending to the bottom of a 'loss landscape'. Imagine you've got to the bottom, and on your journey have acquired a healthy dose of altitude sickness. There, you encounter a strange man filled with murderous intent. Where would you run to get away from him? That's where the minimally disruptive curve will go. -->

 



## How it's useful


## Getting started

or code-blocks `inline` or with highlighting (note the `@def hascode = true` in the source to allow [highlight.js](https://highlightjs.org/) to do its job):

```julia
abstract type Point end
struct PointR2{T<:Real} <: Point
    x::T
    y::T
end
struct PointR3{T<:Real} <: Point
    x::T
    y::T
    z::T
end
function len(p::T) where T<:Point
  sqrt(sum(getfield(p, η)^2 for η ∈ fieldnames(T)))
end
```

You can also quote stuff

> You must have chaos within you to ...

or have tables:

| English         | Mandarin   |
| --------------- | ---------- |
| winnie the pooh | 维尼熊      |

Note that you may have to do a bit of CSS-styling to get these elements to look the way you want them (the same holds for the whole page in fact).

### Symbols and html entities

If you want a dollar sign you have to escape it like so: \$, you can also use html entities like so: &rarr; or &pi; or, if you're using Juno for instance, you can use `\pi[TAB]` to insert the symbol as is: π (it will be converted to a html entity).[^1]

If you want to show a backslash, just use it like so: \ ; if you want to force a line break, use a ` \\ ` like \\ so (this is on a new line).[^blah]

If you want to show a backtick, escape it like so: \` and if you want to show a tick in inline code use double backticks like ``so ` ...``.

Footnotes are nice too:

[^1]: this is the text for the first footnote, you can style all this looking at `.fndef` elements; note that the whole footnote definition is _expected to be on the same line_.
[^blah]: and this is a longer footnote with some blah from veggie ipsum: turnip greens yarrow ricebean rutabaga endive cauliflower sea lettuce kohlrabi amaranth water spinach avocado daikon napa cabbage asparagus winter purslane kale. Celery potato scallion desert raisin horseradish spinach carrot soko.

## Basic Franklin extensions

### Divs

It is sometimes useful to have a short way to make a part of the page belong to a div so that it can be styled separately.
You can do this easily with Franklin by using `@@divname ... @@`.
For instance, you could want a blue background behind some text.

@@colbox-blue
Here we go! (this is styled in the css sheet with name "colbox-blue").
@@

Since it's just a `<div>` block, you can put this construction wherever you like and locally style your text.


\newcommand{\E}[1]{\mathbb E\left[#1\right]}

Now we can write something like

$$  \varphi(\E{X}) \le \E{\varphi(X)}. \label{equation blah} $$

since we've given it the label `\label{equation blah}`, we can refer it like so: \eqref{equation blah} which can be convenient for pages that are math-heavy.

In a similar vein you can cite references that would be at the bottom of the page: \citep{noether15, bezanson17}.

**Note**: the LaTeX commands you define can also incorporate standard markdown (though not in a math environment) so for instance let's define a silly `\bolditalic` command.

\newcommand{\bolditalic}[1]{_**!#1**_} <!--_ ignore this comment, it helps atom to not get confused by the trailing underscore when highlighting the code but is not necessary.-->

and use it \bolditalic{here for example}.

Here's another quick one, a command to change the color:

\newcommand{\col}[2]{~~~<span style="color:#1">#2</span>~~~}

This is \col{blue}{in blue} or \col{#bf37bc}{in #bf37bc}.

### A quick note on whitespaces

For most commands you will use `#k` to refer to the $k$-th argument as in LaTeX.
In order to reduce headaches, this forcibly introduces a whitespace on the left of whatever is inserted which, usually, changes nothing visible (e.g. in a math settings).
However there _may be_ situations where you do not want this to happen and you know that the insertion will not clash with anything else.
In that case, you should simply use `!#k` which will not introduce that whitespace.
It's probably easier to see this in action:

\newcommand{\pathwith}[1]{`/usr/local/bin/#1`}
\newcommand{\pathwithout}[1]{`/usr/local/bin/!#1`}

* with: \pathwith{script.jl}, there's a whitespace you don't want 🚫
* without: \pathwithout{script.jl} here there isn't ✅




* [Installation](/menu1/)
* [menu 2](/menu2/)
* [menu 3](/menu3/)

## References 

* \biblabel{noether15}{Noether (1915)} **Noether**,  Körper und Systeme rationaler Funktionen, 1915.
* \biblabel{bezanson17}{Bezanson et al. (2017)} **Bezanson**, **Edelman**, **Karpinski** and **Shah**, [Julia: a fresh approach to numerical computing](https://julialang.org/research/julia-fresh-approach-BEKS.pdf), SIAM review 2017.


