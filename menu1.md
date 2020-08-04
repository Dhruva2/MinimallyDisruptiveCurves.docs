@def title = "How it works"
@def tags = ["syntax", "code"]
# How it works
\toc


## Notation
- Let's say our model has $N$ parameters. 
- The set of allowable parameter values is denoted $\Theta \subseteq \mathbb{R}^N$. For instance, maybe parameter values must be positive, in which case $\Theta$ is the positive orthant.
- Any particular model configuration can be defined by a vector of $N$ parameters: $\theta \in \Theta$.
- We denote the cost(/loss) function as 
$$ C: \Theta \to \mathbb{R}. $$
    $C(\theta)$ tells us 'how badly' the model behaves when using the parameter vector $\theta$. The smaller $C(\theta)$ is, the better the model behaves. 


## Background: model-fitting

Let's quickly recap the problem of model-fitting (optimising the model), and define some notation for subsequent use.
### Graphically
@@backgroundbox
\\ \\
~~~
<div class="row">
  <div class="container">
    <img class="center" src="/assets/loss_schematic.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
*(bracketed blocks are optional. Ideal outputs could be experimental data)*
\\ \\
**But we can abstract away fixed blocks to get our cost function:**
\\ 

~~~
<div class="row">
  <div class="container">
    <img class="center" src="/assets/reduced_loss_schematic.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
@@
\\

### Verbally:
- When we make a model of a system, we want it recreate some desired behaviour(s). 
- To do so, we make a **cost** (also called loss) function, that takes in model behaviour, and spits out how bad it is. Larger loss <=> less desirable model behaviour.
-  Of course, the modeller has to define the measure of bad behaviour. For instance, squared deviation from data the model is supposed to fit (*is used too much*). 

@@infobox
**By the way**
- Most systems one would model in Biology and Engineering (my interests) are input-output systems. You turn the steering wheel, the car veers. You inject current, the neuron fires. You add a cytokine, the cell goes metabolically manic. 
- Therefore reasonable cost functions should penalise bad model behaviour **for multiple inputs**. I want my car to turn left when I steer left. I also want my car to turn right when I steer right. 
- Don't conveniently forget this to make the modelling/analysis easier. Too many biological models do. I'm also guilty. MinimallyDisruptiveCurves.jl allows you easily sum multiple cost functions (one for each input). It evaluates them in a multithreaded way, so that performance doesn't take such a hit.
@@

### Mathematically

**Model fitting** (also known as training, optimisation, calibration, ...) is running:

$$ \theta^* = \arg\min_\theta C(\theta) $$ 
subject to $\theta \in \Theta$. 
\\
- Since $C(\theta)$ is a scalar function (returns a scalar), we can think of it as a landscape, where height is analogous to $C(\theta)$, and lateral co-ordinates are the parameters $\theta$ of the problem. Then model fitting is just finding a path to the deepest point of the landscape! (see below)

