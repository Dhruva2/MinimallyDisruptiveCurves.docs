@def title = "How it works"
@def hascode = true
@def date = Date(2019, 3, 22)
@def rss = "A short description of the page which would serve as **blurb** in a `RSS` feed; you can use basic markdown here but the whole description string must be a single line (not a multiline string). Like this one for instance. Keep in mind that styling is minimal in RSS so for instance don't expect maths or fancy styling to work; images should be ok though: ![](https://upload.wikimedia.org/wikipedia/en/3/32/Rick_and_Morty_opening_credits.jpeg)"

@def tags = ["syntax", "code"]
# How it works
\toc

## Background: model-fitting

Think about the problem of tuning a model parameters to fit data/recreate a desired behaviour. The flow diagram might look something like this:
\\ \\
~~~
<div class="row">
  <div class="container">
    <img class="center" src="/assets/loss_schematic.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
\\ \\
- A fixed model structure has variable parameters. The behaviour of the model changes with the parameters. Some behaviours are more desirable than others. We quantify this through a loss function, which reduces model behaviour to a scalar number. 
- For instance if we are fitting a model to data, the loss function might measure the discrepancy between model predictions and data. The lower the value of the loss, the more desirable the model behaviour.
- So we can factor out the model, and just consider a function mapping parameter values to 'loss' (ie an inverse measure of model desirability).
\\ \\
~~~
<div class="row">
  <div class="container">
    <img class="right" src="/assets/reduced_loss_schematic.svg">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
\\ \\



- **Model fitting** (also known as training, optimisation, calibration, ...) consists of minimising the loss as a function of the parameters. So, finding the set of parameters that results in the 'most desirable' model behaviour. 
- We can consider the loss function as a landscape, where height represents the value of the loss, and the x-y co-ordinates represent parameter values. Model fitting then consists of getting to the deepest point of the landscape (see schematic below).

~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/Optimizers7.gif">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
*This gif was taken from an Intro to Optimization blog post, [website here](https://blog.paperspace.com/intro-to-optimization-momentum-rmsprop-adam/)*



~~~
<a id="optimization_skip">
</a>
~~~

## Including scripts

Another approach is to include the content of a script that has already been executed.
This can be an alternative to the description above if you'd like to only run the code once because it's particularly slow or because it's not Julia code.
For this you can use the `\input` command specifying which language it should be tagged as:


\input{julia}{/_assets/scripts/script1.jl} <!--_-->


these scripts can be run in such a way that their output is also saved to file, see `scripts/generate_results.jl` for instance, and you can then also input the results:

\output{/_assets/scripts/script1.jl} <!--_-->

which is convenient if you're presenting code.

**Note**: paths specification matters, see [the docs](https://tlienart.github.io/franklindocs/code/index.html#more_on_paths) for details.

Using this approach with the `generate_results.jl` file also makes sure that all the code on your website works and that all results match the code which makes maintenance easier.
