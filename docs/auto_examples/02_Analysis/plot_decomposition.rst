

.. _sphx_glr_auto_examples_02_Analysis_plot_decomposition.py:


Decomposition
=============

Here we demonstrate how to perform a decomposition of an imaging dataset.
All you need to do is specify the algorithm. Currently, we have several
different algorithms implemented from
`scikit-learn <http://scikit-learn.org/stable/>`_
('PCA','ICA','Factor Analysis','Non-Negative Matrix Factorization').



Load Data
---------

First, let's load the pain data for this example.  We need to specify the
training levels.  We will grab the pain intensity variable from the data.X
field.



.. code-block:: python


    from nltools.datasets import fetch_pain

    data = fetch_pain()







Center within subject
---------------------

Next we will center the data.  However, because we are combining three pain
image intensities, we will perform centering separately for each participant.



.. code-block:: python


    data_center = data.empty()
    for s in data.X['SubjectID'].unique():
        sdat = data[data.X['SubjectID']==s]
        data_center = data_center.append(sdat.standardize())








Decomposition with Factor Analysis
----------------------------------

We can now decompose the data into a subset of factors. For this example,
we will use factor analysis, but we can easily switch out the algorithm with
either 'pca', 'ica', or 'nnmf'. Decomposition can be performed over voxels
or alternatively over images.  Here we perform decomposition over images,
which means that voxels are the observations and images are the features. Set
axis='voxels' to decompose voxels treating images as observations.



.. code-block:: python


    n_components = 5

    output = data_center.decompose(algorithm='fa', axis='images',
                                    n_components=n_components)








Display the available data in the output dictionary. The output contains
a Brain_Data instance with the brain factors (e.g., output['components']),
the feature by component weighting matrix (output['weights']), and the
scikit-learn decomposition object (output['decomposition_object'].
The Decomposition object contains the full set of information, including
the parameters, the components, and the explained variance.



.. code-block:: python


    print(output.keys())





.. rst-class:: sphx-glr-script-out

 Out::

    dict_keys(['decomposition_object', 'components', 'weights'])


Next, we can plot the results.  Here we plot a heatmap of how each
brain image loads on each component.  We also plot the degree to which
each voxel loads on each component.



.. code-block:: python


    import seaborn as sns
    import matplotlib.pylab as plt

    with sns.plotting_context(context='paper', font_scale=2):
        sns.heatmap(output['weights'])
        plt.ylabel('Images')
        plt.xlabel('Components')

    output['components'].plot(limit=n_components)




.. rst-class:: sphx-glr-horizontal


    *

      .. image:: /auto_examples/02_Analysis/images/sphx_glr_plot_decomposition_001.png
            :scale: 47

    *

      .. image:: /auto_examples/02_Analysis/images/sphx_glr_plot_decomposition_002.png
            :scale: 47

    *

      .. image:: /auto_examples/02_Analysis/images/sphx_glr_plot_decomposition_003.png
            :scale: 47

    *

      .. image:: /auto_examples/02_Analysis/images/sphx_glr_plot_decomposition_004.png
            :scale: 47

    *

      .. image:: /auto_examples/02_Analysis/images/sphx_glr_plot_decomposition_005.png
            :scale: 47

    *

      .. image:: /auto_examples/02_Analysis/images/sphx_glr_plot_decomposition_006.png
            :scale: 47




Finally, we can examine if any of the components track the intensity of
pain.  We plot the average loading of each component onto each pain
intensity level. Interestingly, the first component with positive weights
on the bilateral insula, s2, and  ACC monotonically tracks the pain
intensity level.



.. code-block:: python


    import pandas as pd

    wt =  pd.DataFrame(output['weights'])
    wt['PainIntensity'] = data_center.X['PainLevel'].replace({1:'Low',
    														  2:'Medium',
    														  3:'High'}
    														 ).reset_index(drop=True)

    wt_long = pd.melt(wt,
    				  value_vars=range(n_components),
    				  value_name='Weight',
    				  var_name='Component',
    				  id_vars='PainIntensity')

    with sns.plotting_context(context='paper', font_scale=2):
        sns.factorplot(data=wt_long,
                        y='Weight',
                        x='PainIntensity',
                        hue='Component',
                        order=['Low','Medium','High'],
                        aspect=1.5)



.. image:: /auto_examples/02_Analysis/images/sphx_glr_plot_decomposition_007.png
    :align: center




**Total running time of the script:** ( 0 minutes  37.744 seconds)



.. only :: html

 .. container:: sphx-glr-footer


  .. container:: sphx-glr-download

     :download:`Download Python source code: plot_decomposition.py <plot_decomposition.py>`



  .. container:: sphx-glr-download

     :download:`Download Jupyter notebook: plot_decomposition.ipynb <plot_decomposition.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.readthedocs.io>`_
