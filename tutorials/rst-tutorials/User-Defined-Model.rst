.. meta::
    :keywords: filterTutorials, filterModeling, filterUserDefinedModel, filterCustomModels, filterCompoundModels






.. raw:: html

    <a href="../_static/User-Defined-Model/User-Defined-Model.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/User-Defined-Model/User-Defined-Model.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _User-Defined-Model:

Create a User Defined Model using astropy.modeling
==================================================

Authors
-------

Rocio Kiman, Lia Corrales and Zé Vinícius.

Learning Goals
--------------

-  Know and understand tools to make user defined models with
   ``astropy`` and in which cases it could be useful
-  We will define models usign two different tools:

   -  Compound models
   -  Custom models

This tutorial assumes the student knows how to fit data using
``astropy.modeling``. This topic is covered in the `Models-Quick-Fit
tutorial <https://astropy-tutorials.readthedocs.io/en/latest/rst-tutorials/Models-Quick-Fit.html>`__.

Keywords
--------

Modeling, User Defined Model, Custom Models, Compound Models

Summary
-------

In this tutorial, we will learn how to define a new model in two ways:
with a compound model and with a custom model.

Imports
~~~~~~~


:inputnumrole:`In[1]:`


.. code:: python

    import numpy as np
    import matplotlib.pyplot as plt
    from astropy.io import fits
    from astropy.modeling import models, fitting
    from astropy.modeling.models import custom_model
    from astropy.modeling import Fittable1DModel, Parameter
    from astroquery.sdss import SDSS


:outputnumrole:`Out[1]:`


.. parsed-literal::

    /home/circleci/project/venv/lib/python3.6/site-packages/astroquery/sdss/__init__.py:29: UserWarning: Experimental: SDSS has not yet been refactored to have its API match the rest of astroquery (but it's nearly there).
      warnings.warn("Experimental: SDSS has not yet been refactored to have its API "


Fit an Emission Line from a star spectrum with ``astropy.modeling``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

M dwarfs are low mass stars (less than half of the mass of the sun).
Currently we do not understand completely the physics inside low mass
stars because they do not behave the same way higher mass stars do. For
example, they stay magnetically active longer than higher mass stars.
One way to measure magnetic activity is the height of the
`:math:`H\alpha` <https://en.wikipedia.org/wiki/H-alpha>`__ emission
line. It is located at :math:`6563` Angstroms at the spectrum.

Let's search for a spectrum of an M dwarf in the Sloan Digital Sky
Survey (SDSS). First, we are going to look for the spectrum in the `SDSS
database <https://dr12.sdss.org/basicSpectra>`__. SDSS has a particular
way to identify the stars it observes: it uses three numbers: Plate,
Fiber and MJD (Modified Julian Date). The star we are going to use has:
\* Plate: 1349 \* Fiber: 216 \* MJD: 52797

So go ahead, put this numbers in the website and click on Plot to
visualize the spectrum. Try to localize the :math:`H\alpha` line.

We could download the spectrum by hand from this website, but we are
going to import it using the
`SDSSClass <http://astroquery.readthedocs.io/en/latest/api/astroquery.sdss.SDSSClass.html>`__
from
```astroquery.sdss`` <https://astroquery.readthedocs.io/en/latest/sdss/sdss.html#module-astroquery.sdss>`__.
We can get the spectrum using the plate, fiber and mjd in the following
way:


:inputnumrole:`In[2]:`


.. code:: python

    spectrum = SDSS.get_spectra(plate=1349, fiberID=216, mjd=52797)[0]


:outputnumrole:`Out[2]:`


.. parsed-literal::

    /home/circleci/project/venv/lib/python3.6/site-packages/astroquery/sdss/core.py:856: VisibleDeprecationWarning: Reading unicode strings without specifying the encoding argument is deprecated. Set the encoding, use None for the system default.
      comments='#'))


.. parsed-literal::

    Downloading http://data.sdss3.org/sas/dr12/sdss/spectro/redux/26/spectra/1349/spec-1349-52797-0216.fits [Done]


Now that we have the spectrum...
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

One way to check what is inside the fits file ``spectrum`` is the
following:


:inputnumrole:`In[3]:`


.. code:: python

    spectrum[1].columns


:outputnumrole:`Out[3]:`




