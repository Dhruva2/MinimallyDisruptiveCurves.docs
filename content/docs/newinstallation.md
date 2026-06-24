---
title: "Installation"
date: 2019-03-22
rss: "How to do a Julia installation of MinimallyDisruptiveCurves.jl"
weight: 2
tags:
  - syntax
  - code
hascode: true
---

# Installation

{{< figure src="/images/python_environment.png" >}}
(*Credit: https://xkcd.com/1987/*)

### Dependencies
- Julia v1.10 or newer

### Steps
1. Open a julia session (REPL) 
2. If you want, make a new environment (see [package management documentation](https://docs.julialang.org/en/v1/stdlib/Pkg/index.html) for more details), with:
   ```julia
   ] activate .
   ```
   Here, `]` switches from the julia REPL to the package manager
3. Add *MinimallyDisruptiveCurves.jl* to your current environment with
   ```julia
   ] add MinimallyDisruptiveCurves
   ```
4. Finally preface any julia code running in your environment, with:
   ```julia
   using MinimallyDisruptiveCurves
   ```


