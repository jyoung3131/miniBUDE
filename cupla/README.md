# miniBUDE CUPLA

This is an implementation of miniBUDE using CUPLA.

## Building

On Spock
```
#Create config and makefiles
cmake -Bbuild_hip_m100 -H. -DALPAKA_ACC_GPU_HIP_ENABLE=ON -DALPAKA_HIP_ARCH=908 -DCMAKE_BUILD_TYPE=Release
#Build
cmake --build build_hip_m100 --target bude --config Release 
```

## Running

This implementation has no special run-time options.
The `-n` and `-i` parameters are available.
Run `bude -h` for a help message.
