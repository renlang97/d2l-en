# Multivariable Calculus
:label:`sec_multivariable_calculus`

Now, that we have a fairly strong understanding of what happens if we had a function of a single variable, let us return to our original question where we were considering a loss function of potentially billions of weights.  

## Generalizing Differentiation
What :numref:`sec_single_variable_calculus` tells us is that if we change a single one of these billions of weights leaving every other one fixed, we know what will happen!  This is nothing more than a function of a single variable, so we can write

$$
L(w_1+\epsilon_1,w_2,\ldots,w_N) \approx L(w_1,w_2,\ldots,w_N) + \epsilon_1 \frac{d}{dw_1} L(w_1,w_2,\ldots,w_N).
$$

We will call the derivative in one variable while fixing the other the *partial derivative*, and we will use the notation $\frac{\partial}{\partial w_1}$.

Now, let us take this, and change $w_2$ a little bit to $w_2 + \epsilon_2$:

$$
\begin{aligned}
L(w_1+\epsilon_1,w_2+\epsilon_2,\ldots,w_N) & \approx L(w_1,w_2+\epsilon_2,\ldots,w_N) + \epsilon_1 \frac{\partial}{\partial w_1} L(w_1,w_2+\epsilon_2,\ldots,w_N) \\
& \approx L(w_1,w_2,\ldots,w_N) \\
& \quad + \epsilon_2\frac{\partial}{\partial w_2} L(w_1,w_2,\ldots,w_N) \\
& \quad + \epsilon_1 \frac{\partial}{\partial w_1} L(w_1,w_2,\ldots,w_N) \\
& \quad + \epsilon_1\epsilon_2\frac{\partial}{\partial w_2}\frac{\partial}{\partial w_1} L(w_1,w_2,\ldots,w_N) \\
& \approx L(w_1,w_2,\ldots,w_N) \\
& \quad + \epsilon_2\frac{\partial}{\partial w_2} L(w_1,w_2,\ldots,w_N) \\
& \quad + \epsilon_1 \frac{\partial}{\partial w_1} L(w_1,w_2,\ldots,w_N).
\end{aligned}
$$

where we have used the idea that $\epsilon_1\epsilon_2$ is a higher order term that we can discard in the same way we could discard $\epsilon^{2}$ in the previous section.  By continuing in this manner, we may write that

$$
L(w_1+\epsilon_1,w_2+\epsilon_2,\ldots,w_N+\epsilon_N) \approx L(w_1,w_2,\ldots,w_N) + \sum_i \epsilon_i \frac{\partial}{\partial w_i} L(w_1,w_2,\ldots,w_N).
$$

This may look like a mess, but we can make this more familiar by noting that the sum on the right looks exactly like a dot product, so if we let

$$
\boldsymbol{\epsilon} = [\epsilon_1, \ldots, \epsilon_N]^\top, \text{ and }
\nabla_{\mathbf{x}} L = \left[\frac{\partial L}{\partial x_1}, \ldots, \frac{\partial L}{\partial x_N}\right]^\top,
$$

then

$$
L(\mathbf{w} + \boldsymbol{\epsilon}) \approx L(\mathbf{w}) + \boldsymbol{\epsilon}\cdot \nabla_{\mathbf{w}} L(\mathbf{w}).
$$

We will call the vector $\nabla_{\mathbf{w}} L$ the *gradient* of $L$.

This formula is worth pondering for a moment.  It has exactly the format that we encountered in one dimension, just we have converted everything to vectors and dot products.  It allows us to tell approximately how the function $L$ will change given any perturbation to the input.  As we will see in the next section, this will provide us with an important tool in understanding geometrically how we can learn using information contained in the gradient.

But first, let us see this approximation at work with an example.  Suppose we are working with the function

$$
f(x,y) = \log(e^x + e^y) \text{ with gradient } \nabla f (x,y) = \left[\frac{e^x}{e^x+e^y}, \frac{e^y}{e^x+e^y}\right].
$$

If we look at a point like $(0,\log(2))$, we see that

$$
f(x,y) = \log(3) \text{ with gradient } \nabla f (x,y) = \left[\frac{1}{3}, \frac{2}{3}\right].
$$

Thus, if we want to approximate $f$ at $(\epsilon_1,\log(2) + \epsilon_2)$,  we see that we should have

$$
f(\epsilon_1,\log(2) + \epsilon_2) \approx \log(3) + \frac{1}{3}\epsilon_1 + \frac{2}{3}\epsilon_2.
$$

We can test this in code to see how good the approximation is.

```{.python .input}
%matplotlib inline
import d2l
from IPython import display
from mxnet import np, npx
npx.set_np()
```

```{.python .input}
f = lambda x,y : np.log(np.exp(x) + np.exp(y))
grad_f = lambda x,y : np.array([np.exp(x)/(np.exp(x)+np.exp(y)), 
                                np.exp(y)/(np.exp(x)+np.exp(y))])

epsilon = np.array([0.01,-0.03])
grad_approx = f(0,np.log(2)) + epsilon.dot(grad_f(0,np.log(2)))
true_value = f(0+epsilon[0],np.log(2)+epsilon[1])
"Approximation: {}, True Value: {}".format(grad_approx, true_value)
```

## Geometry of gradients and gradient descent
Consider the formula 

