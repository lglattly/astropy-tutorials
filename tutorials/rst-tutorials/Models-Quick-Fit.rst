.. meta::
    :keywords: filterTutorials, filterModeling, filterModelFitting, filterAstroquery, filterMatplotlib, filterAstrostatistics






.. raw:: html

    <a href="../_static/Models-Quick-Fit/Models-Quick-Fit.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks/Models-Quick-Fit/Models-Quick-Fit.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

    <div id="spacer"></div>

.. role:: inputnumrole
.. role:: outputnumrole

.. _Models-Quick-Fit:

Make a quick fit using astropy.modeling
=======================================

Authors
-------

Rocio Kiman, Lia Corrales, and Zé Vinícius.

Learning Goals
--------------

-  Know basic models in Astropy Modeling
-  Learn common functions to fit
-  Be able to make a quick fit of your data
-  Visualize the fit

Keywords
--------

modeling, model fitting, astroquery, matplotlib, astrostatistics

Summary
-------

In this tutorial, we will become familiar with the models available in
``astropy.modeling`` and learn how to make a quick fit to our data.

Check http://docs.astropy.org/en/stable/modeling/ for more information

Imports
~~~~~~~


:inputnumrole:`In[1]:`


.. code:: python

    import numpy as np
    import matplotlib.pyplot as plt
    from astropy.modeling import models, fitting
    from astroquery.vizier import Vizier
    import scipy.optimize
    # Make plots display in notebooks
    %matplotlib inline 

1) Fit a Linear model: Three steps to fit data using astropy.modeling
---------------------------------------------------------------------

We are going to start with a **linear fit to real data**. The data comes
from the paper `Bhardwaj et al.
2017 <https://ui.adsabs.harvard.edu/?#abs/2017A%26A...605A.100B>`__.
This is a catalog of **Type II Cepheids**, which is a type of **variable
stars** that pulsate with a period between 1 and 50 days. In this part
of the tutorial, we are going to measure the **Cepheids
Period-Luminosity** relation using ``astropy.modeling``. This relation
states that if a star has a longer period, the luminosity we measure is
higher.

To get it, we are going to import it from
`Vizier <http://vizier.u-strasbg.fr/viz-bin/VizieR>`__ using
`astroquery <http://astroquery.readthedocs.io/en/latest/vizier/vizier.html>`__.


:inputnumrole:`In[2]:`


.. code:: python

    catalog = Vizier.get_catalogs('J/A+A/605/A100')

This catalog has a lot of information, but for this tutorial we are
going to work only with periods and magnitudes. Let's grab them using
the keywords ``'Period'`` and ``__Ksmag__``. Note that ``'e__Ksmag_'``
refers to the error bars in the magnitude measurements.


:inputnumrole:`In[3]:`


.. code:: python

    period = np.array(catalog[0]['Period']) 
    log_period = np.log10(period)
    k_mag = np.array(catalog[0]['__Ksmag_'])
    k_mag_err = np.array(catalog[0]['e__Ksmag_'])

Let's take a look at the magnitude measurements as a function of period:


:inputnumrole:`In[4]:`


.. code:: python

    plt.errorbar(log_period, k_mag, k_mag_err, fmt='k.')
    plt.xlabel(r'$\log_{10}$(Period [days])')
    plt.ylabel('Ks')


:outputnumrole:`Out[4]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7f9066cbae48>




.. image:: nboutput/Models-Quick-Fit_10_1.png



One could say that there is a linear relationship between log period and
magnitudes. To probe it, we want to make a fit to the data. This is
where ``astropy.modeling`` is useful. We are going to understand how in
three simple lines we can make any fit we want. We are going to start
with the linear fit, but first, let's understand what a model and a
fitter are.

Models in Astropy
~~~~~~~~~~~~~~~~~

`Models <http://docs.astropy.org/en/stable/modeling/#using-models>`__ in
Astropy are known parametrized functions. With this format they are easy
to define and to use, given that we do not need to write the function
expression every time we want to use a model, just the name. They can be
linear or non-linear in the variables. Some examples of models are:

