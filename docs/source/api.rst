Begin measurement
=====

.. _installation:


Pinchoff
------------

For pinchoff:

.. code-block:: python

      import nanoqtt.analysis.utils
      from nanoqtt.analysis.utils import analysePinchOff
      
      #start with the backbone
      # Here I will save the pinchoffs
      pinch_dict = dict()

      # Bias 500 uV
      Vsensor_scaled(0.5)
      scanjob = scanjob_t({'sweepdata': dict({'param': station.gates.Backbone,
                                              'start': 0.0, 'end':3000.0, 'step':5.0,
                                              'wait_time': 1e-3,
                                              'wait_time_startscan': 1}),
                                              'minstrument': [station.dmm_curr_dot],
                                              'dataset_label': 'Backbone_pinch_off'})

      data1d = qtt.measurements.scans.scan1D(station, scanjob, liveplotwindow= None, location=None, verbose=0)
      plot_nanoqtt(data1d, scanjob)

      # Analysis part
      result = analysePinchOff(dd=data1d, fig=1, minthr = 1e-10, verbose=3, checks=False)


      # Save in dict values
      keys = ['goodgate', 'midpoint', 'I_pinchoff_value', 'V_pinchoff_value']
      gate_dict = {x:result[x] for x in keys}
      pinch_dict['Backbone'] = gate_dict
      
For automatic pinchoffs

.. code-block:: python

      from nanoqtt.measurements.tuning import pinch_offs
      from nanoqtt.analysis.utils import analysePinchOff
      #pinch_dict = dict()

      Vdot_scaled(0.5) # Bias 500 uV
      gates_to_sweep = ['BL', 'PL', 'B12', 'PM', 'B23', 'PR', 'BR']
      init = 0.0
      final = 1800.0
      step = 10.0
      minstrument = station.dmm_curr_dot
      pinch_offs(station, gates_to_sweep, init, final, step, minstrument, pinch_dict)


Visualisation of pinchoffs

.. code-block:: python

  def get_pinchoffs(dataset, desired_order_list, pinch_offs_dict):
       '''
       Given the dataset the function returns a bar plot of the value of the gates
       '''


       #Plot
       colors= ['blue']*(len(reordered_dict)-3)+['orange']*3

       names = list(reordered_dict.keys())
       values = list(reordered_dict.values())

       plt.figure(figsize=(12,6))
       plt.bar(range(len(reordered_dict)), values, tick_label = names, color=colors)
       plt.bar(range(len(reordered_dict)), pinch_offs, tick_label = names, width=0.5, color='red')


       plt.ylabel('Voltage (mV)', size='large')
       plt.ylim([0, 4000])

       plt.xticks(rotation=45, size='large')
       plt.yticks(size='large')
       plt.grid()
       plt.title(dataset.location)
       plt.tight_layout()
       plt.show()


   desired_order_list = ('Backbone', 'BL', 'PL', 'B12', 'PM', 'B23', 'PR', 'BR', 'SL', 'SP', 'SR')

   pinch_dict['Backbone']['Operational'] = gates.Backbone.get()
   pinch_dict

   # Get keys
   gates_keys = list(pinch_dict.keys())
   analysis_keys = pinch_dict[gates_keys[0]].keys()


   midpoint_list = []

   for ii in gates_keys:
       good_list = pinch_dict[ii]['goodgate']
       midpoint_list.append(pinch_dict[ii]['midpoint'])

   import json
   from datetime import datetime
   # Save dictionary of pinch offs
   with open(datadir + '/' + datetime.now().strftime("%Y-%m-%d-%H-%M-%S") + '_pinchoffs_all.json', 'w') as fp:
           json.dump(pinch_dict, fp)


   plt.figure(figsize=(12,6))
   plt.bar(range(len(pinch_dict.keys())), midpoint_list, tick_label = list(pinch_dict.keys()))#, color=colors)
   plt.bar(0, gates.Backbone.get(), width=0.4)#, color=colors)

   plt.ylabel('Voltage (mV)', size='large')
   plt.ylim([0, 2700])

   plt.xticks(rotation=45, size='large')
   plt.yticks(size='large')
   plt.grid()
   plt.title('Pinch_offs')
   plt.tight_layout()
   plt.show()
   plt.savefig(datetime.now().strftime("%Y-%m-%d-%H-%M-%S") + 'Pinch_offs_all.png')
      
      

