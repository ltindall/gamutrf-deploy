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

# Standalone scenario

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
