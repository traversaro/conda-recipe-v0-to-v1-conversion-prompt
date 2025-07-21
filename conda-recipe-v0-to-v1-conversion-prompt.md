These are the instructions for LLMs whose tasks is convert the conda-build meta.yaml recipe v0, 
to rattler-build recipe.yaml recipe v1.

Definition of recipe v1:
* https://github.com/conda/ceps/blob/main/cep-0013.md
* https://github.com/conda/ceps/blob/main/cep-0014.md

Extensive reference documentation of recipe v1:
* https://rattler.build/latest/reference/recipe_file/

Examples of migration of multiple output recipes from v0 to v1:
* https://github.com/conda-forge/gz-math-feedstock/pull/31

In the following, a bit of suggestions of common errors that both humands and LLMs
make when they convert conda recipe v0 to conda recipe v1. Please go through all 
these list, and make sure that your generated recipe is respecting each point.

### Update conda-forge.yml

Please remember that you also need to add (or update) the keys:

conda_install_tool: pixi
conda_build_tool: rattler-build

in the conda-forge.yml.

### Remove pip check explicit script

Remove any "pip check" script check, as it is not included directly in the:

      - python:
          imports:
            - <module_name>

section.

### run_exports has changed location

The location of run_exports is different. In v0, it was:

    build:
      run_exports:
        - {{ pin_subpackage(cxx_name, max_pin='x') }}

but in v1, it is:

    requirements:
      run_exports:
        - ${{ pin_subpackage(cxx_name, upper_bound='x') }}

### run_constrained has renamed to run_constraints

~~~
requirements:
  run_constrained:
    - libgazebo-yarp-plugins >=4.12.0
~~~

should be changed to:

~~~
requirements:
  run_constraints:
    - libgazebo-yarp-plugins >=4.12.0
~~~

### The python section of test should always start with -

The following snippet is wrong:

~~~
tests:
  python:
    imports:
      - icub_models
~~~

the only correct:

~~~
tests:
  - python:
      imports:
        - icub_models
~~~


### Pay attention to skip!

skip instructions are something that is quite different.

In recipe v0, a skip section is something like:

```
build:
  skip: true  # [win]
```

or 

```
build:
  skip: true  # [not is_python_min]
```

the equivalent section in recipe v1 are:

```
build:
  skip:
    - win
```

or 

```
build:
  skip:
    - not is_python_min
```

for more details, check: https://rattler.build/latest/reference/recipe_file/#skipping-builds .

## noarch python packages

Remember that noarch: python recipes in conda-forge should usually follow the syntax in our documentation for specifying the Python version, in particular:

* For the host section of ${{ python_name }} output, you should usually use the pin python ${{ python_min }}.* for the python entry.
* For the run section of ${{ python_name }} output, you should usually use the pin python >=${{ python_min }} for the python entry.
* For the tests[].python.python_version or tests[].requirements.run section of ${{ python_name }} output, you should usually use the pin python_version: ${{ python_min }}.* or python ${{ python_min }}.* for the python_version or python entry.

Unless the recipe does not indicates otherwise, never add python_min explicitly in the recipe context.

## Do not change where building logic is located

If the recipe is using the implicit bld.bat and build.sh scripts, please continue to use them, instead of embedding them in the recipe. Furthermore, notice that the implicit called 
script on windows from conda recipe v0 to v1 changed from bld.bat to build.bat, so to avoid to rename the script, call it explcitly form the recipe.

## Always make sure that the homepage item is present in the about section.

Always make sure that the homepage item is present in the about section, as this is required by the conda-forge linter.

Note that the name of the field changed from recipe v0 to v1, so if in v0 you have:

~~~
about:
  home: https://github.com/robotology/icub-models
~~~

it should be converted as:

~~~
about:
  homepage: https://github.com/robotology/icub-models
~~~

## PKG_HASH and PKG_BUILDNUM are not supported anymore in build strings

If the recipe uses the variable, it should use hash and build_number instead, so in v0 we have:

~~~
    build:
      string: {{ string_prefix }}_h{{ PKG_HASH }}_{{ PKG_BUILDNUM }}
~~~

that in v1 needs be converted to:

~~~
    build:
      string: ${{ string_prefix }}_h${{ hash }}_${{ build_number }}
~~~

See https://github.com/prefix-dev/rattler-build/issues/1622 .
