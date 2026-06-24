---
title: "Q&A with a sceptic"
weight: 7
---



### This is Marna

{{< figure src="/images/rndimg.jpg" >}}

- We met online, as is common these days. She was hanging out on the website template from which I built this user guide.  
- Marna is a gentleman and a scholar. She is also a **mar**ine igua**na**. *Amblyrhynchus cristatus*. Do you know what *Ambly* means? Blunt.  
- Her native disposition was further blunted by a traumatic childhood spent [fleeing snakes](https://www.youtube.com/watch?v=Rv9hn4IGofM).  
- As such, she has acquired a healthy dislike for all things curve‑shaped. Especially those that claim to be minimally disruptive.


*🦎: This doesn't make sense. The gradient of a function \\(C: \mathbb{R}^n \to \mathbb{R}\\) gives the direction of maximal **sensitivity**. It doesn't say much about the remaining \\(n-1\\) directions, or how **insensitive** they are. But you're trying to use gradient information alone to evolve curves in a maximally **insensitive** direction of the cost function*  

{{< box type="section" >}}
You're correct. Usually the Hessian (second derivative) of a function gives the directions of local **insensitivity**. By Taylor expansion:

$$
 C[\theta + \delta \theta ] = C[\theta] + \langle \delta \theta, \nabla_{\theta} C \rangle + \frac{1}{2} \langle \delta \theta, \nabla^2_{\theta} C[\theta] \delta \theta \rangle + \mathcal{O}(\|\delta\theta\|_2^3).
$$

As you say, there are \\(n-1\\) potential directions for \\(\delta \theta\\) that are uncorrelated with the gradient \\(\nabla_{\theta} C[\theta]\\). So the gradient doesn't give information on their sensitivity, while the Hessian does (we can see if \\(\langle \delta \theta, \nabla^2_{\theta} C[\theta] \delta \theta \rangle\\) is small).

However, Hessians are generally very costly to compute, so we wanted a way of evolving insensitive directions using the gradient alone.

We can think again of the minimally disruptive curve as a ball rolling along a valley in the loss landscape (where height is cost, and lateral co‑ordinates are parameter values). Once given an initial push, the ball just maintains constant speed in a flat (insensitive) direction. It only changes its course when the valley curves, and it starts to creep up the valley side. What makes it turn? Gravity pushes it back down in the direction of the valley slope (the gradient). So the trajectory curves along with the valley.  
{{< /box >}}


*🦎: OK but then numerical error will prevent this algorithm from working.  If you're near a local minimum of the cost, then the gradient \\(\nabla_{\theta} C \approx 0\\). Which means that small numerical errors will massively bias the direction of the gradient.*  

{{< box type="section" >}}
Your intuition is good! But we bypass this problem somewhat. If you want to see this in action, look at our simplest mass‑spring model example:

Evaluate the gradient at the minimum and you'll notice that it isn’t zero as it should be. The entries are \\(\approx 1e^{-5}\\). Nevertheless the MD curve (which we know *a priori* in that problem), is extremely accurate. How come?

Roughly speaking, we could divide numerical errors into systematic, and unsystematic errors.  

**Unsystematic errors**  
We can think again of the minimally disruptive curve as a ball rolling along a valley in the loss landscape (where height is cost, and lateral co‑ordinates are parameter values). Our ball has weight, and thus some momentum. This is the costate variable \\(\lambda(t)\\) described in the [How it works](/menu1) section. We can think of the ball being buffeted by the wind (unsystematic numerical errors). The momentum of the ball prevents this buffeting from affecting the trajectory too much **unless** the direction is systematic….

**Systematic errors**  
Every so often, the minimally disruptive curve generator resets the momentum (costate) variable. This factors out numerical error that has integrated over time. You can set the numerical tolerance at which this reset happens, or switch it off entirely (by setting `callback=nothing`, not recommended).

```julia
MDCSolve(sys::MDCProblem; callback=mdc_momentum_readjustment(sys, tol=1e-3))
```

{{< /box >}}
  

*🦎: Hmm, I see. But do we have to use a gradient at all? Why not just wiggle parameters, see which wiggles don't change model behaviour, and so on?*  

{{< box type="section" >}}
The profile likelihood method introduced in [Venzon et al, 1988](https://www.jstor.org/stable/pdf/2347496.pdf?casa_token=FWVhlSa5GRcAAAAA:pPzSK4NeRprge0-dzlZp7ae5bCIiEEnJRn2L8Qq_s6ND57YmippIDaKDg8Pxq64QH_7_DW0h4bonSbnaLm4dFhsEuPaefVJoYCv1gNUhs_qhgmObZQ), and applied, with a software toolbox, for Systems Biology in [Raue et al, 2009](https://academic.oup.com/bioinformatics/article/25/15/1923/213246) does precisely this! And doesn't require a differentiable cost function.

Essentially they pick a parameter (\\(j\\)), and iteratively change it. At each step, all the other parameters are re‑optimised to minimise the cost function. This solves a slightly different problem: making effective confidence intervals on the parameters for a particular model behaviour. I informally tried to compare this method to mine on the examples of the 2017 paper, and couldn't use it to extract functional relationships between parameters. The reason? MD Curves have momentum in a particular (potentially curving) insensitive direction. The iterative re‑optimisation procedure can and will change parameters in a different insensitive direction at each iteration.  
{{< /box >}}
  

*🦎: All these numerical methods though… what about numerical error?*  

{{< box type="section" >}}
The differential equation that MinimallyDisruptiveCurves.jl attempts to solve can be ill‑conditioned. I've spent some effort alleviating this, but I haven't fully explored methods of making it as numerically stable as possible. Or even characterising the numerical stability of the solution. This would be a good research topic. In the [Tips & Tricks](/menu3/) section, I give some guidelines.

Fortunately I think it is acceptable to be much more tolerant than usual of numerical error in this type of problem. Why?

The success of an MD curve is not defined by whether the numerical solution of the ODE defining curve evolution was accurate. It is defined by whether you generated a curve that doesn't disrupt model behaviour much. If this toolbox gives you that…great. This is different from usual use cases of ODE solvers, where you want to approximate as closely as possible an unknown ‘ground truth’ dynamical system.  
{{< /box >}}
  

* Venzon, D. J., and S. H. Moolgavkar. "[A method for computing profile‑likelihood‑based confidence intervals](https://www.jstor.org/stable/pdf/2347496.pdf?casa_token=FWVhlSa5GRcAAAAA:pPzSK4NeRprge0-dzlZp7ae5bCIiEEnJRn2L8Qq_s6ND57YmippIDaKDg8Pxq64QH_7_DW0h4bonSbnaLm4dFhsEuPaefVJoYCv1gNUhs_qhgmObZQ)." *Journal of the Royal Statistical Society: Series C (Applied Statistics)* 37.1 (1988): 87‑94.

* "[Structural and practical identifiability analysis of partially observed dynamical models by exploiting the profile likelihood.](https://academic.oup.com/bioinformatics/article/25/15/1923/213246)" *Bioinformatics* 25.15 (2009): 1923‑1929.  
