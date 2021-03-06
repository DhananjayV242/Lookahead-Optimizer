# Lookahead Optimizer Analysis

[Interactive project report on Notion.so ( by Akash Palrecha ) ](https://www.notion.so/akashpalrecha/Lookahead-Optimizer-Project-913e45b63e9a4528bee56a588e477f9f)


---

### Lookahead Optimizer: *K* Steps forward, 1 step back

[Lookahead Optimizer: k steps forward, 1 step back](https://arxiv.org/abs/1907.08610)

This repository contains code to run experiments which closely simulate those run by Geoffrey Hinton in the paper linked to above. 

*The results of the project, along with a very detailed report can be interactively viewed at Akash Palrecha's repo (akashpalrecha)  :* 

---

The optimizer has been tested on CIFAR10, CIFAR100, and Imagenette (a smaller subset of the Imagenet dataset created by Jeremy Howard, the founder of Fast.ai)

The experiments have been run primarily using the FastAI library.

### Lookahead Algorithm (Simplified):

- Choose inner optimizer (SGD, AdamW, etc.)
- Perform *K* iterations in the usual way
- Linearly interpolate between weights of 1st and Kth iteration using a parameter *alpha/*
- Repeat till terminating condition (convergence, epochs, etc.)

---

## Conclusions from the experiments:

![Valid Loss for imagenette](/imagenette_valid_loss.png)

- AdamW almost always overfits the data with a very low training loss, high validation loss, and lower accuracy.
- Lookahead consistently gets the least validation accuracy showing the best generalizability, which is more important than having a lower training loss.
- Lookahead almost always gets the highest accuracy and only sometimes loses to SGD with a very small margin.
- Lookahead/SGD offers very small, marginal advantages over vanilla SGD. Lookahead/AdamW consistently performs badly. Possible reason is the behavior of the exponential average and squared average states stored for each parameter after doing lookahead updates.
- RMSProp does not seem like a good choice as an optimizer for computer vision problems.
- The experiments reinforce the fact: lower training loss isn’t always better.
- Again, Lookahead/SGD generalizes the best, and very consistently so.
- Resetting the state in both Lookahead/SGD and Lookahead/AdamW increased accuracies by about 3%. But the tables show the same increase in all other optimizers too. Hence, we cannot surely say if resetting the state helped here. It can simply be related to using a different random
seed for the particular experiment. Issues with getting things to work right:

---

## Issues with getting it to work right":

### Stateful Optimizers:

- The ”inner optimizer”, let’s call it IO, can sometimes have other hyperparameters which may not all behave very well when working under Lookahead.
- For example, in SGD + Momentum, each parameter of the model will have an individual state with it’s momentum. The paper suggests resetting the momentum after each Lookahead update.
We experimented with a version that does not reset momentum. The results follow in the presentation.
- For Adam based optimizers, states like the exponential averages, exponential moving averages, etc need to be dealt with carefully too. Unfortunately, the paper does not discuss Lookahead + Adam
and neither does it suggest a strategy for this specific case. We tried both resetting the state and not altering it after lookahead updates, but either approach did not give good results.

### Computational Graph:

- Current implementations of popular deep learning libraries such as PyTorch, TensorFlow, MXNet, etc. all involve computational graphs being built as calculations are carried out so as to support
features like automatic differentiation and network optimization by compilers.
- When storing the slow-weights at each 1st step in the lookahead algorithm, we need to make sure to not just clone the parameter weights that we’re copying but also to detach them from the
graph. This can be simply achieved in PyTorch by a statement such as: `x.detach()`
- This is a very important detail to get right for two reasons:
    1. The network will still train to an acceptable accuracy even if this weren’t taken care of, but it won’t be as accurate.
    2. This makes it hard to debug this issue, as there are no errors during
    training.

---

# Instructions to run:

*It is assumed that you have a working copy of Conda installed*

Open up the terminal and type:

    git clone https://github.com/akashpalrecha/Lookahead.git
    cd Lookahead
    conda env create -f environment.yml
    conda activate nnfl_project // optional
    jupyter notebook

Open up `start_off.ipynb` and `imagenette.ipynb` to run the respective experiments

In the notebook's menu bar, choose the `nnfl_project` kernel.

You can create a free ID on [www.wandb.ai](http://www.wandb.ai) to track the progress of the experiment.

If you do not want such tracking, comment out the lines corresponding to the `wandb` library and remove the `WandbCallback` from the `cnn_learner` call in the `fit_and_record` function.
