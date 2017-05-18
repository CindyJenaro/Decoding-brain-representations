# https://github.com/umuguc/Decoding-brain-representations

# Decoding-brain-representations
DONDERS (f)MRI TOOL-KIT: From Image Acquisition to Computation Model

In this hands-on session, you will implement a neural decoder for reconstructing perceived stimuli from brain responses. We will be using the dataset that was previously used in a number of papers. These papers along with lecture notes on neural decoding can be found in the *./papers* folder. You can refer them for more details on the dataset and/or the method.

The dataset contains fMRI data acquired from the early visual cortex of one subject as the subject was presented with 100 grayscale images of handwritten sixes and nines (50 sixes and 50 nines). The fMRI data has been realigned and slice time corrected. Furthermore, stimulus specific response amplitudes have been estimated with a general linear model.

Let's first familiarize ourselves with the dataset. It contains a number of variables:

**X** -> This is a 100 x 784 matrix. The *i*th row contains the pixel values of the stimulus that was presented in the *i*th trial of the experiment. Note that the stimuli are 28 pixel x 28 pixel images, which were reshaped to 1 x 784 vectors.

**Y** -> This is a 100 x 3092 matrix. The *i*th row contains the voxel values of the responses that were measured in the *i*th trial of the experiment.

and

**X**_prior -> This is a 2000 x 784 matrix. Each row contains the pixel values of a different stimulus, which was not used in the experiment. Note that the stimuli are 28 pixel x 28 pixel images, which were reshaped to 1 x 784 vectors.

## Task 1

- Load the dataset.
- Visualize some of the stimuli. Tip: You can use reshape and imagesc functions.
- Normalize X and Y to have zero mean and unit variance. Tip: Recall that normalization means subtracting the mean of each pixel/voxel from itself and dividing it by its standard deviation. You can use zscore function.
- Split X and Y in two parts called X_training and X_test, and Y_training and Y_test. The training set should contain 80 stimulus-response pairs (40 pairs for sixes and 40 pairs for nines). The test set should contain 20 stimulus-response pairs (10 pairs for sixes and 10 pairs for nines).

(The solution of the task is provided in *./solutions/task_1.m*. However, it is recommended that you try to solve the task by yourself before referring to the solution.)

---

Note: In the remainder of this document, we will use **x** for referring to a 784 x 1 stimulus vector and **y** for referring to a 3092 x 1 response vector.

Our goal is to solve the problem of reconstructing **x** from **y**.

One possible approach to solve this problem is to use a *discriminative* model.

Discriminative models predict **x** as a function of **y**. That is:

**x** = f(**y**)

We will assume that f is a linear function. That is:

**x** = **B'** **y**

We can estimate **B** with ridge regression. That is:

**B** = inv(**Y**\_training' **Y**\_training + lambda **I**) **Y**\_training' **X**\_training

where lambda is the regularization coefficient, **I** is the *q* x *q* identity matrix, and *q* is the number of voxels.

Note that we can safely ignore the intercept since we normalized our data to have zero mean and unit variance.

## Task 2

- Estimate **B** on the training set. Tip: Normally, you should use cross validation to estimate lambda. For simplicity, you can assume that lambda = 10 ^ -6.
- Reconstruct **x** from **y** in the test set.
- Visualize the reconstructions. Tip: You can use reshape and imagesc functions.

(The solution of the task is provided in *./solutions/task_2.m*. However, it is recommended that you try to solve the task by yourself before referring to the solution.)

---

The remaining tasks are considered optional for this hands-on session.

---

Another possible approach to solve the problem of reconstructing **x** from **y** is to use a *generative* model and invert it with Bayesian inference.

We reformulate the problem as finding the most probable **x** that could have caused **y**. That is:

argmax_**x** P(**x** | **y**)

where P(**x** | **y**) is called the posterior (probability of the stimulus being **x** if the observation is **y**). In other words, we have to define the posterior, estimate its parameters and find the argument that maximizes it, which will be the reconstruction of **x** from **y**. While, this may seem daunting, it actually has a simple solution. The posterior assigns a probability to an event by combining our observations and beliefs about it, and can be decomposed with Bayes' theorem as the product of how likely our observations are given the event (probability of observing **y** if the stimulus is **x**) and how likely the event is independent of our observations (probability of the stimulus being **x**). That is:

P(**x** | **y**) ~ P(**y** | **x**) * P(**x**)

where P(**y** | **x**) is called the likelihood and P(x) is called the prior.

We will assume that the likelihood and the prior are multivariate Gaussian distributions. A Gaussian is characterized by two parameters: a mean vector and a covariance matrix.

In the case of the likelihood, the mean of the Gaussian is given by:

--> **mu**\_likelihood = **B'** **x**

As before, we can estimate **B** with ridge regression:

**B** = inv(**X**\_training' **X**\_training + lambda **I**) **X**\_training' **Y**\_training

where lambda is the regularization coefficient, I is the *p* x *p* identity matrix, and *p* is the number of pixels.

The covariance matrix of the likelihood is given by:

--> **Sigma**_\likelihood = diag(E[||**y** - **B'** **x**|| ^ 2]). 

In the case of the prior, the mean of the Gaussian is given by:

--> **mu**\_prior = **0** (which is a vector of zeros)

The covariance matrix of the prior is given by:

--> **Sigma**\_prior = **X**\_prior' * **X**\_prior / (n - 1)

where n is the length of **X**\_prior.

## Task 3

- Estimate **B** on the training set. Tip: Normally, you should use cross-validation to estimate lambda and Sigma_likelihood. For simplicity, you can assume that lambda = 10 ^ -6 and Sigma_likelihood = 10 ^ -3 **I**.
- Estimate **Sigma**\_prior. Tip: Add 10 ^ -6 to the diagonal of Sigma_prior for regularization.
- Visualize **Sigma**\_prior. Can you explain what it shows? Tip: you can use imagesc function.

(The solution of the task is provided in *./solutions/task_3.m*. However, it is recommended that you try to solve the task by yourself before referring to the solution.)

---

Having defined the likelihood and the prior as Gaussians, we can derive the posterior by multiplying them. It turns out that the product of two Gaussians is another Gaussian, whose mean vector is given by:

**mu**\_posterior = inv(inv(**Sigma**\_prior) + **B** inv(**Sigma**\_likelihood) **B**') **B** * inv(**Sigma**\_likelihood) **y**

We are almost done. Recall that the reconstruction of **x** from **y** is the argument that maximizes the posterior, which we derived to be a Gaussian. We will be completely done once we answer the following question:

Question: What is the argument that maximizes a Gaussian?

.  
.  
.  
.  
.  
.  

The answer is its mean vector, which is the solution of our initial problem. That is:

argmax_**x** P(**x** | **y**) =  
**mu**\_posterior =  
inv(inv(**Sigma**\_prior) + **B** inv(**Sigma**\_likelihood) **B**') **B** * inv(**Sigma**\_likelihood) **y**

Now, we can plug any **y** in the above equation and reconstruct the most probable **x** that could have caused it.

## Task 4

- Reconstruct **x** from **y** in the test set.
- Visualize the reconstructions. Tip: You can use reshape and imagesc functions.
- Compare the reconstructions with the earlier reconstructions. Which one is better? Why? Can you think of ways to improve the results?

(The solution of the task is provided in *./solutions/task_4.m*. However, it is recommended that you try to solve the task by yourself before referring to the solution.)

---

Congratulations, you have reached the end! If everything went according to the plan, your visualizations should look like the following:

### Task 1

![alt text](https://github.com/umuguc/Decoding-brain-representations/raw/master/figures/figure_1.png "Figure 1")

### Task 2

![alt text](https://github.com/umuguc/Decoding-brain-representations/raw/master/figures/figure_2.png "Figure 2")
![alt text](https://github.com/umuguc/Decoding-brain-representations/raw/master/figures/figure_3.png "Figure 3")

### Task 3

![alt text](https://github.com/umuguc/Decoding-brain-representations/raw/master/figures/figure_4.png "Figure 4")

### Task 4

![alt text](https://github.com/umuguc/Decoding-brain-representations/raw/master/figures/figure_5.png "Figure 5")
![alt text](https://github.com/umuguc/Decoding-brain-representations/raw/master/figures/figure_6.png "Figure 6")
