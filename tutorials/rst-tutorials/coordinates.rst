.. meta::
    :keywords: filterTutorials






.. raw:: html

    <a href="../_static/coordinates/coordinates.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/coordinates/coordinates.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _coordinates:

Using ``astropy.coordinates`` to Match Catalogs and Plan Observations
=====================================================================

In this tutorial, we will explore how the ``astropy.coordinates``
package and related astropy functionality can be used to help in
planning observations or other exercises focused on large coordinate
catalogs.


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
    import matplotlib.pyplot as plt
    %matplotlib inline

Describing on-sky locations with ``coordinates``
------------------------------------------------

Let's start by considering a field around the picturesque Hickson
Compact Group 7. To do anything with this, we need to get an object that
represents the coordinates of the center of this group.

In Astropy, the most common object you'll work with for coordinates is
``SkyCoord``. A ``SkyCoord`` can be created most easily directly from
angles as shown below. It's also wise to explicitly specify the frame
your coordinates are in, although this is not strictly necessary because
the default is ICRS.

(If you're not sure what ICRS is, it's basically safe to think of it as
an approximation to an equatorial system at the J2000 equinox).


:inputnumrole:`In[3]:`


.. code:: python

    hcg7_center = SkyCoord(9.81625*u.deg, 0.88806*u.deg, frame='icrs')
    hcg7_center


:outputnumrole:`Out[3]:`




.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>



SkyCoord will also accept string-formatted coordinates either as
separate strings for ra/dec or a single string. You'll have to give
units, though, if they aren't part of the string itself.


:inputnumrole:`In[4]:`


.. code:: python

    SkyCoord('0h39m15.9s', '0d53m17.016s', frame='icrs')


:outputnumrole:`Out[4]:`




.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>




:inputnumrole:`In[5]:`


.. code:: python

    SkyCoord('0:39:15.9 0:53:17.016', unit=(u.hour, u.deg), frame='icrs')


:outputnumrole:`Out[5]:`




.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>



If the object you're interested in is in
`SESAME <http://cdsweb.u-strasbg.fr/cgi-bin/Sesame>`__, you can also
look it up directly from its name using the ``SkyCoord.from_name()``
class method1. Note that this requires an internet connection. It's safe
to skip if you don't have one, because we defined it above explicitly.

*If you don't know what a class method is, think of it like an
alternative constructor for a ``SkyCoord`` object -- calling
``SkyCoord.from_name()`` with a name gives you a new ``SkyCoord``
object. For more detailed background on what class methods are and when
they're useful, see `this
page <https://julien.danjou.info/blog/2013/guide-python-static-class-abstract-methods>`__.*


:inputnumrole:`In[6]:`


.. code:: python

    hcg7_center = SkyCoord.from_name('HCG 7')
    hcg7_center


:outputnumrole:`Out[6]:`




.. parsed-literal::

    <SkyCoord (ICRS): (ra, dec) in deg
        (9.81625, 0.88806)>



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

    hcg7_center.dec


:outputnumrole:`Out[8]:`




.. math::

    0^\circ53{}^\prime17.016{}^{\prime\prime}




:inputnumrole:`In[9]:`


.. code:: python

    hcg7_center.ra


:outputnumrole:`Out[9]:`




.. math::

    9^\circ48{}^\prime58.5{}^{\prime\prime}




:inputnumrole:`In[10]:`


.. code:: python

    hcg7_center.ra.hour


:outputnumrole:`Out[10]:`




.. parsed-literal::

    0.6544166666666668



Now that we have a ``SkyCoord`` object, we can try to use it to access
data from the `Sloan Digitial Sky Survey <http://www.sdss.org/>`__
(SDSS). Let's start by trying to get a picture using the SDSS image
cutout service to make sure HCG7 is in the SDSS footprint and has good
image quality.

This requires an internet connection, but if it fails, don't worry: the
file is included in the repository so you can just let it use the local
file\ ``'HCG7_SDSS_cutout.jpg'``, defined at the top of the cell.


:inputnumrole:`In[11]:`


.. code:: python

    impix = 1024
    imsize = 12*u.arcmin
    cutoutbaseurl = 'http://skyservice.pha.jhu.edu/DR12/ImgCutout/getjpeg.aspx'
    query_string = urlencode(dict(ra=hcg7_center.ra.deg, 
                                  dec=hcg7_center.dec.deg, 
                                  width=impix, height=impix, 
                                  scale=imsize.to(u.arcsec).value/impix))
    url = cutoutbaseurl + '?' + query_string
    
    # this downloads the image to your disk
    urlretrieve(url, 'HCG7_SDSS_cutout.jpg')


:outputnumrole:`Out[11]:`




.. parsed-literal::

    ('HCG7_SDSS_cutout.jpg', <http.client.HTTPMessage at 0x7f18b37c07f0>)



Now lets take a look at the image.


