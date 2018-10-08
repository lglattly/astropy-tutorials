.. meta::
    :keywords: filterTutorials, filterUnits, filterMatplotlib, filterRadioAstronomy






.. raw:: html

    <a href="../_static/quantities/quantities.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/quantities/quantities.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _quantities:

Using Astropy Quantities for astrophysical calculations
=======================================================

Authors
-------

Ana Bonaca, Erik Tollerud, Jonathan Foster

Learning Goals
--------------

-  TODO

Keywords
--------

units, matplotlib, radio astronomy

Summary
-------

In this tutorial we present some examples showing how astropy's
``Quantity`` object can make astrophysics calculations easier. The
examples include calculating the mass of a galaxy from its velocity
dispersion and determining masses of molecular clouds from CO intensity
maps. We end with an example of good practices for using quantities in
functions you might distribute to other people.

For an in-depth discussion of ``Quantity`` objects, see the `astropy
documentation
section <http://docs.astropy.org/en/stable/units/quantity.html>`__.

Preliminaries
-------------

We start by loading standard libraries and set up plotting for ipython
notebooks.


:inputnumrole:`In[1]:`


.. code:: python

    import numpy as np
    import matplotlib.pyplot as plt
    
    # You shouldn't use the `seed` function in real science code, but we use it here for example purposes.
    # It makes the "random" number generator always give the same numbers wherever you run it.
    np.random.seed(12345)
    
    # Set up matplotlib
    import matplotlib.pyplot as plt
    %matplotlib inline

It is conventional to load the astropy ``units`` module as the variable
``u``, demonstrated below. This will make working with ``Quantity``
objects much easier.

Astropy also has a ``constants`` module, where typical physical
constants are available. The constants are stored as objects of a
subclass of ``Quantity``, so they behave just like a ``Quantity``. Here,
we'll only need the gravitational constant ``G``, Planck's constant
``h``, and Boltzmann's constant, ``k_B``.


:inputnumrole:`In[2]:`


.. code:: python

    import astropy.units as u
    from astropy.constants import G, h, k_B

1. Galaxy mass
--------------

In this first example, we will use ``Quantity`` objects to estimate a
hypothetical galaxy's mass, given its half-light radius and radial
velocities of stars in the galaxy.

Lets assume that we measured the half light radius of the galaxy to be
29 pc projected on the sky at the distance of the galaxy. This radius is
often called the "effective radius", so we will store it as a
``Quantity`` object with the name ``Reff``. The easiest way to create a
``Quantity`` object is just by multiplying the value with its unit.
Units are accessed as u."unit", in this case u.pc.


:inputnumrole:`In[3]:`


.. code:: python

    Reff = 29 * u.pc

A completely equivalent (but more verbose) way of doing the same thing
is to use the ``Quantity`` object's initializer, demonstrated below. In
general, the simpler form (above) is preferred, as it is closer to how
such a quantity would actually be written in text. The initalizer form
has more options, though, which you can learn about from the `astropy
reference documentation on
Quantity <http://docs.astropy.org/en/stable/api/astropy.units.quantity.Quantity.html>`__.


:inputnumrole:`In[4]:`


.. code:: python

    Reff = u.Quantity(29, unit=u.pc)

We can access the value and unit of a ``Quantity`` using the ``value``
and ``unit`` attributes.


:inputnumrole:`In[5]:`


.. code:: python

    print("""Half light radius
    value: {0}
    unit: {1}""".format(Reff.value, Reff.unit))


:outputnumrole:`Out[5]:`


.. parsed-literal::

    Half light radius
    value: 29.0
    unit: pc


The ``value`` and ``unit`` attributes can also be accessed within the
print function.


:inputnumrole:`In[6]:`


.. code:: python

    print("""Half light radius
    value: {0.value}
    unit: {0.unit}""".format(Reff))


:outputnumrole:`Out[6]:`


.. parsed-literal::

    Half light radius
    value: 29.0
    unit: pc


Furthermore, we can convert the radius in parsecs to any other unit of
length using the ``to()`` method. Here, we convert it to meters.


:inputnumrole:`In[7]:`


.. code:: python

    print("{0:.3g}".format(Reff.to(u.m)))


:outputnumrole:`Out[7]:`


.. parsed-literal::

    8.95e+17 m


Next, we will first create a synthetic dataset of radial velocity
measurements, assuming a normal distribution with a mean velocity of 206
km/s and a velocity dispersion of 4.3 km/s.


