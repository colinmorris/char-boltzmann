# char-boltzmann

Character-level RBMs for short text. For more information, check out [my blog post](https://colinmorris.github.io/blog/dreaming-rbms).

# Requirements

scikit-learn and its dependencies (numpy, scipy) is the big one. Also enum34. `pip install -r requirements.txt` might be all you need to do.

# How-to

The two important scripts are:

- `train.py`: trains an RBM model on a text file with one short text per line. It has a whole bunch of command line options you can supply, but the defaults are all pretty reasonable. The one you're most likely to need to change is `--extra-chars` - the default behaviour is to use only `[a-z ]` (and `[A-Z]` implicitly downcased), which is definitely not appropriate for some datasets having lots of numerals/punctuation.
- `sample.py`: generates new short texts given a pickled model file generated by `train.py`

(The last script, `compare_models.py` is only really relevant if you're training a bunch of different models on the same dataset and enjoy spreadsheets.)

More details on the arguments to these scripts can be seen by running them with '-h'.

README-datasets.md has pointers to some suitable datasets. 

# Example

To train a small model on first names:

    wget http://www.cs.cmu.edu/afs/cs/project/ai-repository/ai/areas/nlp/corpora/names/other/names.txt
    python train.py --maxlen 10 --extra-chars '' --hid 100 names.txt
    python sample.py names__nh100.pickle
    
This should give you some output like...

    wietzer     
    sarnimono   
    buttheo     
    ressinosoo  
    bernington

# Interpreting train.py output

During training, you'll see debug output like...

    [CharBernoulliRBMSoftmax] Iteration 3/5 t = 14.46s
    Pseudo-log-likelihood sum: -115047.96   Average per instance: -2.13
    E(vali):        -14.00  E(train):       -14.07  difference: 0.07
    Fantasy samples: moll$$$$$$|anderd$$$$|gronbel$$$

Without going into too much detail, the pseudo-log-likelihood (-2.13 above), is a pretty decent estimation of how well the model is currently fitting the training data. The lower the better.

The next line compares the energy assigned to the training data vs. the validation set. The difference (0.07 in this case) gives an idea of how much the model is overfitting. The higher the difference, the worse. A difference of 0 implies no overfitting. 

The final line has string representions of a few of the "fantasy particles" used for the [persistent contrastive divergence](http://www.cs.toronto.edu/~tijmen/pcd/pcd.pdf) training.

# More details

The core RBM code is cannibalized from scikit-learn's [BernoulliRBM](http://scikit-learn.org/stable/modules/generated/sklearn.neural_network.BernoulliRBM.html#sklearn.neural_network.BernoulliRBM) implementation. I tacked on some additional features including:

- L2 weight cost
- softmax sampling
- sampling with temperature (for simulated annealing)
- flag to gradually reduce learning rate
- initializing visible biases to the training set means

This code has the same performance limitations as the base sklearn implementation. In particular, it can't run on a GPU.

The 'workspace' branch has a lot of extra scripts and data files which *might* be useful to someone, but which are kind of messy (even relative to the already-kinda-messy master). They mostly relate to model visualization and experiments with different sampling techniques.