:inputnumrole:`In[12]:`


.. code:: python

    Image('HCG7_SDSS_cutout.jpg')


:outputnumrole:`Out[12]:`




.. image:: nboutput/coordinates_20_0.jpeg



Very pretty!

Exercises
~~~~~~~~~

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



Using ``coordinates`` and ``table`` to match and compare catalogs
-----------------------------------------------------------------

At the end of the last section, we determined that HCG7 is in the SDSS
imaging survey, so that means we can use the cells below to download
catalogs of objects directly from the SDSS. Later on, we will match this
catalog to another catalog covering the same field, allowing us to make
plots using the combination of the two catalogs.

We will access the SDSS SQL database using the
`astroquery <https://astroquery.readthedocs.org>`__ affiliated package.
This will require an internet connection and a working install of
astroquery. If you don't have these you can just skip down two cells,
because the data files are provided with the repository. Depending on
your version of astroquery it might also issue a warning, which you
should be able to safely ignore.


:inputnumrole:`In[13]:`


.. code:: python

    from astroquery.sdss import SDSS
    sdss = SDSS.query_region(coordinates=hcg7_center, radius=20*u.arcmin, 
                             spectro=True, 
                             photoobj_fields=['ra','dec','u','g','r','i','z'])


:outputnumrole:`Out[13]:`


