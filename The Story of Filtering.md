# The Story of Filtering

Author: Amber Lord

## First Chapter: The Establishment of Gauss's world

The story started from a long time ago, when the statistics was still hiding in the dark. Before the 1800s, people knew that there were errors in everywhere, but didn't know what rules the errors follow. In 1794 German, a young man sitting in front of the desk tried to figure out what's wrong in his astronomical observation data's error. He noticed a vary interesting thing:

```python
        *
       ***
      *****
     *******
    *********
   ***********
  *************
```

It was a bell shaped mountain. So the young man-Gauss found out **least squares method** in his 18. This brings the first formula to the Gauss's World.
$$
Estimation_{best}=argminΣ(Observation-Real)^2 \tag{1}
$$

If the error follows a bell shaped distribution, then the most likely true value is the one that minimizes the sum of squared errors.

Through the young man growing up, he wrote down the formula which changed the world:
$$
f(x) = \frac{1}{\sqrt{2\pi\sigma^2}} \times e^{-\frac{(x-\mu)^2}{2\sigma^2}} \tag{2}
$$
The normal distribution is the true character of the bell shaped mountain.

After the central limit theorem has been proven, the Gauss's world has finished right now which is supported by three basic rule.
$$
Gauss + Gauss = Gauss \\
A \times Gauss = Gauss \tag{Rule 1}
$$

$$
(\mu,\sigma^2)=Everything \tag{Rule 2}
$$

$$

E[X \mid Y] = \mu_X + \frac{\sigma_{XY}}{\sigma_Y^2} \left( Y - \mu_Y \right) \tag{Rule 3}
$$

Here we will skip the useless explanation  of three basic rules, by the way, the rule 3 means it can be linearly represented by $Y$. The three rules and the three pillars (Random Variable, Probability space, Gauss Measure) created the new world together.

Our major role is in a castle in this world, the castle's first person which born from the nature with the help of wiener filter. His name is Kalman Filter.

## Second Chapter: Kalman Filter

Well through the story of the Gauss's World, what can we do to use the power of it? To solve this problem, let's started from two fundamental equations that describe any linear dynamic system:
$$
x_k = A_k x_{k-1} + \overbrace{B_k u_k}^{Independ} + w_k\\w_k\sim N(0,Q_k)
$$

$$
z_k = H_k x_k + v_k \\v_k\sim N(0,R_k)
$$

where: $A_k$, $B_k$, $H_k$ are all the transform matrix, $w_k$ and $v_k$ are the noise

I've seen many tellers started from the five sacred formulas, but this time, I want try something new. Just as I told in the first chapter, let me show you how it came from nature.

The aim of us is to show $E[x_k|z_{1:k-1}]$, which can help us use the history to predict the future.
$$
\begin{align*}
E[x_k|z_{1:k-1}] &= E[A_k x_{k-1} + B_k u_k + w_k|z_{1:k-1}] \\
&= A_k E[x_{k-1}|z_{1:k-1}] + B_k u_k + E[w_k|z_{1:k-1}] \\
&= A_k E[x_{k-1}|z_{1:k-1}] + B_k u_k
\end{align*}
$$
As a result:
$$
\hat{x}_{k|k-1} = A_k \hat{x}_{k-1|k-1} + B_k u_k \tag{1}
$$
The control of the error is the key to predict, here we define the error:
$$
\begin{align*}
e_{k|k-1} &= x_k - \hat{x}_{k|k-1} \\
&= (A_k x_{k-1} + B_k u_k + w_k)- (A_k \hat{x}_{k-1|k-1} + B_k u_k)\\
&= A_k(x_{k-1}-\hat{x}_{k-1|k-1})+w_k\\
&= A_ke_{k-1|k-1}+w_k
\end{align*}
$$
Through the error formula, we can explore the covariance:
$$
\begin{align*}
P_{k|k-1} &= E[e_{k|k-1} e_{k|k-1}^T]\\
&= E[(A_k e_{k-1|k-1} + w_k)(A_k e_{k-1|k-1} + w_k)^T]\\
&=\text{skip and the the expection of w is 0} \\
&= A_k E[e_{k-1|k-1} e_{k-1|k-1}^T] A_k^T + E[w_k w_k^T]\\
&= A_k P_{k-1|k-1} A_k^T + Q_k 
\end{align*}
$$
As a result:
$$
P_{k|k-1}= A_k P_{k-1|k-1} A_k^T + Q_k \tag{2}
$$
The second formula is quiet interesting, it contains the old uncertainty from the history and the new uncertainty from the noise. The information in the uncertainty will be passed forever.

