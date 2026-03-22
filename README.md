# The Story of Filtering

Author: Amber Lord

Extended by: Amiya

---

## Prologue: Before the Castle — Wiener Filter

Before Kalman built his castle in the Gauss's world, there was already a guardian standing at the gate of time-invariant systems. His name was **Norbert Wiener**.

The year was 1949. World War II had just ended. Radar systems needed to track enemy aircraft, and anti-aircraft guns needed to predict where planes would be. The problem was simple to state but hard to solve:

> Given noisy observations of a signal, how do we recover the true signal?

Wiener's approach was elegant in its own way. He worked entirely in the **frequency domain**.

### The Wiener-Hopf Equation

Given a signal $x(t)$ corrupted by noise $n(t)$, we observe:
$$
z(t) = x(t) + n(t)
$$

We want to design a filter $h(t)$ such that the estimate:
$$
\hat{x}(t) = \int_{-\infty}^{+\infty} h(\tau) z(t-\tau) d\tau
$$

minimizes the mean squared error:
$$
E[(x(t) - \hat{x}(t))^2]
$$

The optimal solution satisfies the **Wiener-Hopf equation**:
$$
\int_{0}^{+\infty} h(\tau) R_{zz}(t-\tau) d\tau = R_{xz}(t), \quad t \geq 0
$$

where $R_{zz}$ is the autocorrelation of the observation and $R_{xz}$ is the cross-correlation between signal and observation.

In the frequency domain, this becomes:
$$
H(\omega) = \frac{S_{xz}(\omega)}{S_{zz}(\omega)}
$$

where $S$ denotes power spectral density.

### The Limitations of Wiener

Wiener filter was revolutionary, but it had fundamental constraints:

| Constraint                | Why it matters                                        |
| ------------------------- | ----------------------------------------------------- |
| **Stationarity**          | Requires signal and noise to be wide-sense stationary |
| **Infinite history**      | Needs all past observations, not recursive            |
| **Frequency domain**      | Not suitable for real-time applications               |
| **Linear time-invariant** | Cannot handle time-varying systems                    |

In practice, the world is:

- Non-stationary (systems change over time)
- Demanding real-time processing
- Time-varying (parameters drift, dynamics shift)

Something new was needed. And that's when Rudolf Kalman entered the stage.

---

## First Chapter: The Establishment of Gauss's World

The story started from a long time ago, when statistics was still hiding in the dark. Before the 1800s, people knew there were errors everywhere, but didn't know what rules the errors follow. In 1794 Germany, a young man sitting in front of his desk tried to figure out what was wrong in his astronomical observation data's error. He noticed a very interesting thing:

```
        *
       ***
      *****
     *******
    *********
   ***********
  *************
```

It was a bell-shaped mountain. So the young man—Gauss—found the **least squares method** at age 18. This brought the first formula to the Gauss's World:
$$
\text{Estimation}_{\text{best}} = \arg\min \sum (\text{Observation} - \text{Real})^2 \tag{1}
$$

If the error follows a bell-shaped distribution, then the most likely true value is the one that minimizes the sum of squared errors.

As the young man grew up, he wrote down the formula which changed the world:
$$
f(x) = \frac{1}{\sqrt{2\pi\sigma^2}} \times e^{-\frac{(x-\mu)^2}{2\sigma^2}} \tag{2}
$$

The normal distribution is the true character of the bell-shaped mountain.

After the central limit theorem was proven, the Gauss's world was complete, supported by three basic rules:

$$
\text{Gauss} + \text{Gauss} = \text{Gauss} \\
A \times \text{Gauss} = \text{Gauss} \tag{Rule 1}
$$

$$
(\mu, \sigma^2) = \text{Everything} \tag{Rule 2}
$$

$$
E[X \mid Y] = \mu_X + \frac{\sigma_{XY}}{\sigma_Y^2} \left( Y - \mu_Y \right) \tag{Rule 3}
$$

