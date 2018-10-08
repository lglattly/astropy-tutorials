.. meta::
    :keywords: filterTutorials, filterTable, filterFileInputOutput, filterMatplotlib, filterFitsImage






.. raw:: html

    <a href="../_static/FITS-tables/FITS-tables.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/FITS-tables/FITS-tables.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _FITS-tables:

Viewing and manipulating data from FITS tables
==============================================

Authors
-------

Lia Corrales

Learning Goals
--------------

-  TODO

Keywords
--------

table, file input/output, matplotlib, FITS image

Summary
-------

Demonstrates the use of ``astropy.utils.data`` to download a data file,
uses ``astropy.io.fits`` and ``astropy.table`` to open the file, uses
``matplotlib`` to visualize the data.


:inputnumrole:`In[1]:`


.. code:: python

    import numpy as np
    from astropy.io import fits
    from astropy.table import Table
    from matplotlib.colors import LogNorm
    
    # Set up matplotlib
    import matplotlib.pyplot as plt
    %matplotlib inline

The following line is needed to download the example FITS files used in
this tutorial.


:inputnumrole:`In[2]:`


.. code:: python

    from astropy.utils.data import download_file

FITS files can often contain large amount of multi-dimensional data and
tables.

In this particular example, I will open a FITS file from a Chandra
observation of the Galactic Center. The file contains a list of events
with x and y coordinates, energy, and various other pieces of
information.


:inputnumrole:`In[3]:`


.. code:: python

    event_filename = download_file('http://data.astropy.org/tutorials/FITS-tables/chandra_events.fits', 
                                   cache=True)


:outputnumrole:`Out[3]:`


.. parsed-literal::

    Downloading http://data.astropy.org/tutorials/FITS-tables/chandra_events.fits [Done]


Opening the FITS file and viewing table contents
------------------------------------------------

Since the file is big, I will open with ``memmap=True`` to prevent RAM
storage issues.


:inputnumrole:`In[4]:`


.. code:: python

    hdu_list = fits.open(event_filename, memmap=True)


:inputnumrole:`In[5]:`


.. code:: python

    hdu_list.info()


:outputnumrole:`Out[5]:`


.. parsed-literal::

    Filename: /home/circleci/.astropy/cache/download/py3/26e9900d731d08997d99ada3973f4592
    No.    Name      Ver    Type      Cards   Dimensions   Format
      0  PRIMARY       1 PrimaryHDU      30   ()      
      1  EVENTS        1 BinTableHDU    890   483964R x 19C   [1D, 1I, 1I, 1J, 1I, 1I, 1I, 1I, 1E, 1E, 1E, 1E, 1J, 1J, 1E, 1J, 1I, 1I, 32X]   
      2  GTI           3 BinTableHDU     28   1R x 2C   [1D, 1D]   
      3  GTI           2 BinTableHDU     28   1R x 2C   [1D, 1D]   
      4  GTI           1 BinTableHDU     28   1R x 2C   [1D, 1D]   
      5  GTI           0 BinTableHDU     28   1R x 2C   [1D, 1D]   
      6  GTI           6 BinTableHDU     28   1R x 2C   [1D, 1D]   


I'm interested in reading EVENTS, which contains information about each
X-ray photon that hit the detector.

To find out what information the table contains, I will print the column
names.


:inputnumrole:`In[6]:`


.. code:: python

    print(hdu_list[1].columns)


:outputnumrole:`Out[6]:`


