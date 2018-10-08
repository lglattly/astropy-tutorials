.. meta::
    :keywords: filterTutorials, filterMatplotlib, filterFitsImage, filterTable






.. raw:: html

    <a href="../_static/FITS-images/FITS-images.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/FITS-images/FITS-images.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _FITS-images:

Viewing and manipulating FITS images
====================================

Authors
-------

Lia Corrales

Learning Goals
--------------

-  TODO

Keywords
--------

matplotlib, FITS image, table

Summary
-------

Demonstrates the use of ``astropy.utils.data`` to download a data file,
uses ``astropy.io.fits`` to open the file, uses ``matplotlib`` to view
the image with different color scales and stretches and to make
histrograms. Also includes a demonstration of simple image stacking.


:inputnumrole:`In[1]:`


.. code:: python

    import numpy as np
    
    # Set up matplotlib
    import matplotlib.pyplot as plt
    %matplotlib inline
    
    from astropy.io import fits

The following cell is needed to download the example FITS files used
here.


:inputnumrole:`In[2]:`


.. code:: python

    from astropy.utils.data import download_file
    image_file = download_file('http://data.astropy.org/tutorials/FITS-images/HorseHead.fits', cache=True )


:outputnumrole:`Out[2]:`


.. parsed-literal::

    Downloading http://data.astropy.org/tutorials/FITS-images/HorseHead.fits [Done]


Opening FITS files and loading the image data
---------------------------------------------

I will open the FITS file and find out what it contains.


:inputnumrole:`In[3]:`


.. code:: python

    hdu_list = fits.open(image_file)
    hdu_list.info()


:outputnumrole:`Out[3]:`


.. parsed-literal::

    Filename: /home/circleci/.astropy/cache/download/py3/2c9202ae878ecfcb60878ceb63837f5f
    No.    Name      Ver    Type      Cards   Dimensions   Format
      0  PRIMARY       1 PrimaryHDU     161   (891, 893)   int16   
      1  er.mask       1 TableHDU        25   1600R x 4C   [F6.2, F6.2, F6.2, F6.2]   


Generally the image information is located in the PRIMARY block. The
blocks are numbered and can be accessed by indexing hdu\_list.


:inputnumrole:`In[4]:`


.. code:: python

    image_data = hdu_list[0].data

You data is now stored as a 2-D numpy array. Want to know the dimensions
of the image? Just look at the ``shape`` of the array.


:inputnumrole:`In[5]:`


.. code:: python

    print(type(image_data))
    print(image_data.shape)


:outputnumrole:`Out[5]:`


.. parsed-literal::

    <class 'numpy.ndarray'>
    (893, 891)


At this point, we can just close the FITS file. We have stored
everything we wanted to a variable.


:inputnumrole:`In[6]:`


.. code:: python

    hdu_list.close()

SHORTCUT
~~~~~~~~

If you don't need to examine the FITS header, you can call
``fits.getdata`` to bypass the previous steps.


:inputnumrole:`In[7]:`


.. code:: python

    image_data = fits.getdata(image_file)
    print(type(image_data))
    print(image_data.shape)


:outputnumrole:`Out[7]:`


.. parsed-literal::

    <class 'numpy.ndarray'>
    (893, 891)


Viewing the image data and getting basic statistics
---------------------------------------------------


:inputnumrole:`In[8]:`


.. code:: python

    plt.imshow(image_data, cmap='gray')
    plt.colorbar()
    
    # To see more color maps
    # http://wiki.scipy.org/Cookbook/Matplotlib/Show_colormaps


:outputnumrole:`Out[8]:`




.. parsed-literal::

    <matplotlib.colorbar.Colorbar at 0x7fc34b9a0ba8>




.. image:: nboutput/FITS-images_17_1.png



Let's get some basic statistics about our image


:inputnumrole:`In[9]:`


.. code:: python

    print('Min:', np.min(image_data))
    print('Max:', np.max(image_data))
    print('Mean:', np.mean(image_data))
    print('Stdev:', np.std(image_data))


:outputnumrole:`Out[9]:`


.. parsed-literal::

    Min: 3759
    Max: 22918
    Mean: 9831.481676287574
    Stdev: 3032.3927542049046


Plotting a histogram
~~~~~~~~~~~~~~~~~~~~

To make a histogram with ``matplotlib.pyplot.hist()``, I need to cast
the data from a 2-D to array to something one dimensional.

In this case, I am using the ndarray.flatten() to return a 1-D numpy
array.


:inputnumrole:`In[10]:`


.. code:: python

    print(type(image_data.flatten()))


:outputnumrole:`Out[10]:`


.. parsed-literal::

    <class 'numpy.ndarray'>



