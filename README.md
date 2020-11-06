# LDSO: Direct Sparse Odometry with Loop Closure

## Related Publications

 * **LDSO: Direct Sparse Odometry with Loop Closure**, *X. Gao, R. Wang, N. Demmel, D. Cremers*,
   In International Conference on Intelligent Robots and Systems (IROS), 2018.
 * **Direct Sparse Odometry**, *J. Engel, V. Koltun, D. Cremers*,
   In IEEE Transactions on Pattern Analysis and Machine Intelligence (PAMI), 2018
 * **A Photometrically Calibrated Benchmark For Monocular Visual Odometry**, *J. Engel, V. Usenko, D. Cremers*,
   In arXiv:1607.02555, 2016

## Dependencies
 
 ```
sudo apt-get install libeigen3-dev libgoogle-glog-dev libgtest-dev libsuitesparse-dev libopencv-dev libzip-dev libboost-all-dev
 ```
 
build DBoW3 in the thirdparty folder first.
```
cd thirdparty/DBoW3
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=$BUILD_TYPE ..
make
```

### Other libraries

Compile and install
[Pangolin](https://github.com/stevenlovegrove/Pangolin) for
visualization.
 
### Compile

This will build the thirdparty library and also ldso library for
you. You can also follow the steps in this script manually (will
compile DBoW3 and g2o first, and the ldso).

## Run

**TUM-Mono:**

To run LDSO on TUM-Mono dataset sequence 34, execute:

```
rosrun ldso ldso
    preset=0 \
    files=XXXXX/TUMmono/sequences/sequence_34/images.zip \
    vignette=XXXXX/TUMmono/sequences/sequence_34/vignette.png \
    calib=XXXXX/TUMmono/sequences/sequence_34/camera.txt \
    gamma=XXXXX/TUMmono/sequences/sequence_34/pcalib.txt
```

## Notes

 - LDSO is a monocular VO based on DSO with Sim(3) loop closing
   function. Note we still **cannot** know the real scale of
   mono-slam. We only make it more consistent in long trajectories.
   
 - The red line in pangolin windows shows the trajectory before loop
   closing, and the yellow line shows the trajectory after
   optimization.
   
 - If you are looking for code instructions, take a look at
   `doc/notes_on_ldso.pdf` and see if it can help you.
 
 - Set `setting_enableLoopClosing` to true/false to turn on/off loop
   closing function. `setting_fastLoopClosing` will record less data and
   make the loop closing faster.
   
 - If you need loop closing, please set `setting_pointSelection=1` to
   make the program compute feature descriptors. If
   `setting_pointSelection=0`, the program acts just like DSO, and
   `setting_pointSelection=2` means random point selection, which is
   faster but unstable.
   
 - Some of the GUI buttons may not work.
   
 - You might also want to have a look DSO's README:
   https://github.com/JakobEngel/dso/blob/master/README.md

## License

LDSO, like DSO, is licensed under GPLv3. It makes use of several
third-party libraries. Among other's it comes with the source of:
 - Sophus ([MIT](https://github.com/strasdat/Sophus/blob/master/LICENSE.txt))
 - g2o ([BSD, GPL, LGPL, ...](https://github.com/RainerKuemmerle/g2o/blob/master/README.md))
 - DBoW3 ([BSD](https://github.com/raulmur/ORB_SLAM2/blob/master/Thirdparty/DBoW2/LICENSE.txt))