Rule 3 means the conditional expectation can be linearly represented by $Y$. The three rules and three pillars (Random Variable, Probability Space, Gauss Measure) created the new world together.

Our major role lives in a castle in this world. The castle's first inhabitant, born from nature with the help of Wiener's legacy, was named **Kalman Filter**.

---

## Second Chapter: Kalman Filter

Through the story of Gauss's World, what can we do to use its power? Let's start from two fundamental equations that describe any linear dynamic system:

$$
x_k = A_k x_{k-1} + \overbrace{B_k u_k}^{\text{Control}} + w_k, \quad w_k \sim N(0, Q_k)
$$

$$
z_k = H_k x_k + v_k, \quad v_k \sim N(0, R_k)
$$

where $A_k$, $B_k$, $H_k$ are transformation matrices, and $w_k$, $v_k$ are process and measurement noise.

Many tellers start from the five sacred formulas. But this time, let me show you how it emerges naturally from Gauss's world.

### Prediction: The Time Update

Our goal is to compute $E[x_k | z_{1:k-1}]$ — using history to predict the future.

$$
\begin{align*}
E[x_k|z_{1:k-1}] &= E[A_k x_{k-1} + B_k u_k + w_k|z_{1:k-1}] \\
&= A_k E[x_{k-1}|z_{1:k-1}] + B_k u_k + \underbrace{E[w_k|z_{1:k-1}]}_{=0} \\
&= A_k \hat{x}_{k-1|k-1} + B_k u_k
\end{align*}
$$

Thus:
$$
\boxed{\hat{x}_{k|k-1} = A_k \hat{x}_{k-1|k-1} + B_k u_k} \tag{KF-1}
$$

Now, the error covariance. Define:
$$
e_{k|k-1} = x_k - \hat{x}_{k|k-1} = A_k e_{k-1|k-1} + w_k
$$

Then:
$$
\begin{align*}
P_{k|k-1} &= E[e_{k|k-1} e_{k|k-1}^T]\\
&= A_k P_{k-1|k-1} A_k^T + Q_k
\end{align*}
$$

Thus:
$$
\boxed{P_{k|k-1} = A_k P_{k-1|k-1} A_k^T + Q_k} \tag{KF-2}
$$

This formula is beautiful: it carries old uncertainty forward and adds new uncertainty from process noise. Information flows through time.

### Update: The Measurement Correction

Now we have a prediction $\hat{x}_{k|k-1}$ and a measurement $z_k$. How do we combine them?

A natural approach in Gauss's world: **linear combination**.
$$
\hat{x}_{k|k} = M\hat{x}_{k|k-1} + Nz_k
$$

For unbiasedness ($E[\hat{x}_{k|k}] = E[x_k]$), we need:
$$
M = I - NH_k
$$

Therefore:
$$
\hat{x}_{k|k} = \hat{x}_{k|k-1} + N(z_k - H_k\hat{x}_{k|k-1})
$$

The term $(z_k - H_k\hat{x}_{k|k-1})$ is the **innovation** — the "surprise" from the measurement.

Now we find $N$ that minimizes $\text{tr}(P_{k|k})$. After derivation (skipped here for brevity):

$$
\boxed{K_k = P_{k|k-1}H_k^T(H_kP_{k|k-1}H_k^T + R_k)^{-1}} \tag{KF-3}
$$

This is the **Kalman Gain** — the magical balance between prediction uncertainty and measurement uncertainty.

Finally:
$$
\boxed{\hat{x}_{k|k} = \hat{x}_{k|k-1} + K_k(z_k - H_k\hat{x}_{k|k-1})} \tag{KF-4}
$$

$$
\boxed{P_{k|k} = (I - K_kH_k)P_{k|k-1}} \tag{KF-5}
$$

### The Kalman Gain: An Intuitive View

Let's look at $K_k$ more carefully:
$$
K_k = \frac{P_{k|k-1}H_k^T}{H_kP_{k|k-1}H_k^T + R_k}
$$