.. parsed-literal::

    ColDefs(
        name = 'time'; format = '1D'; unit = 's'
        name = 'ccd_id'; format = '1I'
        name = 'node_id'; format = '1I'
        name = 'expno'; format = '1J'
        name = 'chipx'; format = '1I'; unit = 'pixel'; coord_type = 'CPCX'; coord_unit = 'mm'; coord_ref_point = 0.5; coord_ref_value = 0.0; coord_inc = 0.023987
        name = 'chipy'; format = '1I'; unit = 'pixel'; coord_type = 'CPCY'; coord_unit = 'mm'; coord_ref_point = 0.5; coord_ref_value = 0.0; coord_inc = 0.023987
        name = 'tdetx'; format = '1I'; unit = 'pixel'
        name = 'tdety'; format = '1I'; unit = 'pixel'
        name = 'detx'; format = '1E'; unit = 'pixel'; coord_type = 'LONG-TAN'; coord_unit = 'deg'; coord_ref_point = 4096.5; coord_ref_value = 0.0; coord_inc = 0.00013666666666667
        name = 'dety'; format = '1E'; unit = 'pixel'; coord_type = 'NPOL-TAN'; coord_unit = 'deg'; coord_ref_point = 4096.5; coord_ref_value = 0.0; coord_inc = 0.00013666666666667
        name = 'x'; format = '1E'; unit = 'pixel'; coord_type = 'RA---TAN'; coord_unit = 'deg'; coord_ref_point = 4096.5; coord_ref_value = 266.41519201128; coord_inc = -0.00013666666666667
        name = 'y'; format = '1E'; unit = 'pixel'; coord_type = 'DEC--TAN'; coord_unit = 'deg'; coord_ref_point = 4096.5; coord_ref_value = -29.012248288366; coord_inc = 0.00013666666666667
        name = 'pha'; format = '1J'; unit = 'adu'; null = 0
        name = 'pha_ro'; format = '1J'; unit = 'adu'; null = 0
        name = 'energy'; format = '1E'; unit = 'eV'
        name = 'pi'; format = '1J'; unit = 'chan'; null = 0
        name = 'fltgrade'; format = '1I'
        name = 'grade'; format = '1I'
        name = 'status'; format = '32X'
    )


Now I'll we'll take this data and convert it into an `astropy
table <http://docs.astropy.org/en/stable/table/>`__. While it is
possible to access FITS tables directly from the ``.data`` attribute,
using
`Table <http://docs.astropy.org/en/stable/api/astropy.table.Table.html#astropy.table.Table>`__
tends to make a variety of common tasks more convenient.


:inputnumrole:`In[7]:`


.. code:: python

    evt_data = Table(hdu_list[1].data)

For example, a preview of the table is easily viewed by simply running a
cell with the table as the last line:


:inputnumrole:`In[8]:`


.. code:: python

    evt_data


:outputnumrole:`Out[8]:`




