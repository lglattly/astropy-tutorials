.. meta::
    :keywords: filterTutorials, filterCoordinates, filterOop






.. raw:: html

    <a href="../_static/Coordinates-Intro/Coordinates-Intro.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/Coordinates-Intro/Coordinates-Intro.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _Coordinates-Intro:

Getting Started with astropy.coordinates
========================================

Authors
-------

Erik Tollerud, Kelle Cruz, Stephen Pardy

Learning Goals
--------------

-  Create ``astropy.coordinates.SkyCoord`` objects using names and
   coordinates
-  Interact with a ``SkyCoord`` object and access its attributes
-  Use a ``SkyCoord`` object to query a database

Keywords
--------

coordinates, OOP

Summary
-------

In this tutorial, we're going to investigate the area of the sky around
the picturesque group of galaxies named "Hickson Compact Group 7",
download an image and do something with its coordinates.

Imports
-------


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
    
    # Set up matplotlib and use a nicer set of plot parameters
    from astropy.visualization import astropy_mpl_style
    import matplotlib.pyplot as plt
    plt.style.use(astropy_mpl_style)
    %matplotlib inline

Describing on-sky locations with ``coordinates``
------------------------------------------------

The ``SkyCoord`` class in the ``astropy.coordinates`` package is used to
represent celestial coordinates. First, we'll make a SkyCoord object
based on our object's name, "Hickson Compact Group 7", or "HCG 7" for
short. Most astronomical object names can be found by
`SESAME <http://cdsweb.u-strasbg.fr/cgi-bin/Sesame>`__, a service which
queries Simbad, NED, and VizieR and returns the object's type and its
J2000 position. This service can be used via the
``SkyCoord.from_name()`` `class
method <https://julien.danjou.info/blog/2013/guide-python-static-class-abstract-methods>`__:


:inputnumrole:`In[2]:`


.. code:: python

    # initialize a SkyCood object named hcg7_center at the location of HCG 7
    hcg7_center = SkyCoord.from_name('HCG 7')

.. raw:: html

   <div class="alert alert-info">

Note that this requires an internet connection. If you don't have one,
execute this line instead:

.. raw:: html

   </div>


:inputnumrole:`In[3]:`


.. code:: python

    # uncomment and run this line if you don't have an internet connection
    # hcg7_center = SkyCoord(9.81625*u.deg, 0.88806*u.deg, frame='icrs')


:inputnumrole:`In[4]:`


.. code:: python

    type(hcg7_center)


:outputnumrole:`Out[4]:`




.. parsed-literal::

    astropy.coordinates.sky_coordinate.SkyCoord



Show the available methods and attributes of the SkyCoord object we've
created called ``hcg7_center``


:inputnumrole:`In[5]:`


.. code:: python

    dir(hcg7_center)


:outputnumrole:`Out[5]:`




.. parsed-literal::

    ['T',
     '__abstractmethods__',
     '__bool__',
     '__class__',
     '__delattr__',
     '__dict__',
     '__dir__',
     '__doc__',
     '__eq__',
     '__format__',
     '__ge__',
     '__getattr__',
     '__getattribute__',
     '__getitem__',
     '__gt__',
     '__hash__',
     '__init__',
     '__init_subclass__',
     '__iter__',
     '__le__',
     '__len__',
     '__lt__',
     '__module__',
     '__ne__',
     '__new__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasshook__',
     '__weakref__',
     '_abc_cache',
     '_abc_negative_cache',
     '_abc_negative_cache_version',
     '_abc_registry',
     '_apply',
     '_extra_frameattr_names',
     '_parse_inputs',
     '_sky_coord_frame',
     'altaz',
     'apply_space_motion',
     'barycentrictrueecliptic',
     'cache',
     'cartesian',
     'cirs',
     'copy',
     'data',
     'dec',
     'default_differential',
     'default_representation',
     'diagonal',
     'differential_type',
     'distance',
     'equinox',
     'fk4',
     'fk4noeterms',
     'fk5',
     'flatten',
     'frame',
     'frame_attributes',
     'frame_specific_representation_info',
     'from_name',
     'from_pixel',
     'galactic',
     'galacticlsr',
     'galactocentric',
     'galcen_coord',
     'galcen_distance',
     'galcen_v_sun',
     'gcrs',
     'geocentrictrueecliptic',
     'get_constellation',
     'get_frame_attr_names',
     'get_representation_cls',
     'get_representation_component_names',
     'get_representation_component_units',
     'guess_from_table',
     'has_data',
     'hcrs',
     'heliocentrictrueecliptic',
     'icrs',
     'info',
     'is_equivalent_frame',
     'is_frame_attr_default',
     'is_transformable_to',
     'isscalar',
     'itrs',
     'location',
     'lsr',
     'match_to_catalog_3d',
     'match_to_catalog_sky',
     'name',
     'ndim',
     'obsgeoloc',
     'obsgeovel',
     'obstime',
     'obswl',
     'pm_dec',
     'pm_ra_cosdec',
     'position_angle',
     'precessedgeocentric',
     'pressure',
     'proper_motion',
     'ra',
     'radial_velocity',
     'radial_velocity_correction',
     'ravel',
     'realize_frame',
     'relative_humidity',
     'replicate',
     'replicate_without_data',
     'represent_as',
     'representation',
     'representation_component_names',
     'representation_component_units',
     'representation_info',
     'representation_type',
     'reshape',
     'roll',
     'search_around_3d',
     'search_around_sky',
     'separation',
     'separation_3d',
     'set_representation_cls',
     'shape',
     'size',
     'skyoffset_frame',
     'spherical',
     'spherical_offsets_to',
     'sphericalcoslat',
     'squeeze',
     'supergalactic',
     'swapaxes',
     'take',
     'temperature',
     'to_pixel',
     'to_string',
     'transform_to',
     'transpose',
     'v_bary',
     'velocity',
     'z_sun']



