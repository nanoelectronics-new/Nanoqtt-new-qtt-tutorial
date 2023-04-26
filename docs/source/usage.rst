Start a jupyter notebook
=====

.. _installation:


Importation
------------

At the begining of the jupyter notebook import:

.. code-block:: python
   
      import sys, os, tempfile, time, datetime
      import numpy as np
      %matplotlib inline
      %gui qt
      import matplotlib.pyplot as plt
      import qcodes
      from qcodes.plots.qcmatplotlib import MatPlot
      from qcodes.plots.pyqtgraph import QtPlot
      from qcodes.data.data_set import DataSet
      import qtt
      from qtt.measurements.scans import scanjob_t  
      import scipy.optimize
      import cv2

      from qcodes.utils.validators import Numbers
      from qcodes.instrument.base import Instrument
      from qcodes.instrument.parameter import ManualParameter,Parameter

      import pyvisa as visa #for the instrument
      
      from qcodes.utils.validators import Numbers
      from qcodes.instrument.base import Instrument
      from qcodes.instrument.parameter import ManualParameter
      from qcodes import ScaledParameter

      from qtt.algorithms.ohmic import fitOhmic
      
      
Instruments
----------------

**Adress**

.. code-block:: python

      rm = pyvisa.ResourceManager()
      rm.list_resources()
      

**DAC**

.. code-block:: python

      DAC = FastDuck('DAC','COM4', dac_step=30.0, dac_delay=0.0)
      station = qcodes.Station(DAC)
           
      #set limits to the values of dacs (safety measure)

       #Setting min and max values for each dac
       dac.parameters['dac1'].vals._min_value = 0.0
       dac.parameters['dac1'].vals._max_value = 1010.0
       dac.parameters['dac2'].vals._min_value = -4000.0 #Ftransi-10.0
       dac.parameters['dac2'].vals._max_value = 4000.0 # 10.0
       dac.parameters['dac3'].vals._min_value = -4000.0 #-10.0
       dac.parameters['dac3'].vals._max_value = 4000.0 #10.0
       dac.parameters['dac4'].vals._min_value = 0.0 #0.0
       dac.parameters['dac4'].vals._max_value = 4000.0 #4000.0
       dac.parameters['dac5'].vals._min_value = 0.0#0.0
       dac.parameters['dac5'].vals._max_value = 4000.0 #4000.0
       dac.parameters['dac6'].vals._min_value = 0.0 #0.0
       dac.parameters['dac6'].vals._max_value = 4000.0 #4000.0
       dac.parameters['dac7'].vals._min_value = 0.0 #0.0
       dac.parameters['dac7'].vals._max_value = 4000.0 #4000.0
       dac.parameters['dac8'].vals._min_value = 0.0 #0.0
       dac.parameters['dac8'].vals._max_value = 4000.0 #4000.0
       dac.parameters['dac9'].vals._min_value = 0.0 #0.0
       dac.parameters['dac9'].vals._max_value = 4000.0 #4000.0
       dac.parameters['dac10'].vals._min_value = 0.0 #0.0
       dac.parameters['dac10'].vals._max_value = 4000.0
       dac.parameters['dac11'].vals._min_value = 0.0
       dac.parameters['dac11'].vals._max_value = 4000.0
       dac.parameters['dac12'].vals._min_value = 0.0
       dac.parameters['dac12'].vals._max_value = 4000.0
       dac.parameters['dac13'].vals._min_value = 0.0
       dac.parameters['dac13'].vals._max_value = 4000.0
       dac.parameters['dac14'].vals._min_value = 0.0
       dac.parameters['dac14'].vals._max_value = 4000.0
       dac.parameters['dac15'].vals._min_value = 0.0
       dac.parameters['dac15'].vals._max_value = 4000.0
       dac.parameters['dac16'].vals._min_value = 0.0
       dac.parameters['dac16'].vals._max_value = 4000.0
       
       
       #give meaninful names to the gates
       gates = VirtualDAC('gates', 
                   instruments = [DAC], 
                   gate_map ={'Vdot': (0, 2), 
                              'Vsensor': (0, 3), 
                              'Backbone': (0, 4),  
                              'SL': (0, 6), 
                              'SP': (0, 7), 
                              'SR': (0,8), 
                              'BL':(0,9),
                              'PL': (0, 10), 
                              'B12': (0, 11), 
                              'PM': (0, 12), 
                              'B23': (0,13), 
                              'PR':(0,14),
                              'BR': (0, 15)
                              },
                   
                   rc_times=None)

         station.add_component(gates)
         

