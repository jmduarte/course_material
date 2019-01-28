### Check out SDAccel and setup environment

```
git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR  
cd $AWS_FPGA_REPO_DIR                                         
source sdaccel_setup.sh #every time you log in
```

More detailed information [here](https://github.com/aws/aws-fpga/tree/master/SDAccel)

### Clone hls4ml wrapper for SDAccel

```
cd ../../
git clone https://github.com/FPGA4HEP/hls4ml_c.git
```

Edit Makefile in the hls4ml_c directory to change default input directory name.


### Run software simulation, hardware emulation and build FPGA binary

```
cd ~/hls4ml
source deactivate hls4ml-env
cd ~/hls4ml_c
make clean                                                                 
make check TARGETS=sw_emu DEVICES=$AWS_PLATFORM all                 #software emulation
make check TARGETS=hw_emu DEVICES=$AWS_PLATFORM all                 #hardware emulation
make TARGETS=hw DEVICES=$AWS_PLATFORM all && ./create.sh            #firmware building
```

### Run on real FPGA

Launch a F1 instance and copy the host and binary files (.awsxclbin) from the T2 (nb, first copy to your laptop). 

Setup the SDAccel environment on the F1 as well:

```
git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
cd $AWS_FPGA_REPO_DIR 
source sdaccel_setup.sh
sudo sh
source /home/centos/src/project_data/aws-fpga/sdaccel_runtime_setup.sh
``` 

Now copy the input features file from your hls4ml project directory on the T2 (my-hls-test-FAVOURITE-MODEL) to the F1 to pass it to the FPGA.

Finally, you can accelerate your NN inference on the FPGA running on the input features:

```
./host
```

The application will produce a file with the predictions from the FPGA run. Compare it with HLS and Keras calculations using the extract_roc.py script in the hls4ml directory on the T2 instance (nb, copy the tb_output_data.dat from the F1 to the T2)

```
python extract_roc.py -c keras-config-FAVOURITE-MODEL.yml -f tb_output_data.dat
```