- **Numerator**: How uncertain is our prediction about the measurement?
- **Denominator**: Total uncertainty (prediction + measurement)

If $R_k$ is large (noisy measurements), $K_k$ is small → trust prediction more.

If $P_{k|k-1}$ is large (uncertain prediction), $K_k$ is large → trust measurement more.

The Kalman Gain is an **information balancer**.

### The Flow

```
[Predict]
    x̂_{k|k-1} = A_k x̂_{k-1|k-1} + B_k u_k
    P_{k|k-1} = A_k P_{k-1|k-1} A_k^T + Q_k
    
    ↓
    
[Compute Gain]
    K_k = P_{k|k-1} H_k^T (H_k P_{k|k-1} H_k^T + R_k)^{-1}
    
    ↓
    
[Update]
    x̂_{k|k} = x̂_{k|k-1} + K_k (z_k - H_k x̂_{k|k-1})
    P_{k|k} = (I - K_k H_k) P_{k|k-1}
    
    ↓
    
[Repeat for k+1]
```

That's how KF works — clear, elegant, and optimal (for linear Gaussian systems).

---

## Third Chapter: Extended Kalman Filter

Time kept flowing. Our hero lived happily in the linear castle. But one day, a messenger brought troubling news: **The world is not linear.**

The real world systems look like:
$$
x_k = f(x_{k-1}) + w_k
$$

$$
z_k = h(x_k) + v_k
$$

where $f$ and $h$ are nonlinear functions. Gauss's world rules don't apply directly.

### The EKF Solution: Linearization

The Extended Kalman Filter's philosophy: **If you can't handle a curve, approximate it with a straight line at your current position.**

Use Taylor expansion around the current estimate:
$$
F_k = \frac{\partial f}{\partial x}\Big|_{x=\hat{x}_{k-1|k-1}}
$$

$$
H_k = \frac{\partial h}{\partial x}\Big|_{x=\hat{x}_{k|k-1}}
$$

Then:
$$
x_k \approx f(\hat{x}_{k-1|k-1}) + F_k(x_{k-1} - \hat{x}_{k-1|k-1}) + w_k
$$

$$
z_k \approx h(\hat{x}_{k|k-1}) + H_k(x_k - \hat{x}_{k|k-1}) + v_k
$$

Now we can compute Kalman Gain using $F_k$ and $H_k$ just like before!

### EKF Algorithm

**Predict:**
$$
\hat{x}_{k|k-1} = f(\hat{x}_{k-1|k-1})
$$

$$
P_{k|k-1} = F_k P_{k-1|k-1} F_k^T + Q_k
$$

**Update:**
$$
K_k = P_{k|k-1} H_k^T (H_k P_{k|k-1} H_k^T + R_k)^{-1}
$$

$$
\hat{x}_{k|k} = \hat{x}_{k|k-1} + K_k(z_k - h(\hat{x}_{k|k-1}))
$$

$$
P_{k|k} = (I - K_k H_k) P_{k|k-1}
$$

### The Cost of Linearization

EKF works well for **mildly nonlinear** systems. But it has issues:

1. **Divergence**: Strong nonlinearities cause the linearization to fail
2. **Jacobian computation**: Must compute derivatives analytically or numerically
3. **Local optimum**: Only first-order approximation

The Extended Kalman Filter opened the door to the nonlinear world, but it was just the beginning.

---

## Fourth Chapter: Unscented Kalman Filter

EKF had ruled the curved world for quite some time. But whispers spread through the villages:

> "EKF diverges when the system is really nonlinear."
> "Pretending a curved world is made of tiny straight lines is foolish when the curve is really curved."

Then came a visitor with a radical idea: **The Unscented Transform**.

### The Philosophy: Don't Linearize, Sample!

EKF approximates the **function** (by linearizing). UKF approximates the **distribution** (by sampling).

In Gauss's world, all information is in the mean and covariance. So instead of linearizing $f$, why not propagate a few carefully chosen points through $f$ and capture the transformed mean and covariance?

### Sigma Points