**DMM**

.. code-block:: python

      from qcodes.instrument_drivers.Keysight.Keysight_34465A_submodules import Keysight_34465A  #import the driver of your dmm
      
      dmm_dot = Keysight_34465A('dmm_dot', 'USB0::10893::257::MY54502785::0::INSTR')  #give a name and the adress of the DMM
      station.add_component(dmm_dot)

      class DMM_current(qcodes.Parameter):
          def __init__(self, name, dmm_instance, gain):
              # only name is required
              super().__init__(name, label='1G',
                               #vals=qc.validators.Ints(min_value=0),
                               docstring='measures the current out of the DMM',
                               unit= 'A')
              self.dmm_instance = dmm_instance
              self._gain = gain
          # you must provide a get method, a set method, or both
          def get_raw(self):
              self._current = self.dmm_instance.volt.get()/self._gain
              return self._current

          def set_raw(self, val):
              # StandardParameter handles validation automatically, Parameter doesn't
              self._vals.validate(val)
              self._count = val
        
      dmm_curr_dot = DMM_current('dmm_curr_dot', dmm_instance=dmm_dot, gain=1e9)  #gain of the card, we will measure dmm_curr_dot
      station.add_component(dmm_curr_dot)
      #give the parameter a name and set the gain. 
      #Careful to not call it DMM.curr because it may get confused with 
      #the already existing DMM 'A')




**UHFLI**


.. code-block:: python
    
      import zhinst
      import zhinst.toolkit
      zhinst.toolkit.__version__

      from zhinst.toolkit import Session, Sequence, CommandTable, Waveforms
      from nanoqtt.Drivers.ZI.ZIUHFLI import ZIUHFLI
          
      digitizer = ZIUHFLI('digitizer', 'dev2148')
      dataserver_host = 'localhost'     #Hostname or IP address of the dataserer
      dev_uhf = "DEV2148"                #Device ID of the UHFLI

      # Create a session
      session = Session(dataserver_host)
      device_UHFLI = session.connect_device(dev_uhf)
      
      #parameter
      demod = device_UHFLI.demods[3]       # which demodulator here 4

      with device_UHFLI.set_transaction():
          device_UHFLI.demods['*'].enable(False)
          demod.order(1)
          demod.rate(60e3)
          demod.trigger('continuous')
          demod.timeconstant(1130e-6) # 
          demod.enable(True)

          device_UHFLI.oscs[0].freq(43.65e6)
          
       station.add_component(digitizer)      
          
**AWG**