:inputnumrole:`In[8]:`


.. code:: python

    vmean = 206
    sigin = 4.3
    v = np.random.normal(vmean, sigin, 500)*u.km/u.s


:inputnumrole:`In[9]:`


.. code:: python

    print("""First 10 radial velocity measurements: 
    {0}
    {1}""".format(v[:10], v.to(u.m/u.s)[:10]))


:outputnumrole:`Out[9]:`


.. parsed-literal::

    First 10 radial velocity measurements: 
    [205.11975706 208.05945635 203.76641353 203.61035969 214.45285646
     211.99164508 206.39950387 207.21150846 209.30679704 211.35966937] km / s
    [205119.75706422 208059.45635365 203766.41352526 203610.35969131
     214452.85646176 211991.64508178 206399.50387    207211.50845717
     209306.79704073 211359.66936646] m / s



:inputnumrole:`In[10]:`


.. code:: python

    plt.figure()
    plt.hist(v, bins='auto', histtype="step")
    plt.xlabel("Velocity (km/s)")
    plt.ylabel("N")


:outputnumrole:`Out[10]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7f8ce351ee10>




.. image:: nboutput/quantities_21_1.png



Next, we calculate the velocity dispersion of the galaxy. This
demonstrates how you can perform basic operations like subtraction and
division with ``Quantity`` objects, and also use them in standard numpy
functions such as ``mean()`` and ``size()``. They retain their units
through these operations just as you would expect them to.


:inputnumrole:`In[11]:`


.. code:: python

    sigma = np.sqrt(np.sum((v - np.mean(v))**2) / np.size(v))
    print("Velocity dispersion: {0:.2f}".format(sigma))


:outputnumrole:`Out[11]:`


.. parsed-literal::

    Velocity dispersion: 4.36 km / s


Note how we needed to use ``numpy`` square root function, because the
resulting velocity dispersion quantity is a ``numpy`` array. If we used
the python standard ``math`` library's ``sqrt`` function instead, we get
an error.


:inputnumrole:`In[12]:`


.. code:: python

    sigma_scalar = np.sqrt(np.sum((v - np.mean(v))**2) / len(v))

In general, you should only use ``numpy`` functions with ``Quantity``
objects, *not* the ``math`` equivalents, unless you are sure you
understand the consequences.

Now for the actual mass calculation. If a galaxy is pressure-supported
(for example, an elliptical or dwarf spheroidal galaxy), its mass within
the stellar extent can be estimated using a straightforward formula:
:math:`M_{1/2}=4\sigma^2 R_{eff}/G`. There are caveats to the use of
this formula for science - see Wolf et al. 2010 for details. For
demonstrating ``Quantity``, just accept that this is often good enough.
For the calculation we can just multiply the quantities together, and
``astropy`` will keep track of the units.


:inputnumrole:`In[13]:`


.. code:: python

    M = 4*sigma**2*Reff/G
    M


:outputnumrole:`Out[13]:`




.. math::

    3.3085932 \times 10^{13} \; \mathrm{\frac{km^{2}\,kg\,pc}{m^{3}}}



The result is in a composite unit, so it's not really obvious it's a
mass. However, it can be decomposed to cancel all of the length units
(:math:`km^2 pc/m^3`) using the decompose() method.


:inputnumrole:`In[14]:`


.. code:: python

    M.decompose()


:outputnumrole:`Out[14]:`




.. math::

    1.0209252 \times 10^{36} \; \mathrm{kg}



We can also easily express the mass in whatever form you like - solar
masses are common in astronomy, or maybe you want the default SI and CGS
units.


:inputnumrole:`In[15]:`


.. code:: python

    print("""Galaxy mass
    in solar units: {0:.3g}
    SI units: {1:.3g}
    CGS units: {2:.3g}""".format(M.to(u.Msun), M.si, M.cgs))


:outputnumrole:`Out[15]:`


.. parsed-literal::

    Galaxy mass
    in solar units: 5.13e+05 solMass
    SI units: 1.02e+36 kg
    CGS units: 1.02e+39 g


Or, if you want the log of the mass, you can just use ``np.log10`` as
long as the logarithm's argument is dimensionless.


:inputnumrole:`In[16]:`


.. code:: python

    np.log10(M / u.Msun)


:outputnumrole:`Out[16]:`




.. math::

    5.7104737 \; \mathrm{}



However, you can't take the log of something with units, as that is not
mathematically sensible.


:inputnumrole:`In[17]:`


.. code:: python

    np.log10(M)


:outputnumrole:`Out[17]:`


::


    

    UnitConversionErrorTraceback (most recent call last)

    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity_helper.py in get_converter(from_unit, to_unit)
         28     try:
    ---> 29         scale = from_unit._to(to_unit)
         30     except UnitsError:


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _to(self, other)
        931         raise UnitConversionError(
    --> 932             "'{0!r}' is not a scaled version of '{1!r}'".format(self, other))
        933 


    UnitConversionError: 'Unit("kg km2 pc / m3")' is not a scaled version of 'Unit(dimensionless)'

    
    During handling of the above exception, another exception occurred:


    UnitConversionErrorTraceback (most recent call last)

    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity_helper.py in helper_dimensionless_to_dimensionless(f, unit)
        156     try:
    --> 157         return ([get_converter(unit, dimensionless_unscaled)],
        158                 dimensionless_unscaled)


    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity_helper.py in get_converter(from_unit, to_unit)
         31         return from_unit._apply_equivalencies(
    ---> 32                 from_unit, to_unit, get_current_unit_registry().equivalencies)
         33     except AttributeError:


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _apply_equivalencies(self, unit, other, equivalencies)
        868             "{0} and {1} are not convertible".format(
    --> 869                 unit_str, other_str))
        870 


    UnitConversionError: 'kg km2 pc / m3' (mass) and '' (dimensionless) are not convertible

    
    During handling of the above exception, another exception occurred:


    UnitTypeErrorTraceback (most recent call last)

    <ipython-input-17-598955917a11> in <module>()
    ----> 1 np.log10(M)
    

    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity.py in __array_ufunc__(self, function, method, *inputs, **kwargs)
        618         # consistent units between two inputs (e.g., in np.add) --
        619         # and the unit of the result (or tuple of units for nout > 1).
    --> 620         converters, unit = converters_and_unit(function, method, *inputs)
        621 
        622         out = kwargs.get('out', None)


    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity_helper.py in converters_and_unit(function, method, *args)
        536 
        537         # Determine possible conversion functions, and the result unit.
    --> 538         converters, result_unit = ufunc_helper(function, *units)
        539 
        540         if any(converter is False for converter in converters):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity_helper.py in helper_dimensionless_to_dimensionless(f, unit)
        160         raise UnitTypeError("Can only apply '{0}' function to "
        161                             "dimensionless quantities"
    --> 162                             .format(f.__name__))
        163 
        164 


    UnitTypeError: Can only apply 'log10' function to dimensionless quantities


Exercises
---------

Use ``Quantity`` and Kepler's law in the form given below to determine
the (circular) orbital speed of the Earth around the sun in km/s. You
should not have to look up an constants or conversion factors to do this
calculation - it's all in ``astropy.units`` and ``astropy.constants``.

.. math:: v = \sqrt{\frac{G M_{\odot}}{r}}


:inputnumrole:`In[None]:`



There's a much easier way to figure out the velocity of the Earth using
just two units or quantities. Do that and then compare to the Kepler's
law answer (the easiest way is probably to compute the percentage
difference, if any).


:inputnumrole:`In[None]:`



(Completely optional, but a good way to convince yourself of the value
of Quantity:) Do the above calculations by hand - you can use a
calculator (or python just for its arithmatic) but look up all the
appropriate conversion factors and use paper-and-pencil approaches for
keeping track of them all. Which one took longer?


:inputnumrole:`In[None]:`



2. Molecular cloud mass
-----------------------

In this second example, we will demonstrate how using ``Quantity``
objects can facilitate a full derivation of the total mass of a
molecular cloud using radio observations of isotopes of Carbon Monoxide
(CO).

Setting up the data cube
^^^^^^^^^^^^^^^^^^^^^^^^

Let's assume that we have mapped the inner part of a molecular cloud in
the J=1-0 rotational transition of :math:`{\rm C}^{18}{\rm O}` and are
interested in measuring its total mass. The measurement produced a data
cube with RA and Dec as spatial coordiates and velocity as the third
axis. Each voxel in this data cube represents the brightness temperature
of the emission at that position and velocity. Furthermore, we will
assume that we have an independent measurement of distance to the cloud
:math:`d=250` pc and that the excitation temperature is known and
constant throughout the cloud: :math:`T_{ex}=25` K.


:inputnumrole:`In[18]:`


.. code:: python

    d = 250 * u.pc
    Tex = 25 * u.K

We will generate a synthetic dataset, assuming the cloud follows a
Gaussian distribution in each of RA, Dec and velocity. We start by
creating a 100x100x300 numpy array, such that the first coordinate is
right ascension, the second is declination, and the third is velocity.
We use the ``numpy.meshgrid`` function to create data cubes for each of
the three coordinates, and then use them in the formula for a Gaussian
to generate an array with the synthetic data cube. In this cube, the
cloud is positioned at the center of the cube, with :math:`\sigma` and
the center in each dimension shown below. Note in particular that the
:math:`\sigma` for RA and Dec have different units from the center, but
``astropy`` automatically does the relevant conversions before computing
the exponential.


:inputnumrole:`In[19]:`


.. code:: python

    # Cloud's center
    cen_ra = 52.25 * u.deg
    cen_dec = 0.25 * u.deg
    cen_v = 15 * u.km/u.s
    
    # Cloud's size
    sig_ra = 3 * u.arcmin
    sig_dec = 4 * u.arcmin
    sig_v = 3 * u.km/u.s
    
    #1D coordinate quantities
    ra = np.linspace(52, 52.5, 100) * u.deg
    dec = np.linspace(0, 0.5, 100) * u.deg
    v = np.linspace(0, 30, 300) *u.km/u.s
    
    #this creates data cubes of size for each coordinate based on the dimensions of the other coordinates
    ra_cube, dec_cube, v_cube = np.meshgrid(ra, dec, v)
    
    data_gauss = np.exp(-0.5*((ra_cube-cen_ra)/sig_ra)**2 + 
                        -0.5*((dec_cube-cen_dec)/sig_dec)**2 + 
                        -0.5*((v_cube-cen_v)/sig_v)**2 )

The units of the exponential are dimensionless, so we multiply the data
cube by K to get brightness temperature units. Radio astronomers use a
rather odd set of units [K km/s] as of integrated intensity (that is,
summing all the emission from a line over velocity). As an aside for
experts, we're setting up our artificial cube on the main-beam
temperature scale (T:math:`_{\rm MB}`) which is the closest we can
normally get to the actual brightness temperature of our source.


:inputnumrole:`In[20]:`


.. code:: python

    data = data_gauss * u.K

We will also need to know the width of each velocity bin and the size of
each pixel, so we calculate that now.


:inputnumrole:`In[21]:`


.. code:: python

    # Average pixel size
    # This is only right if dec ~ 0, because of the cos(dec) factor.
    dra = (ra.max() - ra.min()) / len(ra)
    ddec = (dec.max() - dec.min()) / len(dec)
    
    #Average velocity bin width
    dv = (v.max() - v.min()) / len(v)
    print("""dra = {0}
    ddec = {1}
    dv = {2}""".format(dra.to(u.arcsec), ddec.to(u.arcsec), dv))


:outputnumrole:`Out[21]:`


.. parsed-literal::

    dra = 18.0 arcsec
    ddec = 18.0 arcsec
    dv = 0.1 km / s


We are interested in the integrated intensity over all of the velocity
channels, so we will create a 2D quantity array by summing our data cube
along the velocity axis (multiplying by the velocity width of a pixel).


:inputnumrole:`In[22]:`


.. code:: python

    intcloud = np.sum(data*dv, axis=2)
    intcloud.unit


:outputnumrole:`Out[22]:`




.. math::

    \mathrm{\frac{K\,km}{s}}



We can plot the 2D quantity using matplotlib's imshow function, by
passing the quantity's value. Similarly, we can set the correct extent
using the values of :math:`x_i` and :math:`x_f`. Finally, we can set the
colorbar label to have proper units.


:inputnumrole:`In[23]:`


.. code:: python

    #Note that we display RA in the convential way by going from max to min
    plt.imshow(intcloud.value, 
               origin='lower', 
               extent=[ra.value.max(), ra.value.min(), dec.value.min(), dec.value.max()], 
               cmap='hot', 
               interpolation='nearest', 
               aspect='equal')
    plt.colorbar().set_label("Intensity ({})".format(intcloud.unit))
    plt.xlabel("RA (deg)")
    plt.ylabel("Dec (deg)");


:outputnumrole:`Out[23]:`



.. image:: nboutput/quantities_58_0.png



Measuring The Column Density of CO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to calculate the mass of the molecular cloud, we need to
measure its column density. A number of assumptions are required for the
following calculation; the most important are that the emission is
optically thin (typically true for :math:`{\rm C}^{18}{\rm O}`) and that
conditions of local thermodynamic equilibrium hold along the line of
sight. In the case where the temperature is large compared to the
separation in energy levels for a molecule and the source fills the main
beam of the telescope, the total column density for
:math:`{\rm C}^{13}{\rm O}` is

:math:`N=C \frac{\int T_B(V) dV}{1-e^{-B}}`

where the constants :math:`C` and :math:`B` are given by:

:math:`C=3.0\times10^{14} \left(\frac{\nu}{\nu_{13}}\right)^2 \frac{A_{13}}{A} {\rm K^{-1} cm^{-2} \, km^{-1} \, s}`

:math:`B=\frac{h\nu}{k_B T}`

(Rohlfs & Wilson "Tools of Radio Astronomy").

Here we have given an expression for :math:`C` scaled to the values for
:math:`{\rm C}^{13}{\rm O}` (:math:`\nu_{13}` and :math:`A_{13}`). In
order to use this relation for :math:`{\rm C}^{18}{\rm O}`, we need to
rescale the frequencies :math:`{\nu}` and Einstein coefficients
:math:`A`. :math:`C` is in funny mixed units, but that's okay. We'll
define it as a ``Quantities`` object and not have to worry about it.

First, we look up the wavelength for these emission lines and store them
as quantities.


:inputnumrole:`In[24]:`


.. code:: python

    lambda13 = 2.60076 * u.mm
    lambda18 = 2.73079 * u.mm

Since the wavelength and frequency of light are related using the speed
of light, we can convert between them. However, doing so just using the
to() method fails, as units of length and frequency are not convertible:


:inputnumrole:`In[25]:`


.. code:: python

    nu13 = lambda13.to(u.Hz)


:outputnumrole:`Out[25]:`


::


    

    UnitConversionErrorTraceback (most recent call last)

    <ipython-input-25-b4a9b54d7f21> in <module>()
    ----> 1 nu13 = lambda13.to(u.Hz)
    

    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity.py in to(self, unit, equivalencies)
        845         # and don't want to slow down this method (esp. the scalar case).
        846         unit = Unit(unit)
    --> 847         return self._new_view(self._to_value(unit, equivalencies), unit)
        848 
        849     def to_value(self, unit=None, equivalencies=[]):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity.py in _to_value(self, unit, equivalencies)
        817             equivalencies = self._equivalencies
        818         return self.unit.to(unit, self.view(np.ndarray),
    --> 819                             equivalencies=equivalencies)
        820 
        821     def to(self, unit, equivalencies=[]):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in to(self, other, value, equivalencies)
        963             If units are inconsistent
        964         """
    --> 965         return self._get_converter(other, equivalencies=equivalencies)(value)
        966 
        967     def in_units(self, other, value=1.0, equivalencies=[]):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _get_converter(self, other, equivalencies)
        897                             pass
        898 
    --> 899             raise exc
        900 
        901     def _to(self, other):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _get_converter(self, other, equivalencies)
        883         try:
        884             return self._apply_equivalencies(
    --> 885                 self, other, self._normalize_equivalencies(equivalencies))
        886         except UnitsError as exc:
        887             # Last hope: maybe other knows how to do it?


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _apply_equivalencies(self, unit, other, equivalencies)
        867         raise UnitConversionError(
        868             "{0} and {1} are not convertible".format(
    --> 869                 unit_str, other_str))
        870 
        871     def _get_converter(self, other, equivalencies=[]):


    UnitConversionError: 'mm' (length) and 'Hz' (frequency) are not convertible


Fortunately, ``astropy`` comes to the rescue by providing a feature
called "unit equivalencies". Equivalencies provide a way to convert
between two physically different units that are not normally equivalent,
but in a certain context have a one-to-one mapping. For more on
equivalencies, see the `equivalencies section of astropy's
documentation <http://docs.astropy.org/en/stable/units/equivalencies.html>`__.