-  `Gaussian1D <http://docs.astropy.org/en/stable/api/astropy.modeling.functional_models.Gaussian1D.html#astropy.modeling.functional_models.Gaussian1D>`__
-  `Trapezoid1D <http://docs.astropy.org/en/stable/api/astropy.modeling.functional_models.Trapezoid1D.html#astropy.modeling.functional_models.Trapezoid1D>`__
-  `Polynomial1D <http://docs.astropy.org/en/stable/api/astropy.modeling.polynomial.Polynomial1D.html#astropy.modeling.polynomial.Polynomial1D>`__
-  `Sine1D <http://docs.astropy.org/en/stable/api/astropy.modeling.functional_models.Sine1D.html#astropy.modeling.functional_models.Sine1D>`__
-  `Linear1D <http://docs.astropy.org/en/stable/api/astropy.modeling.functional_models.Linear1D.html#astropy.modeling.functional_models.Linear1D>`__
-  The
   `list <http://docs.astropy.org/en/stable/modeling/#module-astropy.modeling.functional_models>`__
   continues.

Fitters in Astropy
~~~~~~~~~~~~~~~~~~

Fitters in Astropy are the classes resposable for making the fit. They
can be linear or non-linear in the parameters (no the variable, like
models). Some examples are:

-  `LevMarLSQFitter() <http://docs.astropy.org/en/stable/api/astropy.modeling.fitting.LevMarLSQFitter.html#astropy.modeling.fitting.LevMarLSQFitter>`__
   Levenberg-Marquardt algorithm and least squares statistic.
-  `LinearLSQFitter() <http://docs.astropy.org/en/stable/api/astropy.modeling.fitting.LinearLSQFitter.html#astropy.modeling.fitting.LinearLSQFitter>`__
   A class performing a linear least square fitting.
-  `SLSQPLSQFitter() <http://docs.astropy.org/en/stable/api/astropy.modeling.fitting.SLSQPLSQFitter.html#astropy.modeling.fitting.SLSQPLSQFitter>`__
   SLSQP optimization algorithm and least squares statistic.
-  `SimplexLSQFitter() <http://docs.astropy.org/en/stable/api/astropy.modeling.fitting.SimplexLSQFitter.html#astropy.modeling.fitting.SimplexLSQFitter>`__
   Simplex algorithm and least squares statistic.
-  More detailles
   `here <http://docs.astropy.org/en/stable/modeling/#id21>`__

Now we continue with our fitting.

Step 1: Model
^^^^^^^^^^^^^

First we need to choose which model we are going to use to fit to our
data. As we said before, our data looks like a linear relation, so we
are going to use a linear model.


:inputnumrole:`In[5]:`


.. code:: python

    model = models.Linear1D()

Step 2: Fitter
^^^^^^^^^^^^^^

Second we are going to choose the fitter we want to use. This choice is
basically which method we want to use to fit the model to the data. In
this case we are going to use the `Linear Least Square
Fitting <https://www.mathworks.com/help/curvefit/least-squares-fitting.html>`__.
In the next exercise we are going to analyze how to choose the fitter.


:inputnumrole:`In[6]:`


.. code:: python

    fitter = fitting.LinearLSQFitter() 

Step 3: Fit Data
^^^^^^^^^^^^^^^^

Finally, we give to our **fitter** (method to fit the data) the
**model** and the **data** to perform the fit. Note that we are
including weights: This means that values with higher error will have
smaller weight (less importance) in the fit, and the contrary for data
with smaller errors. This way of fitting is called *Weighted Linear
Least Squares* and you can find more information about it
`here <https://www.mathworks.com/help/curvefit/least-squares-fitting.html>`__
or
`here <https://en.wikipedia.org/wiki/Least_squares#Weighted_least_squares>`__.


:inputnumrole:`In[7]:`


.. code:: python

    best_fit = fitter(model, log_period, k_mag, weights=1.0/k_mag_err**2)
    print(best_fit)


:outputnumrole:`Out[7]:`


.. parsed-literal::

    Model: Linear1D
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Parameters:
               slope            intercept     
        ------------------- ------------------
        -2.0981402468153076 13.418358848855155


And that's it!

