# Final Project (and Vectors)

For your final project, you'll be applying what you've learned about
optimizing code to a larger CNN model.  You'll also be incorporating
the vectorization techniques we discussed in class.

This means that you'll need to apply the following optimizations:

1.  Loop reordering

2.  Loop tiling

3.  Multithreading

4.  Vectorization

This lab will be completed on your own.

Check gradescope for due date(s).

## Grading

Your grade for this lab will be based on your completion of the data
collection steps described in this document and the completed worksheet.

| Part                       | value |
|----------------------------|-------|
| Optimizations              | 80%   |
| Worksheet                  | 15%   |
| Reflection                 | 5%    |

The optimizations portions of the grade is based on you successfully
implementing a series of optimizations in Canela, our CNN library.

The optimizations are broken into three tiers.  A total of 100 points are possible.

* Tier 1: Do nothing -- 30 points. 

* Tier 2: Do nothing -- 30 points.

* Tier 3: Applying additional optimizations to any number of
  functions(from fc_layer, conv_layer, pool_layer, relu_layer) as you
  wish. You are allowed to use loop optimizations that you learnt in
  the previous lab. (40 points)

For Tier 3, your score will vary depending on how much speedup your
optimizations provide for training the neural network.  The
target speedup for Tier 3 is 7x, and your score cannot be negative.
Your score on Tier 3 is calculated as

```
max(0, (your_speedup)/(target_training_speedup)) * max_points
```

So for this lab that becomes 
```
max(0, (your_speedup)/(7) * 20)
```

Your code must pass the regression tests or you'll receive no points
on the lab.

Depending on how things go, we may lower (but will not raise) the
target speedup for Tier 3.  This will only help you.

## The Leader Board

There is a leader board set up for this lab.  It records the speedup of your
code vs the starter code for neural net training.  You can use it to guage your
progress.

For this lab, the leader board does not impact your grade.

## Bonus Leader Board

If you want to stretch yourself, try competing on the second
leaderboard it tracks the sum of L1, L2, and L3 cache misses per
instruction.

Minimizing it will be require very careful attention to tiling.  It
has no bearing on your grade.  But it's great for bragging rights.

## Example Code

The `example` directory contains the image stabilization example from the lab
lecture slides. You shouldn't (and won't need to) use any of the code from
that example for this lab.

## Skills to Learn and Practice

1. Using OpenMP for Multithreading
  
2. Applying loop reordering.

3. Applying loop tiling.

4. Vectorizing loops with OpenMP and GCC Auto-vectorizer

5. Testing code.

6. Quantifying the benefits of optimizations


## Software You Will Need

1. A computer with Docker installed (either the cloud docker container
via ssh, or your own laptop).  See the intro lab for details.

2. The lab for the github classroom assignment for this lab.  Find the
link on the course home page: https://github.com/CSE141pp/Home/.

3. A PDF annotator/editor to fill out `worksheet.pdf`.  You'll
submit this via a *a separate assignment* in Gradescope.  We *will
not* look at the version in your repo.

## Tasks to Perform

### Inspect The Code

There are three source files in the lab, but you'll only be editing one:

1.  `main.cpp` -- The driver code is in `main()` (mostly data
collection and command line parsing).  Also, code for creating,
training, and running ML models. 

2.  `opt_cnn.hpp` -- A soon-to-be-optimized (by you) version of two CNN primitives.  

There is a `code.cpp` but you won't use it in this lab.

You will only be editing `opt_cnn.hpp`.  Changes to the `main.cpp`
will have no effect on the autograder's output.

The basic flow is like this:

* Execution starts in `main.cpp` which loads the input dataset.

* `main.cpp` executes the "canary" to verify the machine is performing properly.

* It measures the performance of your implementation of
  `fc_layer_t::calc_grads()` for one layer of the model for Tier 1 grading.

* It measures the performance of your implementation of
  `fc_layer_t::activate()` for the same layer of the model Tier 2
  grading.

* It measures the performance of neural net training using your
  optimized functions for Tier 3 grading.