In this case, calling the ``astropy.units.spectral()`` function provides
the equivalencies necessary to handle conversions between wavelength and
frequency. To use it, provide the equivalencies to the ``equivalencies``
keyword of the ``to()`` call:


:inputnumrole:`In[26]:`


.. code:: python

    nu13 = lambda13.to(u.Hz, equivalencies=u.spectral())
    nu18 = lambda18.to(u.Hz, equivalencies=u.spectral())

Next, we look up Einstein coefficients (in units of s\ :math:`^{-1}`),
and calculate the ratios in constant :math:`C`. Note how the ratios of
frequency and Einstein coefficient units are dimensionless, so the unit
of :math:`C` is unchanged.


:inputnumrole:`In[27]:`


.. code:: python

    A13 = 7.4e-8 / u.s
    A18 = 8.8e-8 / u.s
    
    C = 3e14 * (nu18/nu13)**3 * (A13/A18) / (u.K * u.cm**2 * u.km *(1/u.s))
    C


:outputnumrole:`Out[27]:`




.. math::

    2.1792458 \times 10^{14} \; \mathrm{\frac{s}{K\,km\,cm^{2}}}



Now we move on to calculate the constant :math:`B`. This is given by the
ratio of :math:`\frac{h\nu}{k_B T}`, where :math:`h` is Planck's
constant, :math:`k_B` is the Boltzmann's constant, :math:`\nu` is the
emission frequency, and :math:`T` is the excitation temperature. The
constants were imported from ``astropy.constants``, and the other two
values are already calculated, so here we just take the ratio.


