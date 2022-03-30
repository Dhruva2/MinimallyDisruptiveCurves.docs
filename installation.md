@def title = "Installation"
@def hascode = true
@def date = Date(2019, 3, 22)
@def rss = "How to do a Julia installation of MinimallyDisruptiveCurves.jl"

@def tags = ["syntax", "code"]

# Installation
\toc
\\ 
~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/python_environment.png">
    <div style="clear: both"></div>      
  </div>
</div>
~~~
  (*Credit: https://xkcd.com/1987/*)

\\ 
### Dependencies
- Julia v1.6 or newer
\\ 

### Steps
1. Open a julia session (REPL) 
   
2. If you want, make a new environment (see [package management documentation](https://docs.julialang.org/en/v1/stdlib/Pkg/index.html) for more details ), with:
```julia
] activate .
```
Here, `]` switches from the julia REPL to the package manager
3. Add MinimallyDisruptiveCurves.jl to your current environment with
```julia
] add MinimallyDisruptiveCurves
```

4. Finally preface any julia code running in your environment, with:
```julia
using MinimallyDisruptiveCurves
```

### Downloading tutorial notebooks

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

5. Open Pluto notebooks:
```julia
using Pluto; Pluto.run()
```
and then open/run the individual notebooks in the notebook environment. 

 



