.. meta::
    :keywords: filterTutorials, filterCoordinates, filterUnits






.. raw:: html

    <a href="../_static/Coordinates-Transform/Coordinates-Transform.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/Coordinates-Transform/Coordinates-Transform.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _Coordinates-Transform:

Transforming between coordinate systems
=======================================

Authors
-------

Erik Tollerud, Kelle Cruz, Stephen Pardy

Learning Goals
--------------

-  Make coordinate objects
-  Transform to different coordinate systems
-  See how to track an object's altitude from certain observing
   locations

Keywords
--------

coordinates, units

Summary
-------

Demonstrates how to define astronomical coordinates using the
``astropy.coordinates`` "frame" classes. Then shows how to transform
between the different built-in coordinate frames, such as from ICRS (ra,
dec) to Galactic (l, b).

Imports
~~~~~~~


:inputnumrole:`In[1]:`


.. code:: python

    # Python standard-library
    from urllib.parse import urlencode
    from urllib.request import urlretrieve
    
    # Third-party dependencies
    from astropy import units as u
    from astropy.coordinates import SkyCoord
    from astropy.table import Table
    import numpy as np
    from IPython.display import Image


:inputnumrole:`In[2]:`


.. code:: python

    # Set up matplotlib and use a nicer set of plot parameters
    from astropy.visualization import astropy_mpl_style
    import matplotlib.pyplot as plt
    plt.style.use(astropy_mpl_style)
    %matplotlib inline

Section 0: Quickstart
---------------------

.. raw:: html

   <div class="alert alert-info">

**Note:** If you already worked through the first in this series you can
feel free to skip to `Section 1 <#Section-1:>`__.

.. raw:: html

   </div>

In Astropy, the most common object you'll work with for coordinates is
``SkyCoord``. A ``SkyCoord`` can be created most easily directly from
angles as shown below.

In this tutorial we will be converting between frames. Let's start in
the ICRS frame (which happens to be the default).

For much of this tutorial we will work with the Hickson Compact Group 7.
We can create an object either by passing the degrees explicitly (using
the astropy
`units <http://docs.astropy.org/en/stable/units/index.html>`__ library)
or by passing in strings. The two coordinates below are equivalent


:inputnumrole:`In[3]:`


.. code:: python

    hcg7_center = SkyCoord(9.81625*u.deg, 0.88806*u.deg, frame='icrs')  # using degrees directly
    print(hcg7_center)


:outputnumrole:`Out[3]:`


.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>



:inputnumrole:`In[4]:`


.. code:: python

    hcg7_center = SkyCoord('0h39m15.9s', '0d53m17.016s', frame='icrs')  # passing in string format
    print(hcg7_center)


:outputnumrole:`Out[4]:`


.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>


We can get the right ascension and declination components of the object
directly by accessing those attributes.


:inputnumrole:`In[5]:`


.. code:: python

    print(hcg7_center.ra)
    print(hcg7_center.dec)


:outputnumrole:`Out[5]:`


.. parsed-literal::

    9d48m58.5s
    0d53m17.016s


Section 1:
----------

Introducing frame transformations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``astropy.coordinates`` provides many tools to transform between
different coordinate systems. For instance, we can use it to transform
from ICRS coordinates (in ra and dec) to galactic coordinates.

To understand the code in this section, it may help to read over the
`overview of the astropy coordinates
scheme <http://astropy.readthedocs.org/en/latest/coordinates/index.html#overview-of-astropy-coordinates-concepts>`__.
The key bit to understand is that all coordinates in astropy are in
particular "frames", and we can transform between a specific
``SkyCoord`` object from one frame to another. For example, we can
transform our previously-defined center of HCG7 from ICRS to Galactic
coordinates:


:inputnumrole:`In[6]:`


.. code:: python

    hcg7_center = SkyCoord(9.81625*u.deg, 0.88806*u.deg, frame='icrs')

There are three different ways of transforming coordinates. Each has
it's pros and cons, but should give you the same result. The first way
to transform to other built-in frames is by specifying those attributes.
For instance, let's see the location of HCG7 in galactic coordinates.

Transforming coordinates using attributes:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


:inputnumrole:`In[7]:`


.. code:: python

    hcg7_center.galactic


:outputnumrole:`Out[7]:`




.. parsed-literal::

    <SkyCoord (Galactic): (l, b) in deg
        (116.47556813, -61.83099472)>



Transforming coordinates using the transform\_to() method and other Coordinate object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The above is actually a special "quick-access" form which internally
does the same as what's in the cell below: uses the
```transform_to()`` <http://docs.astropy.org/en/stable/api/astropy.coordinates.SkyCoord.html#astropy.coordinates.SkyCoord.transform_to>`__
method to convert from one frame to another. We can pass in an empty
coordinate class to specify what coordinate system to transform into.


