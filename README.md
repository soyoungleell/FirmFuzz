# Setup instructions

- Run `setup.sh` which will set up cross-compilation toolchains and setup the cross-compiled
  FirmFuzz kernels
 
- Fix up path for `HOME_DIR` in `./framework/scripts` and put `PATH` in `TEST_PATH`, this is because
  we run certain scripts as sudo and require location of cross compilation binaries

# Run instructions

Using FirmFuzz on a firmware filesystem is a three-step process:
- Extracting the firmware filesystem from the firmware image
- Create a firmware emulation configuration for the extracted filesystem
- Run the fuzzer on the emulated fimrware image

Instructions for each of these steps is provided below.

## Filesystem extraction

- Use the firmadyne extractor system to extract the linux filesystem from the firmware image. Otherwise
you can provide a filesystem extracted using binwalk as well but I didn't test with that.

Cross compiling kernel [1]
----------------------------

- After installing the cross compilation tool chains make sure to put their locations
  in your $PATH environment variable 
- Go the `kernel_firmfuzz` directory inside /path/to/IoT_Research/framework/kernel_firmfuzz
- For MIPS big-endian kernel
    - `mkdir -p build/mipseb`
    - `cp config.mipseb build/mipseb/.config`
    - `make ARCH=mips CROSS_COMPILE=mipseb-linux-musl- O=./build/mipseb -j8`
- For MIPS little-endian kernel
    - `mkdir -p build/mipsel`
    - `cp config.mipsel build/mipsel/.config`
    - `make ARCH=mips CROSS_COMPILE=mipsel-linux-musl- O=./build/mipsel -j8`

- Make sure to run both sets of commands and make sure the build directory for
  the kernels is built as given in the commands

Requirements
-------------
- Framework components
    - Firmware image (From Firmadyne scraper)
    - makeImage.sh
    - extractor.py
    - $IMAGE_SCRIPTS/env_var.config
    - $IMAGE_SCRIPTS/getArch.sh
    - $IMAGE_SCRIPTS/pack.sh
    - $IMAGE_SCRIPTS/inferNetwork.sh

- Firmware prep files
    - $IMAGE_SCRIPTS/init
    - $IMAGE_LIB/libnvram.so.mipseb
    - $IMAGE_LIB/libnvram.so.mipsel
    - $IMAGE_SCRIPTS/dev.tgz

Running instructions
----------------------
- Before running this make sure the variables in `env_var.config` are pointing to
  existing directories with their paths fixed for your workstation
  
- Take a firmware image scraped using the scraper and then run the FS extractor
  on it 
    - `./extractor.py -b Trendnet -np -nk "fc50b605e2d17ed4bbc2fc9cf7a16fa545afdc15.asp" ./`
    - This will create the tarball in the current dir, with the same name_md5sum.tar.gz
      format

- Pass the image created before with the same name as is to the script which will place the 
  final image along with helper scripts in the $FINAL_IMAGE dir
    - `./makeImage.sh <firmware_image>.tar.gz`

First-time Setup Instructions
----------------------------
- For the first time make the following adjustments:
    - Update `ROOT_DIR` in `env_var.config`, `cleanup_fs.sh`and `cleanup.sh` to point to your FirmFuzz Directory
    - Run `./cleanup_fs.sh` to build all the kernels
    
Batch Run Instructions
-----------------------

- Run the script as given below to extract all linux-based FS into `images/` dir:
    - Copy the image directory into the current dir
    - `./extractor_wrapper.sh /path/to/image/database`
    - Dependencies: `extractor.py`

- Copy the extracted images to the extracted directory

- Run the batch emulation script on the set of images
    - `./run_batch.sh images/`
    - Dependencies: `makeImage_batch.sh` and it's dependencies

- Put the emulated images in the `FINAL_DIR` into a vendor-specific directory

- Run the `check_reachable.sh` script to segregate the emulated images into ones
  which are reachable from the host and not reachable
    - `./check_reachable.sh /path/to/emulated/images`

- Run `./cleanup.sh mipsel/mipseb` if you want to clean a specific build dir or
  `./cleanup.sh` if you want to clean both the build dir. Run this before starting
  the framework or if the emulation process ended abruptly

Batch run instructions with FS
--------------------------------
TODO

Initial Run of the image
-------------------------

- Before running the image, we have to save the stable snapshot of the image to which
  we can revert to during the fuzzing process

- Go to the image dir, run the emulation and then run the following commands
    - `ctrl-a-c` : go to monitor mode
    - `savevm 1` : save the vm

- Exit the emulation

Single-Run Image
=================

- Run `cleanup.sh` before running the script
- Put the image in the same directory as `makeImage.sh` and run the script

References
------------
[1] https://github.com/firmadyne/firmadyne   