Show the RA and Dec.


:inputnumrole:`In[6]:`


.. code:: python

    print(hcg7_center.ra, hcg7_center.dec)
    print(hcg7_center.ra.hour, hcg7_center.dec)


:outputnumrole:`Out[6]:`


.. parsed-literal::

    9d48m58.5s 0d53m17.016s
    0.6544166666666668 0d53m17.016s


We see that, according to SESAME, HCG 7 is located at ra = 9.849 deg and
dec = 0.878 deg.

This object we just created has various useful ways of accessing the
information contained within it. In particular, the ``ra`` and ``dec``
attributes are specialized
```Quantity`` <http://docs.astropy.org/en/stable/units/index.html>`__
objects (actually, a subclass called
```Angle`` <http://docs.astropy.org/en/stable/api/astropy.coordinates.Angle.html>`__,
which in turn is subclassed by
```Latitude`` <http://docs.astropy.org/en/stable/api/astropy.coordinates.Latitude.html>`__
and
```Longitude`` <http://docs.astropy.org/en/stable/api/astropy.coordinates.Longitude.html>`__).
These objects store angles and provide pretty representations of those
angles, as well as some useful attributes to quickly convert to common
angle units:


:inputnumrole:`In[7]:`


.. code:: python

    type(hcg7_center.ra), type(hcg7_center.dec)


:outputnumrole:`Out[7]:`




.. parsed-literal::

    (astropy.coordinates.angles.Longitude, astropy.coordinates.angles.Latitude)




:inputnumrole:`In[8]:`


.. code:: python

    hcg7_center.ra, hcg7_center.dec


:outputnumrole:`Out[8]:`




.. parsed-literal::

    (<Longitude 9.81625 deg>, <Latitude 0.88806 deg>)




:inputnumrole:`In[9]:`


.. code:: python

    hcg7_center


:outputnumrole:`Out[9]:`




.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>




:inputnumrole:`In[10]:`


.. code:: python

    hcg7_center.ra.hour


:outputnumrole:`Out[10]:`




.. parsed-literal::

    0.6544166666666668



SkyCoord will also accept string-formatted coordinates either as
separate strings for ra/dec or a single string. You'll have to give
units, though, if they aren't part of the string itself.


:inputnumrole:`In[11]:`


.. code:: python

    SkyCoord('0h39m15.9s', '0d53m17.016s', frame='icrs')


:outputnumrole:`Out[11]:`




.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>




:inputnumrole:`In[12]:`


.. code:: python

    hcg7_center.ra.hour


:outputnumrole:`Out[12]:`




.. parsed-literal::

    0.6544166666666668



Download an image
-----------------

Now that we have a ``SkyCoord`` object, we can try to use it to access
data from the `Sloan Digitial Sky Survey <http://www.sdss.org/>`__
(SDSS). Let's start by trying to get a picture using the SDSS image
cutout service to make sure HCG7 is in the SDSS footprint and has good
image quality.

This requires an internet connection, but if it fails, don't worry: the
file is included in the repository so you can just let it use the local
file\ ``'HCG7_SDSS_cutout.jpg'``, defined at the top of the cell.


:inputnumrole:`In[13]:`


.. code:: python

    # tell the SDSS service how big of a cutout we want
    im_size = 12*u.arcmin # get a 12 arcmin square
    im_pixels = 1024 
    cutoutbaseurl = 'http://skyservice.pha.jhu.edu/DR12/ImgCutout/getjpeg.aspx'
    query_string = urlencode(dict(ra=hcg7_center.ra.deg, 
                                  dec=hcg7_center.dec.deg, 
                                  width=im_pixels, height=im_pixels, 
                                  scale=im_size.to(u.arcsec).value/im_pixels))
    url = cutoutbaseurl + '?' + query_string
    
    # this downloads the image to your disk
    urlretrieve(url, 'HCG7_SDSS_cutout.jpg')


:outputnumrole:`Out[13]:`




.. parsed-literal::

    ('HCG7_SDSS_cutout.jpg', <http.client.HTTPMessage at 0x7f8793cdd908>)




:inputnumrole:`In[14]:`


.. code:: python

    Image('HCG7_SDSS_cutout.jpg')


:outputnumrole:`Out[14]:`




.. image:: nboutput/Coordinates-Intro_24_0.jpeg



Very pretty!

Exercise 1
~~~~~~~~~~

Create a ``SkyCoord`` of some other astronomical object you find
interesting. Using only a single method/function call, get a string with
the RA/Dec in the form 'HH:MM:SS.S DD:MM:SS.S'. Check your answer
against an academic paper or some web site like
`SIMBAD <http://simbad.u-strasbg.fr/simbad/>`__ that will show you
sexigesimal coordinates for the object.

(Hint: ``SkyCoord.to_string()`` might be worth reading up on)


:inputnumrole:`In[None]:`



Now get an image of that object from the Digitized Sky Survey and
download it and/or show it in the notebook. Bonus points if you figure
out the (one-line) trick to get it to display in the notebook *without*
ever downloading the file yourself.

(Hint: STScI has an easy-to-access `copy of the
DSS <https://archive.stsci.edu/dss/>`__. The pattern to follow for the
web URL is
``http://archive.stsci.edu/cgi-bin/dss_search?f=GIF&ra=RA&dec=DEC``)


:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

