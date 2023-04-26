UHFLI
=======

.. _installation:


Useful commands
------------

.. code-block:: python
   
      
    station.digitizer.demod4_R   #amplitude
    station.digitizer.demod4_phi  #phase
    station.digitizer.oscillator1_freq  #frequency
    station.digitizer.signal_output1_amplitude
   
      
      
Optimisation
----------------

.. code-block:: python

   #go somewhere with Coulomb peak
   scanjob = scanjob_t({'sweepdata': dict({'param': station.gates.CSL_P, 
                                           'start': 700.0, 'end': 1200.0, 'step': 2}), 
                        'minstrument': [station.digitizer.demod4_R], 'wait_time': 1e-3})

   scanjob['stepdata'] = dict({'param': station.digitizer.signal_output1_amplitude, 'start': 1e-3, 'end': 30e-3, 'step': 1e-3, 'wait_time': 1e-3})
   scanjob['dataset_label'] = 'Optimization_ampli'

   data = qtt.measurements.scans.scan2D(station, scanjob, liveplotwindow=None, diff_dir='x', location = None, update_period=1)
   plot_nanoqtt(data, scanjob)
   
   #if error for plotting
   plt.pcolor(data.arrays['CSL_P'],data.arrays['signal_output1_amplitude'],data.arrays['demod4_R'])
   
   #look for the maximum measured amplitude 
   AM=[]
   for i in range(np.shape(data.arrays['demod4_R'])[0]):
       minimum=min(data.arrays['demod4_R'][i])
       maximum=max(data.arrays['demod4_R'][i])
       am=maximum-minimum
       AM.append(am)
   plt.plot(data.arrays['signal_output1_amplitude'],AM)
   
   
   
Impedance
-------------

.. code-block:: python

   R_series = 935.7e3 + 20e3 + 10e3  # Card + line 
   V_electronics = (0.5 + 0.0)*0.001 # including offset and in volts
   R = (V_electronics/(data1d.arrays['dmm_curr_sensor'][:]+1e-12))-R_series

   plt.figure()
   fig, ax1 = plt.subplots(figsize=(12,6))

   ax1.plot(data1d.arrays['SP'], data1d.arrays['demod4_R'][:]*10**6, 'b.-', label=r'Amplitude ($\mu V$)')
   ax2 = ax1.twinx()
   ax2.semilogy(data1d.arrays['SP'], R, 'r.-', label=r'Resistance ($\Omega$)')
   #ax1.get_shared_x_axes().join(ax1, ax2)

   ax1.set_xlabel('SP (mV)')
   ax1.set_ylabel('Amplitude ($\mu V$)', color='b')
   ax2.set_ylabel(r'$R_{dot} (\Omega)$', color='r')

   plt.title(data1d.location)
   plt.grid()
   ax1.legend()
   ax2.legend()

   print('The min. of resistance is {} Kohm'.format(np.min(R/1e3)))
   
.. code-block:: python

   res_freq = 175.2e6 #Hz
   L = 2.2e-6 #H
   C = 1/(4*(np.pi**2)*L*res_freq**2)
   print('Parasitic capacitance is {} pF'.format(C*10**12))

   Z = L/(C*R)

   plt.figure()
   fig, ax1 = plt.subplots(figsize=(12,6))

   ax1.plot(data1d.arrays['SP'], data1d.arrays['demod4_R'][:]*10**6, 'b.-', label=r'Amplitude ($\mu V$)')
   ax2 = ax1.twinx()
   ax2.plot(data1d.arrays['SP'], Z, 'r.-', label=r'Impedance ($\Omega$)')
   #ax1.get_shared_x_axes().join(ax1, ax2)

   ax1.set_xlabel('SP (mV)')
   ax1.set_ylabel('Amplitude ($\mu V$)', color='b')
   ax2.set_ylabel(r'$\frac{L}{CR} (\Omega)$', color='r')

   plt.title(data1d.location)
   plt.grid()

   ax1.legend()
   ax2.legend()
 