:inputnumrole:`In[8]:`


.. code:: python

    from astropy.coordinates import Galactic  # new coordinate baseclass
    hcg7_center.transform_to(Galactic())


:outputnumrole:`Out[8]:`




.. parsed-literal::

    <SkyCoord (Galactic): (l, b) in deg
        (116.47556813, -61.83099472)>



Transforming coordinates using the transform\_to() method and a string
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Finally, we can transform using the ``transform_to()`` method and a
string with the name of a built-in coordinate system.


:inputnumrole:`In[9]:`


.. code:: python

    hcg7_center.transform_to('galactic')


:outputnumrole:`Out[9]:`




.. parsed-literal::

    <SkyCoord (Galactic): (l, b) in deg
        (116.47556813, -61.83099472)>



We can transform to many coordinate frames and equinoxes.

By default these coordinates are available:

-  ICRS
-  FK5
-  FK4
-  FK4NoETerms
-  Galactic
-  Galactocentric
-  Supergalactic
-  AltAz
-  GCRS
-  CIRS
-  ITRS
-  HCRS
-  PrecessedGeocentric
-  GeocentricTrueEcliptic
-  BarycentricTrueEcliptic
-  HeliocentricTrueEcliptic
-  SkyOffsetFrame
-  GalacticLSR
-  LSR
-  BaseEclipticFrame
-  BaseRADecFrame

Let's focus on just a few of these. We can try FK5 coordinates next:


:inputnumrole:`In[10]:`


.. code:: python

    hcg7_center_fk5 = hcg7_center.transform_to('fk5')
    print(hcg7_center_fk5)


:outputnumrole:`Out[10]:`


.. parsed-literal::

    <SkyCoord (FK5: equinox=J2000.000): (ra, dec) in deg
        (9.81625645, 0.88806155)>


And, as with the galactic coordinates, we can acheive the same result by
importing the FK5 class from the astropy.coordinates package. This also
allows us to change the equinox.


:inputnumrole:`In[11]:`


.. code:: python

    from astropy.coordinates import FK5
    hcg7_center_fk5.transform_to(FK5(equinox='J1975'))  # precess to a different equinox  


:outputnumrole:`Out[11]:`




.. parsed-literal::

    <SkyCoord (FK5: equinox=J1975.000): (ra, dec) in deg
        (9.49565759, 0.75084648)>



.. raw:: html

   <div class="alert alert-warning">

**Beware:** Changing frames also changes some of the attributes of the
object, but usually in a way that makes sense. The following code should
fail.

.. raw:: html

   </div>


:inputnumrole:`In[12]:`


.. code:: python

    hcg7_center.galactic.ra  # should fail because galactic coordinates are l/b not RA/Dec


:outputnumrole:`Out[12]:`


::


    

    AttributeErrorTraceback (most recent call last)

    <ipython-input-12-d7bc134707f6> in <module>()
    ----> 1 hcg7_center.galactic.ra  # should fail because galactic coordinates are l/b not RA/Dec
    

    ~/project/venv/lib/python3.6/site-packages/astropy/coordinates/sky_coordinate.py in __getattr__(self, attr)
        693         # Fail
        694         raise AttributeError("'{0}' object has no attribute '{1}'"
    --> 695                              .format(self.__class__.__name__, attr))
        696 
        697     def __setattr__(self, attr, val):


    AttributeError: 'SkyCoord' object has no attribute 'ra'


Instead, we now have access the l and b attributes:


:inputnumrole:`In[13]:`


.. code:: python

    print(hcg7_center.galactic.l, hcg7_center.galactic.b)


:outputnumrole:`Out[13]:`


.. parsed-literal::

    116d28m32.0453s -61d49m51.581s


Section 2:
----------

Transform frames to get to altitude-azimuth ("AltAz")
-----------------------------------------------------

To actually do anything with observability we need to convert to a frame
local to an on-earth observer. By far the most common choice is
horizontal altitude-azimuth coordinates, or "AltAz". We first need to
specify both where and when we want to try to observe.

We will need to import a few more specific modules:


:inputnumrole:`In[14]:`


.. code:: python

    from astropy.coordinates import EarthLocation
    from astropy.time import Time

Lets first see the sky position at Kitt Peak National Observatory in
Arizona.


:inputnumrole:`In[15]:`


.. code:: python

    # Kitt Peak, Arizona
    kitt_peak = EarthLocation(lat='31d57.5m', lon='-111d35.8m', height=2096*u.m)

For known observing sites we can enter the name directly.


:inputnumrole:`In[16]:`