$$
L(\mathbf{w} + \boldsymbol{\epsilon}) \approx L(\mathbf{w}) + \boldsymbol{\epsilon}\cdot \nabla_{\mathbf{w}} L(\mathbf{w}).
$$

Let us suppose I want to minimize this loss.  Let's understand geometrically the algorithm of gradient descent first obtained in  :numref:`sec_autograd`. What we will do is the following:

1. Start with a random choice for the initial parameters $\mathbf{w}$.
2. Find the direction $\mathbf{v}$ that makes $L$ decrease the most rapidly at $\mathbf{w}$.
3. Take a small step in that direction: $\mathbf{w} \rightarrow \mathbf{w} + \epsilon\mathbf{v}$.
4. Repeat.

The only thing we do not know exactly how to do is to compute the vector $\mathbf{v}$ in the second step.  We will call such a direction the *direction of steepest descent*.  Let us look at our approximation.   Using the geometric understanding of dot products from :numref:`sec_linear_algebra`, we see that

$$
L(\mathbf{w} + \mathbf{v}) \approx L(\mathbf{w}) + \mathbf{v}\cdot \nabla_{\mathbf{w}} L(\mathbf{w}) = \|\nabla_{\mathbf{w}} L(\mathbf{w})\|\cos(\theta),
$$

where we have taken our direction to have length one for convenience, and used $\theta$ for the angle between $\mathbf{v}$ and $\nabla_{\mathbf{w}} L(\mathbf{w})$.  If we want to find the direction that decreases $L$ as rapidly as possible, we want to make this as expression as negative as possible.  The only way the direction we pick enters into this equation is through $\cos(\theta)$, and thus we wish to make this cosine as negative as possible.  Now, recalling the shape of cosine, we can make this as negative as possible by making $\cos(\theta) = -1$ or equivalently making the angle between the gradient and our chosen direction to be $\pi$ radians, or equivalently $180$ degrees.  The only way to make this angle so, is to head in the exact opposite direction:  pick $\mathbf{v}$ to point in the exact opposite direction to $\nabla_{\mathbf{w}} L(\mathbf{w})$!

This brings us to one of the most important mathematical concepts in machine learning: the direction of steepest decent points in the direction of $-\nabla_{\mathbf{w}}L(\mathbf{w})$.  Thus our informal algorithm can be rewritten as follows.

1. Start with a random choice for the initial parameters $\mathbf{w}$.
2. Compute $\nabla_{\mathbf{w}} L(\mathbf{w})$.
3. Take a small step in the opposite of that direction: $\mathbf{w} \rightarrow \mathbf{w} - \epsilon\nabla_{\mathbf{w}} L(\mathbf{w})$.
4. Repeat.



This basic algorithm has been modified and adapted many ways by many researchers , but the core concept remains the same in all of them.  Use the gradient to find the direction that decreases the loss as rapidly as possible, and update the parameters to take a step in that direction.

## A note on mathematical optimization
Throughout this book, we focus squarely on numerical optimization techniques for the practical reason that all functions we encounter in the deep learning setting are too complex to minimize explicitly.  

However, it is a useful exercise to consider what the geometric understanding we obtained above tells us about optimizing functions directly.

Suppose we wish to find the value of $\mathbf{x}_0$ which minimizes some function $L(\mathbf{x})$.  Let us suppose moreover that someone gives us a value and tells us that it is the value that minimizes $L$.  Is there anything we can check to see if their answer is even plausible?

Consider the expression
$$
L(\mathbf{x}_0 + \boldsymbol{\epsilon}) \approx L(\mathbf{x}_0) + \boldsymbol{\epsilon}\cdot \nabla_{\mathbf{x}} L(\mathbf{x}_0).
$$

If the gradient is not zero, we know that we can take $\boldsymbol{\epsilon} = -\epsilon \nabla_{\mathbf{x}} L(\mathbf{x}_0)$ to find a value of $L$ that is smaller.  Thus, if we truly are at a minimum, this cannot be the case!  We can conclude that if $\mathbf{x}_0$ is a minimum, then $\nabla_{\mathbf{x}} L(\mathbf{x}_0 = 0$.  We call points with $\nabla_{\mathbf{x}} L(\mathbf{x}_0 = 0$ *critical points*.

This is nice, because in some settings, one explicitly find all the points where the gradient is zero, and find the one with the smallest value.  To be concrete, consider the function
$$
f(x) = 3x^4 - 4x^3 -12x^2.
$$

This function has derivative
$$
\frac{df}{dx} = 12x^3 - 12x^2 -24x = 12x(x-2)(x+1).
$$

The only possible location of minima are at $x = -1, 0, 2$, where the function takes the values $-5,0,-32$ respectively, and thus we can conclude that we minimize our function when $x = 2$.  A quick plot confirms this.

```{.python .input}
x = np.arange(-2,3,0.01)
f = 3*x**4 - 4*x**3 -12*x**2

d2l.plot(x,f, 'x', 'f(x)')
```

This highlights an important fact to know when working either theoretically or numerically: the only possible points where we can minimize (or maximize) a function will have gradient equal to zero, however, not every point with gradient zero is the minimum (or maximum).  

## Multivariate Chain rule
Let us suppose we have a function of four variables ($w,x,y,z$) which we can make by composing many terms:

$$
\begin{aligned}
f(u,v) & = (u+v)^{2} \\
u(a,b) & = (a+b)^{2}, \qquad v(a,b) = (a-b)^{2}, \\
a(w,x,y,z) & = (w+x+y+z)^{2},\qquad b(w,x,y,z) = (w+x-y-z)^2.
\end{aligned}
$$

Such chains of equations are common when working with neural networks, so trying to understand how to compute gradients of such functions is key to advanced techniques in machine learning.  We can start to see visually hints of this connection if we take a look at what variables directly relate to one another.

![The function relations above where nodes represent values and edges show functional dependence.](../img/ChainNet1.svg)

Nothing stops us from just composing everything and writing out that

$$
f(w,x,y,z) = (((w+x+y+z)^2+(w+x-y-z)^2)^2+((w+x+y+z)^2-(w+x-y-z)^2)^2)^2,
$$

and then taking the derivative by just using good old single variable derivatives, but if we did that we would quickly find ourself swamped with terms, many of which are repeats!  Indeed, one can see that, for instance:

$$
\begin{aligned}
\frac{\partial f}{\partial w} & = 2 (2 (2 (w + x + y + z) - 2 (w + x - y - z)) ((w + x + y + z)^{2}- (w + x - y - z)^{2}) + \\
& \quad 2 (2 (w + x - y - z) + 2 (w + x + y + z)) ((w + x - y - z)^{2}+ (w + x + y + z)^{2})) \times \\
& \quad (((w + x + y + z)^{2}- (w + x - y - z)^2)^{2}+ ((w + x - y - z)^{2}+ (w + x + y + z)^{2})^{2}).
\end{aligned}
$$

If we then also wanted to compute $\frac{\partial f}{\partial x}$, we would end up with a similar equation again with many repeated terms, and many *shared* repeated terms between the two derivatives.  This represents a massive quantity of wasted work, and if we needed to compute derivatives this way, the whole deep learning revolution would have stalled out before it began!


Let us start by trying to understand how $f$ changes when we change $a$---essentially assuming that $w,x,y,z$ all do not exist.  We will reason as we did back when we worked with the gradient for the first time.  Let us take $a$ and add a small amount $\epsilon$ to it. 

$$
\begin{aligned}
f(u(a+\epsilon,b),v(a+\epsilon,b)) & \approx f\left(u(a,b) + \epsilon\frac{\partial u}{\partial a}(a,b), v(a,b) + \epsilon\frac{\partial v}{\partial a}(a,b)\right) \\
& \approx f(u(a,b),v(a,b)) + \epsilon\left[\frac{\partial f}{\partial u}(u(a,b),v(a,b))\frac{\partial u}{\partial a}(a,b) + \frac{\partial f}{\partial v}(u(a,b),v(a,b))\frac{\partial v}{\partial a}(a,b)\right].
\end{aligned}
$$

The first line follows from the definition of partial derivative, and the second follows from the definition of gradient.  It is notationally burdensome to track exactly where we evaluate every derivative, as in the expression $\frac{\partial f}{\partial u}(u(a,b),v(a,b))$, so we often abbreviate this to the much more memorable

$$
\frac{\partial f}{\partial a} = \frac{\partial f}{\partial u}\frac{\partial u}{\partial a}+\frac{\partial f}{\partial v}\frac{\partial v}{\partial a}.
$$

It is useful to think about the meaning of the process. We are trying to understand how a function of the form $f(u(a,b),v(a,b))$ changes its value with a change in $a$.  There are two pathways this can occur: there is the pathway where $a \rightarrow u \rightarrow f$ and where $a \rightarrow v \rightarrow f$.  We can compute both of these contributions via the chain rule: $\frac{\partial w}{\partial u} \cdot \frac{\partial u}{\partial x}$ and $\frac{\partial w}{\partial v} \cdot \frac{\partial v}{\partial x}$ respectively, and added up.  

Imagine we have a different network of functions where the functions on the right depend on those they are connected to on the left.

![Another more subtle example of the chain rule.](../img/ChainNet2.svg)

To compute something like $\frac{\partial f}{\partial y}$, we need to sum over all (in this case $3$) paths from $y$ to $f$ giving

$$
\frac{\partial f}{\partial y} = \frac{\partial f}{\partial a} \frac{\partial a}{\partial u} \frac{\partial u}{\partial y} + \frac{\partial f}{\partial u} \frac{\partial u}{\partial y} + \frac{\partial f}{\partial b} \frac{\partial b}{\partial v} \frac{\partial v}{\partial y}.
$$

Understanding the chain rule in this way will pay great dividends when trying to understand how gradients flow through networks, and why various architectural choices like those in LSTMs (:numref:`sec_lstm`) or residual layers (:numref:`sec_resnet`) can help shape the learning process by controlling gradient flow.

## The Backpropagation Algorithm

Let us return to the problem we saw in the previous section where

$$
\begin{aligned}
f(u,v) & = (u+v)^{2} \\
u(a,b) & = (a+b)^{2}, \qquad v(a,b) = (a-b)^{2}, \\
a(w,x,y,z) & = (w+x+y+z)^{2},\qquad b(w,x,y,z) = (w+x-y-z)^2.
\end{aligned}
$$

If we want to compute say $\frac{\partial f}{\partial w}$ we may apply the multi-variate chain rule to see:

$$
\begin{aligned}
\frac{\partial f}{\partial w} & = \frac{\partial f}{\partial u}\frac{\partial u}{\partial w} + \frac{\partial f}{\partial v}\frac{\partial v}{\partial w}, \\
\frac{\partial u}{\partial w} & = \frac{\partial u}{\partial a}\frac{\partial a}{\partial w}+\frac{\partial u}{\partial b}\frac{\partial b}{\partial w}, \\
\frac{\partial v}{\partial w} & = \frac{\partial v}{\partial a}\frac{\partial a}{\partial w}+\frac{\partial v}{\partial b}\frac{\partial b}{\partial w}.
\end{aligned}
$$

Let us try using this decomposition to compute $\frac{\partial f}{\partial w}$.  Notice that all we need here are the various single step partials:

$$
\begin{aligned}
\frac{\partial f}{\partial u} & = 2(u+v),\\
\frac{\partial f}{\partial v} & = 2(u+v),\\
\frac{\partial u}{\partial a} & = 2(a+b),\\
\frac{\partial u}{\partial b} & = 2(a+b),\\
\frac{\partial v}{\partial a} & = 2(a-b),\\
\frac{\partial v}{\partial b} & = -2(a-b),\\
\frac{\partial a}{\partial w} & = 2(w+x+y+z),\\
\frac{\partial b}{\partial w} & = 2(w+x-y-z).
\end{aligned}
$$

If we write this out into code this becomes a fairly manageable expression.

```{.python .input}
### Compute the value of the function from inputs to outputs ###
w = -1; x = 0; y = -2; z = 1
a = (w+x+y+z)**2; b = (w+x-y-z)**2
u = (a+b)**2; v = (a-b)**2
f = (u+v)**2
print("    f at {},{},{},{} is {}".format(w,x,y,z,f))

### Compute the derivative using the decomposition above ###
# First compute the single step partials
df_du = 2*(u+v); df_dv = 2*(u+v)
du_da = 2*(a+b); du_db = 2*(a+b); dv_da = 2*(a-b); dv_db = -2*(a-b)
da_dw = 2*(w+x+y+z); db_dw = 2*(w+x-y-z)

### Now compute the final result from inputs to outputs ###
du_dw = du_da*da_dw + du_db*db_dw; dv_dw = dv_da*da_dw + dv_db*db_dw
df_dw = df_du*du_dw + df_dv*dv_dw
print("df/dw at {},{},{},{} is {}".format(w,x,y,z,df_dw))
```

However, note that this still does not make it easy to compute something like $\frac{\partial f}{\partial x}$.  The reason for that is the *way* we chose to apply the chain rule.  If we look at what we did above, we always kept $\partial w$ in the denominator when we could.  In this way, we chose to apply the chain rule seeing how $w$ changed every other variable.  If that is what we wanted, this would be a good idea.  However, think back to our motivation from deep learning: we want to see how every parameter changes the *loss*.  In essence, we want to apply the chain rule keeping $\partial f$ in the numerator whenever we can!

To be more explicit, note that we can write

$$
\begin{aligned}
\frac{\partial f}{\partial w} & = \frac{\partial f}{\partial a}\frac{\partial a}{\partial w} + \frac{\partial f}{\partial b}\frac{\partial b}{\partial w}, \\
\frac{\partial f}{\partial a} & = \frac{\partial f}{\partial u}\frac{\partial u}{\partial a}+\frac{\partial f}{\partial v}\frac{\partial v}{\partial a}, \\
\frac{\partial f}{\partial b} & = \frac{\partial f}{\partial u}\frac{\partial u}{\partial b}+\frac{\partial f}{\partial v}\frac{\partial v}{\partial b}.
\end{aligned}
$$

Note that this application of the chain rule has us explicitly compute $\frac{\partial f}{\partial u}, \frac{\partial f}{\partial u}, \frac{\partial f}{\partial u}, \frac{\partial f}{\partial u}, \text{ and } \frac{\partial f}{\partial u}$.  Nothing stops us from also including the equations:

$$
\begin{aligned}
\frac{\partial f}{\partial x} & = \frac{\partial f}{\partial a}\frac{\partial a}{\partial x} + \frac{\partial f}{\partial b}\frac{\partial b}{\partial x}, \\
\frac{\partial f}{\partial y} & = \frac{\partial f}{\partial a}\frac{\partial a}{\partial y}+\frac{\partial f}{\partial b}\frac{\partial b}{\partial y}, \\
\frac{\partial f}{\partial z} & = \frac{\partial f}{\partial a}\frac{\partial a}{\partial z}+\frac{\partial f}{\partial b}\frac{\partial b}{\partial z}.
\end{aligned}
$$

and then keeping track of how $f$ changes when we change *any* node in the entire network.  Let us implement it.

```{.python .input}
### Compute the value of the function from inputs to outputs ###
w = -1; x = 0; y = -2; z = 1
a = (w+x+y+z)**2; b = (w+x-y-z)**2
u = (a+b)**2; v = (a-b)**2
f = (u+v)**2
print("    f at {},{},{},{} is {}".format(w,x,y,z,f))

### Compute the derivative using the decomposition above ###
# First compute the single step partials
df_du = 2*(u+v); df_dv = 2*(u+v)
du_da = 2*(a+b); du_db = 2*(a+b); dv_da = 2*(a-b); dv_db = -2*(a-b)
da_dw = 2*(w+x+y+z); db_dw = 2*(w+x-y-z); da_dx = 2*(w+x+y+z); db_dx = 2*(w+x-y-z)
da_dy = 2*(w+x+y+z); db_dy = -2*(w+x-y-z); da_dz = 2*(w+x+y+z); db_dz = -2*(w+x-y-z); 

### Now compute how f changes when we change any value from output to input ###
df_da = df_du*du_da + df_dv*dv_da; df_db = df_du*du_db + df_dv*dv_db
df_dw = df_da*da_dw + df_db*db_dw; df_dx = df_da*da_dx + df_db*db_dx
df_dy = df_da*da_dy + df_db*db_dy; df_dz = df_da*da_dz + df_db*db_dz
print("df/dw at {},{},{},{} is {}".format(w,x,y,z,df_dw))
print("df/dx at {},{},{},{} is {}".format(w,x,y,z,df_dx))
print("df/dy at {},{},{},{} is {}".format(w,x,y,z,df_dy))
print("df/dz at {},{},{},{} is {}".format(w,x,y,z,df_dz))
```

The fact that we compute derivatives from $f$ back towards the inputs rather than from the inputs forward to the outputs (as we did in the first code snippet above) is what gives this algorithm its name: *backpropagation*.  Note that there are two steps:
1. Compute the value of the function, and the single step partials from front to back.  While not done above, this can be combined into a single *forward pass*.
2. Compute the gradient of $f$ from back to front.  We call this the *backwards pass*.

This is precisely what every deep learning algorithm implements to allow the computation of the gradient of the loss with respect to every weight in the network at one pass.  It is an astonishing fact that we have such a decomposition.  Without it, the deep learning revolution could have never occurred.

To see how MXNet has encapsulated this, let us take a quick look at this example.

```{.python .input}
from mxnet import autograd

### Initialize as NDArrays, attaching gradients ###
w = np.array(-1); x = np.array(0); y = np.array(-2); z = np.array(1)
w.attach_grad(); x.attach_grad(); y.attach_grad(); z.attach_grad()

### Do the computation like usual, tracking gradients ###
with autograd.record():
    a = (w+x+y+z)**2; b = (w+x-y-z)**2
    u = (a+b)**2; v = (a-b)**2
    f = (u+v)**2

### Execute backward pass ###
f.backward()

print("df/dw at {},{},{},{} is {}".format(w,x,y,z,w.grad))
print("df/dx at {},{},{},{} is {}".format(w,x,y,z,x.grad))
print("df/dy at {},{},{},{} is {}".format(w,x,y,z,y.grad))
print("df/dz at {},{},{},{} is {}".format(w,x,y,z,z.grad))
```

All of what we did above can be done automatically by calling `f.backwards()`.


## Hessians
As with single variable calculus, it is useful to consider higher-order derivatives in order to get a handle on how we can obtain a better approximation to a function than using the gradient alone.

There is one immediate problem one encounters when working with higher order derivatives of functions of several variables, and that is there are a large number of them.  Indeed if we have a function $f(x_1, \ldots, x_n)$ of $n$ variables, then we can take $n^{2}$ many derivatives, namely for any choice of $i$ and $j$:

$$
\frac{d^2f}{dx_idx_j} = \frac{d}{dx_i}\left(\frac{d}{dx_j}f\right).
$$

This is traditionally assembled into a matrix called the *Hessian*:

$$
\mathbf{H}_f = \begin{bmatrix} 
\frac{d^2f}{dx_1dx_1} & \cdots & \frac{d^2f}{dx_1dx_n} \\
\vdots & \ddots & \vdots \\
\frac{d^2f}{dx_ndx_1} & \cdots & \frac{d^2f}{dx_ndx_n} \\
\end{bmatrix}.
$$

Not every entry of this matrix is independent.  Indeed, we can show that as long as both *mixed partials* (partial derivatives with respect to more than one variable) exist and are continuous, we can say that for any $i,j$, 

$$
\frac{d^2f}{dx_idx_j} = \frac{d^2f}{dx_jdx_i}.
$$

This follows by considering first perturbing a function in the direction of $x_i$, and then perturbing it in $x_j$ and then comparing the result of that with what happens if we perturb first $x_j$ and then $x_i$, with the knowledge that both of these orders lead to the same final change in the output of $f$.

As with single variables, we can use these derivatives to get a far better idea of how the function behaves near a point.  In particular, we can use it to find the best fitting quadratic near a point $\mathbf{x}_0$.

Let us see an example.  Suppose that $f(x_1,x_2) = a + b_1x_1 + b_2x_2 + c_{11}x_1^{2} + c_{12}x_1x_2 + c_{22}x_2^{2}$.  This is the general form for a quadratic in two variables.  If we look at the value of the function, its gradient, and its Hessian, all at the point zero:

$$
\begin{aligned}
f(0,0) & = a, \\
\nabla f (0,0) & = \begin{bmatrix}b_1 \\ b_2\end{bmatrix}, \\
\mathbf{H} f (0,0) & = \begin{bmatrix}2 c_{11} & c_{12} \\ c_{12} & 2c_{22}\end{bmatrix}.
\end{aligned}
$$

If we from this, we see we can get our original polynomial back by saying

$$
f(\mathbf{x}) = f(0) + \nabla f (0) \cdot \mathbf{x} + \frac{1}{2}\mathbf{x}^\top \mathbf{H} f (0) \mathbf{x}.
$$

In general, if we computed this expansion any point $\mathbf{x}_0$, we see that

$$
f(\mathbf{x}) = f(\mathbf{x}_0) + \nabla f (\mathbf{x}_0) \cdot (\mathbf{x}-\mathbf{x}_0) + \frac{1}{2}(\mathbf{x}-\mathbf{x}_0)^\top \mathbf{H} f (\mathbf{x}_0) (\mathbf{x}-\mathbf{x}_0).
$$

This works for any dimensional input, and provides the best approximating quadratic to any function at a point.  To give an example, let's plot the function 

$$
f(x,y) = xe^{-x^2-y^2}.
$$

One can compute that the gradient and Hessian are
$$
\nabla f(x,y) = e^{-x^2-y^2}\begin{pmatrix}1-2x^2 \\ -2xy\end{pmatrix} \text{ and } \mathbf{H}f(x,y) = e^{-x^2-y^2}\begin{pmatrix} 4x^3 - 6x & 4x^2y - 2y \\ 4x^2y-2y &4xy^2-2x\end{pmatrix}
$$

And thus, with a little algebra, see that the approximating quadratic at $[-1,0]^\top$ is

$$
f(x,y) \approx e^{-1}(-1 - (x+1) +2(x+1)^2+2y^2).
$$


```{.python .input}
from mpl_toolkits import mplot3d

# Construct grid and compute function
x, y = np.meshgrid(np.linspace(-2, 2, 101), np.linspace(-2, 2, 101), indexing='ij')
z = x*np.exp(- x**2 - y**2)

# Compute gradient and Hessian at (1,0)
z_quad = np.exp(-1)*(-1 - (x+1) + 2*(x+1)**2 + 2*y**2)

# Plot Function
ax = d2l.plt.figure().add_subplot(111, projection='3d')
ax.plot_wireframe(x, y, z, **{'rstride': 10, 'cstride': 10})
ax.plot_wireframe(x, y, z_quad, **{'rstride': 10, 'cstride': 10}, color = 'purple')
d2l.plt.xlabel('x')
d2l.plt.ylabel('y')
ax.set_xlim(-2, 2); ax.set_ylim(-2, 2); ax.set_zlim(-1, 1);
```

This forms the basis for Newton's Algorithm discussed in :numref:`sec_gd`, where we perform numerical optimization iteratively finding the best fitting quadratic, and then exactly minimizing that quadratic.

## A Little Matrix Calculus
Derivatives of functions involving matrices turn out to be particularly nice.  This section can become notationally heavy, so may be skipped in a first reading, but it is useful to know how derivatives of functions involving common matrix operations are often much cleaner than one might initially anticipate, particularly given how central matrix operations are to deep learning applications.

Let us begin with an example.  Suppose we have some fixed row vector $\boldsymbol{\beta}$, and we want to take the product function $f(\mathbf{x}) = \boldsymbol{\beta}\mathbf{x}$, and understand how the dot product changes when we change $\mathbf{x}$.  A bit of notation that will be useful when working with matrix derivatives in ML is called the *denominator layout matrix derivative* where we assemble our partial derivatives into the shape of whatever vector, matrix, or tensor is in the denominator of the differential.  In this case, we will write

$$
\frac{df}{d\mathbf{x}} = \begin{bmatrix}
\frac{df}{dx_1} \\
\vdots \\
\frac{df}{dx_n}
\end{bmatrix}.
$$

where we matched the shape of the column vector $\mathbf{x}$. 

If we write out our function into components this is

$$
f(\mathbf{x}) = \sum_{i = 1}^{n} \beta_ix_i = \beta_1x_1 + \cdots + \beta_nx_n.
$$

If we now take the partial derivative with respect to say $\beta_1$, note that everything is zero but the first term, which is just $x_1$ multiplied by $\beta_1$, so the we obtain that

$$
\frac{df}{dx_1} = \beta_1,
$$

or more generally that 

$$
\frac{df}{dx_i} = \beta_i.
$$

We can now reassemble this into a matrix to see

$$
\frac{df}{d\mathbf{x}} = \begin{bmatrix}
\frac{df}{dx_1} \\
\vdots \\
\frac{df}{dx_n}
\end{bmatrix} = \begin{bmatrix}
\beta_1 \\
\vdots \\
\beta_n
\end{bmatrix} = \boldsymbol{\beta}^\top.
$$

This illustrates a few factors about matrix calculus that we will often counter throughout this section:

* First, The computations will get rather involved.
* Second, The final results are much cleaner than the intermediate process, and will always look similar to the single variable case.  In this case, note that $\frac{d}{dx}(bx) = b$ and $\frac{d}{d\mathbf{x}} (\boldsymbol{\beta}\mathbf{x}) = \boldsymbol{\beta}^\top$ are both similar. 
* Third, transposes can often appear seemingly from nowhere.  The core reason for this is the convention that we match the shape of the denominator, thus when we multiply matrices, we will need to take transposes to match back to the shape of the original term.

To keep building intuition, let us try a computation that is a little harder.  Suppose we have a column vector $\mathbf{x}$, and a square matrix $A$ and we want to compute 

$$
\frac{d}{d\mathbf{x}}(\mathbf{x}^\top A \mathbf{x}).
$$

To drive towards easier to manipulate notation, let us consider this problem using Einstein notation.  In this case we can write the function as

$$
\mathbf{x}^\top A \mathbf{x} = x_ia_{ij}x_j.
$$

To compute our derivative, we need to understand for every $k$, what the value of

$$
\frac{d}{dx_k}(\mathbf{x}^\top A \mathbf{x}) = \frac{d}{dx_k}x_ia_{ij}x_j.
$$

By the product rule, this is

$$
\frac{d}{dx_k}x_ia_{ij}x_j = \frac{dx_i}{dx_k}a_{ij}x_j + x_ia_{ij}\frac{dx_j}{dx_k}.
$$

For a term like $\frac{dx_i}{dx_k}$, it is not hard to see that this is one when $i=k$ and zero otherwise.  This means that every term where $i$ and $k$ are different vanish from this sum, so the only terms that remain in that first sum are the ones where $i=k$.  The same reasoning holds for the second term where we need $j=k$.  This gives

$$
\frac{d}{dx_k}x_ia_{ij}x_j = a_{kj}x_j + x_ia_{ik}.
$$

Now, the names of the indices in Einstein notation are arbitrary---the fact that $i$ and $j$ are different is immaterial to this computation at this point, so we can re-index so that they both use $i$ to see that

$$
\frac{d}{dx_k}x_ia_{ij}x_j = a_{ki}x_i + x_ia_{ik} = (a_{ki} + a_{ik})x_i.
$$

Now, here is where we start to need some practice to go further.  Let us try and identify this outcome in terms of matrix operations.  $a_{ki} + a_{ik}$ is the $k,i$-th component of $\mathbf{A} + \mathbf{A}^\top$.  This gives

$$
\frac{d}{dx_k}x_ia_{ij}x_j = [\mathbf{A} + \mathbf{A}^\top]_{ki}x_i.
$$

Similarly, this term is now the product of the matrix $\mathbf{A} + \mathbf{A}^\top$ by the vector $\mathbf{x}$, so we see that

$$
\left[\frac{d}{d\mathbf{x}}(\mathbf{x}^\top A \mathbf{x})\right]_k = \frac{d}{dx_k}x_ia_{ij}x_j = [(\mathbf{A} + \mathbf{A}^\top)\mathbf{x}]_k.
$$

Thus, we see that the $k$-th entry of the desired derivative is just the $k$-th entry of the vector on the right, and thus the two are the same.  Thus yields

$$
\frac{d}{d\mathbf{x}}(\mathbf{x}^\top A \mathbf{x}) = (\mathbf{A} + \mathbf{A}^\top)\mathbf{x}.
$$

This required significantly more work than our last one, but the final result is small.  More than that, consider the following computation for traditional single variable derivatives:

$$
\frac{d}{dx}(xax) = \frac{dx}{dx}ax + xa\frac{dx}{dx} = (a+a)x.
$$

Equivalently $\frac{d}{dx}(ax^2) = 2ax$.  Again, we get a result that looks suspiciously like the single variable result but with a transpose tossed in.  

At this point, the pattern should be looking rather suspicious, so let us try to figure out why.  When we take matrix derivatives like this, let us first assume that the expression we get will be another matrix expression---by which I mean we can write it in terms of products and sums of matrices and their transposes.  If such an expression exists, it will need to be true for all matrices.  In particular, it will need to be true of $1 \times 1$ matrices, in which case the matrix product is just the of the numbers, the matrix sum is just the sum, and the transpose does nothing at all!  In other words, whatever expression we get *must* match the single variable expression.  This means that, with some practice, one can often guess matrix derivatives just by knowing what the associated single variable expression must look like!

Let us try this out.  Suppose $\mathbf{X}$ is a $n \times m$ matrix, $\mathbf{U}$ is an $n \times r$ and $\mathbf{V}$ is an $r \times m$.  Let us try to compute 

$$
\frac{d}{d\mathbf{V}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2} = \;?
$$

This computation is important in an area called matrix factorization.  For us, however, it is just a derivative to compute.  Let us try to imaging what this would be for $1\times1$ matrices.  In that case, we get the expression

$$ 
\frac{d}{dv} (x-uv)^{2}= 2(x-uv)u,
$$

where, the derivative is rather standard.  If we try to convert this back into a matrix expression we get

$$
\frac{d}{d\mathbf{V}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= 2(\mathbf{X} - \mathbf{U}\mathbf{V})\mathbf{U}.
$$

However, if we look at this it does not quite work.  Recall that $\mathbf{X}$ is $n \times m$, as is $\mathbf{U}\mathbf{V}$, so the matrix $2(\mathbf{X} - \mathbf{U}\mathbf{V})$ is $n \times m$.  On the other hand $\mathbf{U}$ is $n \times r$, and we cannot multiply a $n \times m$ and a $n \times r$ matrix since the dimensions do not match! 

We want to get $\frac{d}{d\mathbf{V}}$, which is the same shape of $\mathbf{V}$, which is $r \times m$.  So somehow we need to take a $n \times m$ matrix and a $n \times r$ matrix, multiply them together (perhaps with some transposes) to get a $r \times m$. We can do this by multiplying $U^\top$ by $(\mathbf{X} - \mathbf{U}\mathbf{V})$.  Thus, we can guess

$$
\frac{d}{d\mathbf{V}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= 2\mathbf{U}^\top(\mathbf{X} - \mathbf{U}\mathbf{V}).
$$

To show we that this works, I would be remiss to not provide a detailed computation.  If we already believe that this rule-of-thumb works, feel free to skip past this derivation.  To compute 

$$
\frac{d}{d\mathbf{V}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^2,
$$

we must find for every $a,b$

$$
\frac{d}{dv_{ab}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= \frac{d}{dv_{ab}} \sum_{i,j}(x_{ij} - \sum_k u_{ik}v_{kj})^2.
$$

Recalling that all entries of $\mathbf{X}$ and $\mathbf{U}$ are constants as far as $\frac{d}{dv_{ab}}$ is concerned, we may push the derivative inside the sum, and apply the chain rule to the square to get

$$
\frac{d}{dv_{ab}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= \sum_{i,j}2\left(x_{ij} - \sum_k u_{ik}v_{kj}\right)\left(\sum_k u_{ik}\frac{dv_{kj}}{dv_{ab}} \right).
$$

As in the previous derivation, we may note that $\frac{dv_{kj}}{dv_{ab}}$ is only non-zero if the $k=a$ and $j=b$.  If either of those conditions do not hold, the term in the sum is zero, and we may freely discard it.  We see that

$$
\frac{d}{dv_{ab}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= \sum_{i}2\left(x_{ib} - \sum_k u_{ik}v_{kb}\right)u_{ia}.
$$

An important subtlety here is that the requirement that $k=a$ does not occur inside the inner sum since that $k$ is a dummy variable which we are summing over inside the inner term.  For a notationally cleaner example, consider why

$$
\frac{d}{dx_1} \left(\sum_i x_i \right)^{2}= 2\left(\sum_i x_i \right).
$$

From this point, we may start identifying components of the sum.  First, 

$$
\sum_k u_{ik}v_{kb} = [\mathbf{U}\mathbf{V}]_{ib}.
$$

So the entire expression in the inside of the sum is

$$
\left(x_{ib} - \sum_k u_{ik}v_{kb}\right) = [\mathbf{X}-\mathbf{U}\mathbf{V}]_{ib}.
$$

This means we may now write our derivative as

$$
\frac{d}{dv_{ab}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= 2\sum_{i}[\mathbf{X}-\mathbf{U}\mathbf{V}]_{ib}u_{ia}.
$$

We want this to look like the $a,b$ element of a matrix so we can use the technique as in the previous example to arrive at a matrix expression, which means that we need to exchange the order of the indices on $u_{ia}$.  If we notice that $u_{ia} = [\mathbf{U}^\top]_{ai}$, we can then write

$$
\frac{d}{dv_{ab}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= 2\sum_{i} [\mathbf{U}^\top]_{ai}[\mathbf{X}-\mathbf{U}\mathbf{V}]_{ib}.
$$

This is a matrix product, and thus we can conclude that

$$
\frac{d}{dv_{ab}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= [2\mathbf{U}^\top(\mathbf{X}-\mathbf{U}\mathbf{V})]_{ab}.
$$

and thus

$$
\frac{d}{d\mathbf{V}} \|\mathbf{X} - \mathbf{U}\mathbf{V}\|_2^{2}= 2\mathbf{U}^\top(\mathbf{X} - \mathbf{U}\mathbf{V}).
$$

This matches the solution we guessed above!

It is reasonable to ask at this point, "Why can I not just write down matrix versions of all the calculus rules I have learned?  It is clear this is still mechanical.  Why do we not just get it over with!"  And indeed there are such rules, and [The Matrix Cookbook](https://www.math.uwaterloo.ca/~hwolkowi/matrixcookbook.pdf) provides an excellent summary.  However, due to the plethora of ways matrix operations can be combined compared to single values, there are many more matrix derivative rules than single variable ones.  It is often the case that it is best to work with the indices, or leave it up to automatic differentiation when appropriate.

## Summary

* In higher dimensions, we can define gradients which serve the same purpose as derivatives in one dimension.  These allow us to see how a multi-variable function changes when we make an arbitrary small change to the inputs.
* The backpropagation algorithm can be seen to be a method of organizing the multi-variable chain rule to allow for the efficient computation of many partial derivatives.
* Matrix calculus allows us to write the derivatives of matrix expressions in concise ways.

## Exercises
1. Given a row vector $\boldsymbol{\beta}$, compute the derivatives of both $f(\mathbf{x}) = \boldsymbol{\beta}\mathbf{x}$ and $g(\mathbf{x}) = \mathbf{x}^\top\boldsymbol{\beta}^\top$.  Why do you get the same answer?
2. Let $\mathbf{v}$ be an $n$ dimension vector. What is $\frac{\partial}{\partial\mathbf{v}}\|\mathbf{v}\|_2$?
3. Let $L(x,y) = \log(e^x + e^y)$.  Compute the gradient.  What is the sum of the components of the gradient?
4. Let $f(x,y) = x^2y + xy^2$. Show that the only critical point is $(0,0)$. By considering $f(x,x)$, determine if $(0,0)$ is a maximum, minimum, or neither.
5. Suppose we are minimizing a function $f(\mathbf{x}) = g(\mathbf{x}) + h(\mathbf{x})$.  How can we geometrically interpret the condition of $\nabla f = 0$ in terms of $g$ and $h$?