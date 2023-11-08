Qcodes-Start a jupyter notebook
=====

.. _installation:


Importation
------------

At the begining of the jupyter notebook import:

.. code-block:: python

   import os
   import numpy as np
   import qcodes as qc
   from qcodes import load_by_run_spec, ScaledParameter
   from qcodes import Station, initialise_or_create_database_at, \
       load_or_create_experiment, Measurement
   # from qcodes.tests.instrument_mocks import DummyInstrument, \
   #    DummyInstrumentWithMeasurement
   from qcodes.utils.dataset.doNd import do1d,do2d,do0d
   from qcodes.dataset.plotting import plot_dataset, plot_by_id
   qc.config.plotting['default_color_map']='Blues_r'  #color nap for the qcodes 2d plots
   qc.config.plotting['fontsize']=16 #fontsize for the qcodes plots
   qc.logger.start_all_logging()

   #for charge sensor 
   from scipy.optimize import curve_fit
   from scipy.signal import find_peaks
      
Instruments
----------------

**Adress**

.. code-block:: python

   rm = pyvisa.ResourceManager()
   rm.list_resources()
      

**DAC**

.. code-block:: python

   from qcodes.instrument_drivers.IST_devices.FastDuck import FastDuck
   DAC_CS=FastDuck('IVVI', 'COM4', dac_step=10.0, dac_delay=0.001)
   station = qc.Station(DAC_CS)

.. code-block:: python

   #set limits to the values of dacs (safety measure)

   DAC.parameters['dac1'].vals._min_value = -4000.0
   DAC.parameters['dac1'].vals._max_value = 4000.0
   DAC.parameters['dac2'].vals._min_value = -4000.0 #Ftransi-10.0
   DAC.parameters['dac2'].vals._max_value = 4000.0 # 10.0
   DAC.parameters['dac3'].vals._min_value = -4000.0 #-10.0
   DAC.parameters['dac3'].vals._max_value = 4000.0 #10.0
   DAC.parameters['dac4'].vals._min_value = 0.0 #0.0
   DAC.parameters['dac4'].vals._max_value = 1050.0 #0.0
   DAC.parameters['dac5'].vals._min_value = -3000.0
   DAC.parameters['dac5'].vals._max_value = 4000.0
   DAC.parameters['dac6'].vals._min_value = -2000
   DAC.parameters['dac6'].vals._max_value = 4000.0
   DAC.parameters['dac7'].vals._min_value = -4000.0
   DAC.parameters['dac7'].vals._max_value = 4000.0
   DAC.parameters['dac8'].vals._min_value = -4000.0
   DAC.parameters['dac8'].vals._max_value = 4000.0
   DAC.parameters['dac9'].vals._min_value = -2000.0
   DAC.parameters['dac9'].vals._max_value = 4000.0
   DAC.parameters['dac10'].vals._min_value = -4000.0
   DAC.parameters['dac10'].vals._max_value = 4000.0
   DAC.parameters['dac11'].vals._min_value = -2000.0
   DAC.parameters['dac11'].vals._max_value = 3600.0
   DAC.parameters['dac12'].vals._min_value = -2000.0
   DAC.parameters['dac12'].vals._max_value = 3200.0
   DAC.parameters['dac13'].vals._min_value = -4000.0
   DAC.parameters['dac13'].vals._max_value = 4000.0
   DAC.parameters['dac14'].vals._min_value = -4000.0
   DAC.parameters['dac14'].vals._max_value = 4000.0
   DAC.parameters['dac15'].vals._min_value = -4000.0
   DAC.parameters['dac15'].vals._max_value = 4000.0
   DAC.parameters['dac16'].vals._min_value = -4000.0
   DAC.parameters['dac16'].vals._max_value = 4000.0
       