.. parsed-literal::

    ColDefs(
        name = 'flux'; format = 'E'
        name = 'loglam'; format = 'E'
        name = 'ivar'; format = 'E'
        name = 'and_mask'; format = 'J'
        name = 'or_mask'; format = 'J'
        name = 'wdisp'; format = 'E'
        name = 'sky'; format = 'E'
        name = 'model'; format = 'E'
    )



To plot the spectrum we need the flux of light as a function of the
wavelength (usually called lambda or :math:`\lambda`). Note that the
wavelength is in log scale: loglam, so we are going to calculate
:math:`10^\lambda` to remove this scale.


:inputnumrole:`In[4]:`


.. code:: python

    flux = spectrum[1].data['flux']
    lam = 10**(spectrum[1].data['loglam'])

Each fits file is different according to what the person who made it
wanted to include and how the observation was made. The information
about the file usually is in ``fitsfile[0].header``. We would like to
have the units from the flux and wavelength. For SDSS spectrum we found
where the units are in this `SDSS
tutorial <https://www.sdss.org/dr12/tutorials/quicklook/#python>`__:


:inputnumrole:`In[5]:`


.. code:: python

    #Units of the flux
    units_flux = spectrum[0].header['bunit']
    print(units_flux)


:outputnumrole:`Out[5]:`


.. parsed-literal::

    1E-17 erg/cm^2/s/Ang



:inputnumrole:`In[6]:`


.. code:: python

    #Units of the wavelegth
    units_wavelength_full = spectrum[0].header['WAT1_001']
    print(units_wavelength_full)


:outputnumrole:`Out[6]:`


.. parsed-literal::

    wtype=linear label=Wavelength units=Angstroms


We are going to select only the characters of the unit we care about:
Angstroms


:inputnumrole:`In[7]:`


.. code:: python

    units_wavelength = units_wavelength_full[36:]
    print(units_wavelength)


:outputnumrole:`Out[7]:`


.. parsed-literal::

    Angstroms


Now we are ready to plot the spectrum with all the information.


:inputnumrole:`In[8]:`


.. code:: python

    plt.plot(lam, flux, color='k')
    plt.xlim(6300,6700)
    plt.axvline(x=6563, linestyle='--')
    plt.xlabel('Wavelength ({})'.format(units_wavelength))
    plt.ylabel('Flux ({})'.format(units_flux))
    plt.show()


:outputnumrole:`Out[8]:`



.. image:: nboutput/User-Defined-Model_18_0.png



Fit an Emission Line with a Gaussian Model
------------------------------------------

We just plotted our spectrum! Check different ranges of wavelength to
see how the full spectrum looks like in comparison to the one we saw
before.

The blue dashed line marks the :math:`H\alpha` emission line. We can
tell this is an active star because it has a strong emission line.

Now, we would like to measure the height of this line. Let's use
``astropy.modeling`` to fit a gaussian to the :math:`H\alpha` line. We
are going to initialize a gaussian model at the position of the
:math:`H\alpha` line. The idea is that the gaussian amplitude will tell
us the height of the line.

We are going to go quickly over this part of the tutorial because it
involves fitting with ``astropy.modeling`` and this was explained in the
`Models-Quick-Fit
tutorial <https://astropy-tutorials.readthedocs.io/en/latest/rst-tutorials/Models-Quick-Fit.html>`__.


:inputnumrole:`In[9]:`


.. code:: python

    gausian_model = models.Gaussian1D(1, 6563, 10)
    fitter = fitting.LevMarLSQFitter()
    gaussian_fit = fitter(gausian_model, lam, flux)

Let's plot the results.


:inputnumrole:`In[10]:`


.. code:: python

    plt.figure(figsize=(8,5))
    plt.plot(lam, flux, color='k')
    plt.plot(lam, gaussian_fit(lam), color='darkorange')
    plt.xlim(6300,6700)
    plt.xlabel('Wavelength (Angstroms)')
    plt.ylabel('Flux ({})'.format(units_flux))
    plt.show()


:outputnumrole:`Out[10]:`



.. image:: nboutput/User-Defined-Model_23_0.png



We can see the fit is not doing a good job. Let's print the parameters
of this fit:


:inputnumrole:`In[11]:`


.. code:: python

    print(gaussian_fit)


:outputnumrole:`Out[11]:`


