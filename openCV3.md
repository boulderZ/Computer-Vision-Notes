### OpenCV3 install and compile for SIFT and SURF

I followed the following from the excellent blogger Adrian Rosebrock
[install openCV 3.0](http://www.pyimagesearch.com/2015/07/20/install-opencv-3-0-and-python-3-4-on-ubuntu/) in order to attempt to get SIFT and SURF capability. You must install and compile openCV from source to get the opencv_contrib modules that can do SIFT and SURF and other very useful computer vision tasks.

I wanted to do this from my existing conda environment where I already had openCV 3.1.0 installed but without opencv_contrib. I could not find a good tutorial on how to do this with conda but did find this which I may try later:
[cmake anaconda](https://groups.google.com/a/continuum.io/forum/#!topic/anaconda/R9gWjl09UFs)

So I followed the Rosebrock post and got all the way to 84% on the compile before it had fatal errors. See file opencv_error.txt for  details. Turns out this seems to be a known issue when using openCV 3.0 with cuda 8.0 and there is a patch.
[openCV and Cuda 8.0](https://github.com/opencv/opencv/issues/6677)

Then followed the steps in the post above
`git clone https://github.com/daveselinger/opencv`
`git checkout 3.1.0-with-cuda8`


and then had a cmake error:

CMake Error: The following variables are used in this project, but they are set to NOTFOUND.
Please set them or make sure they are set and tested correctly in the CMake files:
HDF5_C_INCLUDE_DIR (ADVANCED)
   used as include directory in directory /home/ai/opencv/modules/python/python2
   used as include directory in directory /home/ai/opencv/modules/python/python3

-- Configuring incomplete, errors occurred!
See also "/home/ai/opencv/build/CMakeFiles/CMakeOutput.log".
See also "/home/ai/opencv/build/CMakeFiles/CMakeError.log".

I then did the following also from the post
`sudo apt-get install libhdf5-dev`

Then everything built all the way. The `sudo make install` completed. The `sudo ldconfig` seemed ok too.

However then there was no ``/usr/local/lib/python3.4/site-packages/`` directory and only a `dist-packages` which did not contain the `cv2.cpython-34m.so` file that is supposed to be there. There were two different threads suggesting what was wrong and how to fix. I went with the install later version of cmake which said to remove old version first using `sudo apt remove cmake`. I did this and it looks like it removed much of my ROS install?? Will find out later.

What I did (substitute latest release for below)

```
  Check your current version with cmake --version
  Uninstall it with sudo apt remove cmake  #### Caution!
  Visit https://cmake.org/download/ and download the latest binaries
      In my case cmake-3.6.2-Linux-x86_64.sh is sufficient
  chmod +x /path/to/cmake-3.6.2-Linux-x86_64.sh (use your own file location here, but chmod makes the script executable)

  ./path_to_cmake/cmake-3.6.2-Linux-x86_64.sh (you'll need to press y twice)

 make a symbolic link:

  sudo ln -s /path_to_cmake/cmake-x.x.x-Linux-x86_64/bin/* /usr/local/bin

  Test your results with cmake --version
  ```


There was an error coming up whenever I typed `workon cv` where `cv` was the name of my virtualenv. This is a conflict with anaconda deactivate. The fix is below:

```
So, in the virtualenvwrapper.sh script, we can change the following two lines which test for whether deactivate is merely callable:

type deactivate >/dev/null 2>&1
if [ $? -eq 0 ]

with the stricter test for whether it's a shell function:

nametype="$(type -t deactivate)"
if [ "${nametype##* }" == "function" ]
```
So now hoping that upgrade to cmake will make the install work so back to start again.

Installing newer version of cmake fixed the problem. So three things done differently from the blog post.

1) `sudo apt-get install libhdf5-dev`
2) `git clone https://github.com/daveselinger/opencv`
`git checkout 3.1.0-with-cuda8`
3) Install newer version of cmake

 Next problem was that matplotlib would not import.
Need to do the following:
```
pip install scipy
pip install matplotlib
```
Then `sudo apt-get install tcl-dev tk-dev python-tk python3-tk`

I was then able to import and use cv2  with the SIFT algorithm. I modified the tutorial code to use `cv2.BFMatcher()` instead of `cv2.FlannBasedMatcher` as I got errors trying to use the latter. Looks like there is a known bug with python version of this: [opencv-issues-5667](https://github.com/opencv/opencv/issues/5667). The modified code is in the file epipolar_test.py. 
