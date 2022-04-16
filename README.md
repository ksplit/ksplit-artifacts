# Artifact Evaluation - OSDI'22

Thank you for your time and picking our paper for the artifact evaluation.

This documentation contains steps necessary to reproduce the artifacts for our
paper titled **KSplit: Automating Device Driver Isolation**.

All the experiments are evaluated on a Dell PowerEdge R820 machine on the
[Emulab Infrastructure](https://www.emulab.net/apt/show-hardware.php?type=d820)


## Setting up the hardware

* Create an account on [Cloudlab](https://www.cloudlab.us/) and login.

### Configuring the experiment

* The easiest way to setup our experiment is to use "Repository based profile".

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

* *NOTE* You can select different branches on the git repository. Please select
  `master` branch.

* For a more descriptive explanation and its inner details, consult the
  cloudlab documentation on [repo based profiles](https://docs.cloudlab.us/creating-profiles.html#(part._repo-based-profiles)

* The profile git repository contains a bootstrapping script which
  automatically clones and builds the following repositories, upon a successful bootup of the node.
  - `pdg` - Static analyses
  - `lvd-linux` - modified LVDs kernel
  - `bflank` - modified bareflank hypervisor
  - `bc-files` - llvm bit-code files for driver modules

* Please allow sometime to clone and build all the source code. You can check
  the progress by tailing the log file.

* A short log is located at `/users/geniuser/ksplit-setup.log`. If the setup is
  successful, the short log should contain something similar to the one below
```
[Thu Apr 14 00:55:13 CDT 2022] Begin setup!
[Thu Apr 14 00:55:13 CDT 2022] Installing dependencies...
[Thu Apr 14 00:55:21 CDT 2022] Downloading llvm script to /users/geniuser/llvm.sh
[Thu Apr 14 00:55:28 CDT 2022] Preparing local partition ...
[Thu Apr 14 00:55:39 CDT 2022] Cloning PDG
[Thu Apr 14 00:55:39 CDT 2022] Cloning Bareflank
[Thu Apr 14 00:55:40 CDT 2022] Cloning LVD linux
[Thu Apr 14 00:57:14 CDT 2022] Cloning bc-files
[Thu Apr 14 01:07:51 CDT 2022] Building SVF
[Thu Apr 14 01:07:51 CDT 2022] Building PDG
[Thu Apr 14 01:07:52 CDT 2022] Building bareflank
[Thu Apr 14 01:10:41 CDT 2022] Building Linux
[Thu Apr 14 01:15:46 CDT 2022] Building lcd-domains
[Thu Apr 14 01:18:56 CDT 2022] Done Setting up!
```

* A detailed log is available at `/users/geniuser/ksplit-verbose.log`

* The script that gets executed after startup is available
  [here](https://github.com/mars-research/ksplit-cloudlab/blob/ksplit-test/ksplit-top.sh)

* *NOTE* The automated script is executed by a different user (`geniuser`). If
  you need to manually build something under `/opt/ksplit` make sure to change
  ownership.
  ```
  pushd /opt/ksplit
  sudo chown -R <your-user-name>:<your-group> .
  ```

## Experiments

### Prerequisites
* The following steps assume that the experiment profile is created using the
  aforementioned steps (i.e., using the git repo for creating the profile).

* To create this setup manually,
  - Clone https://github.com/mars-research/ksplit-cloudlab
  - Make sure `/opt/ksplit` is writable
  - Execute `ksplit-top.sh`

### Building KSplit Static Analyses
* The pdg sources should automatically be built for you. If not, refer to the
  Prerequisites subsection above to set it up manually.

### Table 1.a - 1.e Replication
 ```bash
 pushd /opt/ksplit/bc-files
 sudo run_benchmarks.sh # run the 10 isolated benchmarks, wait for all benchmarks to terminate
 sudo bash collect_benchmarks.sh # after all experiments finish, collect the stats
 pushd benchmark_stats # the experiment number for each benchmark is included in the corresponding file name (dummy is null_net).
 sudo python3 merge_to_csv.py # merge all the benchmark stats into one file "merged_stats.csv"
 popd
 popd
 ```

* The remaining numbers (1.f - 1.g) are part of the manual effort.
  Unfortunately, we do not have an automated way to replicate these numbers as
  many of them requires domain knowledge (IDL syntax, understanding kernel -
  driver interaction patterns to classify pointer misclassifications etc).

### Table 2 (Subsystem stats)

Enter each subsystem's directory and run script to iterate through drivers

```bash
for subsys in char tty block net edac hwmon spi i2c usb; do
  pushd ${subsys};
  sudo bash ../run_subsystem.sh # run ksplit analysis on all the drivers in the subsystem
  sudo python3 ../summarize_module_stats.py # summarize the average number for all the drivers in the subsystem into file stats_summary
  popd;
done
```

### Table 3 (Similarity)
```bash
cd table_3_IDL/net/
cd table_3_IDL/edac/
```
Compare the IDL between `ixgbe`, `null_net` and `alx`.
Compare the IDL between `skx_edac` and `sb_edac`.

### Table 4
1. Following the above steps, the node should be running the `4.8.4-lvd` kernel.
```
uname -r
4.8.4-lvd
```

2. Insert bareflank hypervisor module
```
cd /opt/ksplit/bflank/build
make driver_quick && make quick
```

You should see similar messages on the console
```
Scanning dependencies of target driver_quick
Scanning dependencies of target driver_unload
Built target driver_unload
Scanning dependencies of target driver_clean
Built target driver_clean
Scanning dependencies of target driver_build
Built target driver_build
Scanning dependencies of target driver_load
Built target driver_load
Built target driver_quick
Scanning dependencies of target quick
Built target quick
```

In `dmesg`, you should see
```
[ 2682.962046] [BAREFLANK DEBUG]: dev_init succeeded
[ 2683.051737] [BAREFLANK DEBUG]: dev_open succeeded
[ 2683.081715] [BAREFLANK DEBUG]: IOCTL_ADD_MODULE_LENGTH: succeeded
[ 2683.082097] [BAREFLANK DEBUG]: IOCTL_ADD_MODULE: succeeded
[ 2683.494023] [BAREFLANK DEBUG]: IOCTL_LOAD_VMM: succeeded
[ 2683.494046] [BAREFLANK DEBUG]: dev_release succeeded
[ 2683.501491] [BAREFLANK DEBUG]: dev_open succeeded
[ 2694.136286] [BAREFLANK DEBUG]: IOCTL_START_VMM: succeeded
[ 2694.136330] [BAREFLANK DEBUG]: dev_release succeeded
```

4. Loading the test module.

* Load the microkernel module
```
cd /opt/ksplit/lvd-kernel/lcd-domains
# load lcd-domains.ko
./scripts/mk
```

* Load the `vmfunc_klcd` module
```
# load the test module that runs marshalling overheads test
./scripts/loadex vmfunc_klcd
```

* In another terminal, check the dmesg with `sudo dmesg -w`

* If successful, you should see something  like
```
message from lcd ffff881421cbd800: ===============
message from lcd ffff881421cbd800:   LCD BOOTED
message from lcd ffff881421cbd800: ===============
Running marshal_empty test!
test_marshal_empty: 1000000 iterations of marshal none took 530368116 cycles (avg: 530 cycles)
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

### End-to-End examples

#### `msr` driver

`msr` driver is located at `/opt/ksplit/lvd-linux/arch/x86/kernel/msr.c`

1) First step is to gather the bc files. For performing the analysis, we need
  two bc files, `msr_driver.bc` and `msr_kernel.bc`. The bc files for msr are
  available at `/opt/ksplit/bc-files/arch_x86/msr/`

2) Run static analysis to generate the idl.
```bash
pushd /opt/ksplit/bc-files/arch_x86/msr/
sudo bash ../../run_nescheck.sh msr
popd
```

3) Create a new test module folder in `lvd-linux` and copy the generated IDL
```
pushd /opt/ksplit/lvd-linux/lcd-domains/test_mods
mkdir msr_autogen
# copy the auto generated idl here
cp /opt/ksplit/bc-files/arch_x86/msr/kernel.idl msr_autogen.idl
```

4) Patch the auto generated IDL that require manual intervention.
```
patch -p0 < msr-idl-changes.patch
```

5) Generate rpc stubs. If successful, this should generate `common.{c,h}`,
`client.c`, `server.c` and a linker script.
```
/opt/ksplit/lcds-idl/build/idlc ./msr_autogen.idl
```
6) Create entries in the configuration script and Kbuild.
- Add config entry to `/opt/ksplit/lvd-linux/lcd-domains/scripts/defaultconfig`
```
msr_autogen/boot nonisolated
msr_autogen/msr_lcd isolated
msr_autogen/msr_klcd nonisolated
```

- Add Kbuild entry to `/opt/ksplit/lvd-linux/lcd-domains/scripts/Kbuild.test_mods`
```
obj-m += msr_autogen/
```

7) The isolated module needs a bunch of boilerplate code to bootstrap. All the
necessary files are available in this repo under `msr/msr_autogen` folder. After
the above steps, you can copy all the files under `msr/msr_autogen` to the
directory created in step2.

*NOTE* The boilerplate code assumes that the directory created is named
`msr_autogen`. If you have created it with some other name, it needs to be
reflected in the boilerplate code.

- Before compiling the module, a few lines need to be patched in the auto-generated code.
The patch is located at `msr/msr-driver-changes.patch`.

```
cd /opt/ksplit/lvd-linux/lcd-domains/test_mods
patch -p0 < msr/msr-driver-changes.patch
```

8) Compile the newly auto-generated module
```
cd /opt/ksplit/lvd-linux/lcd-domains/
rm test_mods/config
make test_mods
```

9) Load the hypervisor. Follow step 2 under Table4 experiments above

10) Load the microkernel and msr_autogen
```
cd /opt/ksplit/lvd-linux/lcd-domains/
./scripts/mk
./scripts/loadex msr_autogen
```

11) Test the msr driver
```
sudo apt install msr-tools
# This should trigger an msr read through the isolated driver
sudo rdmsr 0x1a4
```
