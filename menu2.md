@def title = "Examples"
@def hascode = true


<!-- @def tags = ["syntax", "code", "image"] -->

# Examples

\toc

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

## Suggested order to go through notebooks
Here is a good order to run through the notebooks:

1. `Transforming_cost_functions.ipynb`
2. `MassSpringExample.ipynb`
3. `NFKB_example.ipynb`
4. `stg_neuron_prelim_collocation.ipynb`
5. `CircadianOscillator.ipynb`

- No. 1 just shows you how to play around with cost functions
- No. 2 is an exanple on an extremely simple model. Running curves takes fractions of a second here, so it's a good playground for playing with curve settings.
- No. 3 is fairly detailed. It's useful if you want to see how scientific insight can be gained from looking at Minimally Disruptive Curves. Each curve on this notebook takes 2-10 mins to generate on my laptop (2017 macbook pro).
- Nos 4. and 5. show you how to use the (much quicker to evaluate) collocation cost. I've generated, but haven't really had time to properly analyse, the minimally disruptive curves.

## A ready-made, minimal example

Will arrive shortly. For now, go through the ones on the repository above!