Noise measurement
--------------------
.. code-block:: python

   from nanoqtt.measurements.NoiseMeas import noise_measurement_current,background_noise_current
   
   #1D plot of one Coulomb peak   including zone with 0 current at the end 
   scanjob = scanjob_t({'sweepdata': dict({'param':station.gates.CSL_P ,
                                        'start':300.0, 'end':360, 'step':2.0,
                                        'wait_time': 1e-3,
                                        'wait_time_startscan': 10e-3}),
                                        'minstrument': [dmm_curr_dot],
                                        'dataset_label': 'CO_CSL_P'})

   data1d = qtt.measurements.scans.scan1D(station, scanjob, liveplotwindow= None, location=None, verbose=0)
   plot_nanoqtt(data1d, scanjob)
   
.. code-block:: python


   #Background measurement
   for key in list(data1d.arrays.keys())[:len(scanjob['minstrument'])]:
    _ = MatPlot(data1d.arrays[key])
    
   scanjob['meas_time'] = 300 #s
   scanjob['dmm'] = 'station.dmm_dot'

   data1d_time_back = background_noise_current(dataset=data1d, scanjob=scanjob, station=station)
   
.. code-block:: python

   
   #peak measurement 
   ##check again the peak 
   
   dmm_dot.NPLC(1) #change NPLC because noise function change it 

   scanjob = scanjob_t({'sweepdata': dict({'param': station.gates.CSL_P,
                                           'start': 340.0, 'end':400.0, 'step':0.2,
                                           'wait_time': 1e-3,
                                           'wait_time_startscan': 1}),
                                           'minstrument': [station.dmm_curr_dot],
                                           'dataset_label': 'CO_CS'})

   data1d = qtt.measurements.scans.scan1D(station, scanjob, liveplotwindow=plotQ, location=None, verbose=0)

   plot_nanoqtt(data1d, scanjob)
   
   
   #detect the peak(s)

   from scipy.signal import find_peaks

   key_gate = list(data1d.arrays.keys())[1]
   key_I = list(data1d.arrays.keys())[0]
   gate = data1d.arrays[key_gate][:]
   I = data1d.arrays[key_I][:]

   peaks, _ = find_peaks(I, height=1.4e-11, distance=60)

   plt.figure()
   plt.plot(gate, I)
   plt.plot(gate[peaks[:]], I[peaks[:]], '*')
   plt.show()

   end = np.min(gate)
   end = np.append(end, ((gate[peaks[:]][:-1] + gate[peaks[:]][1:])/2))
   end = np.append(end, np.max(gate))

   print(end)

.. code-block:: python

   #noise
   for idx, peak in enumerate(peaks):
    # Sweep just that peak
    scanjob['sweepdata']['start'] = end[idx]
    scanjob['sweepdata']['end'] = end[idx+1]
    scanjob['wait_time_startscan']=1.0
    
    scanjob['step']=0.07
    station.dmm_dot.NPLC.set(1)

    data1d = qtt.measurements.scans.scan1D(station, scanjob, liveplotwindow=plotQ, location=None, verbose=0)
    plot_nanoqtt(data1d, scanjob)

    # Do noise measurement
    scanjob['meas_time'] = 300#s
    #scanjob['meas_time'] = 5#s
    scanjob['dmm'] = 'station.dmm_dot'  #name of the DMM when dmm = Keysight_34465A('dmm', 'USB0::0x2A8D::0x0101::MY54506631::INSTR')

    
    gates.CSL_P(scanjob['sweepdata']['start'])
    data1d_time, scanjob, dy_s = noise_measurement_current(dataset=data1d, scanjob=scanjob, station=station)
    
    
Visualisation