Given mean $\bar{x}$ and covariance $P$, we select $2n+1$ **sigma points**:

$$
\mathcal{X}_0 = \bar{x}
$$

$$
\mathcal{X}_i = \bar{x} + \left(\sqrt{(n+\lambda)P}\right)_i, \quad i=1,\ldots,n
$$

$$
\mathcal{X}_{i+n} = \bar{x} - \left(\sqrt{(n+\lambda)P}\right)_i, \quad i=1,\ldots,n
$$

with weights:
$$
W_0^{(m)} = \frac{\lambda}{n+\lambda}, \quad W_0^{(c)} = \frac{\lambda}{n+\lambda} + (1-\alpha^2+\beta)
$$

$$
W_i^{(m)} = W_i^{(c)} = \frac{1}{2(n+\lambda)}, \quad i=1,\ldots,2n
$$

where $\lambda = \alpha^2(n+\kappa) - n$ controls the spread.

### UKF Algorithm

**Predict:**

1. Form sigma points $\mathcal{X}_{i|k-1}$ from $\hat{x}_{k-1|k-1}$ and $P_{k-1|k-1}$
2. Propagate: $\mathcal{Y}_{i|k-1} = f(\mathcal{X}_{i|k-1})$
3. Compute statistics:
   - $\hat{x}_{k|k-1} = \sum_{i=0}^{2n} W_i^{(m)} \mathcal{Y}_{i|k-1}$
   - $P_{k|k-1} = \sum_{i=0}^{2n} W_i^{(c)} (\mathcal{Y}_{i|k-1} - \hat{x}_{k|k-1})(\cdot)^T + Q_k$

**Update:**

1. Reform sigma points from $\hat{x}_{k|k-1}$ and $P_{k|k-1}$
2. Propagate through observation: $\mathcal{Z}_{i|k-1} = h(\mathcal{X}_{i|k-1})$
3. Compute:
   - $\hat{z}_{k|k-1} = \sum W_i^{(m)} \mathcal{Z}_{i|k-1}$
   - $P_{zz} = \sum W_i^{(c)} (\mathcal{Z}_{i|k-1} - \hat{z}_{k|k-1})(\cdot)^T + R_k$
   - $P_{xz} = \sum W_i^{(c)} (\mathcal{X}_{i|k-1} - \hat{x}_{k|k-1})(\mathcal{Z}_{i|k-1} - \hat{z}_{k|k-1})^T$
4. Kalman Gain: $K_k = P_{xz} P_{zz}^{-1}$
5. Update: $\hat{x}_{k|k} = \hat{x}_{k|k-1} + K_k(z_k - \hat{z}_{k|k-1})$

### Why UKF Works Better

- **No Jacobians**: Don't need to compute derivatives
- **Higher-order accuracy**: Captures 2nd order (covariance) exactly, 3rd order approximately
- **Better for strong nonlinearities**: Sigma points "see" the curve, not just the tangent

But UKF still assumes **Gaussianity**. For truly non-Gaussian systems, we need something more powerful.

---

## Fifth Chapter: Particle Filter

UKF handled moderate nonlinearities well. But reports came from the outer territories — systems so wildly nonlinear, so non-Gaussian, that even UKF would fail.

Enter the **Particle Filter** — an outsider to Gauss's world.

### 1. Bayesian Filtering Framework

Consider the general system:
$$
x_k = f_k(x_{k-1}, v_{k-1})
$$

$$
y_k = h_k(x_k, n_k)
$$

Our goal: compute the posterior $p(x_k | y_{1:k})$ recursively.

**Prediction (Chapman-Kolmogorov):**
$$
p(x_k | y_{1:k-1}) = \int p(x_k | x_{k-1}) p(x_{k-1} | y_{1:k-1}) dx_{k-1}
$$