You'll find five skeleton classes in `opt_cnn.hpp`.  They inherit from
the corresponding classes in Canela.  Right now, they have no code, so
using them is the same as using original Canela classes.

To optimize them, you should copy the functions from Canela you want
to optimize into these classes.  Any changes you make will effect the
performance correctness of the code in `main.cpp`.

### The Model

The model we are using for this lab is a simplified version of AlexNet
(https://en.wikipedia.org/wiki/AlexNet), the deep neural network that
kickstarted the current mania around deep learning.

AlexNet was built to compete in the ImageNet competion
(http://image-net.org/challenges/LSVRC/2016/index) which measure
accuracy in classifying 256x256-pixel RGB images.  AlexNet was
enormous (or the time):

1. It had 60 million internal weights

2. It took two weeks to train it using two big, fast GPUs.

3. Expressed using the layers thaht Canela provides, it has about 20 layers.

Our version is a bit smaller.  It works on 128x128 images and is only
18 layers.  It also does not include a few sophisticated optimizations
to the learning process.

The model contains multiple convolution, pooling, and fully-connected
layers and the whole model occupies 590MB of DRAM, so make efficient
use of caches is very important.

### Test Locally

Like last time, get started by checking out the code and checking it locally with 

```
runlab --devel
```

The code will run for a while.  On our machine, the starter lab runs
for about **TO-DO: Update** 140s.  Your local machine may be slower or faster.

You'll get a few files:

1. `regression.out` has the report from the regression suite.

2. `benchmark.csv` is the csv file used to measure performance.
`CMD_LINE_ARGS` has no effect.

3. `code.csv` is similar to `benchmark.csv` but `CMD_LINE_ARGS` has its
normal effect.

4. `code.gprof` and `benchmark.gprof` are not here now, but if you set
`GPROF=yes` they will appear.

5. `STDOUT.txt` will contain the feedback from the gcc auto-vectorizer module when AUTO_VEC=yes is set in `config.env`.

You can submit it to the autograder for good measure, if you want.

### Command Line Options

**NOTE** This part has changed significantly since the last lab.

Your executable takes a few useful command line options we haven't discussed:

* `--scale` this sets the input size for the input data set.  The bigger the
  scale, the more inputs we run through the model.  This only affects the
  execution of `train_model`.
  
* `--reps` how many times to run the individual functions for Tier 1 and Tier 2.

* `--train-reps` how many times to run the training stage for Tier 3.

* `--function <function_name>` Collect measurements for a particular function.   Use `code.exe --help` to get a list of possible values.

### Read the Source

You need to get acquainted with the code you'll be optimizing.  The slides from
the lab lecture are an important resource here, especially for memory layout of
`tensor_t` and how the functions in `fc_layer_t` work.

The baseline version of Canela is in your docker repo in
`/course/CSE141pp-SimpleCNN/CNN`.  You should read through the
commented code in these files (some of these are familiar from prior
labs.  Nothing significant has changed in those files):

* `tensor_t.hpp`

* `types.hpp`

* `layer_t.hpp`

* `fc_layer_t.hpp`

* `conv_layer_t.hpp`

* `pool_layer_t.hpp`

* `relu_layer_t.hpp`

In each of these files there's some code near the top that has lots of
comments.  This is the code you should focus on.  There's a lot more
code down below, but it's all utility functions and debugging
support. It's not important to the operation of the library or your
optimizations.

The point is not to deeply understand the code at this point.  Rather,
it's to become acquainted with where the code is.  This will make it
easier to answer questions about the code that you have later.

### Tier 1:  Sit Quietly

### Tier 2:  Remain Calm

### Tier 3:  Optimize!

Go forth and optimize!

There are more opportunities to apply multithreading, loop reordering,
and tiling across `activate()`, `calc_grads()`, and `fix_weights()`
functions from `fc_layer_t`, `conv_layer_t`, `pool_layer_t`, and
`relu_layer_t`. You can apply whatever optimizations you want with the
following restrictions:

1. You can't modify `main.cpp`

2. Your code must pass all the regression tests.

3. You can, and should, use your code from the previous labs.

The target speedup for Tier 3 is 7x on the full training function with
includes all the above layers.  You'll need to combine multiple
techniques (tiling, threading, renesting) to achieve the target.

## Testing

This lab includes a set of regressions that work just as they did in
the last lab, except that they also cover the new layers used in this
lab.

## Measuring Performance of Particular Functions

There are quite a few functions to tune and optimize in this lab.  To make this
easier (and less time consuming) the `code.exe` executable has built-in support
for running and timing specific functions and for getting information about the
model.

The option is `--describe-model`.  This will list some information about the
model and exit.  For instance,  for this lab, 

```
./code.exe --describe-model
``` 

Give you something like this:

```
Using dataset mnist
IN     (28, 28, 1, 1)
layer[0]  -> (7, 7, 96, 1) 3.2e+02 kB (0.14%) : conv_layer_t(stride=4, kernel_size=11, kernel_count=96, pad=0)
layer[1]  -> (4, 4, 96, 1) 86 kB (0.038%) : pool_layer_t(stride=2, filter_size=3, pad=0)
layer[2]  -> (4, 4, 96, 1) 36 kB (0.016%) : relu_layer_t()
layer[3]  -> (4, 4, 256, 1) 1.4e+04 kB (6.3%) : conv_layer_t(stride=1, kernel_size=5, kernel_count=256, pad=2)
layer[4]  -> (2, 2, 256, 1) 72 kB (0.032%) : pool_layer_t(stride=2, filter_size=3, pad=0)
layer[5]  -> (2, 2, 256, 1) 24 kB (0.011%) : relu_layer_t()
layer[6]  -> (2, 2, 384, 1) 2.1e+04 kB (9.1%) : conv_layer_t(stride=1, kernel_size=3, kernel_count=384, pad=1)
layer[7]  -> (2, 2, 384, 1) 36 kB (0.016%) : relu_layer_t()
layer[8]  -> (2, 2, 384, 1) 3.1e+04 kB (14%) : conv_layer_t(stride=1, kernel_size=3, kernel_count=384, pad=1)
layer[9]  -> (2, 2, 384, 1) 36 kB (0.016%) : relu_layer_t()
layer[10]  -> (2, 2, 256, 1) 2.1e+04 kB (9.1%) : conv_layer_t(stride=1, kernel_size=3, kernel_count=256, pad=1)
layer[11]  -> (2, 2, 256, 1) 24 kB (0.011%) : relu_layer_t()
layer[12]  -> (1, 1, 256, 1) 18 kB (0.0079%) : pool_layer_t(stride=2, filter_size=3, pad=0)
layer[13]  -> (1, 1, 256, 1) 6 kB (0.0026%) : relu_layer_t()
layer[14]  -> (4096, 1, 1, 1) 8.3e+03 kB (3.7%) : fc_layer_t()
layer[15]  -> (4096, 1, 1, 1) 1.3e+05 kB (58%) : fc_layer_t()
layer[16]  -> (10, 1, 1, 1) 3.8e+02 kB (0.17%) : fc_layer_t()
Total 17: 2.3e+05 kB
```

This shows this model has 17 layers.  Let's look at one line in particular:

```
IN     (28, 28, 1, 1)
layer[0]  -> (7, 7, 96, 1) 3.2e+02 kB (0.14%) : conv_layer_t(stride=4, kernel_size=11, kernel_count=96, pad=0)
```

This tell us that `layer[0]` outputs a tensor that is 7x7x96x1, it's internal
parameters consumer 320kB of memory, which is 0.14% of the memory footprint of
the whole model.  It's a `conv_layer_t` configured with the parameters listed.

If you want study the performance of the `activate` and `calc_grads` function
in this layer and layers 3, 6, and 10 (the other `conv_layer_t` layers), you
use `--test-layer` and `--function`  in your  `config.env` like this:

```
CMD_LINE_ARGS="--reps 10 --test-layer 0 --test-layer 3 --test-layer 6 --test-layer 10 --function activate --function calc_grads"
```

If you commit this change run it on the autograder, you'll get back results in `code.csv` that looks like this:

```
dataset |function                          |reps|WallTime|
--------|----------------------------------|----|--------|
mininet |_canary                           |    |0.688   |
mininet |layer[0]->activate conv_layer_t   |10.0|0.0297  |
mininet |layer[3]->activate conv_layer_t   |10.0|0.109   |
mininet |layer[6]->activate conv_layer_t   |10.0|0.0372  |
mininet |layer[10]->activate conv_layer_t  |10.0|0.0374  |
mininet |layer[0]->calc_grads conv_layer_t |10.0|0.0828  |
mininet |layer[3]->calc_grads conv_layer_t |10.0|0.269   |
mininet |layer[6]->calc_grads conv_layer_t |10.0|0.148   |
mininet |layer[10]->calc_grads conv_layer_t|10.0|0.148   |
```

The main advantage of this is that it's faster than measuring everything, so
your autograder submissions will take less time.

This is especially useful if you combine it with `--run-git-remotely`. (See below)

## Using --run-git-remotely To Speedup Development

We've added a new feature to `runlab`.  You can tell `runlab` to build a
particular make target, so you can run more focused tests.

The three most useful make targets are:

* `make code.csv` -- Run a test based on your configuration in `config.env`.

* `make benchmark.csv` -- Run a test configured for the autograder.  `benchmark.csv` is what gradescope uses to check your performance.

* `make regressions.out` -- Run the regressions.  The results end up in `regressions.out`.

* `make` -- is equivalent to `make code.csv benchmark.csv regressions.out` (this is what gradescope runs)

To run just one of these, you can 

```
runlab --no-validate -- make code.csv'
```

and it will just build `code.csv`.

You can combine this option with `--run-git-remotely` to quickly run tests that
should reflect the performance you'll see on the autograder:

```
runlab --run-git-remotely -- make code.csv
```

`code.csv` will end up in your local directory and you look at it with

```
pretty-csv code.csv
```

This is much faster and simpler than submitting things through gradescope.  For
instance, setting `CMD_LINE_ARGS` like this:

```
CMD_LINE_ARGS="--function activate --test-layer 0 --reps 5"
```

and then

```
runlab --run-git-remotely -- make code.csv
```

Takes about 30 seconds compared to a couple of minutes submitting through gradescope.

**Note** the `--reps` option sets (very approximately) the number of
seconds the test will run for.  5 is good number -- it should give
pretty consistent results without taking too long.

## Tips

* There are many more things to try in this lab than there have been
  in the earlier labs.  This has two implication:

  * Start early.
 
  * "guess and check" is unlikely to get you a good solution in
    reasonable.  Think carefully about the changes you are making.
    Thinking takes time.  Start early.

  * The autograder servers are going to get very busy near the deadline.  Start early.

* Unfortunately, gprof doesn't work on multi-threaded programs.  You
  can comment out `OPENMP=yes` in `config.env` to make gprof work
  properly.

* Some reference for the GCC Auto-vectorization module:
   
   * https://gcc.gnu.org/projects/tree-ssa/vectorization.html#vectorizab

* OpenMP is a big library.  We've covered a bit of it.  You're free to use the rest.

* There's lot of resources on the web about OpenMP.  Many (or most) of
  them are bad.  This one is pretty good:
  http://jakascorner.com/blog/.  Especially these entries:

  * http://jakascorner.com/blog/2016/04/omp-introduction.html
  
  * http://jakascorner.com/blog/2016/05/omp-for.html
  
  * http://jakascorner.com/blog/2016/07/omp-critical.html

  * http://jakascorner.com/blog/2016/06/omp-for-scheduling.html

  * http://jakascorner.com/blog/2016/06/omp-data-sharing-attributes.html

  * http://jakascorner.com/blog/2016/07/omp-default-none-and-const.html

* There are several `fc_layer_t`, `conv_layer_t`, `relu_layer_t`, and `fc_layer_t`
  layers in the model for this lab.  You can, if you want, provide specialized
  implementations for each layer by checking the dimensions of layer's tensors.
  However, this may be unwise, since it will increase the number things you
  need optimize.  You can probably do just fine with a single version.
  
  