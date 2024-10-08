# A template for a collaborative benchmark repository

You can use this template to create a public, collaborative benchmarks of multi-component signal processing methods based on the benchmarking toolbox [```mcsm-benchs```](https://github.com/jmiramont/mcsm-benchs).

## Quickstart

1. Use this template to [create a new repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template).

>[!NOTE]
> Make sure you include all the branches of the template by checking the *"Include all branches"* option.

2. Create a local version of your new collaborative benchmark by [cloning the new repository](#benchmarking-a-new-method).
3. [Launch](#installation-using-poetry) a new python virtual environment and install the main dependencies using [```poetry```](https://python-poetry.org/docs/):

    ```bash
    poetry install --only main
    ```

4. [Add your methods](#adding-a-new-method-to-benchmark) by moving files that represent them in ```/src/methods``` folder and [adding new dependencies](#adding-dependencies).
5. [Configure](#changing-the-benchmark-configuration) your benchmark by changing [```config.yaml```](config.yaml).
6. [Run](#running-the-benchmark-with-new-methods) the script ```run_this_benchmark.py```.
7. Push your changes to your repository and wait for the results to get published.
8. [Share](#sharing-results) interactive results to others

>[!TIP]
>:bulb: Collaborators can add new methods to your benchmark [via a pull-request](#contributing-new-methods-to-an-online-benchmark).

## Benchmarking a new method

The instructions below will help you to add a new method and run the benchmark afterwards.
First you should have a local copy of this repository to add and modify files. Open a terminal in a directory of your preference and use

```bash
git clone https://github.com/USERNAME/REPOSITORY
```

Make sure to replace `USERNAME` and `REPOSITORY` with your GitHub username and repository name, respectively.

### Installation using ```poetry```

We use [```poetry```](https://python-poetry.org/docs/), a tool for dependency management and packaging in python to install the benchmarking framework. You can install ```poetry``` following the steps described [here](https://python-poetry.org/docs/#installation).
Then, make ```poetry``` create a virtual environment and install the main dependencies of the benchmarks using:

```bash
poetry install --only main
```

>[!NOTE]
>If you have [`Anaconda`](https://www.anaconda.com/) or [`Miniconda`](https://docs.conda.io/en/latest/miniconda.html) installed please disable the auto-activation of the base environment and your conda environment using:
>
>```bash
>conda config --set auto_activate_base false
>conda deactivate
>```

Benchmarking MATLAB-implemented methods is possible thanks to the incorporated [Matlab's Python engine](https://fr.mathworks.com/help/matlab/matlab-engine-for-python.html), that allows communication between `Python` and a `MATLAB`/`Octave` session. This module's version must be compatible with your local `MATLAB` installation, please [modify the dependencies for this package accordingly](#adding-dependencies).
Additionally, `matlabengine` is only compatible with certain `Python` versions, depending on the local `MATLAB` installation you are running. [Check that your versions of matlab and Python are compatible](https://www.mathworks.com/content/dam/mathworks/mathworks-dot-com/support/sysreq/files/python-compatibility.pdf).

You can now add your new method. You can run the benchmarks with only new added approaches.
However, if you want to reproduce the current results, [you will need extra dependencies](#reproducing-current-benchmarks).

## Adding a new method to benchmark

Whether your method is implemented in Python or Matlab, you must create a new ```.py``` file the name of which must start with *method_* and have certain content to be automatically discovered by the toolbox. The purpose of this file is to encapsulate your method in a new class. This is much easier than it sounds!.

To make it simpler, [a file called *method_new_basic_template.py* is made available](./new_method_example/method_new_basic_template.py) (for Python users) which you can use as a template. You just have to fill in the parts that implement your method. Matlab users can also find a template [here](./new_method_example/method_new_basic_template_matlab.py), as well as Octave users [here](./new_method_example/method_new_basic_template_octave.py)
A new method can then be tested against others by adding this file into the folder [src/methods](./src/methods). We shall see how to do this using a template file in the following sections.

### Python-based methods

First, the function implementing your method must have the following signature if you're working in python:

```python
    def a_new_method(signal, *args, **kwargs):
        ...
```

Methods should receive an ```(N,)``` numpy array representing a discrete-time signal, where and `N` is the number of time samples. Additionally, they should receive a variable number of input arguments to allow testing different combinations of input parameters. The ouput of the function must be a ```numpy``` array of [predefined dimensions according to the task.](#size-of-outputs-according-to-the-task)

In the first section of the template file [*method_new_basic_template.py*](./new_method_example/method_new_basic_template.py), you can import a function with your method or implement everything in the same file:

```python
""" First section ----------------------------------------------------------------------
| Import here all the modules you need.
| Remark: Make sure that neither of those modules starts with "method_".
"""
from mcsm_benchs.benchmark_utils import MethodTemplate
```

<!-- Additionally, the [abstract class](https://docs.python.org/3/library/abc.html) `MethodTemplate` is imported here. Abstract classes are not implemented, but they serve the purpose of establishing a template for new classes, by forcing the implementation of certain *abstract* methods. We will see later that the class that encapsulates your method must inherit from this template. -->

The second section of the file should include all the functions your method needs to work. This functions could also be defined in a separate module imported in the previous section as well.

```python
""" Second section ---------------------------------------------------------------------
| Put here all the functions that your method uses.
| 
| def a_function_of_my_method(signal, *args, **kwargs):
|   ...
"""
```

In the third and final section, your method is encapsulated in a new class called `NewMethod` (you can change this name if you prefer to, but it is not strictly necessary). The only requisite for the class that represents your method is that it inherits from the [abstract class](https://docs.python.org/3/library/abc.html) `MethodTemplate`. This simply means that you will have to implement the class constructor and a class function called -unsurprisingly- `method()`:

```python
""" Third section ----------------------------------------------------------------------
| Create here a new class that will encapsulate your method.
| This class should inherit the abstract class MethodTemplate.
| You must then implement the class function: 

def method(self, signal, params)
    ...
| which should receive the signals and any parameters that you desire to pass to your
| method.
"""

class NewMethod(MethodTemplate):
    def __init__(self):
        self.id = 'a_new_method'
        self.task = 'denoising'  # Should be either 'denoising', 'detection' or 'misc'

    def method(self, signals, params = None): # Implement this method.
        ...

    # def get_parameters(self):            # Use it to parametrize your method.
    #     return [None,]

```

The constructor function ```__init__(self)``` must initialize the attributes ```self.id``` and ```self.task```. The first is a string to identify your method in the benchmark. The second is the name of the task your method is devoted to.

Lastly, you have to implement the class function ```method(self, signals, *args, **kwargs)```. This function may act as a wrapper of your method, i.e. you implement your method elsewhere and call it from this function passing all the parameters in the process, or you could implement it directly here.

If you want to test your method using different sets of parameters, you can also implement the function `get_parameters()` to return a list with the desired input parameters (you can find an example of this [here](./new_method_example/method_new_with_parameters.py)).

Finally, **you have to move the file** with all the modifications to the folder [/src/methods](./src/methods).

>[!WARNING]
>Changing the name of the file is possible, but keep in mind that **the file's name must start with "*method_*" to be recognizable**.

### `MATLAB` and `Octave`-based methods

The `MATLAB`/`Octave` function implementing your method must have a particular signature. For example, for a method with two input parameters should be:

```matlab
    function [X]  = a_matlab_method(signal, param_1, param_2)
```

Your method can have all the (positional) input arguments you need. The output of the function must be a Matlab matrix of [predefined dimensions according to the task.](#size-of-outputs-according-to-the-task)

>[!NOTE]
> If your method returns more than one parameter, only the first one is taken as the output for the benchmark.

We now can see how to benchmark a method implemented in `MATLAB`/`Octave`.
Similarly to the case of Python-based methods, a template file is given [here](./new_method_example/method_new_basic_template_matlab.py) for interested users.

In the first section of this file, the class ```MatlabInterface``` is imported, which will simply act as an interface between `Python` and a `MATLAB` session where your method will be run:

```python

from mcsm_benchs.benchmark_utils import MethodTemplate
from mcsm_benchs.MatlabInterface import MatlabInterface
# You must import the MethodTemplate abstract class and the MatlabInterface class.

```

Then, you must  **move the ```.m``` file with your method to the folder ```src\methods```**. A convenient and neat way of doing this is by creating a folder with all the ```.m``` files related to your method, for example called ```a_matlab_method_utils```. After this you can now create a ```MatlabInterface``` instance that represents your method, by passing a string to the ```MatlabInterface``` creator with the name of the previously defined function. For example:

```python
# After moving a file called 'a_matlab_method.m' to src\methods, create an interface with the Matlab's Python engine by passing the name of the file (without the .m extension). Then get the matlab function as:
mlint = MatlabInterface('a_matlab_method') 
matlab_function = mlint.matlab_function # A python function handler to the method.

# Paths to additional code for the method to add to Matlab path variable.
paths = [
        'src\methods\a_matlab_method_utils',
        '..\src\methods\a_matlab_method_utils'
        ]
```

The last lines make sure that if you created a new folder named ```new_method_utils``` inside ```src\methods``` with the files related to your Matlab-implemented approach, these are available to the `MATLAB` session.

Now we are ready to complete the third section of the file. This can be used exactly as it is in the template file, provided you have done all the precedent steps.

```python
class NewMethod(MethodTemplate):

    def __init__(self):
        self.id = 'a_matlab_method'
        self.task = 'denoising'
        

    def method(self, signal, *params):
        """ A class method encapsulating a matlab function.

        Args:
            signals (numpy array): A signal.
            params: Any number of positional parameters.
        """
        signal_output = matlab_function(signal, *params) # Only positional args.

        return signal_output

```

<!-- The constructor function ```__init__(self)``` must initialize the attributes ```self.id``` and ```self.task```. The first is a string to identify your method in the benchmark. The second is the name of the task your method is devoted to. -->

>[!NOTE]
>The ```MatlabInterface``` class provided by [```mcsm-benchs```](https://github.com/jmiramont/mcsm-benchs) will cast the input parameters in the appropriate `MATLAB` types.

## Running the benchmark with new methods

Once the new methods are added, you can run a benchmark by executing the files ```run_this_benchmark.py``` located in the repository.
You can do this using the local environment created with ```poetry``` by running:

```bash
poetry run python run_this_benchmark_denoising.py
```

This will run the benchmark using new added methods, avoiding previously explored ones and saving time. You can change this from the configuration files as explained in the next section.

## Changing the benchmark configuration

The benchmark parameters can be modified using the config file [```config.yaml```](./config.yaml) located in the repository.
This file defines the input parameters of the benchmark:

```yaml
N: 1024 # Number of time samples
SNRin: [-20,-10,0,10,20] # Values of SNR to evaluate.
repetitions: 30 #
parallelize: 4  # If False, run the benchmark in a serialized way. If True or int, runs the benchmark in parallel with the indicated number of cores. 
verbosity: 4    # Controls the messages appearing in the console.
using_signals: [
                'McDampedCos',
                'McCrossingChirps',
                'McSyntheticMixture5',
              ]

add_new_methods: True # Run again the same benchmark but with new methods:
```

## Adding dependencies

Your method might need particular modules as dependencies that are not currently listed in the dependencies of the default benchmark. You can add all your dependencies by modifying the ```.toml``` file in the folder, under the ```[tool.poetry.dependencies]``` section. For example:

```bash
[tool.poetry.dependencies]
python = ">=3.8,<3.11"
numpy = "^1.22.0"
matplotlib = "^3.5.1"
pandas = "^1.3.5"

```

A more convenient and interactive way to do this interactively is by using ```poetry```, for instance:

```bash
poetry add numpy
```

and following the instructions prompted in the console.

## Contributing new methods to an online benchmark

To add new methods to an online benchmark, you first need to [fork the collaborative benchmark repository](https://docs.github.com/en/get-started/quickstart/fork-a-repo).
One easy way you can do this is by using the "Fork" button above:

![Repository header](docs/readme_figures/header_repo.png)

This will create a copy of the repository in your own GitHub account, the URL of which should look like

```bash
https://github.com/YOUR-USERNAME/FORKED-REPO-NAME
```

Now, let's create a local copy, i.e. in your computer, of the repository you have just forked. Open a terminal in a directory of your preference and use

```bash
git clone https://github.com/YOUR-USERNAME/FORKED-REPO-NAME
```

When a repository is forked, a copy of all the branches existing in the original one are also created.

Now, follow the instructions to [add your method to a benchmark](#adding-a-new-method-to-benchmark), in this case in a local copy of this repository, in order to be able to add and modify files.

After this, you can get your method added to the benchmark via a [Pull Request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests).
For this, you need first to update the remote version of your fork by committing the changes and then pushing them to your remote repository:

```bash
git commit --all -m "Uploading a new method"
git push origin new_method
```

Now you can create a [new pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork) by using the "Contribute" button from the fork created before:

![Start a pull request.](docs/readme_figures/start_a_pull_request.png)

and then *"Open a Pull Request"*. There you will need to select the branch where your changes are going to be made in the original repository of the benchmark.

Please choose here `new_methods``:

![Choose the right branch.](docs/readme_figures/pull_request_branch.png)

Finally, send the pull request by clicking on *"Create pull request"*:

![Create you pull request.](docs/readme_figures/finishing_pull_request.png)

You can add an short comment in the "Write" field.

An explanation of the new method and related references (notes, articles, etc.) will be appreciated.

## Modify ```matlabengine``` module version

Check the version of the ```matlabengine``` module you have to install to use run the benchmarks in the next table:

| Matlab Version | Python Version | ```matlabengine``` version |   |
|----------------|----------------|----------------------------|---|
| 2022b          | 3.8, 3.9, 3.10 | 9.13.6                    |   |
| 2022a          | 3.8, 3.9       | 9.12.17                    |   |
| 2021b          | 3.7, 3.8, 3.9  | 9.11.19                    |   |

Then, look for the ```matlabengine``` line in [```pyproject.toml```](./pyproject.toml), it should look like this:

```python
matlabengine = "9.12.17"
```

Make sure to change the version with the one corresponding to your Python and Matlab current versions. If you have an older version of Matlab or Python, you can search for the correct version of the ```matlabengine``` module [here](https://pypi.org/project/matlabengine/#history).

After this, run

```bash
poetry update
```

<!-- ### Checking everything is in order with ```pytest```

Once your dependencies are ready, you should check that everything is in order using the ```pytest``` testing suit. To do this, simply run the following in a console located in your local version of the repository:

```bash
poetry run pytest
```

This will check a series of important points for running the benchmark online, mainly:

1. Your method class inherits the ```MethodTemplate``` abstract class.
2. The inputs and outputs of your method follows the required format according to the designated task.

Once the tests are passed, you can now either create a pull request to run the benchmark remotely, or [run the benchmark locally](#running-this-benchmark-locally).
 -->

## Sharing results

## Reproducing current benchmarks

## Size of outputs according to the task

The shape and type of the output depends on the task.

- For `task = denoising`: The output must be a vector array with the same length as the signal.
- For `task = detection`: The output of the method must be a boolean variable indicating if a signal has been detected (true) or not (false).
- For `task = misc`: The output of the method has not size limitation, but a performance function capable of taking the output must be passed as an input parameter to the benchmark.