**Update (Bayes' rule):**
$$
p(x_k | y_{1:k}) = \frac{p(y_k | x_k) p(x_k | y_{1:k-1})}{p(y_k | y_{1:k-1})}
$$

Beautiful in theory. Impossible in practice for nonlinear, non-Gaussian systems — the integrals are intractable.

### 2. Monte Carlo Approximation

The key insight: **Approximate the distribution with samples (particles).**

$$
p(x_k | y_{1:k}) \approx \frac{1}{N} \sum_{i=1}^{N} \delta(x_k - x_k^{(i)})
$$

where $x_k^{(i)}$ are particles sampled from the posterior.

Expectations become sample averages:
$$
E[f(x_k)] \approx \frac{1}{N} \sum_{i=1}^{N} f(x_k^{(i)})
$$

### 3. Importance Sampling

But we can't sample from the posterior directly. Solution: sample from a **proposal distribution** $q(x_k | y_{1:k})$ and weight accordingly.

$$
E[f(x_k)] = \sum_{i=1}^{N} \tilde{w}_k^{(i)} f(x_k^{(i)})
$$

where normalized weights:
$$
\tilde{w}_k^{(i)} = \frac{w_k^{(i)}}{\sum_{j=1}^{N} w_k^{(j)}}
$$

### 4. Sequential Importance Sampling (SIS)

The recursive weight update:
$$
w_k^{(i)} \propto w_{k-1}^{(i)} \cdot \frac{p(y_k | x_k^{(i)}) p(x_k^{(i)} | x_{k-1}^{(i)})}{q(x_k^{(i)} | x_{k-1}^{(i)}, y_k)}
$$

### 5. Particle Degeneracy and Resampling

Problem: After many steps, most particles have negligible weight — only a few carry all the information.

Measure of degeneracy:
$$
N_{\text{eff}} = \frac{1}{\sum_{i=1}^{N} (w_k^{(i)})^2}
$$

When $N_{\text{eff}}$ drops below a threshold, **resample**: replicate high-weight particles, discard low-weight ones.

### 6. SIR Filter (Sampling Importance Resampling)

The simplest practical particle filter uses:
$$
q(x_k^{(i)} | x_{k-1}^{(i)}, y_k) = p(x_k^{(i)} | x_{k-1}^{(i)})
$$

This simplifies the weight to:
$$
w_k^{(i)} \propto p(y_k | x_k^{(i)})
$$

**SIR Algorithm:**

1. **Propagate**: $x_k^{(i)} \sim p(x_k | x_{k-1}^{(i)})$
2. **Weight**: $w_k^{(i)} = p(y_k | x_k^{(i)})$
3. **Normalize**: $\tilde{w}_k^{(i)} = w_k^{(i)} / \sum w_k^{(j)}$
4. **Resample** if $N_{\text{eff}} < N_{\text{threshold}}$
5. **Estimate**: $\hat{x}_k = \sum \tilde{w}_k^{(i)} x_k^{(i)}$

Particle Filter breaks free from Gaussian assumptions. But it comes at a cost: **computational complexity** — you need many particles for high-dimensional systems.

---

## Sixth Chapter: Deep Kalman Filter

The 2010s brought a new player to the field: **Deep Learning**. And with it, a question emerged:

> What if we use neural networks to learn the filtering dynamics?

### The Problem with Classical Filters

All classical filters assume we **know** the system model:

- State transition: $f(x_{k-1})$
- Observation model: $h(x_k)$

But in many real-world scenarios, we don't know these functions. We only have data.

### Deep Kalman Filter (DKF)

**Key idea**: Parameterize the unknown functions with neural networks and learn them from data.

The Deep Kalman Filter replaces:

- $f(x_{k-1}) \rightarrow f_\theta(x_{k-1})$ (neural network)
- $h(x_k) \rightarrow h_\phi(x_k)$ (neural network)

The parameters $\theta, \phi$ are learned by maximizing the likelihood of observations:
$$
\mathcal{L} = \sum_{k=1}^{T} \log p(y_k | y_{1:k-1}; \theta, \phi)
$$

### Variational Approach

Since exact inference is intractable, we use **variational inference**:

1. Define a variational posterior $q_\psi(z_{1:T} | y_{1:T})$
2. Optimize the ELBO (Evidence Lower Bound):

$$
\mathcal{L}_{\text{ELBO}} = E_{q}[\log p(y_{1:T} | z_{1:T})] - D_{KL}(q_\psi(z_{1:T}) || p(z_{1:T}))
$$

This is essentially a **Variational Autoencoder (VAE)** for sequential data.

### Deep Variational Bayes Filter (DVBF)

A more sophisticated approach learns a structured latent dynamics model:

1. **Encoder**: $q(z_k | z_{k-1}, y_k)$ — infers latent state from observation
2. **Transition**: $p(z_k | z_{k-1})$ — learned dynamics
3. **Decoder**: $p(y_k | z_k)$ — generates observation from latent state

The model is trained end-to-end using the reparameterization trick.

### Limitations

- **Black box**: Hard to interpret what the network learns
- **Data hungry**: Needs lots of training data
- **No guarantees**: May not respect physical constraints
- **Unstable**: Can diverge in ways classical filters don't

Deep Kalman Filters opened new possibilities, but raised concerns about reliability and interpretability.

---

## Seventh Chapter: KalmanNet

The Deep Kalman Filter threw away the entire Kalman structure and replaced it with neural networks. But what if we could **combine** the best of both worlds?

In 2022, Revach et al. proposed **KalmanNet**: a hybrid architecture that preserves the Kalman filter's structure while using neural networks to learn what we don't know.

### The Key Insight

The Kalman filter has five equations. Some we know (from physics), some we don't.

The innovation of KalmanNet: **Only learn the unknown part with a neural network.**

Specifically, the **Kalman Gain** computation:
$$
K_k = P_{k|k-1}H_k^T(H_kP_{k|k-1}H_k^T + R_k)^{-1}
$$

requires knowing $H_k$ (observation model) and $R_k$ (observation noise). If we don't know these, we can't compute $K_k$.

KalmanNet's solution: **Learn $K_k$ directly with a neural network.**

### Architecture

```
                    ┌─────────────────────────────────────┐
                    │           KalmanNet                 │
                    │                                     │
    z_k ──────────►│  ┌─────────────────────────────┐   │
                    │  │    Neural Network (LSTM)    │   │
                    │  │                             │   │
    Δy_k ──────────►│  │   Input: Innovation Δy_k    │   │
    (innovation)    │  │   Hidden: State history     │   │
                    │  │   Output: Kalman Gain K_k   │   │
                    │  └─────────────────────────────┘   │
                    │                 │                  │
                    │                 ▼                  │
    x̂_{k|k-1} ────►│            [Gain K_k]────────────►│───► x̂_{k|k}
    (prediction)    │                 │                  │     (estimate)
                    │                 ▼                  │
                    │        x̂_{k|k} = x̂_{k|k-1} + K_k·Δy_k │
                    └─────────────────────────────────────┘
```

### What We Need to Know

KalmanNet requires:

- $A_k$ (state transition matrix) — or at least partial knowledge
- The fact that the system is approximately linear

We **don't** need:

- $H_k$ (observation model)
- $Q_k$ (process noise)
- $R_k$ (measurement noise)

The neural network learns to compensate for the unknown parts.

### Training

KalmanNet is trained in a supervised manner:

1. Collect trajectories $\{x_k, y_k\}_{k=1}^{T}$
2. Run the known parts of the Kalman filter
3. Train the neural network to predict $K_k$ that minimizes $\|x_k - \hat{x}_{k|k}\|^2$

### Advantages

| Aspect            | Deep Kalman Filter | KalmanNet                                 |
| ----------------- | ------------------ | ----------------------------------------- |
| Interpretability  | Black box          | Partial (known dynamics preserved)        |
| Data efficiency   | Low                | Higher (inductive bias from KF structure) |
| Generalization    | Poor               | Better (respects physical structure)      |
| Sample complexity | High               | Lower                                     |

### Limitations

- Still requires some knowledge of the system
- Neural network is a black box component
- May not work well for highly nonlinear systems

KalmanNet represents a promising direction: **physics-informed deep learning** for filtering. It respects what we know and learns what we don't.

---

## Eighth Chapter: MCMC and Beyond

Particle filters approximate the posterior with samples, but the samples are generated sequentially and may not explore the state space efficiently. **Markov Chain Monte Carlo (MCMC)** offers an alternative approach.

### The MCMC Philosophy

Instead of generating samples sequentially, MCMC:

1. Constructs a Markov chain whose stationary distribution is the target posterior
2. Runs the chain until it converges
3. Uses samples from the converged chain

### Particle MCMC (PMCMC)

Combines particle filtering with MCMC:

1. **Particle Filter**: Generate a particle approximation of $p(x_{1:T} | y_{1:T})$
2. **MCMC Step**: Use the particle approximation within an MCMC sampler to refine estimates

The **Particle Marginal Metropolis-Hastings (PMMH)** algorithm:

- Proposes new parameters $\theta'$ using particle filter likelihood estimate
- Accepts/rejects based on Metropolis-Hastings ratio

### Sequential Monte Carlo (SMC) Samplers

A generalization of particle filtering for:

- Static parameter estimation
- Model selection
- Rare event simulation

### Hamiltonian Monte Carlo (HMC) for Filtering

For continuous state spaces, HMC can efficiently sample from the posterior by:

- Treating the state as a position in a high-dimensional space
- Introducing momentum variables
- Using Hamiltonian dynamics to propose moves

### Variational Inference (VI)

An alternative to MCMC that:

- Approximates the posterior with a simpler distribution $q(x)$
- Optimizes $q$ to be close to the true posterior (minimizing KL divergence)
- Much faster than MCMC but less accurate

**Amortized Inference**: Train a neural network to output the variational parameters given observations.

### Current Frontiers

1. **Normalizing Flows**: Learn complex posterior distributions with invertible neural networks
2. **Score-Based Diffusion**: Use diffusion models for state estimation
3. **Continuous-Time Filtering**: Neural ODEs + filtering for irregularly sampled data
4. **Multi-Modal Fusion**: Combine heterogeneous sensors with learned observation models
5. **Causal Filtering**: Incorporate causal structure into state estimation

---

## Epilogue: The Castle Expands

The story of filtering began with Wiener in the frequency domain, was revolutionized by Kalman in the time domain, extended by EKF and UKF to nonlinear worlds, freed by Particle Filters from Gaussian assumptions, and now embraces Deep Learning as a new ally.

The castle that Kalman built has grown into a vast kingdom. Each generation of filters has:

- Expanded the domain of applicability
- Relaxed assumptions
- Increased computational complexity
- Added new tools to the arsenal

Yet the core philosophy remains:

> **Use all available information optimally to estimate the state of the world.**

Whether through elegant mathematics or powerful neural networks, this is the eternal quest of filtering.

---

## Appendix: Filter Comparison Table

| Filter    | Dynamics        | Noise      | Observations | Pros                      | Cons                      |
| --------- | --------------- | ---------- | ------------ | ------------------------- | ------------------------- |
| Wiener    | LTI             | Stationary | All history  | Optimal (stationary)      | Not recursive             |
| Kalman    | Linear          | Gaussian   | Recursive    | Optimal, efficient        | Linear only               |
| EKF       | Nonlinear       | Gaussian   | Recursive    | Handles mild nonlinearity | Jacobians needed          |
| UKF       | Nonlinear       | Gaussian   | Recursive    | No Jacobians              | Still Gaussian            |
| Particle  | Any             | Any        | Recursive    | Fully general             | Computationally expensive |
| Deep KF   | Unknown         | Learned    | Recursive    | Learns from data          | Black box, data hungry    |
| KalmanNet | Partially known | Learned    | Recursive    | Best of both worlds       | Needs some structure      |

---

*To be continued...*

*The story of filtering is still being written.*
