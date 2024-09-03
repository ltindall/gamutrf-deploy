# gamutrf-deploy

# Deploying GamutRF

This repo contains simple instructions for deploying [GamutRF](https://github.com/iqtlabs/gamutrf). GamutRF supports many other
configuration options (including multiple SDRs, Pi4/5ss, etc). However these instructions should cover common use cases and serve as a starting point.

## Requirements

* [Ettus B200 SDR](https://www.ettus.com/all-products/usrp-b200mini-i-2/) (or similiar)
* Host with NVIDIA GPU supporting [CUDA 12.1](https://developer.nvidia.com/cuda-12-1-0-download-archive) or later

NOTE: you will need some basic docker and networking knowledge to identify IP addresses
and to install [docker with GPU support](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

NOTE: If you do not need inference, omit ```-f torchserve-cuda.yml``` in the commands below, and set ```TORCHSERVE``` to a blank string.

## Single host operation

```
          +------------+
+---+ USB | x64 + GPU  | 
|SDR+---->|(Scanner,   |                
+---+     | Waterfall, |
          | Torchserve)|
          +------------+
```

In this scenario, the complete GamutRF system is run on one machine, typically a laptop with a GPU.

1. edit ```.env``` and set ```FREQ_START```, ```FREQ_END``` (scan range) and ```VOL_PREFIX``` to local directory for storage.
2. ```docker compose -f scanner.yml -f waterfall.yml -f torchserve-cuda.yml up -d```
3. GamutRF waterfall UI will appear on port 9003 (e.g. ```http://localhost:9003``` if running a browser on the same machine)

Once the system is up and running, sample recording and frequency ranges can be controlled from the Waterfall UI.

### Stare mode

Scanning can be disabled such that GamutRF looks at only one frequency continuously. To configure this mode, set ```FREQ_START``` (or use the Waterfall UI) to the lowest frequency to be observed, and ```FREQ_END``` to 0. GamutRF will monitor from ```FREQ_START``` to ```FREQ_START``` plus GamutRF's sample rate (by default 20.48Msps).

### Recording samples

Recording samples is also managed from the Waterfall UI. Enter the number of samples to record (for example, ```20048000``` will record one second's worth of samples at the default sample rate. A recording accompanied by a [SigMF](https://github.com/sigmf/SigMF) file under ```VOL_PREFIX```.

## Troubleshooting

### SoapySDR errors allocating buffers

Run ```echo 0 > /sys/module/usbcore/parameters/usbfs_memory_mb``` as root before starting the scanner(s).

### Containers won't start using Ettus SDRs

##### ```[ERROR] [USB] USB open failed: insufficient permissions```

Ettus SDRs download firmware and switch USB identities when first powered up. Restart the affected container to work around this (if run with docker compose, restart will happen automatically).

##### ```[ERROR] [UHD] An unexpected exception was caught in a task loop.The task loop will now exit, things may not work.boost: mutex lock failed in pthread_mutex_lock: Invalid argument```

UHD driver arguments ```num_recv_frames``` or ```recv_frame_size``` may be too high. The defaults are defined as ETTUS_ARGS in [utils.py](https://github.com/IQTLabs/gamutRF/blob/main/gamutrf/utils.py)
. Try reducing one or both via ```--sdrargs```. For example, ```--sdrargs num_recv_frames=64,recv_frame_size=8200,type=b200```.

##### ```[ERROR] [UHD] EnvironmentError: IOError: usb rx6 transfer status: LIBUSB_TRANSFER_OVERFLOW```

Stop containers, and reset the Ettus as follows:

```
$ /usr/libexec/uhd/utils/b2xx_fx3_utils -D
$ /usr/libexec/uhd/utils/b2xx_fx3_utils -U
$ /usr/libexec/uhd/utils/b2xx_fx3_utils -S
```