.. parsed-literal::

    Model: Gaussian1D
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Parameters:
            amplitude            mean             stddev      
        ----------------- ----------------- ------------------
        16.75070626907759 9456.749523005932 2368.3957026986304


Exercise
--------

Go back to the previous plot and try to make the fit work. Note: **Do
not spend more than 10 minutes** in this exercise. A couple of ideas to
try: \* Is it not working because of the model we chose to fit? You can
find more models to use
`here <http://docs.astropy.org/en/stable/modeling/#module-astropy.modeling.functional_models>`__.
\* Is it not working because of the fitter we chose? \* Is it not
working because of the range of data we are fitting? \* Is it not
working because how we are plotting the data?

Compound models
===============

One model is not enough to make this fit work. We need to combine a
couple of models to make a `compound
model <http://docs.astropy.org/en/stable/modeling/#compound-models>`__
in ``astropy``. The idea is that we can sum, rest, divide or multiply
models that already exist in
```astropy.modeling`` <http://docs.astropy.org/en/stable/modeling/#models-and-fitting-astropy-modeling>`__
and fit the compound model to our data.

For our problem we are going to combine the gaussian with a polynomial
of degree 1 to account for the background spectrum close to the
:math:`H\alpha` line. Take a look at the plot we made before to convince
yourself that this is the case.

Now let's make our compound model!


:inputnumrole:`In[12]:`


.. code:: python

    compound_model = models.Gaussian1D(1, 6563, 10) + models.Polynomial1D(degree=1)

After this point, the algorithm to fit the data works exactly the same
as before except we use a compound model instead of the gaussian model.


:inputnumrole:`In[13]:`


.. code:: python

    fitter = fitting.LevMarLSQFitter()
    compound_fit = fitter(compound_model, lam, flux)


:inputnumrole:`In[14]:`


.. code:: python

    plt.figure(figsize=(8,5))
    plt.plot(lam, flux, color='k')
    plt.plot(lam, compound_fit(lam), color='darkorange')
    plt.xlim(6300,6700)
    plt.xlabel('Wavelength (Angstroms)')
    plt.ylabel('Flux ({})'.format(units_flux))
    plt.show()


:outputnumrole:`Out[14]:`



.. image:: nboutput/User-Defined-Model_32_0.png



It works! Let's take a look to the fit we just made.


:inputnumrole:`In[15]:`


.. code:: python

    print(compound_fit)


:outputnumrole:`Out[15]:`


.. parsed-literal::

    Model: CompoundModel0
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Expression: [0] + [1]
    Components: 
        [0]: <Gaussian1D(amplitude=1., mean=6563., stddev=10.)>
    
        [1]: <Polynomial1D(1, c0=0., c1=0.)>
    Parameters:
           amplitude_0          mean_0       ...         c1_1        
        ----------------- ------------------ ... --------------------
        7.020891699420025 6564.1363171649145 ... 0.003239952049620112


Let's print all the parameters in a fancy way:


:inputnumrole:`In[16]:`


.. code:: python

    for x,y in zip(compound_fit.param_names, compound_fit.parameters):
        print(x,y)


:outputnumrole:`Out[16]:`


.. parsed-literal::

    amplitude_0 7.020891699420025
    mean_0 6564.1363171649145
    stddev_0 1.9776147858047488
    c0_1 -12.793356164944736
    c1_1 0.003239952049620112


We can see that the result includes all the fit parameters from the
gaussian (mean, std and amplitude) and the two coefficients from the
polynomial of degree 1. So now if we want to see just the amplitude:


:inputnumrole:`In[17]:`


.. code:: python

    compound_fit.amplitude_0


:outputnumrole:`Out[17]:`




.. parsed-literal::

    Parameter('amplitude_0', value=7.020891699420025)



**Conclusions:** What was the difference between the first simple
Gaussian and the compound model? The linear model that we added up to
the gaussian model allowed the base of the Gaussian fit to have a slope
and a background level. Normal Gaussians go to zero at :math:`\pm \inf`;
this one doesn't.

More tools to fit the data: fixed and bounded parameters of the model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The mean value of the gaussian from our previous model indicates where
the :math:`H\alpha` line is. In our fit result, we can tell that it is a
little off from :math:`6563` Angstroms. One way to fix this is to fix
some of the parameters of the model. In ``astropy.modeling`` these are
called **`fixed
parameters <http://docs.astropy.org/en/stable/api/astropy.modeling.Parameter.html#astropy.modeling.Parameter.fixed>`__**.


