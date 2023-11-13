Charge sensor
=====

.. _installation:


Automatic setting 
------------

Gaussian fitting 
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Try to fit and then set the plunger at the maximum slope 

.. code-block:: python

   def set_sensor_CS2(id):  

      # Coulomb oscillations
       do1d(
       CS2_BL,1250,1400,200,0.0,
       dmm_CS2_curr,
       show_progress=True,
       do_plot=True,
       exp=exp,
       measurement_name='CS2_CO',
       )

      #Find the highest peak 
       dataset=load_by_run_spec(captured_run_id=id+1)   
       Curr=dataset.get_parameter_data()['CS2_current']['CS2_current']
       x=dataset.get_parameter_data()['CS2_current']['CS2_BL']
       peak_idx,a=find_peaks(Curr,thr)
       ind=peak_idx[np.argmax(a['peak_heights'])]

      #Do a measurement around one peak 
       width=40
       thr=10e-12
       do1d(
       CS2_BL,x[ind]-width/2,x[ind]+width/2,50,0.01,
       dmm_CS2_curr,
       show_progress=True,
       do_plot=True,
       exp=exp,
       measurement_name='CS2_CO',
       )
      #Gaussian fit 
       dataset=load_by_run_spec(captured_run_id=id+2)      
       Curr=dataset.get_parameter_data()['CS2_current']['CS2_current']
       x=dataset.get_parameter_data()['CS2_current']['CS2_BL']
        
       def gaus(x,a,x0,sigma, offset):
           return a*np.exp(-(x-x0)**2/(2*sigma**2)) + offset       
       mean=(x[-1]+x[0])/2
       sigma=1#(x[-1]-x[0])/5
       popt,pcov = curve_fit(gaus,x[1:],Curr[1:],p0=[1,mean,sigma, Curr[0]])
       
       #Derivative
       def deriv_gaus(x,a,x0,sigma, offset):
           b=-(x-x0)/sigma**2
           return a*b*np.exp(-(x-x0)**2/(2*sigma**2))
            
       Deriv=deriv_gaus(x,popt[0],popt[1],popt[2],popt[3])
       #Deriv=gaus(x,popt[0],popt[1],popt[2],popt[3])
       max_slop=max(Deriv)
       index_slop=np.argmax(Deriv)
       CS2_BL(x[index_slop])
       
       #Plot to check if it is correct
       plt.plot(x,Curr)
       plt.plot(x,gaus(x,popt[0],popt[1],popt[2],popt[3]))
       #plt.plot(x,deriv_gaus(x,popt[0],popt[1],popt[2],popt[3]))
       plt.scatter(x[index_slop],Curr[index_slop],s=40)
       plt.show()
   
       return(CS2_BL.get())

   





   

  
      