.. raw:: html

    <i>Table length=483964</i>
    <table id="table140677317747880" class="table-striped table-bordered table-condensed">
    <thead><tr><th>time</th><th>ccd_id</th><th>node_id</th><th>expno</th><th>chipx</th><th>chipy</th><th>tdetx</th><th>tdety</th><th>detx</th><th>dety</th><th>x</th><th>y</th><th>pha</th><th>pha_ro</th><th>energy</th><th>pi</th><th>fltgrade</th><th>grade</th><th>status [32]</th></tr></thead>
    <thead><tr><th>float64</th><th>int16</th><th>int16</th><th>int32</th><th>int16</th><th>int16</th><th>int16</th><th>int16</th><th>float32</th><th>float32</th><th>float32</th><th>float32</th><th>int32</th><th>int32</th><th>float32</th><th>int32</th><th>int16</th><th>int16</th><th>bool</th></tr></thead>
    <tr><td>238623220.9093583</td><td>3</td><td>3</td><td>68</td><td>920</td><td>8</td><td>5124</td><td>3981</td><td>5095.641</td><td>4138.995</td><td>4168.0723</td><td>5087.772</td><td>3548</td><td>3534</td><td>13874.715</td><td>951</td><td>16</td><td>4</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>1</td><td>68</td><td>437</td><td>237</td><td>4895</td><td>3498</td><td>4865.567</td><td>4621.1826</td><td>3662.1968</td><td>4915.9336</td><td>667</td><td>629</td><td>2621.1938</td><td>180</td><td>64</td><td>2</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>2</td><td>68</td><td>719</td><td>289</td><td>4843</td><td>3780</td><td>4814.835</td><td>4340.254</td><td>3935.2207</td><td>4832.552</td><td>3033</td><td>2875</td><td>12119.018</td><td>831</td><td>8</td><td>3</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>0</td><td>68</td><td>103</td><td>295</td><td>4837</td><td>3164</td><td>4807.3643</td><td>4954.385</td><td>3324.4644</td><td>4897.2754</td><td>831</td><td>773</td><td>3253.0364</td><td>223</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>1</td><td>68</td><td>498</td><td>314</td><td>4818</td><td>3559</td><td>4788.987</td><td>4560.3276</td><td>3713.6343</td><td>4832.735</td><td>3612</td><td>3439</td><td>14214.382</td><td>974</td><td>64</td><td>2</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>3</td><td>68</td><td>791</td><td>469</td><td>4663</td><td>3852</td><td>4635.4526</td><td>4268.053</td><td>3985.8496</td><td>4645.93</td><td>500</td><td>438</td><td>1952.7239</td><td>134</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>3</td><td>68</td><td>894</td><td>839</td><td>4293</td><td>3955</td><td>4266.642</td><td>4165.3203</td><td>4044.5469</td><td>4267.605</td><td>835</td><td>713</td><td>3267.5334</td><td>224</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>3</td><td>68</td><td>857</td><td>941</td><td>4191</td><td>3918</td><td>4164.815</td><td>4202.2256</td><td>3995.9353</td><td>4170.818</td><td>975</td><td>804</td><td>3817.0366</td><td>262</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>3</td><td>68</td><td>910</td><td>959</td><td>4173</td><td>3971</td><td>4146.9937</td><td>4149.364</td><td>4046.3376</td><td>4146.9106</td><td>576</td><td>446</td><td>2252.7295</td><td>155</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238623220.9093583</td><td>3</td><td>3</td><td>68</td><td>961</td><td>962</td><td>4170</td><td>4022</td><td>4144.1284</td><td>4098.4976</td><td>4096.515</td><td>4138.09</td><td>1572</td><td>1354</td><td>6154.1094</td><td>422</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td></tr>
    <tr><td>238672393.54971933</td><td>1</td><td>3</td><td>15723</td><td>933</td><td>199</td><td>4933</td><td>5040</td><td>4902.907</td><td>3082.4956</td><td>5212.4995</td><td>4766.2295</td><td>1222</td><td>1181</td><td>4819.8286</td><td>331</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238672393.54971933</td><td>1</td><td>2</td><td>15723</td><td>596</td><td>412</td><td>4720</td><td>4703</td><td>4691.51</td><td>3418.9893</td><td>4853.5117</td><td>4595.8037</td><td>3142</td><td>3020</td><td>12536.866</td><td>859</td><td>10</td><td>6</td><td>False .. False</td></tr>
    <tr><td>238672393.54971933</td><td>1</td><td>3</td><td>15723</td><td>1000</td><td>608</td><td>4524</td><td>5107</td><td>4494.713</td><td>3015.7185</td><td>5230.886</td><td>4353.018</td><td>658</td><td>585</td><td>2599.5652</td><td>179</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238672393.54971933</td><td>1</td><td>1</td><td>15723</td><td>270</td><td>917</td><td>4215</td><td>4377</td><td>4188.3325</td><td>3743.5957</td><td>4472.07</td><td>4134.221</td><td>3861</td><td>3463</td><td>15535.768</td><td>1024</td><td>16</td><td>4</td><td>False .. False</td></tr>
    <tr><td>238672393.54971933</td><td>1</td><td>0</td><td>15723</td><td>232</td><td>988</td><td>4144</td><td>4339</td><td>4117.6147</td><td>3781.8774</td><td>4425.75</td><td>4068.4873</td><td>1680</td><td>1499</td><td>6653.0815</td><td>456</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238672393.59075934</td><td>0</td><td>1</td><td>15723</td><td>366</td><td>103</td><td>3164</td><td>4766</td><td>3140.9048</td><td>3356.3208</td><td>4733.6816</td><td>3048.5664</td><td>3621</td><td>3602</td><td>14362.482</td><td>984</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238672393.59075934</td><td>0</td><td>3</td><td>15723</td><td>937</td><td>646</td><td>3707</td><td>4195</td><td>3681.2122</td><td>3925.5452</td><td>4231.8354</td><td>3651.9724</td><td>3717</td><td>3486</td><td>14653.954</td><td>1004</td><td>8</td><td>3</td><td>False .. False</td></tr>
    <tr><td>238672393.59075934</td><td>0</td><td>1</td><td>15723</td><td>406</td><td>687</td><td>3748</td><td>4726</td><td>3723.4014</td><td>3396.252</td><td>4762.421</td><td>3631.7224</td><td>1676</td><td>1536</td><td>6652.827</td><td>456</td><td>0</td><td>0</td><td>False .. False</td></tr>
    <tr><td>238672393.59075934</td><td>0</td><td>1</td><td>15723</td><td>354</td><td>870</td><td>3931</td><td>4778</td><td>3906.07</td><td>3344.775</td><td>4834.99</td><td>3807.0835</td><td>2436</td><td>2165</td><td>9672.882</td><td>663</td><td>16</td><td>4</td><td>False .. False</td></tr>
    <tr><td>238672393.63179934</td><td>6</td><td>1</td><td>15723</td><td>384</td><td>821</td><td>3259</td><td>2523</td><td>3230.9204</td><td>5596.8496</td><td>2519.2202</td><td>3401.0327</td><td>491</td><td>356</td><td>1875.9359</td><td>129</td><td>0</td><td>0</td><td>False .. False</td></tr>
    </table>