What can these things do for us? In this essay, I do not want to tell you the concept cause I have not finished to combine them all. I'd rather to tell you what can we do through the information world maybe you can better know the filter that helps you step to the further stage. Through the formulas before, we have got the transform function and the function help us observe. If we want to predict, maybe it's the best way to combine the prediction and the observation simply:
$$
\hat{x}_{k|k}=M\hat{x}_{k|k-1}+Nz_k
$$
The linearly combination is very normal in the Gauss's world, but we got the final prediction through the prediction and observation. The two different bell shaped mountain have now combined into a bigger mountain. Through the world basic rules, the combination contains the full key information. This is just like the LSTM, the key information was reserved, and this information will deliver to the next point and pass forever. 

Well, the problem now have change to: What's M or N?This means we now need to figure out a question: How to balance the prediction and observation to make the final prediction more confident? So when you think about this optimize problem, I think we can firstly simplify it , because finding a best point in the 3-D is much more difficult than in the 2-D, what about we represent one variable through the other variable?

To fulfill unbiased, we can deduce that:
$$
\begin{align}
E[\hat{x}_{k|k}] &=E[x_k] \\
E[M\hat{x}_{k|k-1}+Nz_k]&=E[x_k]\\
ME[\hat{x}_{k|k-1}]+NE[z_k]&=E[x_k]\\
ME[x_k]+NH_kE[x_k]+\overbrace{NE[v_k]}^0 &=E[x_k]\\
(M+NH_k)E[x_k]&=E[x_k]\\
M+NH_k&=I\\
M &= I-NH_k
\end{align}
$$
And we get:
$$
\begin{align*}
\hat{x}_{k|k}&=(I-NH_k)\hat{x}_{k|k-1}+Nz_k\\
&= \hat{x}_{k|k-1}+N(z_k-H_k\hat{x}_{k|k-1})
\end{align*}
$$
Through the formula below, now we can start to find the N which can fulfill the  least squares method, make the trace of the covariance smallest.

As the result, we need to deduce the new covariance:
$$
\begin{align*}
e_{k|k}&=x_k -\hat{x}_{k|k}\\
&= x_k -[\hat{x}_{k|k-1}+N(z_k-H_k\hat{x}_{k|k-1})]\\
&= x_k -[\hat{x}_{k|k-1}+N(H_k x_k + v_k-H_k\hat{x}_{k|k-1})]\\
&= (I-NH_k)(x_k-\hat{x}_{k|k-1})-Nv_k
\end{align*}
$$

$$
\begin{align*}
P_{k|k}&=E[e_{k|k}\times e_{k|k}^T]\\
&= E\Big[(I - NH_k)e_{k|k-1} \times e_{k|k-1}^T(I - NH_k)^T 
- (I - NH_k)e_{k|k-1} \times v_k^T N^T 
- Nv_k \times e_{k|k-1}^T(I - NH_k)^T 
+ Nv_k \times v_k^T N^T\Big]\\
&= E[(I - NH_k)e_{k|k-1} e_{k|k-1}^T(I - NH_k)^T] 
- E[(I - NH_k)e_{k|k-1} v_k^T N^T] 
- E[Nv_k e_{k|k-1}^T(I - NH_k)^T] 
+ E[Nv_k v_k^T N^T]\\
&=\text{The independence of the v makes the muti item to 0}\\
&=  E[(I - NH_k)e_{k|k-1} e_{k|k-1}^T(I - NH_k)^T] + E[Nv_k v_k^T N^T]\\
&= (I - NH_k)E[e_{k|k-1} e_{k|k-1}^T](I - NH_k)^T + NE[v_k v_k^T]N^T\\
&= (I - NH_k)P_{k|k-1}(I - NH_k)^T + NR_kN^T\\
&= (I - NH_k)P_{k|k-1}(I^T - H_k^TN^T)+ NR_kN^T\\
&= (I - NH_k)P_{k|k-1}(I - H_k^TN^T)+ NR_kN^T\\
&= P_{k|k-1} - NH_kP_{k|k-1} - P_{k|k-1}H_k^TN^T + NH_kP_{k|k-1}H_k^TN^T+ NR_kN^T
\end{align*}
$$

