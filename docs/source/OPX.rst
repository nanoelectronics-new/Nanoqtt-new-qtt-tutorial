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

      
Plot
----------------

**Derivative**

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
      

**Patch**

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



   

  
      