:inputnumrole:`In[11]:`


.. code:: python

    histogram = plt.hist(image_data.flatten(), bins='auto')


:outputnumrole:`Out[11]:`



.. image:: nboutput/FITS-images_24_0.png



Displaying the image with a logarithmic scale
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Want to use a logarithmic color scale? To do so we need to load the
``LogNorm`` object from ``matplotlib``.


:inputnumrole:`In[12]:`


.. code:: python

    from matplotlib.colors import LogNorm


:inputnumrole:`In[13]:`


.. code:: python

    plt.imshow(image_data, cmap='gray', norm=LogNorm())
    
    # I chose the tick marks based on the histogram above
    cbar = plt.colorbar(ticks=[5.e3,1.e4,2.e4])
    cbar.ax.set_yticklabels(['5,000','10,000','20,000'])


:outputnumrole:`Out[13]:`




.. parsed-literal::

    [<matplotlib.text.Text at 0x7fc348103390>,
     <matplotlib.text.Text at 0x7fc348112d30>,
     <matplotlib.text.Text at 0x7fc348463f28>]




.. image:: nboutput/FITS-images_28_1.png



Basic image math: image stacking
--------------------------------

You can perform math with the image data like any other numpy array. In
this particular example, I will stack several images of M13 taken with a
~10'' telescope.

I open a series of FITS files and store the data in a list, which I've
named ``image_concat``.


:inputnumrole:`In[14]:`


.. code:: python

    base_url = 'http://data.astropy.org/tutorials/FITS-images/M13_blue_{0:04d}.fits'
    
    image_list = [download_file(base_url.format(n), cache=True) 
                  for n in range(1, 5+1)]
    image_concat = [fits.getdata(image) for image in image_list]


:outputnumrole:`Out[14]:`


.. parsed-literal::

    Downloading http://data.astropy.org/tutorials/FITS-images/M13_blue_0001.fits [Done]
    Downloading http://data.astropy.org/tutorials/FITS-images/M13_blue_0002.fits [Done]
    Downloading http://data.astropy.org/tutorials/FITS-images/M13_blue_0003.fits [Done]
    Downloading http://data.astropy.org/tutorials/FITS-images/M13_blue_0004.fits [Done]
    Downloading http://data.astropy.org/tutorials/FITS-images/M13_blue_0005.fits [Done]


Now I'll stack the images by summing my concatenated list.


:inputnumrole:`In[15]:`


.. code:: python

    # The long way
    final_image = np.zeros(shape=image_concat[0].shape)
    
    for image in image_concat:
        final_image += image
    
    # The short way
    # final_image = np.sum(image_concat, axis=0)

I'm going to show the image, but I want to decide on the best stretch.
To do so I'll plot a histogram of the data.


:inputnumrole:`In[16]:`


.. code:: python

    image_hist = plt.hist(final_image.flatten(), bins='auto')


:outputnumrole:`Out[16]:`



.. image:: nboutput/FITS-images_36_0.png



I'll use the keywords ``vmin`` and ``vmax`` to set limits on the color
scaling for ``imshow``.


:inputnumrole:`In[17]:`


.. code:: python

    plt.imshow(final_image, cmap='gray', vmin=2E3, vmax=3E3)
    plt.colorbar()


:outputnumrole:`Out[17]:`




.. parsed-literal::

    <matplotlib.colorbar.Colorbar at 0x7fc342954a58>




.. image:: nboutput/FITS-images_38_1.png



Writing image data to a FITS file
---------------------------------

This is easy to do with the ``writeto()`` method.

You will receive an error if the file you are trying to write already
exists. That's why I've set ``clobber=True``.


:inputnumrole:`In[18]:`


.. code:: python

    outfile = 'stacked_M13_blue.fits'
    
    hdu = fits.PrimaryHDU(final_image)
    hdu.writeto(outfile, overwrite=True)

Exercises
---------

Determine the mean, median, and standard deviation of a part of the
stacked M13 image where there is *not* light from M13. Use those
statistics with a sum over the part of the image that includes M13 to
estimate the total light in this image from M13.


:inputnumrole:`In[None]:`



Show the image of the Horsehead Nebula, but in to units of *surface
brightness* (magnitudes per square arcsecond). (Hint: the *physical*
size of the image is 15x15 arcminutes.)


:inputnumrole:`In[None]:`



Now write out the image you just created, preserving the header the
original image had, but add a keyword 'UNITS' with the value 'mag per sq
arcsec'. (Hint: you may need to read the
`astropy.io.fits <http://docs.astropy.org/en/stable/io/fits/index.html>`__
documentation if you're not sure how to include both the header and the
data)


:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

