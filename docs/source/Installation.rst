Qcodes installation 
=====

.. _installation:


Installation
------------

First, you need to create an environment, install qcodes, jupyter notebook (if you want to use it)

1. conda create -n qcodes_Yona python=3.9
2. conda activate qcodes_Yona
3. pip install qcodes
4. pip install jupyter notebook

Then for the instrument, put the IST_device folder in C:\ProgramData\Anaconda3\envs\qcodes_yona\Lib\site-packages\qcodes\instrument_drivers

5. pip install pyserial   (for FastDuck)
6. pip install plottr[PyQt5]  (for the database?)
7. pip install ipympl  (for matplotlib)

For Zurich instrument

8. pip install zhinst
9. pip install zhinst_qcodes

For Quantum Machine

10. pip install --upgrade qm-qua
11. pip install --upgrade qualang-tools










   

  
      