:inputnumrole:`In[28]:`


.. code:: python

    B = h * nu18 / (k_B * Tex)

The units of :math:`B` are Hz s, which can be decomposed to a
dimensionless unit if you actually care about it's value. Usually this
is not necessary, though. Quantities are at their best if you just use
them without worrying about intermediate units, and only convert at the
very end when you want a final answer.


:inputnumrole:`In[29]:`


.. code:: python

    print('{0}\n{1}'.format(B, B.decompose()))


:outputnumrole:`Out[29]:`


.. parsed-literal::

    0.21074888275227613 Hz s
    0.21074888275227613


At this point we have all the ingredients to calculate the number
density of :math:`\rm CO` molecules in this cloud. We already integrated
(summed) over the velocity channels above to show the integrated
intensity map, but we'll do it again here for clarity. This gives us the
column density of CO for each spatial pixel in our map. We can then
print out the peak column column density.


:inputnumrole:`In[30]:`


.. code:: python

    NCO = C * np.sum(data*dv, axis=2) / (1 - np.exp(-B))
    print("Peak CO column density: ")
    np.max(NCO)


:outputnumrole:`Out[30]:`


.. parsed-literal::

    Peak CO column density: 




.. math::

    8.5782066 \times 10^{15} \; \mathrm{\frac{1}{cm^{2}}}