.. code-block:: python

   from qcodes.plots.qcmatplotlib import MatPlot

   import nanoqtt
   from nanoqtt.plotting import plot_dataset
   #from nanoqtt.analysis.noise import charge_noise

   from qtt.data import plot_dataset

   from qtt.data import load_dataset
   
   
   import scipy
   %matplotlib inline

   def charge_noise(dataset, dataset_BG, segments, lever_arm):
       '''
       Arguments
       ---------
       dataset:
           Dataset of noise measurement
       slope(float):
           Slope of Coulomb peak
       segments (int):
           Number of segments for average with Welch method
       lever_arm (float):
           Lever arm in eV/V
       gain (float):
           Gain in V/A

       Returns
       -------
       '''

       #print(dataset.metadata['gain'])
       #print(dataset.metadata['SR'])

       #dataset.metadata['gain'] = 100000000.0
       #dataset.metadata['SR']=3344.4816053511704

       freq, SI = power_density_spectrum_current(dataset, segments)
       freq_BG, SI_BG = power_density_spectrum_current(dataset_BG, segments, gain=dataset.metadata['gain'], sampling_rate=dataset.metadata['SR'])

       # slope converted to A/V
       SE = (SI-SI_BG) * lever_arm**2 /(dataset.metadata['slope']*1000)**2

       #Size of font
       plt.rcParams['font.size'] = 14

       plt.figure()
       plt.loglog(freq, SE, '.-')

       plt.ylim([10**(-17), 10**-9])

       plt.xlabel('Frequency [Hz]')
       plt.ylabel(r'$S_E [eV^2$/Hz]')
       plt.title('Put title', size=10)
       plt.show()

       # Noise at 1Hz
       ii = np.argmin(np.abs(freq-1)) # Index of 1 Hz
       print(r"\sqrt{S_E}= " + str(np.sqrt(SE[ii])) + ' eV^2 / Hz ')

       return freq, SE

   def power_density_spectrum_current(dataset, segments, gain=None, sampling_rate=None):

       '''
       Function to extract hte power spectral density of the current

       Arguments
       ---------
       dataset:
           Dataset of noise measurement
       segments (int):
           Number of segments for average with Welch method
       gain (float):
           Gain in V/A
       sampling_rate (float):
           Sampling rate of measurement
       Returns
       -------
       '''

       if gain==None:    
           gain = dataset.metadata['gain']
       if sampling_rate==None:
           sampling_rate = dataset.metadata['SR']

       freq, Sint = scipy.signal.welch(dataset.default_parameter_array()[:]/gain, sampling_rate, window='hanning',             nperseg=len(dataset.default_parameter_array())/segments)

       #Size of font
       plt.rcParams['font.size'] = 14

       plt.figure()
       plt.loglog(freq, Sint*(10**12)**2, '.-')

       plt.xlabel('Frequency [Hz]')
       plt.ylabel(r'$S_I$ [$pA^2$/Hz]')
       plt.title(dataset.location, size=10)
       plt.show()

       return freq, Sint
       
.. code-block:: python


      #get lever arm from a Coulomb diamond plt
      dataset_diamonds = load_dataset(location=r'K:\Measurement\Yona\11044_S08\20220224_Bottom\20220224_Bottom\2023-03-02\13-49-16_qtt_CD_CSL')
      y = dataset_diamonds.arrays['CSL_P']
      x = dataset_diamonds.arrays['V_dot_scaled']
      z = dataset_diamonds.arrays['dmm_curr_dot']
      plt.figure()
      plt.pcolor(x, y, z)
      
.. code-block:: python

      # Distance/2 between extremes of the diamonds  #bias
      Ec1 = (3.07+1.58)/2

      # Distance between extremes of the diamonds plunger
      Ec2 = (267.3-209.3)

      #Lever arms
      alpha1 = Ec1 / Ec2 # eV/V
      print(alpha1)
      
.. code-block:: python
      
      #noise 
      dataset = load_dataset(location=r'K:\Measurement\Yona\11044_S08\20220224_Bottom\2023-03-02\15-29-43_qtt_Noise_Measurement')
      dataset_BG = load_dataset(location=r'K:\Measurement\Yona\11044_S08\20220224_Bottom\2023-03-02\15-02-47_qtt_Noise_Measurement')
      f, SE = charge_noise(dataset, dataset_BG, 20, lever_arm=alpha1)
      
            
.. code-block:: python

      #can save it on a file with the value for several samples
      np.savetxt(r'K:\Measurement\Yona\Data_noise\11044_S08_Bottom_a_0_035.txt', np.c_[f, SE], fmt='%.7e')
      
      %matplotlib inline

      import glob
      df = pd.DataFrame(columns=('Wafer', 'Sample', 'Device', '$S_E$ (1Hz) $\mu eV / \sqrt{Hz}$'))

      plt.figure(figsize=(10,8))
      for idx, f in enumerate(glob.glob('K:\Measurement\Yona\Data_noise\*.txt', recursive=True)):
          print(idx, f)
          x, y = np.loadtxt(f, unpack=True)

          ii = np.argmin(np.abs(x-1)) # Index of 1 Hz
          SE_1Hz = np.sqrt(y[ii])

          df.loc[idx] = [f[5:10], f[11:14], f[15:-4], SE_1Hz*10**6]

          plt.loglog(x, y, label=f)

      plt.xlabel('Frequency [Hz]')
      plt.ylabel(r'$S_E  (eV^2/Hz)$')

      freq_delft = np.logspace(-1, 2, 100)
      Se_delft_22nm = 1.96e-12/freq_delft
      Se_delft_55nm = 3.8e-13/freq_delft**0.92

      plt.loglog(freq_delft, Se_delft_22nm, '--',label = 'Delft 22nm' )
      plt.loglog(freq_delft, Se_delft_55nm, '--', label = 'Delft 55nm' )
      plt.legend(bbox_to_anchor=(1.05, 1.0), loc='upper left')


      plt.xlim([10**-1,10**2])
      plt.ylim([10**-18,10**-10])



