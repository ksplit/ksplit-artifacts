# Artifact Evaluation - OSDI'22

Thank you for your time and picking our paper for the artifact evaluation.

This documentation contains steps necessary to reproduce the artifacts for our
paper titled **KSplit: Automating Device Driver Isolation**.

All the experiments are evaluated on a Dell PowerEdge R820 machine on the
[Emulab Infrastructure](https://www.emulab.net/apt/show-hardware.php?type=d820)

## Setting up the hardware

* Create an account on [Cloudlab](https://www.cloudlab.us/) and login.

### Configuring the experiment

#### Automated setup
* The easiest way to setup our experiment is to use "Repository based profile".

* Create an experiment profile by selecting
  `Experiments > Create Experiment profile`

* Select `Git Repo` and use this repository. The profile comes pre-installed
  with source code for evaluating ksplit's static analysis.
```
https://github.com/ksplit/ksplit-cloudlab
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
  [here](https://github.com/ksplit/ksplit-cloudlab/blob/ksplit-test/ksplit-top.sh)

* *NOTE* The automated script is executed by a different user (`geniuser`). If
  you need to manually build something under `/opt/ksplit` make sure to change
  ownership.
  ```
  pushd /opt/ksplit
  sudo chown -R <your-user-name>:<your-group> .
  ```

#### Manual setup
* If you want to set up the source repos manually for some reason,
```
mkdir /local
# make sure you have permissions or use chown
git clone https://github.com/ksplit/ksplit-cloudlab.git /local/respository
# invoke the top level script
/local/respository/ksplit-top.sh
```

## Experiments

### Prerequisites
* The following steps assume that the experiment profile is created using the
  aforementioned steps (i.e., using the git repo for creating the profile).

* To create this setup manually,
  - Clone https://github.com/ksplit/ksplit-cloudlab
  - Make sure `/opt/ksplit` is writable
  - Execute `ksplit-top.sh`

### Building KSplit Static Analyses
* The pdg sources should automatically be built for you. If not, refer to the
  Prerequisites subsection above to set it up manually.

* Clone the artifacts repository in your `${HOME}` directory. The scripts have
  this path hardcoded.
```
cd ${HOME}
git clone https://github.com/ksplit/ksplit-artifacts.git
```

### Table 1.a - 1.e Replication

* To replicate Table1 from the paper, please invoke this script which runs the
  analyses, collects the results and present it in a csv file.
 ```bash
 cd /opt/ksplit/bc-files
 ./run_benchmarks.sh table1
 ```

* The above step runs the analysis in the background and should take 1-2 hours
  to complete. If successful, the results for table1 should be available in
  `/opt/ksplit/bc-files/benchmark_stats/merged_stats.csv`. The format is
  similar to our Table1 on paper. Inorder to visually compare, one can paste
  the `merged_stats.csv` into a csv viewer (e.g., google sheets).

* The remaining numbers (1.f - 1.g) are part of the manual effort.
  Unfortunately, we do not have an automated way to replicate these numbers as
  many of them requires domain knowledge (IDL syntax, understanding kernel -
  driver interaction patterns to classify pointer misclassifications etc).

### Table 2 (Subsystem stats)

* Similar to Table1, Table2 script can be invoked with, 
 ```bash
 cd /opt/ksplit/bc-files
 ./run_benchmarks.sh table2
 ```
  This script runs the analyses for the subsystems listed under
  `/opt/ksplit/bc-files/subsystems_list` collects the results and present it in
  a csv file.

* The above snippet should take around 5-6 hours to finish. Upon completion,
  running the below script would accumulate summarize all the individual stats
  into a summary file for the subsystem. To cut down on the total time, we
  picked only a few subsystems out of the whole list, namely block, edac, hwmon
  and usb. However, the same steps can be used to collect metrics for all the
  available subsystems under our `bc-files` repository.

* If successful, the script generates a merged csv file at
  `/opt/ksplit/bc-files/benchmark_stats/table2/merged_stats_table2.csv`

### Table 3 (Similarity)
```bash
cd table_3_IDL/net/
cd table_3_IDL/edac/
```
Compare the IDL between `ixgbe`, `null_net` and `alx`.
Compare the IDL between `skx_edac` and `sb_edac`.

> Counting method:
- **Shared rpc comparison**: compare rpc stubs (function start with rpc/rpc_ptr) among the IDLs and count the common ones.
- **Shared rpcs IDL delta**: compare the rpc stubs and count the different ones.
- **Shared rpcs Annotation**: compare the annotations among the IDLs and count the differences.
- **New IDL**: Count the IDL difference between the referenced IDL and the comparison IDL. The IDLs exist in the compared IDL but do not exist in the referenced IDL are considered as new IDL. For example, the referenced IDL is ixgbe and the null_net IDL is used for comparison. Then, we go through the IDL code in the null_net IDL and then find the part of code that are not exist in the ixgbe. The lines of these code are counted as new IDL.


### Table 4

* Make sure you have cloned the artifacts repo in your `${HOME}` folder. The script should
  output the numbers from `dmesg` in csv format.
```
cd ${HOME}
cd ksplit-artifacts/table4
./table4.sh
```

*NOTE:* There are some known issues in loading two LVD modules at the same time
or back-to-back. To avoid these troubles and to have a clean setup, please reboot
before loading the second test module.

### End-to-End examples

#### `msr` driver

`msr` driver is located at `/opt/ksplit/lvd-linux/arch/x86/kernel/msr.c`

* The push-button script does the following
  * Generates the IDL file from `msr.ko.bc` and `msr_kernel.bc`
  * Patch the IDL and driver
  * Load and test by reading/writing to `/dev/cpu/x/msr`

* invoke the script
```
cd ${HOME}/ksplit-artifacts/msr
./end-to-end-msr.sh
```

* The script performs a read, write and a read back from the msr device file.
  Upon successful execution, you should see this message at the end
```
=====================================
Autogenerated msr module test passed
=====================================
```

### Figure 4

At the moment, this artifact package does not contain the scripts necessary for
replicating the memcached benchmarks. It is still a work in progress.

### Approximate running time

Here is an estimate for how long it takes to replicate each benchmark

* setup -> 30-40 minutes (one-time)
* table1 -> ~1 hour
* table2 -> 5-6 hours
* table3 -> 1-2 hours (manual effort)
* table4 -> 10-20 minutes
* End-to-End example -> 1-2 hours
