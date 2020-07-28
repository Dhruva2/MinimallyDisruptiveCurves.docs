@def title = "How it works"
@def tags = ["syntax", "code"]
# How it works
\toc




## Background: model-fitting

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
**But we can abstract away fixed blocks to get this:**
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
- Often, the overall cost function is a sum of sub cost functions. EG we want the model to do $n$ different things for $n$ different inputs. Each ideal input-output behaviour specifies a cost function. 

### Mathematically:

- **Vector of model parameters:** $\theta$.
- **Space of allowable parameters:** $\Theta$  ($\ni \theta$). 
- **Cost function**: $C(\theta)$. *Remember that lower cost <=> better fitting parameter vector.*
\\ 
**Model fitting** (also known as training, optimisation, calibration, ...) is then running:
\\
@@centremaths
$\theta^*$ = Minimise $C(\theta)$ 

subject to $\theta \in \Theta$.  
@@
\\
- Since $C(\theta)$ is a scalar function (returns a scalar), we can think of it as a landscape, where height is analogous to $C(\theta)$, and lateral co-ordinates are the parameters of the problem. Then model fitting is just finding a path to the deepest point of the landscape! (see below)

![](https://blog.paperspace.com/content/images/2018/06/optimizers7.gif)

*This gif was taken from an Intro to Optimization blog post, [website here](https://blog.paperspace.com/intro-to-optimization-momentum-rmsprop-adam/)*



## What are we trying to do

Recall the goal. Given a fitted model, we want to:

> Find functional relationships between model parameters that best preserve model behaviour.


How do we start? First let's gather up what we obtained from the model fitting step:

1. A **cost function** $C(\theta)$ that maps plausible vectors of model parameter values ($\theta$) to cost (how badly the model performs).
   
2. A **locally optimal** set of parameters $\theta^*$. Basic vector calculus gives: $\nabla C(\theta^*) = 0$. *Translation: the slope of a landscape at the deepest point is zero*
  

> **How do we restate our goal in terms of the cost function??**