.. code-block:: python

   #give meaninful names to the gates by using the scaledParameter with a gain of 1
   V_dot = ScaledParameter(DAC.dac1, name='QD_Bias', gain=1e-2, unit='mV')
   cryo_amp = ScaledParameter(DAC.dac2, name='cry_amp', gain=1, unit='mV')
   
   Backbone = ScaledParameter(DAC.dac5, name='Backbone', gain=1, unit='mV')
   
   BL = ScaledParameter(DAC.dac6, name='BL', gain=1, unit='mV')
   P1 = ScaledParameter(DAC.dac7, name='P1', gain=1, unit='mV')
   B12 = ScaledParameter(DAC.dac8, name='B12', gain=1, unit='mV')
   P2 = ScaledParameter(DAC.dac9, name='P2', gain=1, unit='mV')
   B23 = ScaledParameter(DAC.dac10, name='B23', gain=1, unit='mV')
   P3 = ScaledParameter(DAC.dac11, name='P3', gain=1, unit='mV')
   B34 = ScaledParameter(DAC.dac12, name='B34', gain=1, unit='mV')
   P4 = ScaledParameter(DAC.dac13, name='P4', gain=1, unit='mV')
   B45 = ScaledParameter(DAC.dac14, name='B45', gain=1, unit='mV')
   P5 = ScaledParameter(DAC.dac15, name='P5', gain=1, unit='mV')
   BR = ScaledParameter(DAC.dac16, name='BR', gain=1, unit='mV')
   
   
   V_CS = ScaledParameter(DAC_CS.dac1, name='CS_Bias', gain=1e-2, unit='mV')
   
   CS1_BL = ScaledParameter(DAC_CS.dac6, name='CS1_BL', gain=1, unit='mV')
   CS1_P = ScaledParameter(DAC_CS.dac7, name='CS1_P', gain=1, unit='mV')
   CS1_BR = ScaledParameter(DAC_CS.dac8, name='CS1_BR', gain=1, unit='mV')
   CS2_BL = ScaledParameter(DAC_CS.dac9, name='CS2_BL', gain=1, unit='mV')
   CS2_P = ScaledParameter(DAC_CS.dac10, name='CS2_P', gain=1, unit='mV')
   CS2_BR = ScaledParameter(DAC_CS.dac11, name='CS2_BR', gain=1, unit='mV')
         

**DMM**

.. code-block:: python

   from qcodes.instrument_drivers.Keysight.Keysight_34465A_submodules import Keysight_34465A

   dmm_dot = Keysight_34465A('dmm_dot', 'USB0::0x2A8D::0x0101::MY54505188::INSTR')  #give a name and the adress of the DMM
   station.add_component(dmm_dot)

   dmm_dot_curr = ScaledParameter(dmm_dot.volt, name='QD_Current', gain=1e-9, unit='A')  #the gain that is on the card to convert V in A





**UHFLI**


.. code-block:: python
    
   import zhinst
   import zhinst.toolkit
   zhinst.toolkit.__version__
   
   from qcodes.instrument_drivers.zurich_instruments.ZIUHFLI import ZIUHFLI
   digitizer = ZIUHFLI('digitizer', 'dev2148')
   
   station.add_component(digitizer)

**OPX**

.. code-block:: python

   from qm.qua import *
   from qualang_tools.external_frameworks.qcodes.opx_driver import OPX
   from configuration import *  #configuration file of the opx, here it needs to be in the same folder as the jupyter notebook 

   opx_instrument = OPX(config, name="OPX_demo", host='10.21.42.178')  #ip address of the opx
   station.add_component(opx_instrument)
   opx_instrument.readout_pulse_length(config['pulses']['measure']['length']) 
           

**Where to save your data**

.. code-block:: python

   sample_name = 'W11168_S23_top2'
   experiment_name = '5_dots'
   
   path = os.getcwd()
   path=os.path.join(path, sample_name)
   print(path)
   try:
       os.mkdir(path)
   except OSError:
       print ("Creation of the directory %s failed. Folder already exists?" % path)
   else:
       print ("Successfully created the directory %s." % path)
   db_file_path = os.path.join(path, sample_name + '.db')
   
   qc.config.user.mainfolder=path
   
   
**Live plot and Database**
 
.. code-block:: python

   import IPython.lib.backgroundjobs as bg
   from plottr.apps import inspectr
   
   jobs = bg.BackgroundJobManager()
   jobs.new(inspectr.main, db_file_path)
      

   

  
      
