@def title = "Constructing Systems of ODEs: A Practical Guide"
@def published = "13 November 2025"
@def tags = ["ODE"]

# Constructing Systems of ODEs: A Practical Guide

## The Core Philosophy

You're exactly right! Constructing ODEs is about **observing a system and translating its behavior into mathematics**. The basic idea is beautifully simple:

> **Rate of change = What affects the change**

If something is changing over time, you write down what causes it to change.

## The Starting Point: Three Questions

When facing any system, ask yourself:

1. **What quantities are changing?** (These become your variables)
2. **What is the rate of change?** (These become your derivatives)
3. **What influences this rate?** (These become the right-hand side of your equation)

### ODEs vs PDEs: When Do You Need Partials?

**Here's the key insight:** You naturally think in ODEs when tracking **discrete objects** or **well-mixed systems**. You need PDEs when things vary across **space**.

#### You're Already Thinking in ODEs

Most everyday observations are ODE-friendly:
- "The coffee is cooling over time" → ODE (one temperature for the whole cup)
- "The population is growing" → ODE (total population count)
- "My bank account balance changes" → ODE (one balance value)

**Why ODEs work here:** Each quantity has a single value at any moment.

#### When You Need PDEs

PDEs emerge when you ask: **"But what if it's different over here vs over there?"**

**Example progression:**

*ODE thinking:*
- "The room is heating up" → $\frac{dT}{dt} = \text{heat input}$

*PDE thinking:*
- "But wait, it's warmer near the heater and cooler by the window..."
- Now temperature is $T(x, y, z, t)$ → need $\frac{\partial T}{\partial t}$
- Heat flows from hot to cold regions → need $\frac{\partial T}{\partial x}$, etc.

**The heat equation:**
$\frac{\partial T}{\partial t} = \alpha \left(\frac{\partial^2 T}{\partial x^2} + \frac{\partial^2 T}{\partial y^2} + \frac{\partial^2 T}{\partial z^2}\right)$

#### Common ODE → PDE Transitions

| Scenario | ODE Version | PDE Version |
|----------|-------------|-------------|
| **Population** | Total population $P(t)$ | Population density $\rho(x,y,t)$ |
| **Concentration** | Well-mixed tank $C(t)$ | Concentration field $C(x,t)$ |
| **Vibration** | Single mass-spring $x(t)$ | Vibrating string $u(x,t)$ |
| **Disease** | Total infected $I(t)$ | Infection density $I(x,y,t)$ |

#### The Observation Difference

**For ODEs, you observe:**
- "How much is there?"
- "How fast is it changing?"
- Time is usually the only independent variable

**For PDEs, you observe:**
- "How is it distributed in space?"
- "How does it flow from place to place?"
- "Does it diffuse? Advect? Wave-like propagate?"

**More nuanced observations needed:**
- **Diffusion:** Things spread from high to low concentration
- **Advection:** Things get carried along (like wind)
- **Waves:** Disturbances propagate
- **Boundary effects:** What happens at edges matters!

#### Why PDEs Are Harder to "See"

**ODEs:** Follow one thing through time
$\frac{d}{dt}[\text{one quantity}] = \text{influences}$

**PDEs:** Track how spatial patterns evolve
$\frac{\partial}{\partial t}[\text{field}] = \text{spatial operators}[\text{field}] + \text{sources}$

You need to think about:
- **Gradients:** $\nabla T$ (which way does it change in space?)
- **Laplacians:** $\nabla^2 T$ (is it locally peaked or flat?)
- **Divergence/Curl:** For vector fields

#### The Beautiful Pattern

Most physical laws have this structure:

$\frac{\partial}{\partial t}[\text{density}] + \nabla \cdot [\text{flux}] = \text{sources} - \text{sinks}$

This is a **conservation law**:
- Rate of change in a region
- Plus what flows out of the region  
- Equals what's created or destroyed