.. code-block:: python

      from qtt.instrument_drivers.virtualAwg.sequencer import DataTypes
      from qtt.instrument_drivers.virtualAwg.virtual_awg import VirtualAwg
      from nanoqtt.Drivers.ZI.HDAWG4 import ZIHDAWG4
      
      from qcodes.utils.validators import Numbers

      class HardwareType(qcodes.Instrument):

          def __init__(self, name, awg_map, awg_scalings={}, **kwargs):
              super().__init__(name, **kwargs)

              self.awg_map = awg_map
              for gate in self.awg_map.keys():
                  p = 'awg_to_%s' % gate
                  self.add_parameter(p, parameter_class=qcodes.ManualParameter,
                                     initial_value=awg_scalings.get(gate, 1000),
                                     label='{} (factor)'.format(p), unit='mV/V',
                                     vals=Numbers(0, 2000))

          def get_idn(self):
              ''' Overrule because the default VISA command does not work '''
              IDN = {'vendor': 'QuTech', 'model': 'hardwareV2',
                     'serial': None, 'firmware': None}
              return IDN

      import time
      

      def upload_to_AWG(awg, SOURCE = None, awg_sourcefile=None):
          # Create an instance of the AWG Module
          awgModule = awg.daq.awgModule()
          awgModule.set('device', awg.device)
          awgModule.execute()

          # Get the LabOne user data directory (this is read-only).
          data_dir_wave = awgModule.getString('directory')
          # The AWG Tab in the LabOne UI also uses this directory for AWG seqc files.
          src_dir = os.path.join(data_dir_wave, "awg", "src")
          if not os.path.isdir(src_dir):
              # The data directory is created by the AWG module and should always exist. If this exception is raised,
              # something might be wrong with the file system.
              raise Exception("AWG module wave directory {} does not exist or is not a directory".format(src_dir))


          # Note, the AWG source file must be located in the AWG source directory of the user's LabOne data directory.
          if awg_sourcefile is None:
              # Write an AWG source file to disk that we can compile in this example.
              awg_sourcefile = "ziPython_example_awg_sourcefile.seqc"
              with open(os.path.join(src_dir, awg_sourcefile), "w") as f:
                  f.write(SOURCE)
          else:
              if not os.path.exists(os.path.join(src_dir, awg_sourcefile)):
                  raise Exception("The file {} does not exist, this must be specified via an "
                              "absolute or relative path.".format(awg_sourcefile))
                  print("Will compile and load", awg_sourcefile, "from", src_dir)

          # Transfer the AWG sequence program. Compilation starts automatically.
          awgModule.set('compiler/sourcefile', awg_sourcefile)
          # Note: when using an AWG program from a source file (and only then), the compiler needs to
          # be started explicitly:
          awgModule.set('compiler/start', 1)
          timeout = 20
          t0 = time.time()
          while awgModule.getInt('compiler/status') == -1:
              time.sleep(0.1)
              if time.time() - t0 > timeout:
                  Exception("Timeout")
          if awgModule.getInt('compiler/status') == 1:
              # compilation failed, raise an exception
              raise Exception(awgModule.getString('compiler/statusstring'))
          if awgModule.getInt('compiler/status') == 0:
              print("Compilation successful with no warnings, will upload the program to the instrument.")
          if awgModule.getInt('compiler/status') == 2:
              print("Compilation successful with warnings, will upload the program to the instrument.")
              print("Compiler warning: ", awgModule.getString('compiler/statusstring'))

          # Wait for the waveform upload to finish
          time.sleep(0.2)
          i = 0
          while (awgModule.getDouble('progress') < 1.0) and (awgModule.getInt('elf/status') != 1):
              print("{} progress: {:.2f}".format(i, awgModule.getDouble('progress')))
              time.sleep(0.5)
              i += 1
          print("{} progress: {:.2f}".format(i, awgModule.getDouble('progress')))
          if awgModule.getInt('elf/status') == 0:
              print("Upload to the instrument successful.")
          if awgModule.getInt('elf/status') == 1:
              raise Exception("Upload to the instrument failed.")

          print('Success. Enabling the AWG.')
          # This is the preferred method of using the AWG: Run in single mode continuous waveform playback is best achieved by
          # using an infinite loop (e.g., while (true)) in the sequencer program.
          awg.daq.setInt('/' + awg.device + '/awgs/0/single', 1)
          awg.daq.setInt('/' + awg.device + '/awgs/0/enable', 1)
      ###################################################

      # Initialize the arbitrary waveform generator
      awg = ZIHDAWG4(name='ZIHDAWG4', device_id='dev8160')
      
      #add to the session
      device = session.connect_device("DEV8160")
      awg_node = device.awgs[0]
      
      #parameter
      awg_map = {'PL': (0,0), 'PM':(0,1), 'B12':(0,2), 'Ch4':(0,3),'m4i_mk': (0, 0, 0)}
      #awg_scalings = {f'P{ii}': 1000 for ii in range(1,4)}
      awg_scalings = {'PL':2000.0*0.0125, 'B12':2000*0.0125, 'PM':2000*0.0125, 'Ch4':2000*0.0125} # 80 is the attenuation (38dB) of the Rudolph's new probe fast line 4

      hardware = HardwareType(qtt.measurements.scans.instrumentName('hardware'), awg_map, awg_scalings)

      virtual_awg = VirtualAwg([awg], hardware, qtt.measurements.scans.instrumentName('virtual_awg'))
      virtual_awg.digitizer_marker_delay(0)
      virtual_awg.digitizer_marker_uptime(0.2)
      
      station.add_component(virtual_awg)
      station.add_component(hardware)
      
      
      
