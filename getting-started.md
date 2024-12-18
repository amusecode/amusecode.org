---
layout: default
title: Getting started
nav_order: 5
permalink: /getting-started
---

## Introduction

AMUSE is based on python, so if you're new to Python, you'll find the
official [Python documentation](http://docs.python.org/) a valuable
resource. Like with Python, there are basically two ways to use AMUSE.
Firstly, directly via the interactive (Python) command line:

```bash
> python
Python 3.10.10 (main, Feb 24 2023, 04:10:25) [Clang 14.0.0 (clang-1400.0.29.202)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 
>>> quit()    
```

Secondly, by writing (Python) scripts. Suppose you wrote the following
script myscript.py, and saved it in the current working directory:

```python
from amuse.units.units import *
from amuse.units import constants

def convert_to_freq(wavelengths = [355.1, 468.6, 616.5, 748.1, 893.1] | nano(m)):
    """
    This function converts wavelength to frequency, using the speed of
    light in vacuum.
    """
    print(f"The speed of light in vacuum: {constants.c}")
    print("wavelength -->  frequency")
    for wavelength in wavelengths:
        print(f"{wavelength}  --> {(constants.c/wavelength).as_quantity_in(giga(Hz))}")
```

Then this script can be executed from the AMUSE interactive command
line:

```python
>>> import myscript
>>> help(myscript) # Tells you what myscript can do, ...
>>>    # ... for example that it has a function to convert wavelength to frequency.
>>> myscript.convert_to_freq()
The speed of light in vacuum: 299792458.0 m * s**-1
wavelength -->  frequency
355.1 nm   -->  844247.98085 GHz
468.6 nm   -->  639761.967563 GHz
616.5 nm   -->  486281.359286 GHz
748.1 nm   -->  400738.481486 GHz
893.1 nm   -->  335676.24902 GHz
>>> from amuse.units.units import *
>>> myscript.convert_to_freq([21.0, 18.0, 6.0] | cm)
The speed of light in vacuum: 299792458.0 m * s**-1
wavelength -->  frequency
21.0 cm   -->  1.42758313333 GHz
18.0 cm   -->  1.66551365556 GHz
6.0 cm   -->  4.99654096667 GHz
>>> quit()
```

You can also run scripts directly from the terminal prompt. Calling
python with a file name argument will execute the file. For
this you need to add the following line to your script, telling the
script which of its functions to call when executed:

```python
if __name__ == '__main__':
    convert_to_freq()
```

Your script can now be executed directly from the terminal prompt:

```bash
> python myscript.py

The speed of light in vacuum: 299792458.0 m \* s\*\*-1 wavelength --\>
frequency 355.1 nm --\> 844247.98085 GHz 468.6 nm --\> 639761.967563
GHz 616.5 nm --\> 486281.359286 GHz 748.1 nm --\> 400738.481486 GHz
893.1 nm --\> 335676.24902 GHz
```
## Example interactive session

This is an example of an interactive session with AMUSE, showing how the
interface to a typical (gravitational dynamics) legacy code works. Using
the Barnes & Hut Tree code, the dynamics of the Sun-Earth system is
solved. This two-body problem is chosen for simplicity, and is, of
course, not exactly what a Tree code normally is used for. First we
import the necessary AMUSE modules.

```python
>>> from amuse.community.bhtree import Bhtree
>>> from amuse.datamodel import Particles
>>> from amuse.units import nbody_system
>>> from amuse.units import units
```

Gravitational dynamics legacy codes usually work with [N-body
units](https://en.wikipedia.org/wiki/N-body_units) internally. We have
to tell the code how to convert these to the natural units of the
specific system, when creating an instance of the legacy code class.

```python
>>> convert_nbody = nbody_system.nbody_to_si(1.0 | units.MSun, 149.5e6 | units.km)
>>> instance = Bhtree(convert_nbody)
```

Now we can tell the instance to change one of its parameters, before it
initializes itself:

```python
>>> instance.parameters.epsilon_squared = 0.001 | units.au**2
```

Then we create two particles, with properties set to those of the Sun
and the Earth, and hand them over to the BHTree instance.

```python
>>> stars = Particles(2)
>>> sun = stars[0]
>>> sun.mass = 1.0 | units.MSun
>>> sun.position = [0.0, 0.0, 0.0] | units.m
>>> sun.velocity = [0.0, 0.0, 0.0] | units.m / units.s
>>> sun.radius = 1.0 | units.RSun
>>> earth = stars[1]
>>> earth.mass = 5.9736e24 | units.kg
>>> earth.radius = 6371.0 | units.km 
>>> earth.position = [1.0, 0.0, 0.0] | units.au
>>> earth.velocity = [0.0, 29783, 0.0] | units.m / units.s
>>> instance.particles.add_particles(stars)
```

We need to setup a channel to copy values from the code to our model in
python:

```python
>>> channel = instance.particles.new_channel_to(stars)
```

Now the model can be evolved up to a specified end time. The current
values of the particles are retieved from the legacy code by using copy
from the channel.

```python
>>> print(earth.position[0])
149597870691.0 m
>>> print(earth.position.in_(units.au)[0])
1.0 AU
>>> instance.evolve_model(1.0 | units.yr)
>>> print(earth.position.in_(units.au)[0])  # This is the outdated value! (should update_particles first)
1.0 AU
>>> channel.copy()
>>> print(earth.position.in_(units.au)[0])
0.999843742682 au
>>> instance.evolve_model(1.5 | units.yr)
>>> channel.copy()
>>> print(earth.position.in_(units.au)[0])
-1.0024037469 au
```

It's always a good idea to clean up after you're finished:

```python
>>> instance.stop()
```

## Example scripts

In the
[test/examples](https://github.com/amusecode/amuse/tree/master/examples)
subdirectory several example scripts are included. They show how the
different legacy codes can be used. One such example is
[test\_HRdiagram\_cluster.py](https://github.com/amusecode/amuse/blob/master/examples/applications/test_HRdiagram_cluster.py).
It has several optional arguments. The example script can be executed
from the AMUSE command line as well as from the terminal prompt (in the
latter case use -h to get a list of the available command line options):

```python
>>> import test_HRdiagram_cluster
>>> test_HRdiagram_cluster.simulate_stellar_evolution()
The evolution of  1000  stars will be  simulated until t= 1000.0 Myr ...
Using SSE legacy code for stellar evolution.
Deriving a set of  1000  random masses following a Salpeter IMF between 0.1 and 125 MSun (alpha = -2.35).
Initializing the particles
Start evolving...
Evolved model successfully.
Plotting the data...
All done!
>>> from amuse.units.units import *
>>> test_HRdiagram_cluster.simulate_stellar_evolution(end_time=5000 | Myr)
The evolution of  1000  stars will be  simulated until t= 5000 Myr ...
...
```

```bash
> python test_HRdiagram_cluster.py -h
Usage: test_HRdiagram_cluster.py [options]

This script will generate HR diagram for an 
evolved cluster of stars with a Salpeter mass 
distribution.

Options:
  -h, --help            show this help message and exit
...
> python test_HRdiagram_cluster.py
The evolution of  1000  stars will be  simulated until t= 1000.0 Myr ...
...
```

If instead of "Plotting the data..." the script printed "Unable to
produce plot: couldn't find matplotlib.", this probably means you do not
have Matplotlib installed. See the subsection on Matplotlib\_ below.

### Matplotlib

Matplotlib is a python plotting library which produces publication
quality figures. Many of the AMUSE example scripts use this library to
produce graphical output. If you would like to take advantage of this
library, get it from <https://matplotlib.org/> and install it in the
Python site-packages directory. For your own work, it is of course also
possible to print the required output to the terminal and use your
favourite plotting tool to make the figures.


## Further tutorials

Further tutorials, also on more advanced topics, can be found [here](https://amuse.readthedocs.io/en/latest/tutorial/index.html).
