Data
=====

.. _installation:


How to get access to your data 
------------

Import 

.. code-block:: python

   import os
   import numpy as np
   import matplotlib.pyplot as plt
   import qcodes as qc
   from qcodes import load_by_run_spec, ScaledParameter
   from qcodes import Station, initialise_or_create_database_at, \
       load_or_create_experiment, Measurement
   from qcodes.dataset.plotting import plot_dataset, plot_by_id
   qc.config.plotting['default_color_map']='Blues_r'
   qc.logger.start_all_logging()

You need to load the dataset that you used for your measurements

.. code-block:: python

   sample_name = 'W11168_S23_top2'   
   path = os.getcwd()
   path=os.path.join(path, sample_name)
   db_file_path = os.path.join(path, sample_name + '.db') #or put directly the path 
     
   initialise_or_create_database_at(db_file_path)
      
      
Plot
----------------

Derivative
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   dataset=load_by_run_spec(captured_run_id=591)
   x=dataset.get_parameter_data()['QD_Current']['P5']
   y=dataset.get_parameter_data()['QD_Current']['QD_Bias']
   I=dataset.get_parameter_data()['QD_Current']['QD_Current']
   
   deriv_y=np.gradient(I,np.abs(y[0][1]-y[0][0]),axis=1)
   deriv_x=np.gradient(I,np.abs(x[:,0][1]-x[:,0][0]),axis=0)
   
   plt.pcolormesh(x,y,deriv_x)
   plt.colorbar(label='dI/dP')
   plt.xlabel('P5 (mV)')
   plt.ylabel('Bias (mV)')
      

Patch
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   id_image=4434
   plt.figure(figsize=(6,4))
   for i in range(25):
       id_image+=3
       dataset=load_by_run_spec(captured_run_id=id_image)
       x=dataset.get_parameter_data()['CS1_current']['P1']
       y=dataset.get_parameter_data()['CS1_current']['P3']
       R=dataset.get_parameter_data()['CS1_current']['CS1_current']
   
       deriv_y=np.gradient(R,np.abs(y[0][1]-y[0][0]),axis=1)
       deriv_x=np.gradient(R,np.abs(x[:,0][1]-x[:,0][0]),axis=0)
   
       plt.pcolormesh(x,y,deriv_x,cmap='Blues_r')   
   
   #plt.pcolormesh(x,y,I,cmap='hot')
   cb = plt.colorbar()
   cb.set_label(label='dxCS1_current (A)',fontsize=16)
   cb.ax.tick_params(labelsize=16)
   
   #plt.colorbar(label='dI/dP',labelsize=16)
   plt.xlabel('P1 (mV)',fontsize=16)
   plt.ylabel('P3 (mV)',fontsize=16)
   plt.xticks(fontsize=16)
   plt.yticks(fontsize=16)

   plt.show()



   

  
      
