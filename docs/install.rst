Download and Installation
----------------------------

You can `download MoviePy`_ on its Github repository.


Just unzip everything in one folder, open a terminal and type ::
    
    sudo python setup.py install


This should install the python packages listed below, **but NOT OpenCV**, which you will have to download and install manually (see below).

MoviePy is a very small program but it relies on many existing Python packages:

- `OpenCV 2.4.6`_ or more recent, to read and write movie files. 
- PyGame_ for video and sound previews
- `Scipy and Numpy`_, for image and sound manipulation
- `Scikit Image`_ for advanced image manipulation 
- The Decorator_ module for better code readability

MoviePy also needs some external software that you will need to install:

- ffmpeg_, for writing movies and many other useful operations.
- imageMagick_ for text generation, GIF support, and much more in the future.

All these are normally easy to install (on linux, they will certainly be in your repos).

Installing OpenCV 2.4.6
~~~~~~~~~~~~~~~~~~~~~~~~

That will depend on your system. It seems easy for Windows. On linux, here is what I found on the Internet:

- Remove any other version of OpenCV if you installed it through a package manager.
- Unzip the source code of `OpenCV 2.4.6`_ in some folder. open a terminal in this folder.
- Make a new directory and go into this directory: ::
      
      mkdir release
      cd release
      
- Run ``cmake``. Here is the line I used: ::
      
      cmake -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=OFF -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D BUILD_EXAMPLES=ON ..
      
- Run ``make``. This may take a few minutes (15 minutes on my computer). ::
      
      make
      
- Finally, install. ::
      
      sudo make install
      
And voilà !

You can check if it worked by opeing a Python console and typing ::
    
    import cv2
    print cv2.__version__


    





.. _`download MoviePy`: https://github.com/Zulko/moviepy
.. _`OpenCV 2.4.6`: http://sourceforge.net/projects/opencvlibrary/files/
.. _Pygame: http://www.pygame.org/download.shtml
.. _`Scipy and Numpy`: http://www.scipy.org/install.html
.. _`Scikit Image`: http://scikit-image.org/download.html
.. _Decorator: https://pypi.python.org/pypi/decorator


.. _ffmpeg: http://www.ffmpeg.org/download.html 
.. _imageMagick: http://www.imagemagick.org/script/index.php
