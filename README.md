# gamutrf-deply

# Deploying GamutRF

This repo contains simple instructions for deploying [GamutRF](https://github.com/iqtlabs/gamutrf). GamutRF supports many other
configuration options (including multiple SDRs, Pi4s, etc). However these instructions
should cover common use cases and serve as a starting point.

## Requirements

* [Ettus B200 SDR](https://www.ettus.com/all-products/usrp-b200mini-i-2/) (or similiar)
* Host with NVIDIA GPU supporting [CUDA 12.1](https://developer.nvidia.com/cuda-12-1-0-download-archive) or later
* (Optionally) Pi5

NOTE: you will need some basic docker and networking knowledge to identify IP addresses
and to install [docker with GPU support](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

NOTE: If you do not need inference, omit ```-f torchserve-cuda.yml``` in the commands below, and set ```TORCHSERVE``` to a blank string.

# Standalone scenario

```
          +------------+
+---+ USB | x64 + GPU  | 
|SDR+---->|(Scanner,   |                
+---+     | Waterfall, |
          | Torchserve)|
          +------------+
```

In this scenario, the complete GamutRF system is run on one machine, typically a laptop
with a GPU.

1. edit ```.env``` and set ```FREQ_START```, ```FREQ_END``` (scan range) and ```VOL_PREFIX``` to local directory for storage.
2. ```docker compose -f scanner.yml -f waterfall.yml -f torchserve-cuda.yml up -d```
3. GamutRF waterfall UI will appear on port 9003 (e.g. ```http://localhost:9003``` if running a browser on the same machine)

# Remote SDR scenario
         
```
          +---------+            +------------+
+---+ USB |   Pi5   |   Network  | x64 + GPU  |
|SDR+---->|(Scanner)+----------->|(Waterfall, |
+---+     |         |            | Torchserve)|
          +---------+            +------------+
```

In this scenario, the scanner is run remotely on the Pi5, and the waterfall and inference server
are run on another machine with a GPU.

1. edit ```.env``` on the Pi5 to set ```FREQ_START```, ```FREQ_END``` (scan range) and ```VOL_PREFIX``` to local directory for storage (```torchsig_model.mar``` must be in a ```model_store``` subdirectory of that local directory), and ```TORCHSERVE``` to be the address of the GPU machine.
2. edit ```.env``` on the GPU machine to set ```GAMUTRF``` to the IP address of the Pi5, and ```VOL_PREFIX``` to local directory for storage.
3. ```docker compose -f pi5-scanner-inference.yml up -d``` on the Pi5.
4. ```docker compose -f waterfall.yml -f torchserve-cuda.yml up -d``` on the GPU machine.
5. GamutRF waterfall UI will appear on port 9003 (e.g. ```http://localhost:9003``` if running a browser on the same machine)
