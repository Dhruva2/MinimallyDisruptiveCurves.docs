@def title = "MinimallyDisruptiveCurves.jl: Documentation"
@def tags = ["syntax", "code"]

# MinimallyDisruptiveCurves.jl

\tableofcontents <!-- you can use \toc as well -->
\\
\\
~~~
<p style="color:black;font-size:20px;"> Because building a good model is hard</p>  
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
  (*Credit: https://xkcd.com/793/ and https://xkcd.com/2048/*)
\\
## Use Cases  
<!-- **The Context** -->

~~~
<p style="color:black;font-size:18px;"> You have a mathematical model (of any flavour), and you want to answer questions like: </p> 
~~~
<!-- **Next Questions**: -->
- Are some model interactions unnecessary? Which ones?
- Is there a hidden approximation you could make in the model that wouldn't greatly change behaviour? 
- Which model parameters can never be determined by model-fitting to data? Can we instead find some identifiable function of these parameters? 
- (*Dually*) What spaces of parameters are (approximately/exactly) consistent with the data?
- Could changes in parameter $x$ could be continuously compensated for by changes in parameters $y$ and $z$? Or by $u$, $v$, and $w$? If so, what form of compensation?
\\

@@infobox
**By the way**
 - Successfully fitting your model to data does **not** imply that model parameters can be uniquely determined from data. It might not even bound their allowable values. 
@@

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
Each functional relationship is termed a 'minimally disruptive curve'. It's basically a path in parameter space over which model behaviour changes minimally. For a more complete characterisation, see table below:

\\

|          Source                                                                     | Rigorous enough | Excessively rigorous | Excessively verbose |
| ----------------------------------------------------------------------------------- | --------------- | -------------------- | ------------------- |
| [My thesis, ch. 5 (ch. 4 is relevant too)](https://ora.ox.ac.uk/objects/uuid:f58aa335-db0a-495b-8eef-1ddb363cbd19/download_file?file_format=pdf&safe_filename=masterDoc.pdf&type_of_work=Thesis) |                 | x                    | x                   |
| \citep{Raman17}                                                                     | x               |                      | x                   |
| [How it works](/menu1/)                                                             | x               |                      |                     |     


\\

#### Ingredients

- MinimallyDisruptiveCurves.jl doesn't care about / interact with the specific model you are using. It is 'model agnostic'. 
- You only need a differentiable cost function (also called loss/objective), which maps model parameters to 'how bad' the model behaviour is. Pseudocode for required cost function methods below.
- The optimal behaviour could be matching data, oscillating at 3Hz, or doing backflips. Your choice.

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

- You need a locally optimal set of parameters for that cost function. (You get these from 'fitting' the model). You can get these by optimising your cost function (model fitting), using the julia optimisation packages mentioned below, for example.
\\ \\
@@infobox
**By the way**
- Several julia optimisation packages (such as Optim.jl and NLOpt.jl, DiffEqParamEstim.jl) use the same specification for their cost functions (which they call objective functions). So you can exploit their existing APIs for building cost functions. This package contains additional helper functions and examples for building and transforming cost functions.
- Otherwise, Zygote.jl and ForwardDiff.jl can collectively take the gradient of pretty much any differentiable Julia code between them. 
- If all else fails, MinimallyDisruptiveCurves.jl provides a function: `make_fd_differentiable(cost)`, that spits out a function of the above form, with a finite-difference step to calculate the gradient. But using finite diffence differentiation with Julia is like using a Ferrari to do the school run. 
@@  
\\ \\

#### Workflow

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

@@infobox
**Computational complexity**
- Generating a minimally disruptive curve usually requires somewhere between a few hundred, and a few thousand, evaluations of your cost function (and its' gradient). That's the major computational cost. 
- For differential equation-based models, 
@@


## Features



## Place in the literature ecosystem

Structural identifiability, practical unidentifiability
Profile likelihood without gradients. Mbam. 
<!-- http://pengqiu.gatech.edu/software/model_manifold/html/publish_MBAM_example.html -->


## Citing

**The algorithm**: (minus a few tweaks)
```
@article{raman2017delineating,
  title={Delineating parameter unidentifiabilities in complex models},
  author={Raman, Dhruva V and Anderson, James and Papachristodoulou, Antonis},
  journal={Physical Review E},
  volume={95},
  number={3},
  pages={032314},
  year={2017},
  publisher={APS}
}
```

**The software**: as yet unpublished, keep your eye on this!


## References 
* \biblabel{Raman17}{Raman et al. (2017)} **Raman**,  **Anderson**  and **Papachristodoulou**, [Delineating parameter unidentifiabilities in complex models.](https://journals.aps.org/pre/abstract/10.1103/PhysRevE.95.032314), Physical Review E 95.3 (2017).