:inputnumrole:`In[18]:`


.. code:: python

    compound_model_fixed = models.Gaussian1D(1, 6563, 10) + models.Polynomial1D(degree=1)
    compound_model_fixed.mean_0.fixed = True

Now let's use this new model with a fixed parameter to fit the data the
same way we did before.


:inputnumrole:`In[19]:`


.. code:: python

    fitter = fitting.LevMarLSQFitter()
    compound_fit_fixed = fitter(compound_model_fixed, lam, flux)


:inputnumrole:`In[20]:`


.. code:: python

    plt.figure(figsize=(8,5))
    plt.plot(lam, flux, color='k')
    plt.plot(lam, compound_fit_fixed(lam), color='darkorange')
    plt.xlim(6300,6700)
    plt.xlabel('Wavelength (Angstroms)')
    plt.ylabel('Flux ({})'.format(units_flux))
    plt.show()


:outputnumrole:`Out[20]:`



.. image:: nboutput/User-Defined-Model_44_0.png




:inputnumrole:`In[21]:`


.. code:: python

    print(compound_fit_fixed)


:outputnumrole:`Out[21]:`


.. parsed-literal::

    Model: CompoundModel1
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Expression: [0] + [1]
    Components: 
        [0]: <Gaussian1D(amplitude=1., mean=6563., stddev=10.)>
    
        [1]: <Polynomial1D(1, c0=0., c1=0.)>
    Parameters:
           amplitude_0     mean_0 ...         c0_1                c1_1        
        ------------------ ------ ... ------------------- --------------------
        1.5325722844012764 6563.0 ... -12.791029980456878 0.003237448622613733


We can see in the plot that the height of the fit does not match the
:math:`H\alpha` line height. What happend here is that we were too
strict with the mean value, so we did not get a good fit. But the mean
value is where we want it! Let's loosen this condition a little. Another
thing we can do is to define a `**minimum and maximum
value** <http://docs.astropy.org/en/stable/api/astropy.modeling.Parameter.html#astropy.modeling.Parameter.max>`__
for the mean.


:inputnumrole:`In[22]:`


.. code:: python

    compound_model_bounded = models.Gaussian1D(1, 6563, 10) + models.Polynomial1D(degree=1)
    delta = 0.5
    compound_model_bounded.mean_0.max = 6563 + delta
    compound_model_bounded.mean_0.min = 6563 - delta
    
    fitter = fitting.LevMarLSQFitter()
    compound_fit_bounded = fitter(compound_model_bounded, lam, flux)


:inputnumrole:`In[23]:`


.. code:: python

    plt.figure(figsize=(8,5))
    plt.plot(lam, flux, color='k')
    plt.plot(lam, compound_fit_bounded(lam), color='darkorange')
    plt.xlim(6300,6700)
    plt.xlabel('Wavelength (Angstroms)')
    plt.ylabel('Flux ({})'.format(units_flux))
    plt.show()


:outputnumrole:`Out[23]:`



.. image:: nboutput/User-Defined-Model_48_0.png




:inputnumrole:`In[24]:`


.. code:: python

    print(compound_fit_bounded)


:outputnumrole:`Out[24]:`


.. parsed-literal::

    Model: CompoundModel2
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Expression: [0] + [1]
    Components: 
        [0]: <Gaussian1D(amplitude=1., mean=6563., stddev=10.)>
    
        [1]: <Polynomial1D(1, c0=0., c1=0.)>
    Parameters:
           amplitude_0    mean_0 ...         c0_1                c1_1       
        ----------------- ------ ... ------------------- -------------------
        6.657305353992826 6563.5 ... -12.793362521502136 0.00323995002187704


Better! By loosening the condition we added to the mean value, we got a
better fit and the mean of the gaussian is closer to where we want it.

Exercise
--------

Modify the value of delta to change the minimum and maximum values for
the mean of the gaussian. Look for: \* The better delta so the mean is
closer to the real value of the :math:`H\alpha` line. \* What is the
minimum delta for which the fit is still good according to the plot?

Custom model
============

