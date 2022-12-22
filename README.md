# Generative Adversarial Networks

## Introduction and concept

Generative Adversarial Networks (GANs) represent a class of Deep Learning Frameworks which have firstly been introduced by Ian Goodfellow in 2014. While their spectrum of application is becoming larger with research, the main focus of GANs has been image processing.

The main idea behind GANs consists in the competition between 2 neural networks in a zero-sum game, the scope of each network being to improve its prediction accuracy. One network is named the "Generator" and the other the "Discriminator". The Generator's goal is to create fake data, with the ultimate goal of its output to be perceived as real. The goal of the Discrimnator is to look at the data generated by the Generator and label the data as "real" or "fake". As the data that the application of Generative Adversarial Networks is mainly focused on, as well as the data in our study is represented by images, we will focus on implementing Deep Convolutional Generative Adversarial Networks (DCGANs).

## Data 

The dataset used in our study is the Celeba Dataset. It represents a collection of more than 200K photos of celebrities, which come in .jpg format. We can plot some of the images in the training set :
![image](https://user-images.githubusercontent.com/114659655/209142529-fabd6e5a-4f66-407f-b67c-fd4cffc5e7f3.png)




## Deep Convolutional Generative Adversarial networks 

From a general point of view, Convolutional Neural Networks represent a particular type of neural networks that posess an ability to detect patterns in the data (for example, shapes, corners or edges in the case of images). Their hidden layers, called "Convolutional Layers", receive input, transform it and then pass it to the following layers.  The operation that corresponds to the transformation of an input is called a "Convolution".

### Convolution

The object at the basis of the convolution process is the kernel. It is represented by a 2-dimensional matrix of pixels that will be passed along the pixels of the input image. The values that constitute the kernel represent the weights of the network, they are randomly initialized and will be further optimized according to the accuracy and loss of the convolution process. The kernel we are using in our application is 4x4, chosen according to the paper Radford A., Metz L. Chintala S. (2016) : Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks, in order to match the dimensions desired for the feature maps passed through each layer. Its size is the same in the case of the Generator as in the case of the Discriminator.

The kernel is initialized and then passed along the image, row by row and column by column, until it has slid over all the pixels of the image. A dot product is applied between the kernel and the surface on the image on which it slides, resulting in a sum of element by element product between the kernel and the components of the image matrix. The resulting image is the output matrix. Each Kernel position corresponds to a single output pixel, for which the value is calculated by multiplying together the kernel value and the underlying image pixel value for each pixel in the kernel, and summing up the results. As it can be deduced, the size of the output image of the convolution process will be smaller, depending on the size of the initial image and the size of the kernel. In the case of a Convolutional Neural Network, the output obtained by the convolution inside a hidden layer is passed as input to the next hidden layer, after its normalization and passage through the acitvation function. The process of convolution is illustrated below :

![1_VVvdh-BUKFh2pwDD0kPeRA@2x](https://user-images.githubusercontent.com/114659655/209142358-c8b6cd4e-294c-4125-a06d-8483b77f4744.gif)

**source : [Dertat, A. (2017) : Applied Deep Learning - Part 4: Convolutional Neural Networks] (https://towardsdatascience.com/applied-deep-learning-part-4-convolutional-neural-networks-584bc134c1e2)**

Stride and padding are the 2 other elements present in the process of convolution. When kernel is being passed along the image, it can be noticed that some pixels participate in more convolution operations than others. It is the case for the pixels in the center, compared to the ones on the corners. Hence, it is less used in feature detection. Padding helps us solve this issue by adding rows and columns of empty pixels. Stride dictates how the kernel moves along the image. A stride of 1 means that the kernel is moving 1 by 1 pixel each time, while a stride of 2 means the kernel moves 2 by 2 pixels.

### Transposed Convolution

In our application, namely in the case of the Generator, we will come across Transposed Convolution. The purpose of such procedure is the opposite of the basic convolution, in the sense that it is looking to increase the size, height and width of the input image. Its functioning is also different.

The Transposed Convolution begins by padding the input image with zeros. This procedure happens without using the padding specified by us as argument, but using an implicit padding instead. Then, the kernel is convolved over the padded image with a stride of 1, also implicit and not representing a stride that we specify as argument. We can see an illustration of the process below :

![image](https://user-images.githubusercontent.com/114659655/209095000-6a92a857-d70e-43ce-aa52-bc5d8ade49cd.png)
**source : Godoy, D. (2022) : What are Transposed Convolutions ?**


Let us concentrate on the stride and padding that we introduce as arguments. Choosing a stride with a value superior to 1 will modify the process of transposed convolution. Zero-valued pixels will be introduced in both the columns and the rows of the existing image. The stride introduced minus 1 will give the number of columns and rows to be inserted. For example, a stride of 2 will introduce 1 row and 1 column of zeros. Then, the padding is added as mentioned before and usual convolution with a fixed stride of 1 is applied.


### Convolutional Layers

The neural networks we are going to build are going to be formed of Convolutional Layers. Convolutional Layers are meant to process data that is correlated in space, meaning pixels in an image whose measurements are represented by their color value that goes from 0 to 255. The pixels are correlated as they are part of the same image, therefore the value of one pixel wil influence the value of any neighbouring pixel. The output of each convolutional layer is a feature map, which corresponds to a spatial projection where certain features are exposed, which have previously been found by the convolutional layer.

The layers of the network are interconnected : the second layer takes as input the output from the first layer and so on. This can be seen in the values of the arguments that are passed onto each layer. For example, the number of output channels from the first layer will be the number of input channels of the second layer. We implement the layers using they torch.nn module.

### Batch Normalization Layers

Batch Normalization is applied right after the Convolutional Layer. It consists in the normalization of the activation vectors using the mean and standard deviation of the current bach. The term "normalization" refers to constraining the data to having values between 0 and 1. Given that the data points in an image are pixels, which have a value between 0 and 225, it is necessary to normalize them to a range that is manageable, otherwise the weights of the neural network will be affected during trainin by becoming too large and hence producing pixels on a very wide range of values for the output image.

### Activation functions

The activation functions are used depending on the type of output we are looking for. We are defining the functions according to the paper by Radford A., Metz L. Chintala S. (2016) : Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks. For the generator, we are using ReLU activation functions, except for the output layer which uses Tanh. The choice is based on the fact that the authors observed that a bounded activation function such as Tanh allowed quicker learning during training for the network. For the discriminator, the authors used Leaky ReLU activations because of their good performance. As outputs of the Discriminator should be probabilities, the activation function of the last layer wll be sigmoid.

## Generator and Discriminator

Throughout the implementation of our neural networks, we wil use the Pytorch framework. Both the Generator and the Discriminator are Neural Network Object Types. They are defined as classes who inherit from the nn.Module in Pytorch. The structure of both networks will be defined using the __init__() method and applied to computation using the forward() method.


### Generator

The Generator Neural Network's initial input is represented by a latent 1-dimensional noise vector. The length of the latent vector is fixed and can be chosen arbitrarily, but we will choose 100 for our example. We will draw the latent vector from the Gaussian distribution. 

![1_ULAGAYoGGr5eEB2377ArYA](https://user-images.githubusercontent.com/114659655/201658929-c53960f3-1d5d-4e33-b6c6-f6f88d484100.png)

**source : [Inkawich, N - DCGAN Tutorial] (https://pytorch.org/tutorials/beginner/dcgan_faces_tutorial.html)**


Transposed convolution is then applied by sliding the kernel along the noise vector. We need to pass from a 1x100 dimensional vector to the size of the images in the dataset (3x64x64, meaning 3 channels, 64 pixels height and 64 pixels width) for the final output, the convolution applied is transposed in order to upsample the data. Each transposed convolution will produce a feature map inside the generator.

Batch Normalization : After the Convolutional Layer, a batch normalization layer is implemented. It normalizes data at batch level so that it can be passed through the activation function afterwards. We are applying Batch Normalization to 2d images using BatchNorm2d().

### Discriminator

The input of the discriminator is an image in its intial dimensions. The image is processed through Conv2d layers, as the data has to be downsampled. As mentioned before, the filter (Kernel) passes through the set of pixels of the image. Values of the corresponding pixels are multiplied together then summed up to result in the convolved feature. 

The last convolution layer of the discriminator is flattened and passed through a sigmoid function. The Discriminator will therefore output the label of the image, 0 corresponding for 'fake' and 1 corresponding for 'real'. The following image illustrates the architecture of the Discriminator Network.

![image](https://user-images.githubusercontent.com/114659655/208300570-d18620da-22cf-4d00-a430-b6293ade3069.png)

**source : [Tsang, S. (2022) : Review: DCGAN — Deep Convolutional Generative Adversarial Network (GAN)] (https://sh-tsang.medium.com/review-dcgan-deep-convolutional-generative-adversarial-network-gan-ec390cded63c)**




## Weights initializiation

Model weights will be initialized according to the paper by Radford, Metz and Chintala (2015). They are normally distributed with mean 0 and standard deviation 0.02. The function weights_initialization takes as input one of the 2 models and transforms input data within the network's hidden layers, respectively the Convolutional, Transposed Convolutional and Batch Normalization Layer.

For a Convolutional Layer, weights represent each matrix element in the Kernel, that will be trained.



# Training setup 

After the Discriminator and the Generator have been initialized given the hyperparameters defined at the beginning of the code (number of channels and number of features for the discriminator and size of the latent vector, the number of channels and the number of features for the generator), weights are initialized. We use the Binary Cross Entropy loss function and Adam Optimizer for both Networks. In order to understand the choice for these characteristics, we have to look at the principle of interaction between the networks :

![image](https://user-images.githubusercontent.com/114659655/208265282-205b6375-2370-408f-998e-27ed7c864ba5.png)

**source : [Inkawich, N - DCGAN Tutorial] (https://pytorch.org/tutorials/beginner/dcgan_faces_tutorial.html)**

Here D(G(z)) is the probability that the output of the Generator G is classified as a real image. Let us look now at the Binary Cross Entropy Loss : 

![image](https://user-images.githubusercontent.com/114659655/208265308-6345ba98-1718-48be-9d65-2294af4cc528.png)

**source : [Inkawich, N - DCGAN Tutorial] (https://pytorch.org/tutorials/beginner/dcgan_faces_tutorial.html)**

It can be noticed that the 2 functions look alike. Indeed, the value of y can be dictated so as to correspond to the objective function of each of the 2 networks. For the Generator, we are looking for min log(1 - D(G(z))) which is equivalent to max log(D(G(z)), whereas for the Discriminator we are looking for min log(D(x)) + log(1-D(G(z)). 

It is worth reminding that the function BCELoss measures the Binary Cross Entropy between an input tensor and a target tensor. In our model, labelling is done with the value 1 for images classified as real and 0 for images classified as fake by the discriminator, through the functions zeros_like and ones_like. The values are put in a tensor, which is compared to a tensor of the same size filled with 1's through the BCE Loss criterion. The difference between the 2 network consists in the tensor used as input for the BCELoss function.

### Optimizers and their intialization

The training for our Networks  on the CelebA dataset will use the Adam (Adaptative moment estimation) optimizer, an extension of the stochastic gradient descent, given its computation efficiency in large datasets. However, it is not the only choice and we will see later than in some cases other optimizers perform better.

Adam optimizer is an adaptative learning rate method, meaning that it computes individual learning rates for different parameters. It does so by using estimations of the first and second moments of the gradient (mean and uncentered variance) to adapt the learning rate for each weight in the neural network. The procces is done by the usage of exponentially moving averages computed on the current batch.

Hyperparameters for the Adam optimizer are the learning rate, beta1 and beta2. The values of the betas represent the coefficients of the exponentially moving averages. We will take their values suggested in Radford A., Metz L. Chintala S. (2016) : Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks. These values are a 0.0002 for the learning rate (found empirically), 0.5 for beta1 and 0.999 for beta2. It should be noticed that conventional values for the betas are 0.9 and 0.999, but the authors found that a "value of 0.9 resulted in training oscillation and instability while reducing it to 0.5 helped
stabilize training" (Radford A., Metz L. Chintala S. (2016) : Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks, p.4).


### Training loop

The Generator is trained to minimize the loss with respect to the fake images it generates. By looking at its loss function log(1 - D(G(z))), we can see that its minimization is equivalent to max(D(G(z))), which will be the optimization problem in our training loop. The Discriminator's loss has 2 components as we have seen in its formula. We are maximizing the probability that it labels real images as real and fake images as fake. Through each batch the following operations are applied :

- A number of real images equal to the batch size are taken from the dataset and and sent to CUDA
- A number of random noise vectors equal to the batch size is created and inputted to the Generator for the creation of fake images
- Training of the Discriminator : On the real images, Discriminator classifies the image as "real" or "fake", according it the value 0 for "fake" and "1" for real. The obtained result is flattened into a 1 dimensional vector with values 0 or 1, which is compared to a vector composed only of value 1's the same size as the one obtained after the flattening through BCE. The same procedure is applied on the fake images, but this time the obtained tensor for comparison is composed only of value 0's. The obtained BCE loss function is optimized through the Adam optimizer
- Training of the Generator : Discriminator network is applied to the generated fake images. The result of this operation is flattened on a 1-dimensional vector composed of values 0 or 1 depending on the classification by the discriminator. This obtained vector is compared through the BCE to a vector of the same size full of 1's. The obtained objective function will be maximized through the Adam Optimizer, as we want fake image to be classified as real as often as possible.

### Results

Let us look at some of the obtained images after 3 epochs :

![image](https://user-images.githubusercontent.com/114659655/209126836-9ea22860-0a8d-4a70-92fd-95f6d1b27a45.png)

The images look noisy because of the reduced number of epochs, which we have fixed because of ressource constraints. The networks have not been training enough in order to reproduce visible features. However, for some images some shapes are starting to get visible.

Let us now look at the graph of the evolution of the objective functions for both the Generator and the Discriminator :

![image](https://user-images.githubusercontent.com/114659655/209132815-a01c3cce-ecef-4d8b-a450-3ad308c854b7.png)


We can see that the loss function for the discriminator is going down as the number of iterations increases, while the objective function for the Generator is going up. This is showing a normal optimization of the network. We want the objective function of the Generator to increase, given the way we have coded the training loop. We are looking for the maximization of D(G(z)).


## Extension : application on the MNIST Dataset

In order to see the behavior of our Discriminator and Generator, we can apply them to the MNIST dataset. A few differences are present at the level of hyperparameters. MNIST images are only black and white, meaning they have only 1 channel instead of 3. The image size is 32 instead of 64 here. The networks are trained in the same manner as before. We can take a look at the images obtained after 3 epochs :

![image](https://user-images.githubusercontent.com/114659655/209143219-83671e28-94db-47a6-b2b8-79e4a38082b8.png)


Once again, as the networks have not been trained enough, images produced are very noisy.



# Extension : WGAN- Wasserstein Generative Adversarial Networks

We have seen the principles of GANs and the functioning of DCGANS. We are now focusing on Wasserstein Generative Adversarial Networks


## Introduction and concept

Firstly let us understand why such variations of the traditional GANs have been created. While a very performant tool, GANs can be subject to convergence failure (failure to produce optimal results) and mode collapse (model failing to produce unique results and repeating a similar pattern, quality or classes). The advantage of WGANs is that they solve this issue and they offer higher stability for the training in comparison to traditional GANs. Another advantage is that the value of the global loss is meaningful, in the sense that it gives us a termination criterion, contrary to the loss of classic GANs.

We have seen that the main idea in GAN implementation is that we have 2 probability distributions, one for the Generator Pg and one for the Discriminator Pd. The goal is to have similar probability distributions so that generated images look realistic. The issue here is therefore how we define this similarity, or the "distance" between the 2 distributions. In the case of WGAN's, we are focusing on Wasserstein distance measure. 

![image](https://user-images.githubusercontent.com/114659655/209140474-d8b16a4c-9380-4347-ae12-712015f2c301.png)

**source : [Arjovsky M., Chintala S., Bottou L., (2017): Wasserstein GAN] (https://arxiv.org/abs/1701.07875)**

Here the capital PI represents the set of all joint distributions whose marginal distributions are Pr respectively Pg.

In the training cell, we are intializing the parameters the way we did in the GAN section. This time however we are taking the parameters metioned in the paper by Arjovsky M., Chintala S., Bottou L., (2017): Wasserstein GAN. We are taking a learning rate of 0.0005 and batch size is 64. 2 additional parameters are added, the discriminator iterations and the weight clip. Weight clipping prevents the gradient from getting too large in training, making the model unstable. The idea of weight clipping is that if the gradient gets too large, we rescale the parameters by the value of the weight clip.


# References

[1^]  [Inkawich, N - DCGAN Tutorial] (https://pytorch.org/tutorials/beginner/dcgan_faces_tutorial.html)

[2^]  [Radford A., Metz L. Chintala S. (2016) : Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks] (https://arxiv.org/abs/1511.06434)

[3^] [Arjovsky M., Chintala S., Bottou L., (2017): Wasserstein GAN] (https://arxiv.org/abs/1701.07875)

[4^] [WGAN Repository] (https://github.com/aladdinpersson/Machine-Learning-Collection/blob/master/ML/Pytorch/GANs/3.%20WGAN/train.py)

[5^] [Godoy, D. (2022) : What are Transposed Convolutions ?] (https://towardsdatascience.com/what-are-transposed-convolutions-2d43ac1a0771#:~:text=Transposed%20convolutions%20are%20like%20the,the%20generator%20part%20of%20GANs.)

[6^] [Tsang, S. (2022) : Review: DCGAN — Deep Convolutional Generative Adversarial Network (GAN)] (https://sh-tsang.medium.com/review-dcgan-deep-convolutional-generative-adversarial-network-gan-ec390cded63c)

[7^] [Dertat, A. (2017) : Applied Deep Learning - Part 4: Convolutional Neural Networks] (https://towardsdatascience.com/applied-deep-learning-part-4-convolutional-neural-networks-584bc134c1e2)