We can evaluate the fit at our particular x axis by doing
``best_fit(x)``.


:inputnumrole:`In[8]:`


.. code:: python

    plt.errorbar(log_period,k_mag,k_mag_err,fmt='k.')
    plt.plot(log_period, best_fit(log_period), color='g', linewidth=3)  
    plt.xlabel(r'$\log_{10}$(Period [days])')
    plt.ylabel('Ks')


:outputnumrole:`Out[8]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7f9066b843c8>




.. image:: nboutput/Models-Quick-Fit_27_1.png



**Conclusion:** Remember, you can fit data with three lines of code:

1) Choose a
   `model <http://docs.astropy.org/en/stable/modeling/#module-astropy.modeling.functional_models>`__.

2) Choose a
   `fitter <http://docs.astropy.org/en/stable/modeling/#id21>`__.

3) Pass to the fitter the model and the data to perform fit.

Exercise
--------

Use the model ``Polynomial1D(degree=1)`` to fit the same data and
compare the results.


:inputnumrole:`In[None]:`



2) Fit a Polynomial model: Choose fitter wisely
-----------------------------------------------

For second example, lets fit a polynomial of degree more than 1. In this
case, we are going to create fake data to make the fit. Note that we are
adding gaussian noise to the data with the function
``np.random.normal(0,2)`` which gives a random number from a gaussian
distribution with mean 0 and standard deviation 2.


:inputnumrole:`In[9]:`


.. code:: python

    N = 100
    x1 = np.linspace(0, 4, N)  # Makes an array from 0 to 4 of N elements
    y1 = x1**3 - 6*x1**2 + 12*x1 - 9 
    # Now we add some noise to the data
    y1 += np.random.normal(0, 2, size=len(y1)) #One way to add random gaussian noise
    sigma = 1.5
    y1_err = np.ones(N)*sigma 

Let's plot it to see how it looks like:


:inputnumrole:`In[10]:`


.. code:: python

    plt.errorbar(x1, y1, yerr=y1_err,fmt='k.')
    plt.xlabel('$x_1$')  
    plt.ylabel('$y_1$')


:outputnumrole:`Out[10]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7f9066ae02e8>




.. image:: nboutput/Models-Quick-Fit_36_1.png



To fit this data lets remember the three steps: model, fitter and
perform fit.


:inputnumrole:`In[11]:`


.. code:: python

    model_poly = models.Polynomial1D(degree=3)
    fitter_poly = fitting.LinearLSQFitter() 
    best_fit_poly = fitter_poly(model_poly, x1, y1, weights = 1.0/y1_err**2)


:inputnumrole:`In[12]:`


.. code:: python

    print(best_fit_poly)


:outputnumrole:`Out[12]:`


.. parsed-literal::

    Model: Polynomial1D
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Degree: 3
    Parameters:
                c0                 c1                c2                 c3        
        ------------------ ----------------- ------------------ ------------------
        -8.555456509160916 11.75192049661028 -6.296144613797063 1.0975177268488066


What would happend if we use a different fitter (method)? Lets use the
same model but with ``SimplexLSQFitter`` as fitter.


:inputnumrole:`In[13]:`


.. code:: python

    fitter_poly_2 = fitting.SimplexLSQFitter()
    best_fit_poly_2 = fitter_poly_2(model_poly, x1, y1, weights = 1.0/y1_err**2)


:outputnumrole:`Out[13]:`


.. parsed-literal::

    WARNING: Model is linear in parameters; consider using linear fitting methods. [astropy.modeling.fitting]
    WARNING: The fit may be unsuccessful; Maximum number of iterations reached. [astropy.modeling.optimizers]



:inputnumrole:`In[14]:`


.. code:: python

    print(best_fit_poly_2)


:outputnumrole:`Out[14]:`


.. parsed-literal::

    Model: Polynomial1D
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Degree: 3
    Parameters:
                 c0                 c1                  c2                 c3        
        ------------------- ------------------ ------------------- ------------------
        -0.7723060597250879 0.3063605903712064 -1.5217469892353714 0.4933950238618753


Note that we got a warning after using ``SimplexLSQFitter`` to fit the
data. The first line says:

``WARNING: Model is linear in parameters; consider using linear fitting methods. [astropy.modeling.fitting]``

If we look at the model we chose:
:math:`y = c_0 + c_1\times x + c_2\times x^2 + c_3\times x^3`, it is
linear in the parameters :math:`c_i`. The warning means that
``SimplexLSQFitter`` works better with models that are not linear in the
parameters, and that we should use a linear fitter like
``LinearLSQFitter``. The second line says:

``WARNING: The fit may be unsuccessful; Maximum number of iterations reached. [astropy.modeling.optimizers]``

so it is not surprising that the results are different, because this
means that the fitter is not working properly. Lets discuss a method to
choose between fits and remember to **pay attention** when you choose
the **fitter**.

Compare results
^^^^^^^^^^^^^^^

One way to check which model parameters are a better fit is calculating
the `Reduced Chi Square
Value <https://en.wikipedia.org/wiki/Reduced_chi-squared_statistic>`__.
Lets define a function to do that because we are going to use it several
times.


