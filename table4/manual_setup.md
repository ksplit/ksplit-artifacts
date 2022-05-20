## Table 4 benchmarks
--

If you are looking for a push-button script, please invoke `table4.sh`

Here is a detailed explanation of the steps involved to recreate table4 from
the paper.

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
cd /opt/ksplit/lvd-linux/lcd-domains
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