What should you do if you need a model that ``astropy.modeling`` doesn't
provide? To solve that problem, Astropy has another tool called `custom
model <http://docs.astropy.org/en/stable/modeling/new.html>`__. Using
this tool, we can create any model we want.

| We will describe two way to create a custom model: \*
  `basic <http://docs.astropy.org/en/stable/modeling/new.html#basic-custom-models>`__
| \*
  `full <http://docs.astropy.org/en/stable/modeling/new.html#a-step-by-step-definition-of-a-1-d-gaussian-model>`__

We use the basic custom model when we need a simple function to fit and
the full custom model when we need a more complex function. Let's use an
example to understand each one of the custom models.

Basic custom model
------------------

An **Exponential Model** is not provided by Astropy models. Let's see
one example of basic custom model for this case. First, let's simulate a
dataset that follows an exponential:


:inputnumrole:`In[25]:`


.. code:: python

    x1 = np.linspace(0,10,100)
    
    a = 3
    b = -2
    c = 0
    y1 = a*np.exp(b*x1+c)
    y1 += np.random.normal(0., 0.2, x1.shape)
    y1_err = np.ones(x1.shape)*0.2


:inputnumrole:`In[26]:`


.. code:: python

    plt.errorbar(x1 , y1, yerr=y1_err, fmt='.')
    plt.show()


:outputnumrole:`Out[26]:`



.. image:: nboutput/User-Defined-Model_57_0.png



We can define a simple custom model by specifying which parameters we
want to fit.


:inputnumrole:`In[27]:`


.. code:: python

    @custom_model
    def exponential(x, a=1., b=1., c=1.):
        '''
        f(x)=a*exp(b*x + c)
        '''
        return a*np.exp(b*x+c)

Now we have one more available model to use in the same way we fit data
with ``astropy.modeling``.


:inputnumrole:`In[28]:`


.. code:: python

    exp_model = exponential(1.,-1.,1.)  
    fitter = fitting.LevMarLSQFitter()
    exp_fit = fitter(exp_model, x1, y1, weights = 1.0/y1_err**2)


:inputnumrole:`In[29]:`


.. code:: python

    plt.errorbar(x1 , y1, yerr=y1_err, fmt='.')
    plt.plot(x1, exp_fit(x1))
    plt.show()


:outputnumrole:`Out[29]:`



.. image:: nboutput/User-Defined-Model_62_0.png




:inputnumrole:`In[30]:`


.. code:: python

    print(exp_fit)


:outputnumrole:`Out[30]:`


.. parsed-literal::

    Model: exponential
    Inputs: ('x',)
    Outputs: ('x',)
    Model set size: 1
    Parameters:
                a                   b                   c         
        ------------------ ------------------- -------------------
        2.3331408349188885 -2.0175401217207427 0.21857337185694276


The fit looks good in the plot. Let's check the parameters and the
Reduced Chi Square value, which will give us information about the
goodness of the fit.


:inputnumrole:`In[31]:`


.. code:: python

    def calc_reduced_chi_square(fit, x, y, yerr, N, n_free):
        '''
        fit (array) values for the fit
        x,y,yerr (arrays) data
        N total number of points
        n_free number of parameters we are fitting
        '''
        return 1.0/(N-n_free)*sum(((fit - y)/yerr)**2)


:inputnumrole:`In[32]:`


.. code:: python

    calc_reduced_chi_square(exp_fit(x1), x1, y1, y1_err, len(x1), 3)


:outputnumrole:`Out[32]:`




.. parsed-literal::

    1.0091141026674066



The Reduced Chi Square value is close to 1. Great! This means our fit is
good, and we can corroborate it by comparing the values we got for the
parameters and the ones we used to simulate the data.

**Note:** Fits of non-linear parameters (like in our example) are
extremely dependent on initial conditions. Pay attention to the initial
conditions you select.

Exercise
--------

Modify the initial conditions of the fit and check yourself the relation
between the best fit parameters and the initial conditions for the
previous example. You can check it by looking at the Reduced Chi Square
value: if it gets closer to 1 the fit is better and vice versa. To
compare the quality of the fits you can take note of the Reduced Chi
Square value you get for each initial condition.

Full custom model
-----------------

