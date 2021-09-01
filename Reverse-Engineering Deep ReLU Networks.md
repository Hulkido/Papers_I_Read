# **Reverse-Engineering Deep ReLU Networks**

As a brief, the main aim of the paper is to reconstruct a Deep ReLU network by observing its outputs. Earlier, people used to think that it is impossible, but they proved it feasible and devised a method that could reconstruct the ReLU network up to isomorphism (we will describe it later). [Link to the paper.](https://arxiv.org/abs/1910.00744)

## Summary

It has been widely assumed that a neural network cannot be recovered from its outputs, as the network depends on its parameters in a highly nonlinear way. But in this paper, they proved that it is often possible to get the whole network with weights and biases up to greater accuracy and can be determined just by the inputs and outputs of a neural network. Here we will mainly deal with deep ReLU networks and give some insight into generalizing this method to some other architecture with limitations and future directions.

In the biological neural structure, we often have output and input of a group of cells, but we don't know the structure and the calculation between them, so this method can be beneficial to understand the system's internal working.

This approach leverages the fact that a ReLU network is piecewise linear and transitions between linear pieces when one of the ReLUs of the network transitions from its inactive to its active state. No prior work has deduced even the first layer of a fully connected network with two hidden layers. By contrast, their theoretical results and algorithm hold for reverse engineering any layer of a network of any depth. Furthermore, they show empirically that their algorithm can reconstruct the first layer of 2-, 3-, and 4-layer networks, as well as the second layer of 2-layer networks.

 <p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%201st.png" />
</p>

Note that we can create a new ReLU network by just permutating the neurons of a particular layer, and each permutation will develop a new network but is isomorphic to the older one. Also, since ReLU is invariant to multiplication, we can multiply a particular layer with a constant and then scale the next layer with the same constant but inverted to produce another isomorphic architecture. 

The scaling and permutation always create isomorphic architectures for a neural network. But not only these, but some architectures also are isomorphic to the given architecture other than scaling, and permutation proved in some papers. In this work, they showed that, where additional isomorphisms do not occur, it is often possible both in theory and in practice to reverse-engineer ReLU networks up to permutation and scaling.

Now we will discuss the method and some of the theorems that support the method.

 <p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%202nd.png" />
</p>

### Algorithms

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%203rd.png" />
</p>

We will be using two algorithms and a set of functions to complete the process. The first algo calculates the weights and the size of the first layer, and the second algo calculates the rest of the layers.

#### Algorithm 1 - For First Layer
<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%204th.png" />
</p>

We will initialize P<sub>1</sub>, P<sub>2</sub>, and S<sub>1</sub> as empty sets. Note that P<sub>2</sub> has no role in this algo but is the data useful for the second layer, and later we will see both the algos output p<sub>k+1</sub> if the current layer is k to help us gain the weight of the next layer.

**PointsOnLine Function:** This will accept a line segment in the input dimension and then calculate the number of possible points that transition between two linear regions. To know how it is done, the below definition from the paper can be helpful.

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%205_1.png" />
</p> 
<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%205_2.png" />
</p>

They are using a weighted average to move more toward the center. So for a given line segment, we can find the number of points in the set where there is a transition and hence a plane is passing through them. If a hyperplane passes through them (i.e., the plane with no bends), then the number of such hyperplanes equals the number of neurons in the first layer.

**InferHyperplane:** This function is to generate a hyperplane corresponding to those points that have the chance to have a hyperplane through them, i.e., first layer neuron. We do so by sampling the line segment around the point p and then using the PoinstOnLine function to calculate the points corresponding to hyperplane passing through p as we are sampling around it so all the found points will also be the part of the hyperplane, and then we can use them to calculate the hyperplane. The definition in the paper is as below.

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%206.png" />
</p>

**Testing hyperplane:** So till now, we have planes corresponding to the points, but we don’t know whether these planes are hyperplanes or are bend, so this function accepts p and hyperplane as the argument, and then we make a point far away from the p in the direction of the hyperplane and then using InferHyperplane function to infer hyperplane on that point if both the planes are same then it means given plane is a hyperplane. Note that our algorithm takes care of duplicates as it only accepts those arguments which are unique. Also, since we have the hyperplane equation, we can easily find the weights of the first layer now.

#### Algorihtm 2 - For additional layers.

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%207.png" />
</p>

In this part of the algorithm, we choose a p from the unused set p<sub>k</sub>, find a p<sub>z'</sub> and B<sub>z'</sub> using closest Boundary function, then if the p<sub>z'</sub> lies on B<sub>z</sub>, we find another p<sub>2</sub> on hyperplane after the bend then do the same until we have found the intersection point for all z’ in layer k-1 (impossible generally), or the p<sub>z'</sub> don’t lie on B<sub>z</sub> that is p<sub>z'</sub> belong to some higher layer B<sub>z'</sub> or if the number of iterations stops.

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%208_1.png" />
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%208_2.png" />
</p>

Once we have all these values output by the above function, we can use the below-given theorem to ensure the weights can be found. 

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%209.png" />
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/Hulkido/Papers_I_Read/main/images/Reverse-Engineering%20Deep%20ReLU%20Networks%2010.png" />
</p>

#### Main Points

1.  They prove that the architecture, weights, and biases of a deep ReLU network can be recovered (up to isomorphism) from the arrangement of regions on which the network is linear.
2.  They describe an algorithm to recover the network by approximating the boundaries between these linear regions, with the only information given about the network being its output on specified queries.
3.  They demonstrate the success of their algorithm in reverse-engineering both trained and untrained ReLU networks.