CO to Total Mass
^^^^^^^^^^^^^^^^

We are using CO as a tracer for the much more numerous H\ :math:`_2`,
the quantity we are actually trying to infer. Since most of the mass is
in H\ :math:`_2`, we calculate its column density by multiplying the CO
column density with the (known/assumed) H\ :math:`_2`/CO ratio.


:inputnumrole:`In[31]:`


.. code:: python

    H2_CO_ratio = 5.9e6
    NH2 = NCO * H2_CO_ratio
    print("Peak H2 column density: ")
    np.max(NH2)


:outputnumrole:`Out[31]:`


.. parsed-literal::

    Peak H2 column density: 




.. math::

    5.0611419 \times 10^{22} \; \mathrm{\frac{1}{cm^{2}}}



That's a peak column density of roughly 50 magnitudes of visual
extinction (assuming the conversion between N\ :math:`_{\rm H_2}` and
A\ :math:`_V` from Bohlin et al. 1978), which seems reasonable for a
molecular cloud.

We obtain the mass column density by multiplying the number column
density by the mass of an individual H\ :math:`_2` molecule.


:inputnumrole:`In[32]:`


.. code:: python

    mH2 = 2 * 1.008 * u.Dalton  #aka atomic mass unit/amu
    rho = NH2 * mH2