:inputnumrole:`In[15]:`


.. code:: python

    def calc_reduced_chi_square(fit, x, y, yerr, N, n_free):
        '''
        fit (array) values for the fit
        x,y,yerr (arrays) data
        N total number of points
        n_free number of parameters we are fitting
        '''
        return 1.0/(N-n_free)*sum(((fit - y)/yerr)**2)


:inputnumrole:`In[16]:`


.. code:: python

    reduced_chi_squared = calc_reduced_chi_square(best_fit_poly(x1), x1, y1, y1_err, N, 4)
    print('Reduced Chi Squared with LinearLSQFitter: {}'.format(reduced_chi_squared))


:outputnumrole:`Out[16]:`


.. parsed-literal::

    Reduced Chi Squared with LinearLSQFitter: 1.8996166383259825



:inputnumrole:`In[17]:`


.. code:: python

    reduced_chi_squared = calc_reduced_chi_square(best_fit_poly_2(x1), x1, y1, y1_err, N, 4)
    print('Reduced Chi Squared with SimplexLSQFitter: {}'.format(reduced_chi_squared))


:outputnumrole:`Out[17]:`


.. parsed-literal::

    Reduced Chi Squared with SimplexLSQFitter: 4.037322356538704


As we can see, the *Reduced Chi Square* for the first fit is closer to
one, which means this fit is better. Note that this is what we expected
after the discussion of the warnings.

We can also compare the two fits visually.


:inputnumrole:`In[18]:`


.. code:: python

    plt.errorbar(x1, y1, yerr=y1_err,fmt='k.')
    plt.plot(x1, best_fit_poly(x1), color='r', linewidth=3, label='LinearLSQFitter()')  
    plt.plot(x1, best_fit_poly_2(x1), color='g', linewidth=3, label='SimplexLSQFitter()')
    plt.xlabel(r'$\log_{10}$(Period [days])')
    plt.ylabel('Ks')
    plt.legend()


:outputnumrole:`Out[18]:`




.. parsed-literal::

    <matplotlib.legend.Legend at 0x7f90669c79e8>




.. image:: nboutput/Models-Quick-Fit_50_1.png



Results are as espected, the fit performed with the linear fitter is
better than the second one, non linear.

**Conclusion:** Pay attention when you choose the fitter.

3) Fit a Gaussian: Lets compare to scipy
----------------------------------------

Scipy has the function
`scipy.optimize.curve\_fit <https://docs.scipy.org/doc/scipy-1.0.0/reference/generated/scipy.optimize.curve_fit.html>`__
to fit in a similar way we are doing things. Lets compare the two
methods with fake data in the shape of a gaussian.


:inputnumrole:`In[19]:`


.. code:: python

    mu, sigma, amplitude = 0.0, 10.0, 10.0
    N2 = 100
    x2 = np.linspace(-30, 30, N)
    y2 = amplitude * np.exp(-(x2-mu)**2 / (2*sigma**2))
    y2 = np.array([y_point + np.random.normal(0, 1) for y_point in y2])   #Another way to add random gaussian noise
    sigma = 1
    y2_err = np.ones(N)*sigma


