Useful stuff
======

.. _installation:


Useful functions
------------

**Scaled**
If you use a division/amplifer card

.. code-block:: python
   
   Vdot_scaled = ScaledParameter(DAC.dac1, division = 100)  #10mV/V

   station.add_component(Vdot_scaled)
   
   #or
   Varactor = ScaledParameter(DAC.dac8, gain = 5)  #5 V/V
   station.add_component(Varactor)
   
**Sweep several gates**

.. code-block:: python

   #Creating a parameter for sweeping multiple DAC channels together
   #Creating a parameter for sweeping multiple DAC channels together
   class Sweep_multiple_gates(qcodes.Parameter):
       def __init__(self, name, gates_to_sweep=None):
           # only name is required
           super().__init__(name, label='multiple_gates',
                            #vals=qc.validators.Ints(min_value=0),
                            docstring='sweeping_multiple_gates_together')
           self.gates_to_sweep = gates_to_sweep
           if self.gates_to_sweep == None:
               raise Exception('Gates_to_sweep not provided')
       # you must provide a get method, a set method, or both
       def get_raw(self):
           return None

       def set_raw(self, value):
           for gate in self.gates_to_sweep:
               if gate==gates.CSL_BL:
                   gate.set(value+260.0)
               else:
                   gate.set(value)

          def set_raw(self, value):
              for gate in self.gates_to_sweep:
                  gate.set(value)

     barrier = Sweep_multiple_gates('barrier',gates_to_sweep=[station.gates.CSL_BR,station.gates.CSL_BL])
     barrier.set(500.0)
      
      
**UHF**
To set automatically the frequency 
 
.. code-block:: python

 def set_frequency():
    #set frequency min of the dip
    scanjob = scanjob_t({'sweepdata': dict({'param':station.digitizer.oscillator1_freq ,
                                            'start':34e6, 'end':50e6, 'step':0.1e6,
                                            'wait_time': 1e-3,
                                            'wait_time_startscan': 10e-3}),
                                            'minstrument': [station.digitizer.demod4_R, station.digitizer.demod4_phi],
                                            'dataset_label': 'Demod_frequency'})

    data1d = qtt.measurements.scans.scan1D(station, scanjob, liveplotwindow= None, location=None, verbose=0)
    
    L=data1d.arrays['demod4_R']

    eps=4
    for i in range(eps,len(data1d.arrays['demod4_R'])-eps):
        if L[i-eps]>L[i] and L[i]<L[i+eps]:
            index_R=i
            break




    #minimum_R=min(L_deriv)
    #index_R=L_deriv.index(minimum_R)
    frequency_R=data1d.arrays['oscillator1_freq'][index_R]
    station.digitizer.oscillator1_freq.set(frequency_R)


    frequency_R=data1d.arrays['oscillator1_freq'][index_R]
    station.digitizer.oscillator1_freq.set(frequency_R)
    print(frequency_R)
    return frequency_R
      

Useful commands
---------------------

**For the DAC**

.. code-block:: python

   DAC.set_dacs_zero()  #Put everything to 0
   print(station.gates.get_all(verbose=1))  #print all the values of the gates

   
**For the DMM**

.. code-block:: python

   dmm_dot.NPLC(0.2)   #dmm_dot = Keysight_34465A('dmm_dot', 'USB0::10893::257::MY54502785::0::INSTR')
   dmm_dot.range(10)
   

**For the UHF**

.. code-block:: python

    station.digitizer.demod4_R   #amplitude
    station.digitizer.demod4_phi  #phase
    station.digitizer.oscillator1_freq  #frequency
    station.digitizer.signal_output1_amplitude
   
  
Read the data
----------
First import function the load the data and other that can be useful 

.. code-block:: python
   from qtt.data import plot_dataset

   from qtt.data import load_dataset
   
Then load your data, for example 

.. code-block:: python
   dataset= load_dataset(location=r'K:\Measurement\Yona\11044_S08\20220224_Bottom\20220224_Bottom\2023-03-02\13-49-16_qtt_CD_CSL')
   
If you run

.. code-block:: python
   dataset

You will see the name of the axis and the size of the arrays
Then to access the data you can do 
.. code-block:: python

   y = dataset_diamonds.arrays['CSL_P']
   x = dataset_diamonds.arrays['V_dot_scaled']
   z = dataset_diamonds.arrays['dmm_curr_dot']
   plt.figure()
   plt.pcolor(x, y, z)
   
 
**Browse**
 
To have a GUI for easy browsing of our saved datasets.
 
.. code-block:: python
   %gui qt
   import qtt
   #from qtt.data import load_example_dataset
   import qcodes
   #from qcodes.plots.qcmatplotlib import MatPlot
   #from qcodes.plots.pyqtgraph import QtPlot
   #from qcodes.data.data_set import DataSet

   # Set data directory
   path_save = r'K:\\Measurement\\Josip\\Iso-pur_wafer_10820_piece_9\\'
   datadir = os.path.join(path_save, '')
   DataSet.default_io = qcodes.data.io.DiskIO(datadir) 
   
   logviewer = qtt.gui.dataviewer.DataViewer(datadir, verbose=0)
   





