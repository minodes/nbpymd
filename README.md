# nbpymd: [n]ote[b]ooks as [py]thon + [m]ark[d]own

`nbpymd` lets you manage Jupyter notebooks as plain Python code with embedded Markdown text, enabling:

* Higher code quality
* Version control
* Development of notebooks as regular Python code
* Parametrized notebooks
* Programmatic and command line notebook execution
* Reproducible results at any time
* Notebook execution cache

## Installation

`nbpymd` is compatible with `Python >= 3.4`. To install nbpymd on your system:

```
pip install nbpymd
```

## Usage

`nbpymd` can be used in two ways: as a command line tool and as a library. The command line tool `nbapp` is tailored for simplicity and is the fastest way to write & run a notebook. The library access provides a finer control on parametrization and execution.

On MacOS, ignore these warning messages:

```
RuntimeWarning: Failed to set sticky bit on [...]`. It's a known annoying (bug)[https://github.com/jupyter/jupyter_client/pull/201#issuecomment-314269710].
```

### Notebook format

A notebook is defined as a Python function with Markdown text embedded in multi-line string blocks. Notebooks can contain only Python and Markdown cells. Example:

```
# Contents of sum.py


def cells(a, b):
    '''
    # Sum of two integers
    '''

    a, b = int(a), int(b)

    ''''''

    a, b

    '''
    Sum:
    '''

    a + b
```

The parameter names of the `cells` function are mapped to command line arguments. In the example, we have two arguments `a` and `b`.

Lines whose content is either `'''` or `''''''` have a special meaning: Markdown cells are delimited by `'''` and `''''''` serves as separator between Python cells. Markdown cell delimiters serve also as separators between Python and Markdown cells. An empty Markdown cell `'''\n'''` is equivalent to `''''''`. Empty cells are ignored and trailing spaces or empty lines within cells are stripped away.

In presence of parameters, if the first cell is a Markdown cell, it is treated as the title and the injected Python cell with parameters is inserted as second cell.

### Using the `nbapp` command line tool

To run the notebook defined in ``sum.py`:

```
nbapp sum.py --param a=3 --param b=5
```

Parameters `a` and `b` are injected in the notebook at execution time as an additional Python cell.
A different Python function name can be specified by appending `:func_name` to the module pathname. E.g., `sum.py:func_name`.

#### Exporting

Options `--export-html` and `--export-ipynb` let you export to .html and .ipynb file formats, respectively. The special output pathname `-` points to standard output.

#### Caching

The caching system allows you to reuse transparently the Python sessions and outputs of previous notebook executions. Each cell is associated to an hash generated from these fields:

* **uid**: Unique identifier of the notebook. It is either the notebook module name or the class name. An additional id can be appended with the command line parameter `--append-id`.

* **kwargs**: Notebook parameters

* **cell**: Cell content

* **index**: Cell position

Cache hits speed up significantly the notebook execution. Cache misses cause the invalidation of the cache. The cache is maintained in temporary files. To clean the cache, remove the files manually with `rm /tmp/nbpymd-cache-*`.

### Using the library interface

To define a notebook, extend the `Notebook` class and define a `cells` method.
Example:


```
# Contents of sumapp.py

from nbpymd.notebook import Notebook


class SumNotebook(Notebook):
    def cells(self, a, b):
        a + b


if __name__ == "__main__":
    nb = SumNotebook()
    nb.add_argument('--a', default=5, type=int)
    nb.add_argument('--b', type=int)
    nb.add_argument('--print-ipynb', action="store_true", default=False)

    args = nb.run()

    if args.print_ipynb:
        nb.export_ipynb('-')
```

To run `sumapp.py`:

```
python3 sumapp.py --b 3 --print-ipynb
```

Class `SumNotebook` extends `Notebook` and defines the notebook in method `cells`.

Method `Notebook.add_argument` maps to (ArgumentParser.add_argument)[https://docs.python.org/2/library/argparse.html#argparse.ArgumentParser.add_argument] and lets you define additional notebook parameters or custom options.

Method `Notebook.run` takes care of executing the notebook taking into account the command line arguments, and returns the object returned by (ArgumentParser.parse_args)[https://docs.python.org/2/library/argparse.html#argparse.ArgumentParser.parse_args]. The user-defined parameter '--print-ipynb' is handled using it.

There must be an exact match between the parameter names of the `cells` function and the attribute names of the object returned by [ArgumentParser.parse_args].

All notebook parameter values that have no default value must be provided from command line. E.g., parameter `b` in the example above.


## License

The nbpymd project is released under the MIT license. Please see `LICENSE.txt`.


## Development

Tests, builds and releases are managed with `Fabric`.
The build, test and release environment is managed with `Docker`.

Install Docker and Fabric in your system. To install Fabric:

```
pip install Fabric3
```

### Dependencies

For ease of development, the file `requirements.txt` includes the package dependencies.
Any changes to the package dependencies in `setup.py` must be reflected in `requirements.txt`.

### Jupyter server

The Jupyter server is reachable at [http://127.0.0.1:8889/tree].

### Releasing

Create a file `secrets.py` in the project directory with the Pypi credentials in this format:

```
pypi_auth = {
    'user': 'youruser',
    'pass': 'yourpass'
}
```

To release a new version:

```
fab release
```


### Tests

To run the module tests:

```
fab test
```

To run a single test:

```
fab test:tests/test_nbapp.py::test_nbapp_cells
```

To run tests printing output and stopping at first error:

```
fab test_sx
```

To run the pep8 test:

```
fab test_pep8
```

To fix some common pep8 errors in the code:

```
fab fix_pep8
```

To test the pip package after a new release (end-to-end test):
```
fab test_pip
```

### Docker container

To build the Docker image:

```
fab docker_build
```

To force a complete rebuild of the Docker image without using the cache:

```
fab docker_build:--no-cache
```

To start the daemonized Docker container:

```
fab docker_start
```

Top stop the Docker container:

```
fab docker_stop
```

To open a shell in the Docker container:

```
fab docker_sh
```