A final step in going from the column density to mass is summing up over
the area area. If we do this in the straightforward way of length x
width of a pixel, this area is then in units of :math:`{\rm deg}^2`.


:inputnumrole:`In[33]:`


.. code:: python

    dap = dra * ddec
    print(dap)


:outputnumrole:`Out[33]:`


.. parsed-literal::

    2.5e-05 deg2


Now comes an important subtlety: in the small angle approximation,
multiplying the pixel area with the square of distance yields the
cross-sectional area of the cloud that the pixel covers, in *physical*
units, rather than angular units. So it is tempting to just multiply the
area and the square of the distance.


:inputnumrole:`In[34]:`


.. code:: python

    da = dap * d**2  # don't actually do it this way - use the version below instead!
    print(da)


:outputnumrole:`Out[34]:`


.. parsed-literal::

    1.5625 deg2 pc2



:inputnumrole:`In[35]:`


.. code:: python

    dap.to(u.steradian).value * d**2


:outputnumrole:`Out[35]:`




.. math::

    0.00047596472 \; \mathrm{pc^{2}}



But this is **wrong**, because ``astropy.units`` treats angles (and
solid angles) as actual physical units, while the small-angle
approximation assumes angles are dimensionless. So if you, e.g., try to
convert to a different area unit, it will fail:


:inputnumrole:`In[36]:`


.. code:: python

    da.to(u.cm**2)


:outputnumrole:`Out[36]:`


::


    

    UnitConversionErrorTraceback (most recent call last)

    <ipython-input-36-d7c4d4dcf9cc> in <module>()
    ----> 1 da.to(u.cm**2)
    

    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity.py in to(self, unit, equivalencies)
        845         # and don't want to slow down this method (esp. the scalar case).
        846         unit = Unit(unit)
    --> 847         return self._new_view(self._to_value(unit, equivalencies), unit)
        848 
        849     def to_value(self, unit=None, equivalencies=[]):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity.py in _to_value(self, unit, equivalencies)
        817             equivalencies = self._equivalencies
        818         return self.unit.to(unit, self.view(np.ndarray),
    --> 819                             equivalencies=equivalencies)
        820 
        821     def to(self, unit, equivalencies=[]):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in to(self, other, value, equivalencies)
        963             If units are inconsistent
        964         """
    --> 965         return self._get_converter(other, equivalencies=equivalencies)(value)
        966 
        967     def in_units(self, other, value=1.0, equivalencies=[]):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _get_converter(self, other, equivalencies)
        897                             pass
        898 
    --> 899             raise exc
        900 
        901     def _to(self, other):


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _get_converter(self, other, equivalencies)
        883         try:
        884             return self._apply_equivalencies(
    --> 885                 self, other, self._normalize_equivalencies(equivalencies))
        886         except UnitsError as exc:
        887             # Last hope: maybe other knows how to do it?


    ~/project/venv/lib/python3.6/site-packages/astropy/units/core.py in _apply_equivalencies(self, unit, other, equivalencies)
        867         raise UnitConversionError(
        868             "{0} and {1} are not convertible".format(
    --> 869                 unit_str, other_str))
        870 
        871     def _get_converter(self, other, equivalencies=[]):


    UnitConversionError: 'deg2 pc2' and 'cm2' (area) are not convertible


The solution is to use the ``dimensionless_angles`` equivalency, which
allows angles to be treated as dimensionless. This makes it so that they
will automatically convert to radians and become dimensionless when a
conversion is needed.


:inputnumrole:`In[37]:`


.. code:: python

    da = (dap * d**2).to(u.pc**2, equivalencies=u.dimensionless_angles())
    da


:outputnumrole:`Out[37]:`




.. math::

    0.00047596472 \; \mathrm{pc^{2}}




:inputnumrole:`In[38]:`


.. code:: python

    da.to(u.cm**2)


:outputnumrole:`Out[38]:`




.. math::

    4.5318534 \times 10^{33} \; \mathrm{cm^{2}}



Finally, multiplying the column density with the pixel area and summing
over all the pixels gives us the cloud mass.


:inputnumrole:`In[39]:`


.. code:: python

    M = np.sum(rho * da)
    M.decompose().to(u.solMass)


:outputnumrole:`Out[39]:`




.. math::

    317.63786 \; \mathrm{M_{\odot}}



Exercises
---------

The astro material was pretty heavy on that one, so lets focus on some
associated statistics using ``Quantity``'s array capabililities. Compute
the median and mean of the ``data`` with the ``np.mean`` and
``np.median`` functions. Why are their values so different?


:inputnumrole:`In[None]:`



Similarly, compute the standard deviation and variance (if you don't
know the relevant functions, look it up in the numpy docs or just type
np. and a code cell). Do they have the units you expect?


:inputnumrole:`In[None]:`



3. Using Quantities with Functions
----------------------------------

``Quantity`` is also a useful tool if you plan to share some of your
code, either with collaborators or the wider community. By writing
functions that take ``Quantity`` objects instead of raw numbers or
arrays, you can write code that is agnostic to the input unit. In this
way, you may even be able to prevent `the destruction of Mars
orbiters <http://en.wikipedia.org/wiki/Mars_Climate_Orbiter#Cause_of_failure>`__.
Below, we provide a simple example.

Suppose you are working on an instrument, and the bigwig funding it asks
for a function to give an analytic estimate of the response function.
You determine from some tests it's basically a Lorentzian, but with a
different scale along the two axes. Your first thought might be to do
this:


:inputnumrole:`In[40]:`


.. code:: python

    def response_func(xinarcsec, yinarcsec):
        xscale = 0.9
        yscale = 0.85
        xfactor = 1 / (1 + xinarcsec/xscale)
        yfactor = 1 / (1 + yinarcsec/yscale)
        
        return xfactor * yfactor

You meant the inputs to be in arcsec, but you send that to your hapless
collaborator, and they don't look closely and think the inputs are
instead supposed to be in arcmin. So they do:


:inputnumrole:`In[41]:`


.. code:: python

    response_func(1.0, 1.2)


:outputnumrole:`Out[41]:`




.. parsed-literal::

    0.19640564826700893



And now they tell all their friends how terrible the instrument is,
because it's supposed to have arcsecond resolution, but your function
clearly shows it can only resolve an arcmin at best. But you can solve
this by requiring they pass in ``Quantity`` objects. The new function
could simply be:


:inputnumrole:`In[42]:`


.. code:: python

    def response_func(x, y):
        xscale = 0.9 * u.arcsec
        yscale = 0.85 * u.arcsec
        xfactor = 1 / (1 + x/xscale)
        yfactor = 1 / (1 + y/yscale)
        
        return xfactor * yfactor

And your collaborator now has to pay attention. If they just blindly put
in a number they get an error:


:inputnumrole:`In[43]:`


.. code:: python

    response_func(1.0, 1.2)


:outputnumrole:`Out[43]:`


::


    

    UnitsErrorTraceback (most recent call last)

    <ipython-input-43-5d7d1ca80126> in <module>()
    ----> 1 response_func(1.0, 1.2)
    

    <ipython-input-42-c1a22f103934> in response_func(x, y)
          2     xscale = 0.9 * u.arcsec
          3     yscale = 0.85 * u.arcsec
    ----> 4     xfactor = 1 / (1 + x/xscale)
          5     yfactor = 1 / (1 + y/yscale)
          6 


    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity.py in __array_ufunc__(self, function, method, *inputs, **kwargs)
        618         # consistent units between two inputs (e.g., in np.add) --
        619         # and the unit of the result (or tuple of units for nout > 1).
    --> 620         converters, unit = converters_and_unit(function, method, *inputs)
        621 
        622         out = kwargs.get('out', None)


    ~/project/venv/lib/python3.6/site-packages/astropy/units/quantity_helper.py in converters_and_unit(function, method, *args)
        554                                      "argument is not a quantity (unless the "
        555                                      "latter is all zero/infinity/nan)"
    --> 556                                      .format(function.__name__))
        557             except TypeError:
        558                 # _can_have_arbitrary_unit failed: arg could not be compared


    UnitsError: Can only apply 'add' function to dimensionless quantities when other argument is not a quantity (unless the latter is all zero/infinity/nan)


Which is their cue to provide the units explicitly:


:inputnumrole:`In[44]:`


.. code:: python

    response_func(1.0*u.arcmin, 1.2*u.arcmin)


:outputnumrole:`Out[44]:`




.. math::

    0.0001724307 \; \mathrm{}



The funding agency is impressed at the resolution you achieved, and your
instrument is saved. You now go on to win the Nobel Prize due to
discoveries the instrument makes. And it was all because you used
``Quantity`` as the input of code you shared.

Exercise
--------

Write a function that computes the Keplerian velocity you worked out in
section 1 (using ``Quantity`` input and outputs, of course), but
allowing for an arbitrary mass and orbital radius. Try it with some
reasonable numbers for satellites orbiting the Earth, a moon of Jupiter,
or an extrasolar planet. Feel free to use wikipedia or similar for the
masses and distances.


:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