:inputnumrole:`In[20]:`


.. code:: python

    plt.errorbar(x2, y2, yerr=y2_err, fmt='k.')
    plt.xlabel('$x_2$')
    plt.ylabel('$y_2$')


:outputnumrole:`Out[20]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7f90669df908>




.. image:: nboutput/Models-Quick-Fit_55_1.png



Lets do our three steps to make the fit we want. For this fit we are
going to use a non-linear fitter: ``LevMarLSQFitter``, because the model
we need (``Gaussian1D``) is non-linear in the parameters.


:inputnumrole:`In[21]:`


.. code:: python

    model_gauss = models.Gaussian1D()
    fitter_gauss = fitting.LevMarLSQFitter()
    best_fit_gauss = fitter_gauss(model_gauss, x2, y2, weights=1/y2_err**2)


:inputnumrole:`In[22]:`


.. code:: python

    print(best_fit_gauss)


:outputnumrole:`Out[22]:`


.. parsed-literal::

    Model: Gaussian1D
    Inputs: ('x',)
    Outputs: ('y',)
    Model set size: 1
    Parameters:
            amplitude            mean             stddev     
        ----------------- ------------------ ----------------
        9.911353179817407 0.4664808649978389 9.92217762452695


We can get the `covariance
matrix <http://mathworld.wolfram.com/CovarianceMatrix.html>`__ from
``LevMarLSQFitter``, which provides an error for our fit parameters by
doing ``fitter.fit_info['param_cov']``. The elements in the diagonal of
this matrix are the square of the errors. We can check the order of the
parameters using:


:inputnumrole:`In[23]:`


.. code:: python

    model_gauss.param_names


:outputnumrole:`Out[23]:`




.. parsed-literal::

    ('amplitude', 'mean', 'stddev')




:inputnumrole:`In[24]:`


.. code:: python

    cov_diag = np.diag(fitter_gauss.fit_info['param_cov'])
    print(cov_diag)


:outputnumrole:`Out[24]:`


.. parsed-literal::

    [0.05488178 0.07330016 0.07350361]


Then:


:inputnumrole:`In[25]:`


.. code:: python

    print('Amplitude: {} +\- {}'.format(best_fit_gauss.amplitude.value, np.sqrt(cov_diag[0])))
    print('Mean: {} +\- {}'.format(best_fit_gauss.mean.value, np.sqrt(cov_diag[1])))
    print('Standard Deviation: {} +\- {}'.format(best_fit_gauss.stddev.value, np.sqrt(cov_diag[2])))


:outputnumrole:`Out[25]:`


.. parsed-literal::

    Amplitude: 9.911353179817407 +\- 0.2342686105624387
    Mean: 0.4664808649978389 +\- 0.2707400178924447
    Standard Deviation: 9.92217762452695 +\- 0.27111549147772596


We can apply the same method with ``scipy.optimize.curve_fit``, and
compare the results using again the *Reduced Chi Square Value*.


:inputnumrole:`In[26]:`


.. code:: python

    def f(x,a,b,c):
        return a * np.exp(-(x-b)**2/(2.0*c**2))


:inputnumrole:`In[27]:`


.. code:: python

    p_opt, p_cov = scipy.optimize.curve_fit(f,x2, y2, sigma=y1_err)
    a,b,c = p_opt
    best_fit_gauss_2 = f(x2,a,b,c)


:inputnumrole:`In[28]:`


.. code:: python

    print(p_opt)


:outputnumrole:`Out[28]:`


.. parsed-literal::

    [9.91135241 0.46648192 9.92217916]



:inputnumrole:`In[29]:`


.. code:: python

    print('Amplitude: {} +\- {}'.format(p_opt[0], np.sqrt(p_cov[0,0])))
    print('Mean: {} +\- {}'.format(p_opt[1], np.sqrt(p_cov[1,1])))
    print('Standard Deviation: {} +\- {}'.format(p_opt[2], np.sqrt(p_cov[2,2])))


:outputnumrole:`Out[29]:`


.. parsed-literal::

    Amplitude: 9.911352411637719 +\- 0.23426942289047176
    Mean: 0.4664819193778 +\- 0.27073809579932634
    Standard Deviation: 9.922179163732988 +\- 0.2711135199816969