At the present, our work turns into compute the first difference of the trace of the P, and I searched some formula which is helpful:
$$

\frac{\partial \text{tr}(AX)}{\partial X} = A^T\\
\frac{\partial \text{tr}(XBX^T)}{\partial X} = X(B + B^T)\\
   \frac{\partial \text{tr}(X^T X)}{\partial X} = 2X\\
\frac{\partial \text{tr}(AXB)}{\partial X} = A^T B^T\\
\frac{\partial \text{tr}(AX^T BX)}{\partial X} = (BA + B^T A^T) X^T
$$
Before we start taking the derivative, let's simplify P based on the properties of the trace:
$$
\begin{align*}
\text{tr}(P_{k|k}) &= \text{tr}(P_{k|k-1}) - \text{tr}(NH_kP_{k|k-1}) - \text{tr}(P_{k|k-1}H_k^TN^T) 
+ \text{tr}(NH_kP_{k|k-1}H_k^TN^T) + \text{tr}(NR_kN^T)\\
& = \text{tr}(P_{k|k-1}) - 2\text{tr}(NH_kP_{k|k-1}) + \text{tr}(N(H_kP_{k|k-1}H_k^T)N^T) + \text{tr}(NR_kN^T)\\
&= \text{tr}(P_{k|k-1}) - 2\text{tr}(NH_kP_{k|k-1}) + \text{tr}(N(H_kP_{k|k-1}H_k^T + R_k)N^T)
\end{align*}
$$

$$
\begin{align*}
\frac{\partial \text{tr}(P_{k|k})}{\partial N} &= \frac{\partial}{\partial N}\left[\text{tr}(P_{k|k-1}) - 2\text{tr}(NH_kP_{k|k-1}) + \text{tr}(N(H_kP_{k|k-1}H_k^T + R_k)N^T)\right]\\
&= 0 - 2(H_kP_{k|k-1})^T + 2N(H_kP_{k|k-1}H_k^T + R_k)\\
&= -2P_{k|k-1}^TH_k^T + 2N(H_kP_{k|k-1}H_k^T + R_k)
\end{align*}
$$

$$
\begin{align*}
\text{Just a regular minimum value problem}\\
\frac{\partial \text{tr}(P_{k|k})}{\partial N} &= 0 \\
-2P_{k|k-1}^TH_k^T + 2N(H_kP_{k|k-1}H_k^T + R_k) &= 0\\
N &= P_{k|k-1}H_k^T(H_kP_{k|k-1}H_k^T + R_k)^{-1}
\end{align*}
$$

Finally, we get the best balance N, and It's real given name is **Kalman Gain**:
$$
K_k=\frac{P_{k|k-1}H_k^T}{(H_kP_{k|k-1}H_k^T + R_k)}\tag{3}
$$
Now we have found the magical balance point - the Kalman Gain. But what does this N (or as we formally call it Kalman Gain) really mean? 

Look at the formula,you will find that this is actually a competition between two uncertainties. The numerator shows the uncertain we are about our prediction. The denominator shows the uncertainty of the prediction passes through the observation model's equivalent uncertainty. After that, you will understand its magic.

After all, we need to bring the $K_k$ back to the formulas below:
$$
\begin{align*}
\hat{x}_{k|k}&= \hat{x}_{k|k-1}+K_k(z_k-H_k\hat{x}_{k|k-1})\tag{4}
\end{align*}
$$

$$
P_{k|k}= (I - K_kH_k)P_{k|k-1}\tag{5}
$$

Well, the five sacred formulas are all here, and I will show you in a flow:

