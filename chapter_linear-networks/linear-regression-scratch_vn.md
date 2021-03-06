<!-- ===================== Bắt đầu dịch Phần 1 ===================== -->
<!-- ========================================= REVISE PHẦN 1 - BẮT ĐẦU =================================== -->

<!--
# Linear Regression Implementation from Scratch
-->

# *dịch tiêu đề phía trên*
:label:`sec_linear_scratch`

<!--
Now that you understand the key ideas behind linear regression, we can begin to work through a hands-on implementation in code.
In this section, we will implement the entire method from scratch, including the data pipeline, the model, the loss function, and the gradient descent optimizer.
While modern deep learning frameworks can automate nearly all of this work, implementing things from scratch is the only to make sure that you really know what you are doing.
Moreover, when it comes time to customize models, defining our own layers, loss functions, etc., understanding how things work under the hood will prove handy.
In this section, we will rely only on `ndarray` and `autograd`.
Afterwards, we will introduce a more compact implementation, taking advantage of Gluon's bells and whistles.
To start off, we import the few required packages.
-->

*dịch đoạn phía trên*

```{.python .input  n=1}
%matplotlib inline
import d2l
from mxnet import autograd, np, npx
import random
npx.set_np()
```

<!--
## Generating the Dataset
-->

## *dịch tiêu đề phía trên*

<!--
To keep things simple, we will construct an artificial dataset according to a linear model with additive noise.
Out task will be to recover this model's parameters using the finite set of examples contained in our dataset.
We will keep the data low-dimensional so we can visualize it easily.
In the following code snippet, we generated a dataset containing $1000$ examples, each consisting of $2$ features sampled from a standard normal distribution.
Thus our synthetic dataset will be an object $\mathbf{X}\in \mathbb{R}^{1000 \times 2}$.
-->

*dịch đoạn phía trên*

<!--
The true parameters generating our data will be $\mathbf{w} = [2, -3.4]^\top$ and $b = 4.2$
and our synthetic labels will be assigned according to the following linear model with noise term $\epsilon$:
-->

*dịch đoạn phía trên*

$$\mathbf{y}= \mathbf{X} \mathbf{w} + b + \mathbf\epsilon.$$

<!-- ===================== Kết thúc dịch Phần 1 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 2 ===================== -->

<!--
You could think of $\epsilon$ as capturing potential measurement errors on the features and labels.
We will assume that the standard assumptions hold and thus that $\epsilon$ obeys a normal distribution with mean of $0$.
To make our problem easy, we will set its standard deviation to $0.01$.
The following code generates our synthetic dataset:
-->

*dịch đoạn phía trên*

```{.python .input  n=2}
# Saved in the d2l package for later use
def synthetic_data(w, b, num_examples):
    """Generate y = X w + b + noise."""
    X = np.random.normal(0, 1, (num_examples, len(w)))
    y = np.dot(X, w) + b
    y += np.random.normal(0, 0.01, y.shape)
    return X, y

true_w = np.array([2, -3.4])
true_b = 4.2
features, labels = synthetic_data(true_w, true_b, 1000)
```

<!--
Note that each row in `features` consists of a 2-dimensional data point and that each row in `labels` consists of a 1-dimensional target value (a scalar).
-->

*dịch đoạn phía trên*

```{.python .input  n=3}
print('features:', features[0],'\nlabel:', labels[0])
```

<!--
By generating a scatter plot using the second `features[:, 1]` and `labels`, we can clearly observe the linear correlation between the two.
-->

*dịch đoạn phía trên*

```{.python .input  n=18}
d2l.set_figsize((3.5, 2.5))
d2l.plt.scatter(features[:, 1].asnumpy(), labels.asnumpy(), 1);
```

<!-- ===================== Kết thúc dịch Phần 2 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 3 ===================== -->

<!-- ========================================= REVISE PHẦN 1 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 2 - BẮT ĐẦU ===================================-->

<!--
## Reading the Dataset
-->

## *dịch tiêu đề phía trên*

<!--
Recall that training models consists of making multiple passes over the dataset, grabbing one minibatch of examples at a time, and using them to update our model.
Since this process is so fundamental to training machine learning algorithms, its worth defining a utility function to shuffle the data and access it in minibatches.
-->

*dịch đoạn phía trên*

<!--
In the following code, we define a `data_iter` function to demonstrate one possible implementation of this functionality.
The function takes a batch size, a design matrix, and a vector of labels, yielding minibatches of size `batch_size`.
Each minibatch consists of an tuple of features and labels.
-->

*dịch đoạn phía trên*

```{.python .input  n=5}
def data_iter(batch_size, features, labels):
    num_examples = len(features)
    indices = list(range(num_examples))
    # The examples are read at random, in no particular order
    random.shuffle(indices)
    for i in range(0, num_examples, batch_size):
        batch_indices = np.array(
            indices[i: min(i + batch_size, num_examples)])
        yield features[batch_indices], labels[batch_indices]
```

<!--
In general, note that we want to use reasonably sized minibatches to take advantage of the GPU hardware,
which excels at parallelizing operations.
Because each example can be fed through our models in parallel and the gradient of the loss function for each example can also be taken in parallel,
GPUs allow us to process hundreds of examples in scarcely more time than it might take to process just a single example.
-->

*dịch đoạn phía trên*

<!--
To build some intuition, let's read and print the first small batch of data examples.
The shape of the features in each minibatch tells us both the minibatch size and the number of input features.
Likewise, our minibatch of labels will have a shape given by `batch_size`.
-->

*dịch đoạn phía trên*

```{.python .input  n=6}
batch_size = 10

for X, y in data_iter(batch_size, features, labels):
    print(X, '\n', y)
    break
```

<!-- ===================== Kết thúc dịch Phần 3 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 4 ===================== -->

<!--
As we run the iterator, we obtain distinct minibatches successively until all the data has been exhausted (try this).
While the iterator implemented above is good for didactic purposes, it is inefficient in ways that might get us in trouble on real problems.
For example, it requires that we load all data in memory and that we perform lots of random memory access.
The built-in iterators implemented in Apache MXNet are considerably efficient and they can deal both with data stored on file and data fed via a data stream.
-->

*dịch đoạn phía trên*

<!-- ========================================= REVISE PHẦN 2 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 3 - BẮT ĐẦU ===================================-->

<!--
## Initializing Model Parameters
-->

## *dịch tiêu đề phía trên*

<!--
Before we can begin optimizing our model's parameters by gradient descent, we need to have some parameters in the first place.
In the following code, we initialize weights by sampling random numbers from a normal distribution with mean 0 and a standard deviation of $0.01$, setting the bias $b$ to $0$.
-->

*dịch đoạn phía trên*

```{.python .input  n=7}
w = np.random.normal(0, 0.01, (2, 1))
b = np.zeros(1)
```

<!--
Now that we have initialized our parameters, our next task is to update them until they fit our data sufficiently well.
Each update requires taking the gradient (a multi-dimensional derivative) of our loss function with respect to the parameters.
Given this gradient, we can update each parameter in the direction that reduces the loss.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 4 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 5 ===================== -->

<!--
Since nobody wants to compute gradients explicitly (this is tedious and error prone), we use automatic differentiation to compute the gradient.
See :numref:`sec_autograd` for more details.
Recall from the autograd chapter that in order for `autograd` to know that it should store a gradient for our parameters,
we need to invoke the `attach_grad` function, allocating memory to store the gradients that we plan to take.
-->

*dịch đoạn phía trên*

```{.python .input  n=8}
w.attach_grad()
b.attach_grad()
```

<!--
## Defining the Model
-->

## *dịch tiêu đề phía trên*

<!--
Next, we must define our model, relating its inputs and parameters to its outputs.
Recall that to calculate the output of the linear model, we simply take the matrix-vector dot product of the examples $\mathbf{X}$ and the models weights $w$, and add the offset $b$ to each example.
Note that below `np.dot(X, w)` is a vector and `b` is a scalar.
Recall that when we add a vector and a scalar, the scalar is added to each component of the vector.
-->

*dịch đoạn phía trên*

```{.python .input  n=9}
# Saved in the d2l package for later use
def linreg(X, w, b):
    return np.dot(X, w) + b
```

<!-- ===================== Kết thúc dịch Phần 5 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 6 ===================== -->

<!-- ========================================= REVISE PHẦN 3 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 4 - BẮT ĐẦU ===================================-->

<!--
## Defining the Loss Function
-->

## *dịch tiêu đề phía trên*

<!--
Since updating our model requires taking the gradient of our loss function, we ought to define the loss function first.
Here we will use the squared loss function as described in the previous section.
In the implementation, we need to transform the true value `y` into the predicted value's shape `y_hat`.
The result returned by the following function will also be the same as the `y_hat` shape.
-->

*dịch đoạn phía trên*

```{.python .input  n=10}
# Saved in the d2l package for later use
def squared_loss(y_hat, y):
    return (y_hat - y.reshape(y_hat.shape)) ** 2 / 2
```

<!--
## Defining the Optimization Algorithm
-->

## *dịch tiêu đề phía trên*

<!--
As we discussed in the previous section, linear regression has a closed-form solution.
However, this is not a book about linear regression, it is a book about deep learning.
Since none of the other models that this book introduces
can be solved analytically, we will take this opportunity to introduce your first working example of stochastic gradient descent (SGD).
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 6 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 7 ===================== -->

<!--
At each step, using one batch randomly drawn from our dataset, we will estimate the gradient of the loss with respect to our parameters.
Next, we will update our parameters (a small amount) in the direction that reduces the loss.
Recall from :numref:`sec_autograd` that after we call `backward` each parameter (`param`) will have its gradient stored in `param.grad`.
The following code applies the SGD update, given a set of parameters, a learning rate, and a batch size.
The size of the update step is determined by the learning rate `lr`.
Because our loss is calculated as a sum over the batch of examples, we normalize our step size by the batch size (`batch_size`), 
so that the magnitude of a typical step size does not depend heavily on our choice of the batch size.
-->

*dịch đoạn phía trên*

```{.python .input  n=11}
# Saved in the d2l package for later use
def sgd(params, lr, batch_size):
    for param in params:
        param[:] = param - lr * param.grad / batch_size
```

<!-- ========================================= REVISE PHẦN 4 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 5 - BẮT ĐẦU ===================================-->

<!--
## Training
-->

## *dịch tiêu đề phía trên*

<!--
Now that we have all of the parts in place, we are ready to implement the main training loop.
It is crucial that you understand this code because you will see nearly identical training loops over and over again throughout your career in deep learning.
-->

*dịch đoạn phía trên*

<!--
In each iteration, we will grab minibatches of models, first passing them through our model to obtain a set of predictions.
After calculating the loss, we call the `backward` function to initiate the backwards pass through the network, 
storing the gradients with respect to each parameter in its corresponding `.grad` attribute.
Finally, we will call the optimization algorithm `sgd` to update the model parameters.
Since we previously set the batch size `batch_size` to $10$, the loss shape `l` for each minibatch is ($10$, $1$).
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 7 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 8 ===================== -->

<!--
In summary, we will execute the following loop:
-->

*dịch đoạn phía trên*

<!--
* Initialize parameters $(\mathbf{w}, b)$
* Repeat until done
    * Compute gradient $\mathbf{g} \leftarrow \partial_{(\mathbf{w},b)} \frac{1}{\mathcal{B}} \sum_{i \in \mathcal{B}} l(\mathbf{x}^i, y^i, \mathbf{w}, b)$
    * Update parameters $(\mathbf{w}, b) \leftarrow (\mathbf{w}, b) - \eta \mathbf{g}$
-->

*dịch đoạn phía trên*

<!--
In the code below, `l` is a vector of the losses for each example in the minibatch.
Because `l` is not a scalar variable, running `l.backward()` adds together the elements in `l` to obtain the new variable and then calculates the gradient.
-->

*dịch đoạn phía trên*

<!--
In each epoch (a pass through the data), we will iterate through the entire dataset (using the `data_iter` function) 
once passing through every examples in the training dataset (assuming the number of examples is divisible by the batch size).
The number of epochs `num_epochs` and the learning rate `lr` are both hyper-parameters, which we set here to $3$ and $0.03$, respectively.
Unfortunately, setting hyper-parameters is tricky and requires some adjustment by trial and error.
We elide these details for now but revise them later in :numref:`chap_optimization`.
-->

*dịch đoạn phía trên*

```{.python .input  n=12}
lr = 0.03  # Learning rate
num_epochs = 3  # Number of iterations
net = linreg  # Our fancy linear model
loss = squared_loss  # 0.5 (y-y')^2

for epoch in range(num_epochs):
    # Assuming the number of examples can be divided by the batch size, all
    # the examples in the training dataset are used once in one epoch
    # iteration. The features and tags of minibatch examples are given by X
    # and y respectively
    for X, y in data_iter(batch_size, features, labels):
        with autograd.record():
            l = loss(net(X, w, b), y)  # Minibatch loss in X and y
        l.backward()  # Compute gradient on l with respect to [w, b]
        sgd([w, b], lr, batch_size)  # Update parameters using their gradient
    train_l = loss(net(features, w, b), labels)
    print('epoch %d, loss %f' % (epoch + 1, train_l.mean().asnumpy()))
```

<!-- ===================== Kết thúc dịch Phần 8 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 9 ===================== -->

<!--
In this case, because we synthesized the data ourselves, we know precisely what the true parameters are.
Thus, we can evaluate our success in training by comparing the true parameters with those that we learned through our training loop.
Indeed they turn out to be very close to each other.
-->

*dịch đoạn phía trên*

```{.python .input  n=13}
print('Error in estimating w', true_w - w.reshape(true_w.shape))
print('Error in estimating b', true_b - b)
```

<!--
Note that we should not take it for granted that we are able to recover the parameters accurately.
This only happens for a special category problems: strongly convex optimization problems with "enough" data to ensure that the noisy samples allow us to recover the underlying dependency.
In most cases this is *not* the case.
In fact, the parameters of a deep network are rarely the same (or even close) between two different runs, 
unless all conditions are identical, including the order in which the data is traversed.
However, in machine learning, we are typically less concerned with recovering true underlying parameters, 
and more concerned with parameters that lead to accurate prediction.
Fortunately, even on difficult optimization problems, stochastic gradient descent can often find remarkably good solutions, 
owing partly to the fact that, for deep networks, there exist many configurations of the parameters that lead to accurate prediction.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 9 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 10 ===================== -->

<!-- ========================================= REVISE PHẦN 5 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 6 - BẮT ĐẦU ===================================-->

<!--
## Summary
-->

## *dịch tiêu đề phía trên*

<!--
We saw how a deep network can be implemented and optimized from scratch, using just `ndarray` and `autograd`, without any need for defining layers, fancy optimizers, etc.
This only scratches the surface of what is possible.
In the following sections, we will describe additional models based on the concepts that we have just introduced and learn how to implement them more concisely.
-->

*dịch đoạn phía trên*

<!--
## Exercises
-->

## *dịch tiêu đề phía trên*

<!--
1. What would happen if we were to initialize the weights $\mathbf{w} = 0$. Would the algorithm still work?
2. Assume that you are [Georg Simon Ohm](https://en.wikipedia.org/wiki/Georg_Ohm) trying to come up with a model between voltage and current. Can you use `autograd` to learn the parameters of your model.
3. Can you use [Planck's Law](https://en.wikipedia.org/wiki/Planck%27s_law) to determine the temperature of an object using spectral energy density?
4. What are the problems you might encounter if you wanted to extend `autograd` to second derivatives? How would you fix them?
5.  Why is the `reshape` function needed in the `squared_loss` function?
6. Experiment using different learning rates to find out how fast the loss function value drops.
7. If the number of examples cannot be divided by the batch size, what happens to the `data_iter` function's behavior?
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 10 ===================== -->

<!-- ========================================= REVISE PHẦN 6 - KẾT THÚC ===================================-->

<!--
## [Discussions](https://discuss.mxnet.io/t/2332)
-->

## Thảo luận
* [Tiếng Anh](https://discuss.mxnet.io/t/2332)
* [Tiếng Việt](https://forum.machinelearningcoban.com/c/d2l)


### Những người thực hiện
Bản dịch trong trang này được thực hiện bởi:
<!--
Tác giả của mỗi Pull Request điền tên mình và tên những người review mà bạn thấy
hữu ích vào từng phần tương ứng. Mỗi dòng một tên, bắt đầu bằng dấu `*`.

Lưu ý:
* Nếu reviewer không cung cấp tên, bạn có thể dùng tên tài khoản GitHub của họ
với dấu `@` ở đầu. Ví dụ: @aivivn.

* Tên đầy đủ của các reviewer có thể được tìm thấy tại https://github.com/aivivn/d2l-vn/blob/master/docs/contributors_info.md.
-->

* Đoàn Võ Duy Thanh
<!-- Phần 1 -->
*

<!-- Phần 2 -->
*

<!-- Phần 3 -->
*

<!-- Phần 4 -->
*

<!-- Phần 5 -->
*

<!-- Phần 6 -->
*

<!-- Phần 7 -->
*

<!-- Phần 8 -->
*

<!-- Phần 9 -->
*

<!-- Phần 10 -->
*
