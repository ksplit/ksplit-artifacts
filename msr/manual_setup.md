*NOTE:*
For the push-button script, execute `end-to-end-msr.sh`. Please look for instructions in the top-level `README.md

## Detailed manual setup

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