We can extract data from the table by referencing the column name.. For
example, I'll make a histogram for the energy of each photon, giving us
a sense for the spectrum (folded with the detector's efficiency).


:inputnumrole:`In[9]:`


.. code:: python

    energy_hist = plt.hist(evt_data['energy'], bins='auto')


:outputnumrole:`Out[9]:`



.. image:: nboutput/FITS-tables_18_0.png



Making a 2-D histogram with some table data
-------------------------------------------

I will make an image by binning the x and y coordinates of the events
into a 2-D histogram.

This particular observation spans five CCD chips. First we determine the
events that only fell on the main (ACIS-I) chips, which have number ids
0, 1, 2, and 3.


:inputnumrole:`In[10]:`


.. code:: python

    ii = np.in1d(evt_data['ccd_id'], [0, 1, 2, 3])
    np.sum(ii)


:outputnumrole:`Out[10]:`




.. parsed-literal::

    434858



Method 1: Use numpy to make a 2-D histogram and imshow to display it
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This method allowed me to create an image without stretching


:inputnumrole:`In[11]:`


.. code:: python

    NBINS = (100,100)
    
    img_zero, yedges, xedges = np.histogram2d(evt_data['x'][ii], evt_data['y'][ii], NBINS)
    
    extent = [xedges[0], xedges[-1], yedges[0], yedges[-1]]
    
    plt.imshow(img_zero, extent=extent, interpolation='nearest', cmap='gist_yarg', origin='lower')
    
    plt.xlabel('x')
    plt.ylabel('y')
    
    # To see more color maps
    # http://wiki.scipy.org/Cookbook/Matplotlib/Show_colormaps


:outputnumrole:`Out[11]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7ff1f5496b70>




.. image:: nboutput/FITS-tables_25_1.png



Method 2: Use hist2d with a log-normal color scheme
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


:inputnumrole:`In[12]:`


.. code:: python

    NBINS = (100,100)
    img_zero_mpl = plt.hist2d(evt_data['x'][ii], evt_data['y'][ii], NBINS, 
                              cmap='viridis', norm=LogNorm())
    
    cbar = plt.colorbar(ticks=[1.0,3.0,6.0])
    cbar.ax.set_yticklabels(['1','3','6'])
    
    plt.xlabel('x')
    plt.ylabel('y')


:outputnumrole:`Out[12]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7ff1f54bce48>




.. image:: nboutput/FITS-tables_27_1.png



Close the FITS file
-------------------

When you're done using a FITS file, it's often a good idea to close it.
That way you can be sure it won't continue using up excess memory or
file handles on your computer. (This happens automatically when you
close Python, but you never know how long that might be...)


:inputnumrole:`In[13]:`


.. code:: python

    hdu_list.close()

Exercises
---------

Make a scatter plot of the same data you histogrammed above. The
`plt.scatter <http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.scatter>`__
function is your friend for this. What are the pros and cons of doing
this?


:inputnumrole:`In[None]:`



Try the same with the
`plt.hexbin <http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.hexbin>`__
plotting function. Which do you think looks better for this kind of
data?


:inputnumrole:`In[None]:`



Choose an energy range to make a slice of the FITS table, then plot it.
How does the image change with different energy ranges?


:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