![](https://blog.paperspace.com/content/images/2018/06/optimizers7.gif)

*This gif was taken from an Intro to Optimization blog post, [website here](https://blog.paperspace.com/intro-to-optimization-momentum-rmsprop-adam/)*



## What are we trying to do

Recall the goal. Given a fitted model, we want to:

> Find functional relationships between model parameters that best preserve model behaviour.

To do this, our curve generator wants to 
> *get as far away as possible from the initial parameters, while keeping the cost function as low as possible*.

@@infobox
**By the way**
- Optimisation consists of **descending to the lowest point of the cost landscape** (without worrying about your particular trajectory).
-  We, on the other hand, are trying to **flow along the valleys of the landscape**. Like a river. But flowing backwards (since we start at the lowest point). 
@@

Let's express this mathematically. We can define a curve on parameter space as a function:
$$ \gamma: [0, F] \to \Theta. $$
- The input $s \in [0, F]$ to the curve is a scalar number, that specifies 'how far along' the curve we are. So $\gamma(0)$ is the initial point, and $\gamma(F)$ is the final point.

Any point $\gamma(s)$ on the curve has a **derivative**, $\gamma'(s)$. This is the direction in which it is pointing (i.e. the direction of the tangent). Let's fix 
- $\gamma(0) = \theta^*$, the locally optimal parameter vector gained from model fitting.
- $\gamma'(0)$, the initial starting direction of the curve, as a user-defined parameter.

We will denote the set of allowable curves $\gamma$ (i.e. those satisfying the above constraints), as $\Gamma$. 

@@infobox
**By the way**

We are going to iteratively add constraints to the set of allowable curves, and thus shrink $\Gamma$.
@@
\\

Now we want all points on our curve to define low-cost parameter configurations. In other words, we want $C(\gamma(s))$ to be small for any $s \in [0, F]$. How can we distinguish between 'good' and 'bad' curves? How about this:
$$J[\gamma] = \int_{\gamma} C[\gamma(s)] \ ds $$


$J$ is the **line integral** of the cost $C$ along the curve $\gamma$. If $J[\gamma]$ is small, then $C$ is necessarily small along the entire curve. 

@@infobox
**By the way**

Vector calculus 101: we can expand out the line integral as
$$ \int_{\gamma} C[\gamma(s)] \ ds  = \int_0^F C[\gamma(s)] \| \gamma(s) \|_2 \ ds $$
@@
\\ 
So now the **best** curve would be the solution of
$$ \min_{\gamma \in \Gamma} J[\gamma]. $$


@@unobtrusivebox
**But wait!** 

*What's preventing the curve from just buzzing around $\gamma(0) = \theta^*$. Since we know that $C(\theta^*)$ is as low as can be?*

**Absolutely nothing!**
@@
\\
So let's add a constraint that the direction of the curve must always be pointing **away** from $\theta^*$. Mathematically:

$$ \frac{d}{ds} \| \gamma(s) - \theta^* \|_2 > 0, \qquad \forall s \in [0, F]. $$

<!-- $$ \langle \gamma'(s), \gamma(s) - \theta^* \rangle > 0, \quad \quad \forall s \in [0, F]. $$ -->

<!-- Here, $\langle ., . \rangle$  is my notation for the dot product (i.e. correlation), and $\gamma(s) - \theta^*$ is the straight line connecting $\theta^*$ to the current point $\gamma(s)$.  -->
\\
@@unobtrusivebox
**But wait again!**

*What if the curve 'doesn't want to evolve' because every direction increases the cost. So it just...stays short.*

**Absolutely nothing again!**
@@
\\
We want a curve that actually changes the parameters appreciably. So let's constrain the length of the curve, by setting how fast it moves as a function of $s$:
$$ \| \gamma'(s) \|_2 = 1 \qquad \forall s \in [0, F].$$
Now the curve **has** to have length $F$.

So to wrap up, our allowable curves are:

$$ \Gamma = \Big\{\gamma:[0,F] \to \Theta:  \\ \frac{d}{ds} \| \gamma(s) - \theta^* \|_2 > 0; \\ \| \gamma'(s) \|_2 = 1;    
\\ \gamma(0) = \theta^*;
\\ \gamma'(0) = \delta \theta^*
\Big\}, $$
for some fixed initial direction $\delta \theta^* \in \mathbb{R}^N$. Our mathematical problem is 
$$ \min_{\gamma \in \Gamma} J[\gamma], \text{where} \\  J[\gamma] = \int_{\gamma} C[\gamma(s)] \ ds. \label{eq:mainProb}$$


## How we do it

If equation \eqref{eq:mainProb} reminds you of a variational calculus problem, you would be right! Now we just have to solve the Euler-Lagrange equations, right?

@@unobtrusivebox
**Wrong. It's too hard!** Try it yourself if you want. I'm not going to explain why ... lots of potential curves, not so many constraints.
@@
\\
So instead of searching directly over curves, we are going to turn this into a dynamic problem. Stop thinking about curves as static shapes. Start thinking about curves as things that are drawn over time.
<!-- - We previously parameterised curves as $\gamma: [0,F] \to \Theta$. An input $s \in [0,F]$ quantified 'how far along the curve' we are.
-  Instead of thinking of $s$ as a distance-like variable, let's now think of $s$ as a timelike variable.
- Our problem is to draw (over time) a curve. We need some rules for how the curve evolves in time, so that the final curve satisfies equation \eqref{eq:mainProb}. -->

**This turns it into an optimal control problem:**

1. Imagine a rocket sitting at $\theta^*$
2. It needs to traverse a trajectory in parameter space
3. At each point in time, we send it a control input, that determines its current direction (i.e. we steer the rocket).
4. **Problem**: Design a steering law so that the trajectory traced by the rocket is a (locally) optimal solution of equation \eqref{eq:mainProb}.

@@infobox
**By the way**
- The rocket analogy is there for a reason. Optimal control theory got going in the 1950s, when the USSR and the USA suddenly got interested in designing aircraft/rocket trajectories that were optimal for doing various military things. No prizes for guessing why.
- [Here's a history](https://www.math.uni-bielefeld.de/documenta/vol-ismp/48_pesch-hans-josef-cold-war.pdf)
- Two solutions emerged, separated by the iron curtain. Pontryagin's maximum principle, and Bellman's principle of optimality. 
- Using one of these methods, we shall seize the means of curve generation, Comrade!
@@

Let's mathematically express equation \eqref{eq:mainProb} as an optimal control problem:

- At each time $t$, we choose an input $u(t)$. The entire trajectory, from $t = 0$ to $t=F$, we denote $u$. The space of possible trajectories is $\mathcal{U}$. So $u \in \mathcal{U}$.
- The dynamics of the 'rocket' are $\theta'(t) = u(t)$ (i.e. we directly control rocket velocity).
- By transcribing the constraints on $\gamma'(s)$ in the previous section to constraints on $u(s)$, we define the space of possible input trajectories as
  $$ \mathcal{U} = \Bigg\{u: \|u(t)\|_2 = 1; \langle u(t), \gamma(t) - \theta^* \rangle \geq 0; \\ u(0) = \delta \theta^*; \quad \forall t \in [0, F] 
  \Bigg\}$$
- Our minimisation problem \eqref{eq:mainProb} is now
  $$ \min_{u \in \mathcal{U}} J(u), \text{where} \\
  J(u) = \int_\gamma C \ d \gamma, \text{ and } \gamma'(t) = u(t), \ \forall s \in [0,F].
  $$

- You can then apply Pontryagin's maximum principle to get some complicated constraints on $u(s)$. 
- You can work through these constraints (see P135 of [my PhD thesis](https://ora.ox.ac.uk/objects/uuid:f58aa335-db0a-495b-8eef-1ddb363cbd19/download_file?file_format=pdf&safe_filename=masterDoc.pdf&type_of_work=Thesis)) and translate them into a differential equation.

@@infobox
**By the way**
- Usually, solving the optimal control problem using Pontryagin's maximum principle requires solution of a boundary value problem. These are annoying to solve.
- Luckily, by tinkering with constraints (see PhD thesis reference), we factor out the boundary constraints, and end up with an **explicit ordinary differential equation that defines optimal curve evolution**.
@@
\\ \\
@@backgroundbox
**Now, we are going to write down the differential equations for curve evolution**
- They are complicated and you should feel free to skip them. Intuition will be provided subsequently.
@@
... no one more thing.

@@unobtrusivebox
Pontryagin's maximum principle requires dynamics for not only the **state** of the system (i.e. the position $\gamma(s)$ of the rocket) but also the **costate** of the system. What is this? Something roughly analogous to the momentum of the rocket, but also affected by all the input constraints we have set. 

So we get two curves. One for the state $\gamma(s)$, and the other for the costate, which we shall denote $\lambda(s)$.
@@

So now, the coupled dynamics for state and costate. Angle brackets denote inner (dot) products.
1. Let $H := C[\gamma(F)]$ be the final cost. We set this *a priori* as a hyperparameter. Curve evolution stops prematurely if $C[\gamma(s)] \geq H$.

@@infobox
**By the way**
- Pre-setting $H$ is the trick that turns this from a boundary value problem into a (much easier) ODE problem.
@@
\\
2. Let $\mu_2(s) = \frac{C[\gamma(s)] - H}{2}$.
3. Define $\mu_1(s) \geq 0$, and impose the constraint $\mu_1(s)\langle u(s), \gamma(s) - \theta^* \rangle = 0$.
4. Let $\lambda(0) = \big(H - C[\gamma(0)]\big)u(0)$

Then
$$ \lambda'(s) = \mu_1(s) u(s) - \nabla_{\theta} C[\gamma(s)] $$
$$ \gamma'(s) = u(s) $$
$$ u(s) = \frac{\lambda(s) - \mu_1(s)[\gamma(s) - \theta^*]}{2 \mu_2(s)}.$$

Given $C[\gamma(s)]$ (the current cost), $\nabla_{\theta} C[\gamma(s)]$ (and it's gradient), we can solve this as a differential equation in the variables $\gamma(s)$ and $\lambda(s)$.

@@infobox
**By the way**
- Get some intuition by setting $\mu_1(s) = 0$ in your head, then reviewing the equations. This occurs when the curve is currently pointing away from $\theta^*$.
- Then you see that $u(s)$, the curve direction, is proportional to $\lambda(s)$, the costate ($\approx$ momentum).
- Meanwhile, the costate is **integrating** the cost gradient, $\nabla_{\theta} C[\gamma(s)]$
- Notice the analogy with optimisation algorithms based on gradient descent with momentum, if you're into that kind of thing.
@@


## Why it works