.. parsed-literal::

    /home/circleci/project/venv/lib/python3.6/site-packages/astroquery/sdss/__init__.py:29: UserWarning: Experimental: SDSS has not yet been refactored to have its API match the rest of astroquery (but it's nearly there).
      warnings.warn("Experimental: SDSS has not yet been refactored to have its API "


``astroquery`` queries gives us back an ```astropy.table.Table``
object <http://docs.astropy.org/en/stable/table/index.html>`__. We could
just work with this directly without saving anything to disk if we
wanted to. But here we will use the capability to write to disk. That
way, if you quit the session and come back later, you don't have to run
the query a second time.

(Note that this won't work fail if you skipped the last step. Don't
worry, you can just skip to the next cell with ``Table.read`` and use
the copy of this table included in the tutorial.)


:inputnumrole:`In[14]:`


.. code:: python

    sdss.write('HCG7_SDSS_photo.dat', format='ascii')


:outputnumrole:`Out[14]:`


.. parsed-literal::

    WARNING: AstropyDeprecationWarning: HCG7_SDSS_photo.dat already exists. Automatically overwriting ASCII files is deprecated. Use the argument 'overwrite=True' in the future. [astropy.io.ascii.ui]


If you don't have internet, you can read the table into python by
running the cell below. But if you did the astroquery step above, you
could skip this, as the table is already in memory as the ``sdss``
variable.


:inputnumrole:`In[15]:`


.. code:: python

    sdss = Table.read('HCG7_SDSS_photo.dat', format='ascii')

Ok, so we have a catalog of objects we got from the SDSS. Now lets say
you have your own catalog of objects in the same field that you want to
match to this SDSS catalog. In this case, we will use a catalog
extracted from the `2MASS <http://www.ipac.caltech.edu/2mass/>`__. We
first load up this catalog into python.


:inputnumrole:`In[16]:`


.. code:: python

    twomass = Table.read('HCG7_2MASS.tbl', format='ascii')

Now to do matching we need ``SkyCoord`` objects. We'll have to build
these from the tables we loaded, but it turns out that's pretty
straightforward: we grab the RA and dec columns from the table and
provide them to the ``SkyCoord`` constructor. Lets first have a look at
the tables to see just what everything is that's in them.


:inputnumrole:`In[17]:`


.. code:: python

    sdss # just to see an example of the format


:outputnumrole:`Out[17]:`




.. raw:: html

    <i>Table length=679</i>
    <table id="table139744067191696" class="table-striped table-bordered table-condensed">
    <thead><tr><th>ra</th><th>dec</th><th>u</th><th>g</th><th>r</th><th>i</th><th>z</th></tr></thead>
    <thead><tr><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th></tr></thead>
    <tr><td>9.50881763297576</td><td>0.952659480876111</td><td>23.14136</td><td>22.04438</td><td>20.78414</td><td>19.95302</td><td>19.53971</td></tr>
    <tr><td>10.0654707290494</td><td>0.595214029511537</td><td>19.12723</td><td>18.21047</td><td>17.97034</td><td>17.88815</td><td>17.8364</td></tr>
    <tr><td>10.1384468865657</td><td>0.608841745204223</td><td>22.90436</td><td>20.51319</td><td>19.07403</td><td>17.64244</td><td>16.87299</td></tr>
    <tr><td>9.67030199069393</td><td>0.792576991595946</td><td>21.52925</td><td>21.73903</td><td>20.18559</td><td>19.08188</td><td>18.65605</td></tr>
    <tr><td>9.89665095055904</td><td>0.802469605980228</td><td>18.13757</td><td>16.8143</td><td>16.29248</td><td>16.12416</td><td>16.00822</td></tr>
    <tr><td>9.98153607208258</td><td>0.701449510003319</td><td>22.46251</td><td>19.97594</td><td>18.54476</td><td>17.4755</td><td>16.88063</td></tr>
    <tr><td>9.78788692126642</td><td>1.14240364889993</td><td>21.88085</td><td>19.62179</td><td>18.1226</td><td>16.40912</td><td>15.48559</td></tr>
    <tr><td>9.97504512537563</td><td>0.663335072544381</td><td>21.41835</td><td>18.71138</td><td>17.34012</td><td>16.69449</td><td>16.28335</td></tr>
    <tr><td>9.58165174974994</td><td>0.869561518593612</td><td>20.21412</td><td>17.76353</td><td>16.32029</td><td>15.26006</td><td>14.67762</td></tr>
    <tr><td>9.89699525490238</td><td>0.985577903182473</td><td>22.64499</td><td>19.86383</td><td>18.56006</td><td>17.95849</td><td>17.62663</td></tr>
    <tr><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td></tr>
    <tr><td>10.0178113422437</td><td>0.663568617260795</td><td>23.07802</td><td>21.94675</td><td>20.22082</td><td>19.16396</td><td>18.6394</td></tr>
    <tr><td>9.51149604861618</td><td>0.771680075957743</td><td>20.01444</td><td>19.19013</td><td>18.89122</td><td>18.60928</td><td>18.41371</td></tr>
    <tr><td>9.99598169424664</td><td>0.6800681698203</td><td>20.08357</td><td>17.37006</td><td>16.1537</td><td>15.66973</td><td>15.373</td></tr>
    <tr><td>10.1494930726228</td><td>0.863407223678946</td><td>22.77547</td><td>20.00966</td><td>18.54422</td><td>17.67901</td><td>17.22931</td></tr>
    <tr><td>9.81396735807618</td><td>1.07380008389376</td><td>22.42185</td><td>21.83237</td><td>21.5233</td><td>21.65854</td><td>22.50206</td></tr>
    <tr><td>9.92450427008492</td><td>0.96474911896837</td><td>20.88001</td><td>18.17424</td><td>16.75394</td><td>15.76061</td><td>15.25564</td></tr>
    <tr><td>9.61902334567515</td><td>0.573420171591954</td><td>22.68919</td><td>21.59723</td><td>20.61812</td><td>19.84894</td><td>19.4082</td></tr>
    <tr><td>9.98289589103072</td><td>0.828173647873068</td><td>17.57576</td><td>16.40284</td><td>15.99398</td><td>15.84826</td><td>15.80337</td></tr>
    <tr><td>9.79963390205882</td><td>0.863969479176525</td><td>30.63567</td><td>16.55926</td><td>15.60212</td><td>15.30208</td><td>15.59649</td></tr>
    <tr><td>9.96058094193592</td><td>1.09853738590981</td><td>22.89306</td><td>20.91076</td><td>19.48685</td><td>18.88035</td><td>18.52977</td></tr>
    </table>




:inputnumrole:`In[18]:`


.. code:: python

    twomass # just to see an example of the format


:outputnumrole:`Out[18]:`




.. raw:: html

    <i>Table masked=True length=23</i>
    <table id="table139744032877480" class="table-striped table-bordered table-condensed">
    <thead><tr><th>designation</th><th>ra</th><th>dec</th><th>r_k20fe</th><th>j_m_k20fe</th><th>j_msig_k20fe</th><th>j_flg_k20fe</th><th>h_m_k20fe</th><th>h_msig_k20fe</th><th>h_flg_k20fe</th><th>k_m_k20fe</th><th>k_msig_k20fe</th><th>k_flg_k20fe</th><th>k_ba</th><th>k_phi</th><th>sup_ba</th><th>sup_phi</th><th>r_ext</th><th>j_m_ext</th><th>j_msig_ext</th><th>h_m_ext</th><th>h_msig_ext</th><th>k_m_ext</th><th>k_msig_ext</th><th>cc_flg</th><th>dist</th><th>angle</th></tr></thead>
    <thead><tr><th></th><th>deg</th><th>deg</th><th>arcsec</th><th>mag</th><th>mag</th><th></th><th>mag</th><th>mag</th><th></th><th>mag</th><th>mag</th><th></th><th></th><th>deg</th><th></th><th>deg</th><th>arcsec</th><th>mag</th><th>mag</th><th>mag</th><th>mag</th><th>mag</th><th>mag</th><th></th><th>arcsec</th><th>deg</th></tr></thead>
    <thead><tr><th>str16</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>int64</th><th>float64</th><th>float64</th><th>int64</th><th>float64</th><th>float64</th><th>int64</th><th>float64</th><th>int64</th><th>float64</th><th>int64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>float64</th><th>str1</th><th>float64</th><th>float64</th></tr></thead>
    <tr><td>00402069+0052508</td><td>10.086218</td><td>0.880798</td><td>9.4</td><td>13.835</td><td>0.068</td><td>0</td><td>13.01</td><td>0.086</td><td>0</td><td>12.588</td><td>0.089</td><td>0</td><td>0.8</td><td>70</td><td>0.82</td><td>35</td><td>18.62</td><td>13.632</td><td>0.088</td><td>12.744</td><td>0.104</td><td>12.398</td><td>0.105</td><td>0</td><td>972.120611</td><td>91.538952</td></tr>
    <tr><td>00395984+0103545</td><td>9.99935</td><td>1.06514</td><td>12.9</td><td>12.925</td><td>0.035</td><td>0</td><td>12.183</td><td>0.042</td><td>0</td><td>11.89</td><td>0.067</td><td>0</td><td>0.8</td><td>35</td><td>0.7</td><td>40</td><td>35.9</td><td>12.469</td><td>0.048</td><td>11.91</td><td>0.066</td><td>11.522</td><td>0.087</td><td>0</td><td>916.927636</td><td>45.951861</td></tr>
    <tr><td>00401849+0049448</td><td>10.077062</td><td>0.82913</td><td>6.0</td><td>14.918</td><td>0.086</td><td>0</td><td>14.113</td><td>0.107</td><td>0</td><td>13.714</td><td>0.103</td><td>0</td><td>0.6</td><td>-15</td><td>1.0</td><td>90</td><td>11.35</td><td>14.631</td><td>0.121</td><td>13.953</td><td>0.169</td><td>13.525</td><td>0.161</td><td>0</td><td>962.489231</td><td>102.73149</td></tr>
    <tr><td>00395277+0057124</td><td>9.969907</td><td>0.953472</td><td>5.3</td><td>14.702</td><td>0.049</td><td>0</td><td>14.248</td><td>0.069</td><td>0</td><td>13.899</td><td>0.095</td><td>0</td><td>0.6</td><td>-60</td><td>0.44</td><td>-50</td><td>10.59</td><td>14.62</td><td>0.144</td><td>14.15</td><td>0.296</td><td>13.73</td><td>0.2</td><td>0</td><td>601.136444</td><td>66.93659</td></tr>
    <tr><td>00401864+0047245</td><td>10.077704</td><td>0.790143</td><td>7.6</td><td>15.585</td><td>0.134</td><td>1</td><td>15.003</td><td>0.18</td><td>1</td><td>14.049</td><td>0.142</td><td>1</td><td>0.5</td><td>30</td><td>0.46</td><td>30</td><td>14.48</td><td>14.977</td><td>0.138</td><td>14.855</td><td>0.303</td><td>13.653</td><td>0.18</td><td>0</td><td>1004.982128</td><td>110.53147</td></tr>
    <tr><td>00393485+0051355</td><td>9.895219</td><td>0.859882</td><td>39.3</td><td>11.415</td><td>0.031</td><td>3</td><td>10.755</td><td>0.044</td><td>3</td><td>10.514</td><td>0.068</td><td>3</td><td>0.6</td><td>-30</td><td>0.7</td><td>-60</td><td>92.29</td><td>11.415</td><td>0.018</td><td>10.155</td><td>0.054</td><td>9.976</td><td>0.085</td><td>0</td><td>301.813395</td><td>109.639102</td></tr>
    <tr><td>00392964+0103495</td><td>9.873526</td><td>1.063769</td><td>10.9</td><td>14.463</td><td>0.065</td><td>0</td><td>13.618</td><td>0.067</td><td>0</td><td>13.258</td><td>0.091</td><td>0</td><td>0.4</td><td>55</td><td>0.28</td><td>60</td><td>20.35</td><td>14.2</td><td>0.086</td><td>13.363</td><td>0.091</td><td>13.101</td><td>0.133</td><td>0</td><td>665.301415</td><td>18.051526</td></tr>
    <tr><td>00403343+0049079</td><td>10.139293</td><td>0.818865</td><td>5.0</td><td>15.484</td><td>0.15</td><td>0</td><td>--</td><td>--</td><td>--</td><td>13.97</td><td>0.137</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>10.05</td><td>15.035</td><td>0.183</td><td>14.725</td><td>0.0</td><td>13.654</td><td>0.189</td><td>0</td><td>1189.207905</td><td>102.088788</td></tr>
    <tr><td>00393319+0035505</td><td>9.888305</td><td>0.597381</td><td>11.5</td><td>13.156</td><td>0.033</td><td>0</td><td>12.509</td><td>0.043</td><td>0</td><td>12.073</td><td>0.059</td><td>0</td><td>0.6</td><td>-55</td><td>0.52</td><td>-40</td><td>21.64</td><td>13.026</td><td>0.04</td><td>12.247</td><td>0.046</td><td>11.978</td><td>0.065</td><td>0</td><td>1078.11027</td><td>166.0785</td></tr>
    <tr><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td></tr>
    <tr><td>00391798+0041588</td><td>9.824936</td><td>0.699687</td><td>6.1</td><td>15.685</td><td>0.168</td><td>0</td><td>14.89</td><td>0.191</td><td>0</td><td>14.003</td><td>0.155</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>11.4</td><td>15.677</td><td>0.312</td><td>14.415</td><td>0.226</td><td>13.568</td><td>0.19</td><td>0</td><td>678.863209</td><td>177.360117</td></tr>
    <tr><td>00384796+0034572</td><td>9.699858</td><td>0.582578</td><td>5.1</td><td>14.925</td><td>0.077</td><td>0</td><td>14.224</td><td>0.114</td><td>0</td><td>13.536</td><td>0.079</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>10.2</td><td>14.839</td><td>0.133</td><td>14.111</td><td>0.192</td><td>13.461</td><td>0.137</td><td>0</td><td>1176.842625</td><td>200.856597</td></tr>
    <tr><td>00390392+0050579</td><td>9.766345</td><td>0.849419</td><td>5.0</td><td>14.895</td><td>0.07</td><td>0</td><td>14.238</td><td>0.087</td><td>0</td><td>13.834</td><td>0.11</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>10.05</td><td>14.706</td><td>0.107</td><td>14.033</td><td>0.132</td><td>13.75</td><td>0.187</td><td>0</td><td>227.201453</td><td>232.24689</td></tr>
    <tr><td>00391339+0051508</td><td>9.805797</td><td>0.864135</td><td>52.8</td><td>10.362</td><td>0.014</td><td>0</td><td>9.631</td><td>0.017</td><td>0</td><td>9.334</td><td>0.024</td><td>0</td><td>0.3</td><td>-15</td><td>0.4</td><td>-15</td><td>75.02</td><td>10.279</td><td>0.015</td><td>9.527</td><td>0.016</td><td>9.247</td><td>0.023</td><td>0</td><td>93.990015</td><td>203.598476</td></tr>
    <tr><td>00391786+0054458</td><td>9.824418</td><td>0.912743</td><td>27.9</td><td>11.082</td><td>0.016</td><td>0</td><td>10.384</td><td>0.022</td><td>0</td><td>10.147</td><td>0.032</td><td>0</td><td>0.5</td><td>5</td><td>0.7</td><td>5</td><td>42.75</td><td>10.914</td><td>0.018</td><td>10.251</td><td>0.021</td><td>10.031</td><td>0.03</td><td>0</td><td>93.596555</td><td>18.308033</td></tr>
    <tr><td>00385879+0057269</td><td>9.744971</td><td>0.957478</td><td>5.0</td><td>15.535</td><td>0.122</td><td>0</td><td>14.796</td><td>0.145</td><td>0</td><td>14.278</td><td>0.165</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>10.05</td><td>15.535</td><td>0.122</td><td>14.623</td><td>0.227</td><td>14.147</td><td>0.269</td><td>0</td><td>358.163568</td><td>314.246475</td></tr>
    <tr><td>00391879+0053308</td><td>9.828303</td><td>0.891909</td><td>15.4</td><td>13.044</td><td>0.047</td><td>0</td><td>12.412</td><td>0.063</td><td>0</td><td>12.077</td><td>0.094</td><td>0</td><td>0.8</td><td>60</td><td>0.74</td><td>65</td><td>23.62</td><td>12.755</td><td>0.048</td><td>12.283</td><td>0.072</td><td>11.713</td><td>0.096</td><td>0</td><td>45.544562</td><td>72.287562</td></tr>
    <tr><td>00391213+0102408</td><td>9.80055</td><td>1.044691</td><td>5.0</td><td>15.568</td><td>0.126</td><td>0</td><td>15.047</td><td>0.181</td><td>0</td><td>14.356</td><td>0.176</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>10.05</td><td>15.295</td><td>0.181</td><td>15.047</td><td>0.181</td><td>14.067</td><td>0.25</td><td>0</td><td>566.696375</td><td>354.276982</td></tr>
    <tr><td>00383990+0104442</td><td>9.666268</td><td>1.078968</td><td>5.3</td><td>15.255</td><td>0.108</td><td>0</td><td>14.232</td><td>0.121</td><td>0</td><td>13.873</td><td>0.113</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>10.44</td><td>15.151</td><td>0.18</td><td>13.812</td><td>0.149</td><td>13.552</td><td>0.155</td><td>0</td><td>873.946372</td><td>321.851314</td></tr>
    <tr><td>00384916+0050212</td><td>9.704872</td><td>0.839244</td><td>5.1</td><td>15.075</td><td>0.088</td><td>0</td><td>14.651</td><td>0.17</td><td>0</td><td>13.804</td><td>0.101</td><td>0</td><td>1.0</td><td>90</td><td>1.0</td><td>90</td><td>10.2</td><td>15.053</td><td>0.159</td><td>14.651</td><td>0.17</td><td>13.682</td><td>0.171</td><td>0</td><td>437.740484</td><td>246.331036</td></tr>
    </table>



OK, looks like they both have ``ra`` and ``dec`` columns, so we should
be able to use that to make ``SkyCoord``\ s.

You might first think you need to create a separate ``SkyCoord`` for
*every* row in the table, given that up until now all ``SkyCoord``\ s we
made were for just a single point. You could do this, but it will make
your code much slower. Instead, ``SkyCoord`` supports *arrays* of
coordinate values - you just pass in array-like inputs (array
``Quantity``\ s, lists of strings, ``Table`` columns, etc.), and
``SkyCoord`` will happily do all of its operations element-wise.


:inputnumrole:`In[19]:`


.. code:: python

    coo_sdss = SkyCoord(sdss['ra']*u.deg, sdss['dec']*u.deg)
    coo_twomass = SkyCoord(twomass['ra'], twomass['dec'])

Note a subtle difference here: you had to give units for SDSS but *not*
for 2MASS. This is because the 2MASS table has units associated with the
columns, while the SDSS table does not (so you have to put them in
manually).

Now we simply use the ``SkyCoord.match_to_catalog_sky`` method to match
the two catalogs. Note that order matters: we're matching 2MASS to SDSS
because there are many *more* entires in the SDSS, so it seems likely
that most 2MASS objects are in SDSS (but not vice versa).


:inputnumrole:`In[20]:`


.. code:: python

    idx_sdss, d2d_sdss, d3d_sdss = coo_twomass.match_to_catalog_sky(coo_sdss)

``idx`` are the indecies into ``coo_sdss`` that get the closest matches,
while ``d2d`` and ``d3d`` are the on-sky and real-space distances
between the matches. In our case ``d3d`` can be ignored because we
didn't give a line-of-sight distance, so its value is not particularly
useful. But ``d2d`` provides a good diagnosis of whether we actually
have real matches:


:inputnumrole:`In[21]:`


.. code:: python

    plt.hist(d2d_sdss.arcsec, histtype='step', range=(0,2))
    plt.xlabel('separation [arcsec]')
    plt.tight_layout()


:outputnumrole:`Out[21]:`



.. image:: nboutput/coordinates_45_0.png



Ok, they're all within an arcsecond that's promising. But are we sure
it's not just that *anything* has matches within an arcescond? Lets
check by comparing to a set of *random* points.

We first create a set of uniformly random points (with size matching
``coo_twomass``) that cover the same range of RA/Decs that are in
``coo_sdss``.


:inputnumrole:`In[22]:`


.. code:: python

    ras_sim = np.random.rand(len(coo_twomass))*coo_sdss.ra.ptp() + coo_sdss.ra.min()
    decs_sim = np.random.rand(len(coo_twomass))*coo_sdss.dec.ptp() + coo_sdss.dec.min()
    ras_sim, decs_sim


:outputnumrole:`Out[22]:`




.. parsed-literal::

    (<Angle [ 9.87029145,  9.93490892,  9.95462551,  9.78988164,  9.75174236,
              9.82731236,  9.60431398,  9.86440752,  9.53615992, 10.01702578,
              9.86267217,  9.99984224,  9.96609812,  9.74556998,  9.79238976,
              9.99300797,  9.80710402,  9.86056295, 10.05024451, 10.06328443,
              9.92664842,  9.53228843,  9.59892149] deg>,
     <Angle [0.71782275, 1.10843359, 0.65969538, 0.76998109, 1.11159179,
             0.59153497, 1.20911213, 0.98995878, 0.96378493, 1.18735066,
             1.03439977, 0.68870729, 0.72560203, 1.15463665, 1.21611903,
             0.64572407, 1.08784572, 1.0497186 , 0.66867458, 1.11249272,
             0.64293906, 0.81912909, 0.91232126] deg>)



Now we create a ``SkyCoord`` from these points and match it to
``coo_sdss`` just like we did above for 2MASS.

Note that we do not need to explicitly specify units for ``ras_sim`` and
``decs_sim``, because they already are unitful ``Angle`` objects because
they were created from ``coo_sdss.ra``/``coo_sdss.dec``.


:inputnumrole:`In[23]:`


.. code:: python

    coo_simulated = SkyCoord(ras_sim, decs_sim)  
    idx_sim, d2d_sim, d3d_sim = coo_simulated.match_to_catalog_sky(coo_sdss)

Now lets plot up the histogram of separations from our simulated catalog
so we can compare to the above results from the *real* catalog.


:inputnumrole:`In[24]:`


.. code:: python

    plt.hist(d2d_sim.arcsec, bins='auto', histtype='step', label='Simulated', linestyle='dashed')
    plt.hist(d2d_sdss.arcsec, bins='auto', histtype='step', label='2MASS')
    plt.xlabel('separation [arcsec]')
    plt.legend(loc=0)
    plt.tight_layout()


:outputnumrole:`Out[24]:`



.. image:: nboutput/coordinates_51_0.png



Alright, great - looks like randomly placed sources should be more like
an arc\ *minute* away, so we can probably trust that our earlier matches
which were within an arc\ *second* are valid. So with that in mind, we
can start computing things like colors that combine the SDSS and 2MASS
photometry.


:inputnumrole:`In[25]:`


.. code:: python

    rmag = sdss['r'][idx_sdss]
    grcolor = sdss['g'][idx_sdss] - rmag
    rKcolor = rmag - twomass['k_m_ext']
    
    plt.subplot(1, 2, 1)
    plt.scatter(rKcolor, rmag)
    plt.xlabel('r-K')
    plt.ylabel('r')
    plt.xlim(2.5, 4)
    plt.ylim(18, 12) #mags go backwards!
    
    plt.subplot(1, 2, 2)
    plt.scatter(rKcolor, rmag)
    plt.xlabel('r-K')
    plt.ylabel('g-r')
    plt.xlim(2.5, 4)
    
    plt.tight_layout()


:outputnumrole:`Out[25]:`



.. image:: nboutput/coordinates_53_0.png



For more on what matching options are available, check out the
`separation and matching section of the astropy
documentation <http://astropy.readthedocs.org/en/latest/coordinates/matchsep.html>`__.
Or for more on what you can do with ``SkyCoord``, see `its API
documentation <http://astropy.readthedocs.org/en/latest/api/astropy.coordinates.SkyCoord.html>`__.

Exercises
~~~~~~~~~

Check that the ``d2d_sdss`` variable matches the on-sky separations you
get from comaparing the matched ``coo_sdss`` entries to ``coo_twomass``.

Hint: You'll likely find the ``SkyCoord.separation()`` method useful
here.


:inputnumrole:`In[None]:`



Compute the *physical* separation between two (or more) objects in the
catalogs. You'll need line-of-sight distances, so a reasonable guess
might be the distance to HCG 7, which is about 55 Mpc.

Hint: you'll want to create new ``SkyCoord`` objects, but with
``distance`` attributes. There's also a ``SkyCoord`` method that should
do the rest of the work, but you'll have to poke around to figure out
what it is.


:inputnumrole:`In[None]:`



Transforming between coordinate systems and planning observations
-----------------------------------------------------------------

Now lets say something excites you about one of the objects in this
catalog, and you want to know if and when you might go about observing
it. ``astropy.coordinates`` provides tools to enable this, as well.

Introducting frame transformations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To understand the code in this section, it may help to read over the
`overview of the astropy coordinates
scheme <http://astropy.readthedocs.org/en/latest/coordinates/index.html#overview-of-astropy-coordinates-concepts>`__.
The key bit to understand is that all coordinates in astropy are in
particular "frames", and we can transform between a specific
``SkyCoord`` object from one frame to another. For example, we can
transform our previously-defined center of HCG7 from ICRS to Galactic
coordinates:


:inputnumrole:`In[26]:`


.. code:: python

    hcg7_center.galactic


:outputnumrole:`Out[26]:`




.. parsed-literal::

    <SkyCoord (Galactic): (l, b) in deg
        (116.47556813, -61.83099472)>



The above is actually a special "quick-access" form which internally
does the same as what's in the cell below: uses the ``transform_to()``
method to convert from one frame to another.


:inputnumrole:`In[27]:`


.. code:: python

    from astropy.coordinates import Galactic
    hcg7_center.transform_to(Galactic())


:outputnumrole:`Out[27]:`




.. parsed-literal::

    <SkyCoord (Galactic): (l, b) in deg
        (116.47556813, -61.83099472)>



Note that changing frames also changes some of the attributes of the
object, but usually in a way that makes sense:


:inputnumrole:`In[28]:`


.. code:: python

    hcg7_center.galactic.ra  # should fail because galactic coordinates are l/b not RA/Dec


:outputnumrole:`Out[28]:`


::


    

    AttributeErrorTraceback (most recent call last)

    <ipython-input-28-d7bc134707f6> in <module>()
    ----> 1 hcg7_center.galactic.ra  # should fail because galactic coordinates are l/b not RA/Dec
    

    ~/project/venv/lib/python3.6/site-packages/astropy/coordinates/sky_coordinate.py in __getattr__(self, attr)
        693         # Fail
        694         raise AttributeError("'{0}' object has no attribute '{1}'"
    --> 695                              .format(self.__class__.__name__, attr))
        696 
        697     def __setattr__(self, attr, val):


    AttributeError: 'SkyCoord' object has no attribute 'ra'



:inputnumrole:`In[29]:`


.. code:: python

    hcg7_center.galactic.b


:outputnumrole:`Out[29]:`




.. math::

    -61^\circ49{}^\prime51.581{}^{\prime\prime}



Using frame transformations to get to AltAz
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To actually do anything with observability we need to convert to a frame
local to an on-earth observer. By far the most common choice is
horizontal coordinates, or "AltAz" coordinates. We first need to specify
both where and when we want to try to observe.


:inputnumrole:`In[30]:`


.. code:: python

    from astropy.coordinates import EarthLocation
    from astropy.time import Time
    
    observing_location = EarthLocation(lat='31d57.5m', lon='-111d35.8m', height=2096*u.m)  # Kitt Peak, Arizona
    # If you're using astropy v1.1 or later, you can replace the above with this:
    #observing_location = EarthLocation.of_site('Kitt Peak')
    
    observing_time = Time('2010-12-21 1:00')  # 1am UTC=6pm AZ mountain time

Now we use these to create an ``AltAz`` frame object. Note that this
frame has some other information about the atmosphere, which can be used
to correct for atmospheric refraction. Here we leave that alone, because
the default is to ignore this effect (by setting the pressure to 0).


:inputnumrole:`In[31]:`


.. code:: python

    from astropy.coordinates import AltAz
    
    aa = AltAz(location=observing_location, obstime=observing_time)
    aa


:outputnumrole:`Out[31]:`




.. parsed-literal::

    <AltAz Frame (obstime=2010-12-21 01:00:00.000, location=(-1994310.09211632, -5037908.606337594, 3357621.752122168) m, pressure=0.0 hPa, temperature=0.0 deg_C, relative_humidity=0, obswl=1.0 micron)>



Now we can just transform our ICRS ``SkyCoord`` to ``AltAz`` to get the
location in the sky over Kitt Peak at the requested time.


:inputnumrole:`In[32]:`


.. code:: python

    hcg7_center.transform_to(aa)


:outputnumrole:`Out[32]:`




.. parsed-literal::

    <SkyCoord (AltAz: obstime=2010-12-21 01:00:00.000, location=(-1994310.09211632, -5037908.606337594, 3357621.752122168) m, pressure=0.0 hPa, temperature=0.0 deg_C, relative_humidity=0, obswl=1.0 micron): (az, alt) in deg
        (149.19392036, 55.0624736)>



Alright, it's up at 6pm, but that's pretty early to be observing. We
could just try various times one at a time to see if the airmass is at a
darker time, but we can do better: lets try to create an airmass plot.


:inputnumrole:`In[33]:`


.. code:: python

    # this gives a Time object with an *array* of times
    delta_hours = np.linspace(0, 6, 100)*u.hour
    full_night_times = observing_time + delta_hours
    full_night_aa_frames = AltAz(location=observing_location, obstime=full_night_times)
    full_night_aa_coos = hcg7_center.transform_to(full_night_aa_frames)
    
    plt.plot(delta_hours, full_night_aa_coos.secz)
    plt.xlabel('Hours from 6pm AZ time')
    plt.ylabel('Airmass [Sec(z)]')
    plt.ylim(0.9,3)
    plt.tight_layout()


:outputnumrole:`Out[33]:`



.. image:: nboutput/coordinates_78_0.png



Great! Looks like it's at the lowest airmass in another hour or so
(7pm). But might that might still be twilight... When should we start
observing for proper dark skies? Fortunately, astropy provides a
``get_sun`` function that can be used to check this. Lets use it to
check if we're in 18-degree twilight or not.


:inputnumrole:`In[34]:`


.. code:: python

    from astropy.coordinates import get_sun
    
    full_night_sun_coos = get_sun(full_night_times).transform_to(full_night_aa_frames)
    plt.plot(delta_hours, full_night_sun_coos.alt.deg)
    plt.axhline(-18, color='k')
    plt.xlabel('Hours from 6pm AZ time')
    plt.ylabel('Sun altitude')
    plt.tight_layout()


:outputnumrole:`Out[34]:`



.. image:: nboutput/coordinates_80_0.png



Looks like it's just below 18 degrees at 7, so you should be good to go!

Exercises
~~~~~~~~~

Try to actually compute to some arbitrary precision (rather than
eye-balling on a plot) when 18 degree twilight or sunrise/sunset hits on
that night.


:inputnumrole:`In[None]:`



Try converting the HCG7 coordinates to an equatorial frame at some other
equinox a while in the past (like J2000). Do you see the precession of
the equinoxes?

Hint: To see a diagram of the supported frames look
`here <http://docs.astropy.org/en/stable/coordinates/#module-astropy.coordinates>`__.
One of those will do what you need if you give it the right frame
attributes.


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

