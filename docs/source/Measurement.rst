Basic measurements
=====

.. _installation:


1D measurement
------------


.. code-block:: python

  exp = load_or_create_experiment(experiment_name = experiment_name, sample_name = sample_name)

           #gate, ini, end, nb_point, waiting_time, instrument
  data=do1d(V_dot, -1, 1, 20, .1, dmm_CS2_curr , write_period=.1, do_plot=True, 
          measurement_name='link_test', show_progress=True, use_threads=True,)


If you want to fit for the ohmics: 

.. code-block:: python

   dataset=load_by_run_spec(captured_run_id=2)  #put the id of your measurement 
   # plt.figure()
   x=dataset.get_parameter_data()['CS2_current']['QD_Bias']*1e-3  #if you don't know the name run just in a cell dataset 
   y=dataset.get_parameter_data()['CS2_current']['CS2_current']
   m, b = np. polyfit(x, y, 1) 
   print('Resistance: %.2f kOhm' % (1/m*1e-3))
   print(b)
   plt.plot(x*1e3, y*1e9,'.')
   plt.plot(x*1e3, (b+m*x)*1e9)
   plt.xlabel('Bias [mV]')
   plt.ylabel('Current [nA]')
   plt.title('Resistance: %.2f kOhm' % (1/m*1e-3))
      
      
2D measurement
----------------

.. code-block:: python

  exp = load_or_create_experiment(experiment_name = experiment_name, sample_name = sample_name)        
  do2d(P1, x_i,x_f,120, 0.0, P3, y_i, y_f,120, 0.0, 
      dmm_CS1_curr,
      show_progress=True,
      do_plot=True,
      exp=exp,
      measurement_name='5QD',
  )
         
Noise measurement
----------------

from time import sleep


# Find a Coulomb peak 

.. code-block:: python

   scanjob = scanjob_t({'sweepdata': dict({'param': station.DAC.dac5,
                                           'start':0.0, 'end':3000.0, 'step':10.0,
                                           'wait_time': 1e-3,
                                           'wait_time_startscan': 10e-3}),
                                           'minstrument': [station.dmm_curr_sensor],
                                           'dataset_label': 'Leak_all_gates_to_ohmics'})

   data1d = qtt.measurements.scans.scan1D(station, scanjob, liveplotwindow= None, location=None, verbose=0)
   plot_nanoqtt(data1d, scanjob)
   #station.DAC.dac5.set(0.0)

      
**2D**
 
.. code-block:: python
 
   gates_XY = [station.gates.SL, station.gates.SR]
   step_size = 3.0

   name = '2D_plot_SL_SR'

   #offset = 0.0
   #Vdot_scaled(offset)
   #Vsensor_scaled(0.0)

   station.dmm_sensor.NPLC.set(1)

   scanjob = scanjob_t({'sweepdata': dict({'param': gates_XY[0], 
                                           'start': 600.0, 'end': 880.0, 'step': step_size}), 
                        'minstrument': [digitizer.demod4_R, digitizer.demod4_phi], 'wait_time': 1e-3})

   scanjob['stepdata'] = dict({'param': gates_XY[1], 'start': 600.0, 'end': 880.0, 'step': step_size})
   scanjob['dataset_label'] = name

   data = qtt.measurements.scans.scan2D(station, scanjob, liveplotwindow=None, diff_dir=None, location = None, update_period=1)
   plot_nanoqtt(data, scanjob)

  #diff_dir='x' 
   
   
   

  
      
