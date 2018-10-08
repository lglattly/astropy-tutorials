.. meta::
    :keywords: filterTutorials, filterUnits, filterCosmology, filterMatplotlib






.. raw:: html

    <a href="../_static/redshift-plot/redshift-plot.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/redshift-plot/redshift-plot.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _redshift-plot:

Make a plot with both redshift and universe age axes using astropy.cosmology
============================================================================

Authors
-------

Neil Crighton

Learning Goals
--------------

-  TODO

Keywords
--------

units, cosmology, matplotlib

Summary
-------

Each redshift corresponds to an age of the universe, so if you're
plotting some quantity against redshift, it's often useful show the
universe age too. The relationship between the two changes depending the
type of cosmology you assume, which is where ``astropy.cosmology`` comes
in. In this tutorial we'll show how to use the tools in
``astropy.cosmology`` to make a plot like this:


:inputnumrole:`In[1]:`


.. code:: python

    # Set up matplotlib
    import matplotlib.pyplot as plt
    %matplotlib inline


:inputnumrole:`In[2]:`


.. code:: python

    from IPython.display import Image
    Image(filename="ang_dist.png", width=500)


:outputnumrole:`Out[2]:`




.. image:: nboutput/redshift-plot_2_0.png




We start with a cosmology object. We will make a flat cosmology (which
means that the curvature density :math:`\Omega_k=0`) with a hubble
parameter of :math:`70` km/s/Mpc and matter density :math:`\Omega_M=0.3`
at redshift 0. The ``FlatLambdaCDM`` cosmology then automatically infers
that the dark energy density :math:`\Omega_\Lambda` must :math:`=0.7`,
because :math:`\Omega_M + \Omega_\Lambda + \Omega_k = 1`.


:inputnumrole:`In[3]:`


.. code:: python

    from astropy.cosmology import FlatLambdaCDM
    
    # In this case we just need to define the matter density 
    # and hubble parameter at z=0.
    
    # Note the default units for the hubble parameter H0 are km/s/Mpc. 
    # You can also pass an astropy `Quantity` with the units specified. 
    
    cosmo = FlatLambdaCDM(H0=70, Om0=0.3)

Note that we could instead use one of the built-in cosmologies, like
``WMAP9`` or ``Planck13``, in which case we would just redefine the
``cosmo`` variable.

Now we need an example quantity to plot versus redshift. Let's use the
angular diameter distance, which is the physical transverse distance
(the size of a galaxy, say) corresponding to a fixed angular separation
on the sky. To calculate the angular diameter distance for a range of
redshifts:


:inputnumrole:`In[4]:`


.. code:: python

    import numpy as np
    zvals = np.arange(0, 6, 0.1)
    dist = cosmo.angular_diameter_distance(zvals)

Note that we passed an array of redshifts to
``cosmo.angular_diameter_distance`` and it produced a corresponding
array of distance values, one for each redshift. Let's plot them:


:inputnumrole:`In[5]:`


.. code:: python

    plt.rc('xtick.major', size=4)
    plt.rc('ytick.major', size=4)
    plt.rc('xtick.minor', size=2)
    plt.rc('ytick.minor', size=2)
    plt.rc('axes', grid=False)
    plt.rc('xtick.major', width=1)
    plt.rc('xtick.minor', width=1)
    plt.rc('ytick.major', width=1)
    plt.rc('ytick.minor', width=1)
    plt.rc('lines', marker='')
    
    fig = plt.figure(figsize=(6,4))
    ax = fig.add_subplot(111)
    ax.plot(zvals, dist);


:outputnumrole:`Out[5]:`



.. image:: nboutput/redshift-plot_8_0.png



To check the units of the angular diameter distance, take a look at the
unit attribute:


:inputnumrole:`In[6]:`


.. code:: python

    dist.unit


:outputnumrole:`Out[6]:`




.. math::

    \mathrm{Mpc}



Now let's put some age labels on the top axis. We're going to pick a
series of round age values where we want to place axis ticks. You may
need to tweak these depending on your redshift range to get nice, evenly
spaced ticks.


:inputnumrole:`In[7]:`


.. code:: python

    import astropy.units as u
    ages = np.array([13, 10, 8, 6, 5, 4, 3, 2, 1.5, 1.2, 1])*u.Gyr

To link the redshift and age axes, we have to find the redshift
corresponding to each age. The function ``z_at_value`` does this for us.


:inputnumrole:`In[8]:`


.. code:: python

    from astropy.cosmology import z_at_value
    ageticks = [z_at_value(cosmo.age, age) for age in ages]

Now we make the second axes, and set the tick positions using these
values.


:inputnumrole:`In[9]:`