What if we want to use a model from ``astropy.modeling``, but with a
different set of parameters? One example is the `Sine
Model <http://docs.astropy.org/en/stable/api/astropy.modeling.functional_models.Sine1D.html#astropy.modeling.functional_models.Sine1D>`__.
It has a very particular definition of the frequency and phase. Let's
define a new Sine function with a full custom model. Again, first let's
create a simulated dataset.


:inputnumrole:`In[33]:`


.. code:: python

    x2 = np.linspace(0,10,100)
    a = 3
    b = 2
    c = 4
    d = 1
    y2 = a*np.sin(b*x2+c)+d
    y2 += np.random.normal(0., 0.5, x2.shape)
    y2_err = np.ones(x2.shape)*0.3


:inputnumrole:`In[34]:`


.. code:: python

    plt.errorbar(x2, y2, yerr=y2_err, fmt='.')
    plt.show()


:outputnumrole:`Out[34]:`



.. image:: nboutput/User-Defined-Model_72_0.png



For the full custom model we can easily set the **derivative of the
function**, which is used by different
`fitters <http://docs.astropy.org/en/stable/modeling/#id21>`__, for
example the ``LevMarLSQFitter()``.


:inputnumrole:`In[35]:`


.. code:: python

    class SineNew(Fittable1DModel):
        a = Parameter(default=1.)
        b = Parameter(default=1.)
        c = Parameter(default=1.)
        d = Parameter(default=1.)
            
        @staticmethod
        def evaluate(x, a, b, c, d):
            return a*np.sin(b*x+c)+d
        
        @staticmethod
        def fit_deriv(x, a, b, c, d):
            d_a = np.sin(b*x+c)+d
            d_b = a*np.cos(b*x+c)*x
            d_c = a*np.sin(b*x+c)
            d_d = np.ones(x.shape)
            return [d_a, d_b, d_c, d_d]

**Note** Defining default values for the fit parameters allows to define
a model as ``model=SineNew()``

We are going to fit the data with our **new model**. Once more, the fit
is very **sensitive to the initial conditions** due to the non-linearity
of the parameters.


:inputnumrole:`In[36]:`


.. code:: python

    sine_model = SineNew(a=4.,b=2.,c=4.,d=0.)  
    fitter = fitting.LevMarLSQFitter()
    sine_fit = fitter(sine_model, x2, y2, weights = 1.0/y2_err**2)


:inputnumrole:`In[37]:`


.. code:: python

    plt.errorbar(x2, y2, yerr=y2_err, fmt='.')
    plt.plot(x2,sine_fit(x2))
    plt.show()


:outputnumrole:`Out[37]:`



.. image:: nboutput/User-Defined-Model_77_0.png




:inputnumrole:`In[38]:`


.. code:: python

    print(sine_fit)


:outputnumrole:`Out[38]:`


.. parsed-literal::

    Model: SineNew
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Parameters:
                a                 b                  c                 d         
        ----------------- ------------------ ----------------- ------------------
        3.476856137468753 2.0030401432956424 3.800144366275022 0.8967050214673217



:inputnumrole:`In[39]:`


.. code:: python

    calc_reduced_chi_square(sine_fit(x2), x2, y2, y2_err, len(x2), 3)


:outputnumrole:`Out[39]:`




.. parsed-literal::

    5.101513860481203



The Reduced Chi Squared value is showing the same as the plot: this fit
could be improved. The Reduced Chi Squared is not close to 1 and the fit
is off by small phase.

Exercise
--------

Play with the initial values for the last fit and improve the Reduced
Chi Squared value.

**Note:** A fancy way of doing this would be to code a function which
iterates over different initial conditions, optimizing the Reduced Chi
Squared value. No need to do it here, but feel free to try.

Exercise
--------

Custom models are also useful when we want to fit an **unusual
function** to our data. As an example, create a full custom model to fit
the following data.


:inputnumrole:`In[40]:`


.. code:: python

    x3 = np.linspace(-2,3,100)
    y3 = x3**2* np.exp(-0.5 * (x3)**3 / 2**2)
    y3 += np.random.normal(0., 0.5, x3.shape)
    y3_err = np.ones(x3.shape)*0.5


:inputnumrole:`In[41]:`


.. code:: python

    plt.errorbar(x3,y3,yerr=y3_err,fmt='.')
    plt.show()


:outputnumrole:`Out[41]:`



.. image:: nboutput/User-Defined-Model_84_0.png




:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

