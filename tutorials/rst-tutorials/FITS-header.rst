.. meta::
    :keywords: filterTutorials, filterFitsHeader, filterFileInputOutput






.. raw:: html

    <a href="../_static/FITS-header/FITS-header.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/FITS-header/FITS-header.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _FITS-header:

Edit a FITS header
==================

Authors
-------

Adrian Price-Whelan, Adam Ginsburg

Learning Goals
--------------

-  Read a FITS file and retrieve the header metadata
-  Edit the FITS header and write the modified file as a FITS file

Keywords
--------

FITS header, file input/output

Summary
-------

This tutorial describes how to read in, edit a FITS header, and then
write it back out to disk. For this example we're going to change the
``OBJECT`` keyword.

This tutorial uses
`astropy.io.fits <http://docs.astropy.org/en/latest/io/fits/index.html>`__,
which was formerly released separately as ``pyfits``. If you have used
``pyfits`` to manipulate FITS files then you may already be familiar
with the features and syntax of the package. We start by importing the
subpackage into our local namespace, and allows us to access the
functions and classes as ``fits.name_of_function()``. For example, to
access the ``getdata()`` function, we *don't* have to do
``astropy.io.fits.getdata()`` and can instead simple use
``fits.getdata()``. You may run across old documentation or tutorials
that use the name ``pyfits``. Such examples will begin with
``import pyfits`` and then the command ``fits.getdata()`` (for example)
would be written as ``pyfits.getdata()``.


:inputnumrole:`In[1]:`


.. code:: python

    from astropy.io import fits

``astropy.io.fits`` provides a lot of flexibility for reading FITS files
and headers, but most of the time the convenience functions are the
easiest way to access the data. ``fits.getdata()`` reads just the data
from a FITS file, but with the ``header=True`` keyword argument will
also read the header.


:inputnumrole:`In[2]:`


.. code:: python

    data, header = fits.getdata("input_file.fits", header=True)

There is also a dedicated function for reading just the header:


:inputnumrole:`In[3]:`


.. code:: python

    hdu_number = 0 # HDU means header data unit
    fits.getheader('input_file.fits', hdu_number)


:outputnumrole:`Out[3]:`




.. parsed-literal::

    SIMPLE  =                    T / conforms to FITS standard                      
    BITPIX  =                  -64 / array data type                                
    NAXIS   =                    2 / number of array dimensions                     
    NAXIS1  =                  100                                                  
    NAXIS2  =                  100                                                  
    EXTEND  =                    T                                                  
    OBJECT  = 'KITTEN  '                                                            



but ``getdata()`` can get both the data and the header, so it is a
useful command to remember. Since the primary HDU of a FITS file must
contain image data, the data is now stored in a ``numpy`` array. The
header is stored in an object that acts like a standard Python
dictionary.


:inputnumrole:`In[4]:`


.. code:: python

    # But hdu_number = 0 is the PRIMARY HDU.How many HDUs are in this file?
    fits_inf = fits.open("input_file.fits")
    fits_inf.info() 
    fits_inf[0].header


:outputnumrole:`Out[4]:`


.. parsed-literal::

    Filename: input_file.fits
    No.    Name      Ver    Type      Cards   Dimensions   Format
      0  PRIMARY       1 PrimaryHDU       7   (100, 100)   float64   
      1                1 ImageHDU         7   (128, 128)   float64   




.. parsed-literal::

    SIMPLE  =                    T / conforms to FITS standard                      
    BITPIX  =                  -64 / array data type                                
    NAXIS   =                    2 / number of array dimensions                     
    NAXIS1  =                  100                                                  
    NAXIS2  =                  100                                                  
    EXTEND  =                    T                                                  
    OBJECT  = 'KITTEN  '                                                            



Using ``fits.open`` allowed us to look more generally at our data.
``fits_inf[0].header`` gave us the same output as ``fits.getheader``.
What will you learn if you type ``fits_inf[1].header``? Based on
``fits_inf.info()`` can you guess what will happen if you type
``fits_inf[2].header``?

Now let's change the header to give it the correct object:


:inputnumrole:`In[5]:`


.. code:: python

    header['OBJECT'] = "M31"

Finally, we have to write out the FITS file. Again, the convenience
function for this is the most useful command to remember:


:inputnumrole:`In[6]:`


.. code:: python

    fits.writeto('output_file.fits', data, header, overwrite=True)

That's it; you're done!

Two common more complicated cases are worth mentioning (but if your
needs are much more complex, you should consult the full documentation
http://docs.astropy.org/en/stable/io/fits/).

The first complication is that the FITS file you're examining and
editing might have multiple HDU's (extensions), in which case you can
specify the extension like this:


:inputnumrole:`In[7]:`


.. code:: python

    data1, header1 = fits.getdata("input_file.fits", ext=1, header=True)

This will get you the data and header associated with the index=1
extension in the FITS file. Without specifying a number, getdata() will
get the 0th extension (equivalent to saying ``ext=0``).

Another useful tip is if you want to overwrite an existing FITS file. By
default, writeto() won't let you do this, so you need to explicitly give
it permission using the ``clobber`` keyword argument:


:inputnumrole:`In[8]:`


.. code:: python

    fits.writeto('output_file.fits', data, header, overwrite=True)

A final example is if you want to make a small change to a FITS file,
for example updating a header keyword, but you do not want to read in
and write out the whole file, which can take a while. You can use the
``mode='update'`` read mode to do this:


:inputnumrole:`In[9]:`


.. code:: python

    with fits.open('input_file.fits', mode='update') as filehandle:
        filehandle[0].header['MYHDRKW'] = "My Header Keyword"

Exercise
--------

Read in the file you just wrote, and add three header keywords:

1. 'RA' for the Right Ascension of M31
2. 'DEC' for the Declination of M31
3. 'RADECSRC' with text indicating where you found the RA/Dec (web URL,
   textbook name, your photographic memory, etc.).

then write the updated header back out to a new file.


:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