.. code:: python

    kitt_peak = EarthLocation.of_site('Kitt Peak')


:outputnumrole:`Out[16]:`


.. parsed-literal::

    Downloading http://data.astropy.org/coordinates/sites.json [Done]


We can see the list of observing sites:


:inputnumrole:`In[17]:`


.. code:: python

    EarthLocation.get_site_names()


:outputnumrole:`Out[17]:`




.. parsed-literal::

    ['',
     '',
     '',
     'ALMA',
     'Anglo-Australian Observatory',
     'Apache Point',
     'Apache Point Observatory',
     'Atacama Large Millimeter Array',
     'BAO',
     'Beijing XingLong Observatory',
     'Black Moshannon Observatory',
     'CHARA',
     'Canada-France-Hawaii Telescope',
     'Catalina Observatory',
     'Cerro Pachon',
     'Cerro Paranal',
     'Cerro Tololo',
     'Cerro Tololo Interamerican Observatory',
     'DCT',
     'Discovery Channel Telescope',
     'Dominion Astrophysical Observatory',
     'GBT',
     'Gemini South',
     'Green Bank Telescope',
     'Hale Telescope',
     'Haleakala Observatories',
     'Happy Jack',
     'JCMT',
     'James Clerk Maxwell Telescope',
     'Jansky Very Large Array',
     'Keck Observatory',
     'Kitt Peak',
     'Kitt Peak National Observatory',
     'La Silla Observatory',
     'Large Binocular Telescope',
     'Las Campanas Observatory',
     'Lick Observatory',
     'Lowell Observatory',
     'Manastash Ridge Observatory',
     'McDonald Observatory',
     'Medicina',
     'Medicina Dish',
     'Michigan-Dartmouth-MIT Observatory',
     'Mount Graham International Observatory',
     'Mt Graham',
     'Mt. Ekar 182 cm. Telescope',
     'Mt. Stromlo Observatory',
     'Multiple Mirror Telescope',
     'NOV',
     'National Observatory of Venezuela',
     'Noto',
     'Observatorio Astronomico Nacional, San Pedro Martir',
     'Observatorio Astronomico Nacional, Tonantzintla',
     'Palomar',
     'Paranal Observatory',
     'Roque de los Muchachos',
     'SAAO',
     'SALT',
     'SRT',
     'Siding Spring Observatory',
     'Southern African Large Telescope',
     'Subaru',
     'Subaru Telescope',
     'Sutherland',
     'TUG',
     'UKIRT',
     'United Kingdom Infrared Telescope',
     'Vainu Bappu Observatory',
     'Very Large Array',
     'W. M. Keck Observatory',
     'Whipple',
     'Whipple Observatory',
     'aao',
     'alma',
     'apo',
     'bmo',
     'cfht',
     'ctio',
     'dao',
     'dct',
     'ekar',
     'example_site',
     'flwo',
     'gbt',
     'gemini_north',
     'gemini_south',
     'gemn',
     'gems',
     'greenwich',
     'haleakala',
     'irtf',
     'jcmt',
     'keck',
     'kpno',
     'lapalma',
     'lasilla',
     'lbt',
     'lco',
     'lick',
     'lowell',
     'mcdonald',
     'mdm',
     'medicina',
     'mmt',
     'mro',
     'mso',
     'mtbigelow',
     'mwo',
     'noto',
     'ohp',
     'paranal',
     'salt',
     'sirene',
     'spm',
     'srt',
     'sso',
     'tona',
     'tug',
     'ukirt',
     'vbo',
     'vla']



Let's check the altitude at 1am UTC, which is 6pm AZ mountain time


:inputnumrole:`In[18]:`


.. code:: python

    observing_time = Time('2010-12-21 1:00')

Now we use these to create an ``AltAz`` frame object. Note that this
frame has some other information about the atmosphere, which can be used
to correct for atmospheric refraction. Here we leave that alone, because
the default is to ignore this effect (by setting the pressure to 0).


:inputnumrole:`In[19]:`


.. code:: python

    from astropy.coordinates import AltAz
    
    aa = AltAz(location=kitt_peak, obstime=observing_time)
    print(aa)


:outputnumrole:`Out[19]:`


.. parsed-literal::

    <AltAz Frame (obstime=2010-12-21 01:00:00.000, location=(-1994502.6043061386, -5037538.54232911, 3358104.9969029757) m, pressure=0.0 hPa, temperature=0.0 deg_C, relative_humidity=0, obswl=1.0 micron)>


Now we can just transform our ICRS ``SkyCoord`` to ``AltAz`` to get the
location in the sky over Kitt Peak at the requested time.


:inputnumrole:`In[20]:`


.. code:: python

    hcg7_center.transform_to(aa)


