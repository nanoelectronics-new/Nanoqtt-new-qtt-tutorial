OPX
=====

.. _installation:


Documentation
------------

Qua documentation: https://docs.quantum-machines.co/1.1.5/

Configuration: https://docs.quantum-machines.co/1.1.5/assets/qua_config.html

QM github: https://github.com/qua-platform

Qcodes driver: https://github.com/qua-platform/py-qua-tools/tree/main/qualang_tools/external_frameworks/qcodes


Connexion
------------

.. code-block:: python

   from qm.qua import *
   from qualang_tools.external_frameworks.qcodes.opx_driver import OPX
   from configuration import *
   opx_instrument = OPX(config, name="OPX_demo", host='10.21.42.178')
   #station.add_component(opx_instrument) # if you want to use it with qcodes
   #opx_instrument.readout_pulse_length(config['pulses']['measure']['length']) 

Programm
------------
**Example execution of a program**

.. code-block:: python  

   qm=opx_instrument.qm # Open a quantum machine with a given configuration ready to execute a program
   job = qm.execute(prog)

   #if your programm has a infinite loop: stop the program after x times
   time.sleep(30)  # The program will run for 30 seconds
   job.halt()

**Example of a program**

.. code-block:: python 

      gate1='P1'  #P1 and P2 are elements of the OPX defined in the config file 
      gate2='P2'
      
      with program() as prog:
          with infinite_loop_():
              for i in range(0,nb_period):
                  play('jump'*amp(1),gate1,duration=(period_pulse/2)//4)
                  play('jump'*amp(1),gate2,duration=(period_pulse/2)//4)
                  play('jump'*amp(-1),gate1,duration=(period_pulse/2)//4)
                  play('jump'*amp(-1),gate2,duration=(period_pulse/2)//4)

      opx_instrument.qua_program = prog

**Example of a simulation**

.. code-block:: python 

   opx_instrument.qua_program = prog

   # Simulate program
   opx_instrument.sim_time(20_000)
   opx_instrument.simulate()
   opx_instrument.plot_simulated_wf()


.. image:: image/ex_opx_simulation.PNG
   :width: 100px
   :height: 50px
   :scale: 40 %
   :alt: alternate text
   :align: right

      
Qcodes measurement
----------------
If you want to do sweep other parameter than opx via qcode, you need first to create a function that return the qua program and then do the qcodes measurement. The qcodes measurement change the sweeping parameter, run the qua program inner the infinite loop, break it at the pause then change again the external parameter,...


**OD qua**
If you don't want to sweep an OPX parameter
Example for reflectometry measurement
First we define the program

.. code-block:: python

   def OPX_0d_scan(f,simulate=False):
       with program() as prog:
           update_frequency('RF', f)
           I = declare(fixed)
           Q = declare(fixed)
           Q_st = declare_stream()
           I_st = declare_stream()
           with infinite_loop_():
               if not simulate:
                   pause()  # to synchronize the opx measurement with the external parameter, skip the pause in the resume function in the dond
               measure(
                   "measure"*amp(2),
                   "RF",
                   None,  # don't save raw data
                   demod.full("cos", I, "out1"),
                   demod.full("sin", Q, "out1"),
               )
               save(I, I_st)
               save(Q, Q_st)
   
           with stream_processing():
               I_st.save_all("I")
               Q_st.save_all("Q")
    return prog

Then we do the measurement, it can be a 1d or 2d measurement

.. code-block:: python

   opx_instrument.qua_program = OPX_0d_scan(f,simulate=False)
   do1d(CS1_BL,1200,2200,10,0.1,
       opx_instrument.resume,
       opx_instrument.get_measurement_parameter(),
       dmm_CS1_curr,
       enter_actions=[opx_instrument.run_exp],
       exit_actions=[opx_instrument.halt],
       show_progress=True,
       do_plot=True,
       exp=exp,
       measurement_name='CS1_BL_opx',
   )

That will give you I,Q, R and Phase

.. image:: image/exp_opx_0d
   :width: 100px
   :height: 50px
   :scale: 40 %
   :alt: alternate text
   :align: right

**1d**

If you want to sweep an OPX parameter.
Example for a frequency sweep

.. code-block:: python
   from qualang_tools.loops import from_array
   # QUA sequence
   def OPX_frequency_sweep(f_array,n_avg=50,simulate=False): 
       with program() as prog:
           #adc_st=declare_stream(adc_trace=True)
           n = declare(int)
           f = declare(int)
           I = declare(fixed)
           Q = declare(fixed)
           I_st = declare_stream()
           Q_st = declare_stream()
           with infinite_loop_():
               if not simulate:
                   pause()
               with for_(n, 0, n < n_avg, n + 1):
                   with for_(*from_array(f,f_array)):
                       reset_phase('RF')
                       update_frequency('RF', f)
                       #measure('measure'*amp(self.amp()), 'RF', adc_st, demod.full('cos', I, 'out1'), demod.full('sin', Q, 'out1'))
                       measure('measure'*amp(0.2), 'RF', None, demod.full('cos', I, 'out1'), demod.full('sin', Q, 'out1'))
   
                       save(I, I_st)
                       save(Q, Q_st)
                       wait(100)
   
           with stream_processing():
               I_st.buffer(len(f_array)).buffer(n_avg).map(FUNCTIONS.average()).save_all(
                   "I"
               )
               Q_st.buffer(len(f_array)).buffer(n_avg).map(FUNCTIONS.average()).save_all(
                   "Q"
               )
   
       return prog

.. code-block:: python

   f_array=np.arange(20e6,200e6,1e6)
   opx_instrument.set_sweep_parameters("axis1", f_array, "Hz", "f")  #the axis the you want the sweep 
   opx_instrument.qua_program = OPX_frequency_sweep(f_array,n_avg=50,simulate=False)

   exp = load_or_create_experiment(experiment_name = experiment_name, sample_name = sample_name)
   do0d(
       opx_instrument.run_exp,
       opx_instrument.resume,
       opx_instrument.get_measurement_parameter(),
       opx_instrument.halt,
       do_plot=True,
       exp=exp,
   )
That will give you I,Q, R and Phase


.. image:: image/exp_opx_frequency_sweep
   :width: 100px
   :height: 50px
   :scale: 40 %
   :alt: alternate text
   :align: right



Calibration
----------------

**Time of flight**

You need to calibrate the time of flight i.e. the time that the signal need to reach back the opx. 
For that you will need to send a wave and measure it, look at the raw data the see from wich time you start seing the oscillation. 






   

  
      