**Magnet**

.. code-block:: python

      class B_field_param(qcodes.Parameter):
          def __init__(self, name, By = None, Bz = None, phi = None, By_offset = 0.0, Bz_offset = 0.0):
              '''This class makes qcodes paremeter which recieves By and Bz objects and angle phi between then in degrees.
                 Set function of this parameter sets the magnetic field amplitude along the angle phi.
              '''
              super().__init__(name, label='B_field', 
                               #vals=qc.validators.Ints(min_value=0),
                               docstring='Sweep_both_amplitude_and_field_angle')
              self.By = By
              self.Bz = Bz
              self.phi = phi
              self.By_offset = By_offset
              self.Bz.offset = Bz_offset
              if (self.By == None or self.Bz == None or self.phi == None):
                  raise Exception('Du musst By, Bz und phi stellen')

          # you must provide a get method, a set method, or both
          def get_raw(self):
              return None

          def set_raw(self, value):
              '''First element in value is magnetic field and the second one is angle in degrees.
              '''
              r = value
              if abs(value) > 70e-3:
                  raise Exception("Khm, check the magnetic field value you are trying to set.")
              self.By.field.set(self.By_offset+r*np.cos(np.deg2rad(self.phi)))
              self.Bz.field.set(self.Bz.offset+r*np.sin(np.deg2rad(self.phi)))

      from qcodes.instrument_drivers.american_magnetics.AMI430 import AMI430
      By = AMI430('By',address = '10.21.64.65', port=7180)
      Bz = AMI430('Bz',address = '10.21.64.64', port=7180)
      By.ramp_rate.set(0.0004) #T/s

      By.field_limit.set(21e-3)
      Bz.field_limit.set(15e-3)
      
      station.add_component(By)
      station.add_component(Bz)

**Microwave source**

.. code-block:: python

      import qcodes_contrib_drivers

      #import qcodes_contrib_drivers.RohdeSchwarz
      from qcodes_contrib_drivers.drivers.RohdeSchwarz.SMW200A import RohdeSchwarz_SMW200A as smw200a
      SMW = smw200a( name='SMW200A', address='TCPIP::10.21.64.165::hislip0::INSTR' )
      print( "ID:", SMW.get_id() )
      print( "Options:", SMW.get_options() )
      fm = SMW.submodules['fm_channels'][0]
      print('       Deviation:', fm.deviation())
      print('          Source:', fm.source())
      print('Deviation ration:', fm.deviation_ratio())
      print('            Mode:', fm.mode())
      print('           State:', fm.state())
      
      #parameter
      SMW.rfoutput1.frequency.set(250e6)
      SMW.rfoutput1.level() # The output power level in dB
      SMW.rfoutput1.state('OFF')
      SMW.rfoutput1.level.set(-10) # The output power level in dB
      SMW.rfoutput1.level.get()
      
      
      
Almost ready
----------------    

**Where to save your data**

.. code-block:: python

   path_save = r'B:\\group\\katsagrp\\Measurement\\Yona\\W11044_S15_ohmics'
   datadir = os.path.join(path_save, '')
   DataSet.default_io = qcodes.data.io.DiskIO(datadir)
   
   
**Live plot**
 
.. code-block:: python

   mwindows = qtt.gui.live_plotting.setupMeasurementWindows(station, create_parameter_widget=False)
   plotQ = mwindows['plotwindow']
   #may have to run it several time
   
Measurement
----------------

**1D**

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

     #diff_dir='X' ?
   
   
   

  
      
