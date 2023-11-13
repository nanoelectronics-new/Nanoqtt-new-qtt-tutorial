Qcodes installation 
=====

.. _installation:


Installation
------------

First, you need to create an environment, install qcodes, jupyter notebook (if you want to use it)

1. conda create -n qcodes_Yona python=3.9
2. pip install qcodes
3. pip install jupyter notebook

Then for the instrument, put the IST_device folder in C:\ProgramData\Anaconda3\envs\qcodes_yona\Lib\site-packages\qcodes\instrument_drivers
4. pip install pyserial   (for FastDuck)
5. pip install plottr[PyQt5]  (for the database?)
6. pip install ipympl  (for matplotlib)

For Zurich instrument
7. pip install zhinst
8. pip install zhinst_qcodes

For Quantum Machine
9. pip install --upgrade qm-qua
10. pip install --upgrade qualang-tools










   

  
      