:outputnumrole:`Out[20]:`


.. parsed-literal::

    Downloading http://maia.usno.navy.mil/ser7/finals2000A.all [Done]




.. parsed-literal::

    <SkyCoord (AltAz: obstime=2010-12-21 01:00:00.000, location=(-1994502.6043061386, -5037538.54232911, 3358104.9969029757) m, pressure=0.0 hPa, temperature=0.0 deg_C, relative_humidity=0, obswl=1.0 micron): (az, alt) in deg
        (149.19234446, 55.05673074)>



To look at just the altitude we can \`alt' attribute:


:inputnumrole:`In[21]:`


.. code:: python

    hcg7_center.transform_to(aa).alt


:outputnumrole:`Out[21]:`




.. math::

    55^\circ03{}^\prime24.2307{}^{\prime\prime}



Alright, it's at 55 degrees at 6pm, but that's pretty early to be
observing. We could just try various times one at a time to see if the
airmass is at a darker time, but we can do better: lets try to create an
airmass plot.


:inputnumrole:`In[22]:`


.. code:: python

    # this gives a Time object with an *array* of times
    delta_hours = np.linspace(0, 6, 100)*u.hour
    full_night_times = observing_time + delta_hours
    full_night_aa_frames = AltAz(location=kitt_peak, obstime=full_night_times)
    full_night_aa_coos = hcg7_center.transform_to(full_night_aa_frames)
    
    plt.plot(delta_hours, full_night_aa_coos.secz)
    plt.xlabel('Hours from 6pm AZ time')
    plt.ylabel('Airmass [Sec(z)]')
    plt.ylim(0.9,3)
    plt.tight_layout()


:outputnumrole:`Out[22]:`



.. image:: nboutput/Coordinates-Transform_51_0.png



Great! Looks like it's at the lowest airmass in another hour or so
(7pm). But might that might still be twilight... When should we start
observing for proper dark skies? Fortunately, astropy provides a
``get_sun`` function that can be used to check this. Lets use it to
check if we're in 18-degree twilight or not.


:inputnumrole:`In[23]:`


.. code:: python

    from astropy.coordinates import get_sun
    
    full_night_sun_coos = get_sun(full_night_times).transform_to(full_night_aa_frames)
    plt.plot(delta_hours, full_night_sun_coos.alt.deg)
    plt.axhline(-18, color='k')
    plt.xlabel('Hours from 6pm AZ time')
    plt.ylabel('Sun altitude')
    plt.tight_layout()


:outputnumrole:`Out[23]:`



.. image:: nboutput/Coordinates-Transform_53_0.png



Looks like it's just below 18 degrees at 7, so you should be good to go!

We can also look at the object altitude at the present time and date.


:inputnumrole:`In[24]:`


.. code:: python

    now = Time.now()
    hcg7_center = SkyCoord(9.81625*u.deg, 0.88806*u.deg, frame='icrs')
    kitt_peak_aa = AltAz(location=kitt_peak, obstime=now)
    print(hcg7_center.transform_to(kitt_peak_aa))


:outputnumrole:`Out[24]:`


.. parsed-literal::

    <SkyCoord (AltAz: obstime=2018-10-08 16:04:29.144620, location=(-1994502.6043061386, -5037538.54232911, 3358104.9969029757) m, pressure=0.0 hPa, temperature=0.0 deg_C, relative_humidity=0, obswl=1.0 micron): (az, alt) in deg
        (300.24937333, -37.45817894)>


Exercises
---------

Excercise 1
~~~~~~~~~~~

Try to actually compute to some arbitrary precision (rather than
eye-balling on a plot) when 18 degree twilight or sunrise/sunset hits on
that night.


:inputnumrole:`In[None]:`



Excercise 2
~~~~~~~~~~~

Try converting the HCG7 coordinates to an equatorial frame at some other
equinox a while in the past (like J2000). Do you see the precession of
the equinoxes?

Hint: To see a diagram of the supported frames look
`here <http://docs.astropy.org/en/stable/coordinates/#module-astropy.coordinates>`__
or the list above. One of those will do what you need if you give it the
right frame attributes.


:inputnumrole:`In[None]:`



Excercise 3
~~~~~~~~~~~

Try looking at the altitude of HCG7 at another observatory.


:inputnumrole:`In[None]:`



Wrap-up
-------

For lots more documentation on the many other features of
``astropy.coordinates``, check out `its section of the
documentation <http://astropy.readthedocs.org/en/latest/coordinates/index.html>`__.

You might also be interested in `the astroplan affiliated
package <http://astroplan.readthedocs.org/>`__, which uses the
``astropy.coordinates`` to do more advanced versions of the tasks in the
last section of this tutorial.


.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