Compare results
^^^^^^^^^^^^^^^


:inputnumrole:`In[30]:`


.. code:: python

    reduced_chi_squared = calc_reduced_chi_square(best_fit_gauss(x2), x2, y2, y2_err, N2, 3)
    print('Reduced Chi Squared using astropy.modeling: {}'.format(reduced_chi_squared))


:outputnumrole:`Out[30]:`


.. parsed-literal::

    Reduced Chi Squared using astropy.modeling: 1.0608225862428253



:inputnumrole:`In[31]:`


.. code:: python

    reduced_chi_squared = calc_reduced_chi_square(best_fit_gauss_2, x2, y2, y2_err, N2, 3)
    print('Reduced Chi Squared using scipy: {}'.format(reduced_chi_squared))


:outputnumrole:`Out[31]:`


.. parsed-literal::

    Reduced Chi Squared using scipy: 1.0608225862423626


As we can see there is a very small difference in the *Reduced Chi
Squared*. This actually needed to happen, because the fitter in
``astropy.modeling`` uses scipy to fit. The advantage of using
``astropy.modeling`` is you only need to change the name of the fitter
and the model to perform a completely different fit, while scipy require
us to remember the expression of the function we wanted to use.


:inputnumrole:`In[32]:`


.. code:: python

    plt.errorbar(x2, y2, yerr=y2_err, fmt='k.')
    plt.plot(x2, best_fit_gauss(x2), 'g-', linewidth=6, label='astropy.modeling')
    plt.plot(x2, best_fit_gauss_2, 'r-', linewidth=2, label='scipy')
    plt.xlabel('$x_2$')
    plt.ylabel('$y_2$')
    plt.legend()


:outputnumrole:`Out[32]:`




.. parsed-literal::

    <matplotlib.legend.Legend at 0x7f9066844860>




.. image:: nboutput/Models-Quick-Fit_73_1.png



**Conclusion:** Choose the method most convenient for every case you
need to fit. We recomend ``astropy.modeling`` because is easier to write
the name of the function you want to fit, than remember the expression
every time we want to use it. Also, ``astropy.modeling`` becomes useful
with more complicated models like `two
gaussians <http://docs.astropy.org/en/stable/modeling/#compound-models>`__
plus a `black
body <http://docs.astropy.org/en/stable/modeling/#blackbody-radiation>`__,
but that is another tutorial.

Summary:
--------

Lets review the conclusion we got in this tutorial:

1. You can fit data with **three lines of code**:

   -  model
   -  fitter
   -  perform fit to data

2. **Pay attention** when you choose the **fitter**.

3. Choose the method most convenient for every case you need to fit. We
   recomend ``astropy.modeling`` to make **quick fits of known
   functions**.

3) Exercise: Your turn to choose
--------------------------------

Exercise: For the next data: \* Choose model and fitter to fit this
data. \* Compare different options.


:inputnumrole:`In[33]:`


.. code:: python

    N3 = 100
    x3 = np.linspace(0, 3, N3)
    y3 = 5.0 * np.sin(2 * np.pi * x3)
    y3 = np.array([y_point + np.random.normal(0, 1) for y_point in y3])
    sigma = 1.5
    y3_err = np.ones(N)*sigma 


:inputnumrole:`In[34]:`


.. code:: python

    plt.errorbar(x3, y3, yerr=y3_err, fmt='k.')
    plt.xlabel('$x_3$')
    plt.ylabel('$y_3$')


:outputnumrole:`Out[34]:`




.. parsed-literal::

    <matplotlib.text.Text at 0x7f9066832cc0>




.. image:: nboutput/Models-Quick-Fit_79_1.png




:inputnumrole:`In[None]:`




.. raw:: html

    <div id="spacer"></div>

    <a href="../_static//.ipynb"><button id="download">Download tutorial notebook</button></a>
    <a href="https://beta.mybinder.org/v2/gh/astropy/astropy-tutorials/master?filepath=/tutorials/notebooks//.ipynb"><button id="binder">Interactive tutorial notebook</button></a>

