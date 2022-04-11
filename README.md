# Artifact Evaluation - OSDI'22

This documentation contains steps necessary to reproduce the artifacts for our
paper titled **KSplit: Automating Device Driver Isolation**.

All the experiments are evaluated on a Dell PowerEdge R820 machine on the
[Emulab Infrastructure](https://www.emulab.net/apt/show-hardware.php?type=d820)


## Setting up the hardware

* Create an account on [Cloudlab](https://www.cloudlab.us/) and login.

### Configuring the experiment

* Create an experiment profile by selecting
  `Experiments > Create Experiment profile`

* Select `Git Repo` and use this repository. The profile comes pre-installed
  with source code for evaluating ksplit's static analysis.
```
https://github.com/mars-research/ksplit-cloudlab
```

* Populate the name field and click `Create`

* If successful, instantiate the created profile by clicking `Instantiate`
  button on the left pane.



## Experiments

### Building KSplit Static Analyses

Compile Program dependence graph from the cloned sources
```
cd /opt/ksplit/pdg
mkdir build && cd build
cmake ..
make
```


### Table 1.a - 1.e Replication

 ```bash
 ./run_benchmarks.sh # run the 10 isolated benchmarks
 ./collect_benchmarks.sh # after all experiments finish, collect the stats
 cd benchmark_stats # the experiment number for each benchmark is included in the corresponding file name (dummy is null_net).
 ```

### Table 1.f Replication
 (Vikram, gcov)

 ### Table 1.g Replication
 ```bash
 cd IDL_manual_effort
 cd driver_dir
 ```
manual compare manually written IDL with automatically generated IDL.

manual IDL is prefixed with `manual_` and automatically generated IDL is prefixed with `auto_`.
 
### Table 2 (Subsystem stats)

Enter each subsystem's directory and run script to iterate through drivers

```bash
# char/tty
cd char
sudo bash ../run_subsystem.sh
cd ../tty
sudo bash ../run_subsystem.sh
# block
cd ../block
sudo bash ../run_subsystem.sh
# net
cd ../net
sudo bash ../run_subsystem.sh
# edac
cd ../edac
sudo bash ../run_subsystem.sh
# hwmon
cd ../hwmon
sudo bash ../run_subsystem.sh
#spi/i2c
cd ../spi
sudo bash ../run_subsystem.sh
cd ../i2c
sudo bash ../run_subsystem.sh
# usb
cd ../usb
sudo bash ../run_subsystem.sh
```
### Table 3 (Similarity)
```bash
cd ../table_3_IDL/net/
cd ../table_3_IDL/edac/
```
Compare the IDL between ixgbe, null_net and alx.
Compare the IDL between skx_edac and sb_edac.

### Table 4
1. The kernel should automatically be built by the profile creation script. If
  not, you can build the kernel with
```
cd /opt/ksplit/lvd-kernel
cp config_lvd .config
make -j $(nproc)
```

2. Compile the LVD modules (base microkernel and test modules)
```
cd /opt/ksplit/lvd-linux/lcd-domains
make
```

3. Check if we are running the LVDs kernel
```
uname -r
4.8.4-lvd
```

4. Loading the test module

* Load `vmfunc_klcd` module
```
cd ..
# load lcd-domains.ko
sudo ./scripts/mk
# load the test module that runs marshalling overheads test
sudo ./scripts/loadex vmfunc_klcd
```

* In another terminal, check the dmesg with `sudo dmesg`

* If successful, you should see something  like
```
message from lcd ffff881421cbd800: ===============
message from lcd ffff881421cbd800:   LCD BOOTED
message from lcd ffff881421cbd800: ===============
Running marshal_empty test!
test_marshal_empty: 1000000 iterations of marshal int took 538368116 cycles (avg: 538 cycles)
Running marshal_int test!
test_marshal_int: 1000000 iterations of marshal int took 537546720 cycles (avg: 537 cycles)
Running marshal_array test!
message from lcd ffff881421cbd800: marshal_array_callee, allocating an array of size 32
test_marshal_array: 1000000 iterations of marshal array[32] took 706914708 cycles (avg: 706 cycles)
Running marshal_string test!
test_marshal_string: 1000000 iterations of marshal string (len: 255) took 1312815912 cycles (avg: 1312 cycles)
Running marshal_voidptr test!
message from lcd ffff881421cbd800: marshal_voidptr_callee, allocating a buffer of size 4096
test_marshal_voidptr: 1000000 iterations of marshal void ptr (sz=4096) took 886989248 cycles (avg: 886 cycles)
Running marshal_union test!
test_marshal_union: 1000000 iterations of marshal union took 700801680 cycles (avg: 700 cycles)
```