**Examples:**
- Mass conservation → Continuity equation
- Energy conservation → Heat equation
- Momentum conservation → Navier-Stokes

#### When to Use What?

**Stick with ODEs if:**
- Spatial variations don't matter (well-mixed, uniform)
- Tracking discrete counts or totals
- System is 0-dimensional (point masses, lumped systems)

**Move to PDEs when:**
- Spatial patterns are essential
- You care about waves, diffusion, or transport
- Boundary conditions matter
- System has continuous spatial extent

#### The Cognitive Jump

You're right that **we naturally think in ODEs** because we naturally track objects:
- "How many?" 
- "How fast?"
- "What happens next?"

PDEs require thinking about **fields and flows**:
- "How is it distributed?"
- "Where is it flowing?"
- "How do patterns evolve?"

This is why ODEs feel intuitive (tracking your bank account) while PDEs feel abstract (tracking how money flows through an economy's spatial network).

### The Good News

Even complex PDE systems often reduce to ODEs in special cases:
- **Separation of variables:** Splits space and time
- **Spatial averaging:** Get ODEs for total quantities
- **Lumped parameter models:** Discretize space into compartments

So mastering ODEs gives you the foundation. PDEs are "just" ODEs where your variables also depend on position!

## Classic Example: Population Growth

**Observation:** A population grows over time.

**Step 1:** Identify the variable
- Let $P(t)$ = population at time $t$

**Step 2:** What's changing?
- $\frac{dP}{dt}$ = rate of population change

**Step 3:** What influences this rate?
- More people → more births → faster growth
- So growth rate is proportional to current population

**Result:** 
$$\frac{dP}{dt} = rP$$

where $r$ is the growth rate constant.

## Building Complexity: Predator-Prey

**Observation:** Rabbits and foxes interact in an ecosystem.

**Step 1:** Variables
- $R(t)$ = rabbit population
- $F(t)$ = fox population

**Step 2:** What changes each population?

*For rabbits:*
- ✅ Rabbits reproduce → increases $R$
- ❌ Foxes eat rabbits → decreases $R$

*For foxes:*
- ✅ Eating rabbits helps foxes survive → increases $F$
- ❌ Foxes die naturally → decreases $F$

**Step 3:** Translate to math

$$\begin{align}
\frac{dR}{dt} &= aR - bRF \quad \text{(birth rate - predation)} \\
\frac{dF}{dt} &= cRF - dF \quad \text{(gain from eating - death rate)}
\end{align}$$

## The General Recipe

### 1. **Mechanical Systems** (Physics)
Start with Newton's laws: $F = ma$

**Example:** Spring-mass system
- Position: $x(t)$
- Velocity: $v(t) = \frac{dx}{dt}$
- Acceleration: $\frac{dv}{dt} = \frac{d^2x}{dt^2}$

Forces acting:
- Spring force: $-kx$
- Damping: $-c\frac{dx}{dt}$

$$\frac{d^2x}{dt^2} = -kx - c\frac{dx}{dt}$$

Or as a system:
$$\begin{align}
\frac{dx}{dt} &= v \\
\frac{dv}{dt} &= -kx - cv
\end{align}$$

### 2. **Chemical Reactions**
Start with reaction rates and mass balance.

**Example:** $A \rightarrow B$ with rate $k$

$$\begin{align}
\frac{d[A]}{dt} &= -k[A] \\
\frac{d[B]}{dt} &= k[A]
\end{align}$$

### 3. **Compartment Models** (Epidemiology, Economics)
Track flow between categories.

**Example:** SIR disease model
- $S$ = susceptible
- $I$ = infected  
- $R$ = recovered

$$\begin{align}
\frac{dS}{dt} &= -\beta SI \quad \text{(susceptible → infected)} \\
\frac{dI}{dt} &= \beta SI - \gamma I \quad \text{(gains from S, losses to R)} \\
\frac{dR}{dt} &= \gamma I \quad \text{(infected → recovered)}
\end{align}$$

### 4. **Conservation Laws**
If something is conserved, its total derivative is zero.

**Example:** Energy conservation

$$E = E_{\text{kinetic}} + E_{\text{potential}}$$
$$\frac{dE}{dt} = 0$$

## Key Principles to Remember

### Principle 1: Units Must Match
Both sides of your equation must have the same units!

If $\frac{dP}{dt}$ is in "people/day", then the right side must also be "people/day".

### Principle 2: Look for Feedback
- **Positive feedback:** More of $X$ causes more $X$ (exponential growth)
- **Negative feedback:** More of $X$ causes less $X$ (stability, oscillation)

### Principle 3: Start Simple, Add Complexity
- First model: capture the most important effect
- Then add: secondary effects, nonlinearities, coupling between variables

### Principle 4: Check Limiting Behavior
Ask: "What happens when $X = 0$?" or "What happens when $X \to \infty$?"

Your equations should make physical sense in these limits.

## Common Patterns

### Linear Growth/Decay
$$\frac{dy}{dt} = ay + b$$

### Logistic Growth (with carrying capacity)
$$\frac{dy}{dt} = ry\left(1 - \frac{y}{K}\right)$$

### Coupled Oscillators
$$\begin{align}
\frac{dx}{dt} &= -\omega y \\
\frac{dy}{dt} &= \omega x
\end{align}$$

### Mass Action (interactions)
$$\frac{dx}{dt} = -kxy$$

Rate proportional to how often $X$ and $Y$ meet.

## Practice Exercise

**Your turn:** Model a cup of hot coffee cooling in a room.

<details>
<summary>Hint 1</summary>

What's the variable? Temperature $T(t)$.

What affects cooling rate?

</details>

<details>
<summary>Hint 2</summary>

The bigger the temperature difference, the faster it cools.

</details>

<details>
<summary>Solution</summary>

Newton's Law of Cooling:
$$\frac{dT}{dt} = -k(T - T_{\text{room}})$$

When $T > T_{\text{room}}$: $\frac{dT}{dt} < 0$ (cooling)

When $T = T_{\text{room}}$: $\frac{dT}{dt} = 0$ (equilibrium)

</details>

## From Equations to Parameters: Fitting ODEs to Data

Once you've constructed your ODE system, you face a challenge: **What are the parameter values?**

Unlike simple regression where you fit $y = mx + b$ directly, ODEs create an indirect relationship between parameters and data. You can't just use least squares directly!

### The Challenge

Given data points $(t_1, y_1), (t_2, y_2), \ldots, (t_n, y_n)$ and a model:
$\frac{dy}{dt} = f(y, t; \theta)$

where $\theta$ are unknown parameters, you need to find $\theta$ that best fits the data.

**The problem:** You don't have a formula for $y(t)$—you only have the differential equation!

### Approach 1: Solve, Then Fit

**When it works:** Simple ODEs with analytical solutions

**How it works:**
1. Solve the ODE analytically to get $y(t; \theta)$
2. Use standard least squares to minimize:
$\min_{\theta} \sum_{i=1}^n \left(y_i - y(t_i; \theta)\right)^2$

**Example:** For $\frac{dP}{dt} = rP$ with $P(0) = P_0$:

1. Solve: $P(t) = P_0 e^{rt}$
2. Take log: $\ln P(t) = \ln P_0 + rt$
3. Linear regression on $\ln P$ vs $t$ gives you $r$!

**Limitation:** Most ODEs can't be solved analytically.

### Approach 2: Numerical Integration + Optimization

**When it works:** Most real-world problems (the standard approach)

**How it works:**
1. Pick initial guess for parameters $\theta$
2. Numerically solve the ODE with these parameters
3. Compare solution to data, compute error
4. Adjust $\theta$ to reduce error
5. Repeat until convergence

**The algorithm:**
$\min_{\theta} \sum_{i=1}^n \left(y_i - y_{\text{sim}}(t_i; \theta)\right)^2$

where $y_{\text{sim}}(t; \theta)$ is the numerical solution.

**Tools:**
- Python: `scipy.optimize.minimize` + `scipy.integrate.solve_ivp`
- MATLAB: `lsqnonlin` + `ode45`
- R: `optim` + `deSolve`

**Example pseudo-code:**
```python
def simulate(theta, t_points, y0):
    # Solve ODE numerically
    return solve_ivp(lambda t, y: f(y, t, theta), 
                     [t0, tf], y0, t_eval=t_points)

def objective(theta):
    y_sim = simulate(theta, t_data, y0)
    return sum((y_data - y_sim)**2)

theta_best = minimize(objective, theta_initial)
```

### Approach 3: Direct Derivative Estimation

**When it works:** When you have lots of smooth data

**How it works:**
1. Estimate $\frac{dy}{dt}$ directly from data (finite differences, smoothing splines)
2. Use the ODE structure: $\frac{dy}{dt} \approx f(y, t; \theta)$
3. Fit $\theta$ using regression on the derivatives

**Example:** For $\frac{dP}{dt} = rP$:
1. Compute $\frac{dP}{dt} \approx \frac{P_{i+1} - P_i}{t_{i+1} - t_i}$ at each point
2. Regression: $\frac{dP}{dt}$ vs $P$ gives slope $r$

**Advantages:** Fast, doesn't require solving ODE
**Disadvantages:** Very sensitive to noise in data

### Approach 4: Bayesian Parameter Estimation

**When it works:** When you want uncertainty quantification

**How it works:**
- Define prior distributions on parameters
- Use MCMC or other sampling methods
- Get full posterior distribution: $p(\theta | \text{data})$

**Advantages:** 
- Provides uncertainty estimates
- Can incorporate prior knowledge
- Handles model uncertainty

**Disadvantages:** Computationally expensive

### Which Approach Should You Use?

```
Data Quality          | Recommended Approach
---------------------|------------------------
Analytical solution  | Solve + least squares
Lots of clean data   | Direct derivative method
Standard case        | Numerical ODE + optimization
Need uncertainty     | Bayesian methods
Multiple scenarios   | Numerical + optimization
```

### Common Pitfalls

**1. Identifiability Problem**

Sometimes different parameter values give identical predictions!

Example: In $\frac{dP}{dt} = rP(1 - P/K)$, only the ratio $r/K$ might be identifiable from limited data.

**2. Local Minima**

The optimization landscape can be complex. Solutions:
- Try multiple initial guesses
- Use global optimization methods
- Add regularization

**3. Overfitting**

More parameters ≠ better model. Use:
- Cross-validation
- Information criteria (AIC, BIC)
- Physical constraints

**4. Stiff Systems**

Some ODEs are "stiff" and hard to solve numerically. Use specialized solvers (implicit methods).

### Practical Workflow

1. **Explore your data:** Plot it, understand the trends
2. **Start simple:** Fit simplest model first
3. **Check residuals:** Do they look random?
4. **Add complexity:** Only if residuals show systematic patterns
5. **Validate:** Test on held-out data or different initial conditions
6. **Iterate:** Refine model structure and refit

### Reality Check

In practice, parameter estimation for ODEs is often **more art than science**:
- Initial conditions matter (a lot!)
- Measurement noise is real
- Model structure matters more than precise parameters
- Sometimes "eyeballing it" is surprisingly effective

The goal isn't always to find the "true" parameters—it's to build a model that captures the essential dynamics and makes useful predictions.

## The Beautiful Truth

The "basics" of ODE construction ARE simple! The sophistication comes from:

1. Choosing which effects matter most
2. Estimating parameters from data (now you know how!)
3. Analyzing the resulting equations

But the core skill—translating observation into rates of change—is fundamentally intuitive. You're watching the world change and asking "why?"