.. code:: python

    fig = plt.figure(figsize=(6,4))
    ax = fig.add_subplot(111)
    ax.plot(zvals, dist)
    ax2 = ax.twiny()
    ax2.set_xticks(ageticks);


:outputnumrole:`Out[9]:`



.. image:: nboutput/redshift-plot_16_0.png



We have ticks on the top axis at the correct ages, but they're labelled
with the redshift, not the age. Fix this by setting the tick labels by
hand.


:inputnumrole:`In[10]:`


.. code:: python

    fig = plt.figure(figsize=(6,4))
    ax = fig.add_subplot(111)
    ax.plot(zvals, dist)
    ax2 = ax.twiny()
    ax2.set_xticks(ageticks)
    ax2.set_xticklabels(['{:g}'.format(age) for age in ages.value]);


:outputnumrole:`Out[10]:`



.. image:: nboutput/redshift-plot_18_0.png



We need to make sure the top and bottom axes have the same redshift
limits. They may not line up properly in the above plot, for example,
depending on your setup (the age of the universe should be ~13 Gyr at
z=0).


:inputnumrole:`In[11]:`


.. code:: python

    fig = plt.figure(figsize=(6,4))
    ax = fig.add_subplot(111)
    ax.plot(zvals, dist)
    ax2 = ax.twiny()
    ax2.set_xticks(ageticks)
    ax2.set_xticklabels(['{:g}'.format(age) for age in ages.value])
    zmin, zmax = 0.0, 5.9
    ax.set_xlim(zmin, zmax)
    ax2.set_xlim(zmin, zmax);


:outputnumrole:`Out[11]:`



.. image:: nboutput/redshift-plot_20_0.png



We're almost done. We just need to label all the axes, and add some
minor ticks. Let's also tweak the y axis limits to avoid putting labels
right near the top of the plot.


:inputnumrole:`In[12]:`


.. code:: python

    fig = plt.figure(figsize=(6,4))
    ax = fig.add_subplot(111)
    ax.plot(zvals, dist)
    ax2 = ax.twiny()
    ax2.set_xticks(ageticks)
    ax2.set_xticklabels(['{:g}'.format(age) for age in ages.value])
    zmin, zmax = 0, 5.9
    ax.set_xlim(zmin, zmax)
    ax2.set_xlim(zmin, zmax)
    ax2.set_xlabel('Time since Big Bang (Gyr)')
    ax.set_xlabel('Redshift')
    ax.set_ylabel('Angular diameter distance (Mpc)')
    ax.set_ylim(0, 1890)
    ax.minorticks_on()


:outputnumrole:`Out[12]:`



.. image:: nboutput/redshift-plot_22_0.png



Now for comparison, let's add the angular diameter distance for a
different cosmology, from the Planck 2013 results. And then finally, we
save the figure to a png file.


:inputnumrole:`In[13]:`


.. code:: python

    from astropy.cosmology import Planck13
    dist2 = Planck13.angular_diameter_distance(zvals)
    
    fig = plt.figure(figsize=(6,4))
    ax = fig.add_subplot(111)
    ax.plot(zvals, dist2, label='Planck 2013')
    ax.plot(zvals, dist, label=
            '$h=0.7,\ \Omega_M=0.3,\ \Omega_\Lambda=0.7$')
    ax.legend(frameon=0, loc='lower right')
    ax2 = ax.twiny()
    ax2.set_xticks(ageticks)
    ax2.set_xticklabels(['{:g}'.format(age) for age in ages.value])
    zmin, zmax = 0.0, 5.9
    ax.set_xlim(zmin, zmax)
    ax2.set_xlim(zmin, zmax)
    ax2.set_xlabel('Time since Big Bang (Gyr)')
    ax.set_xlabel('Redshift')
    ax.set_ylabel('Angular diameter distance (Mpc)')
    ax.minorticks_on()
    ax.set_ylim(0, 1890)
    fig.savefig('ang_dist.png', dpi=200, bbox_inches='tight')


:outputnumrole:`Out[13]:`



.. image:: nboutput/redshift-plot_24_0.png



``bbox_inches='tight'`` automatically trims any whitespace from around
the plot edges.

And we're done!

Exercise
--------

Well, almost done. Notice that we calculated the times on the upper axis
using the original cosmology, not the new cosmology based on the Planck
2013 results. So strictly speaking, this axis applies only to the
original cosmology, although the difference between the two is small. As
an exercise, you can try plot two different upper axes, slightly offset
from each other, to show the times corresponding to each cosmology. Take
a look at the first answer to `this question on Stack
Overflow <http://stackoverflow.com/questions/7733693/matplotlib-overlay-plots-with-different-scales>`__
for some hints on how to go about this.


:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

