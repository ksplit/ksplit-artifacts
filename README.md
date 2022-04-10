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

### Table 1

* Compile Program dependence graph from the cloned sources
```
cd /opt/ksplit/pdg
mkdir build && cd build
cmake ..
make
```
* Run a simple test program
```

```

### Table 2
* Follow build steps for generating Table 1

*
```
cd /opt/ksplit/bc-files
# run_table2.sh
```

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

4. Load `vmfunc_klcd` module
```
cd ..
# load lcd-domains.ko
sudo ./scripts/mk
# load the test module that runs marshalling overheads test
sudo ./scripts/loadex vmfunc_klcd
```

* In another terminal, check the dmesg
```
sudo dmesg
```
