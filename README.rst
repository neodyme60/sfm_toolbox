This toolbox is a compilation of popular Structure From Motion (SFM) algorithms. As I used more and more SFM over the years, I could never find anything that goes beyond simple SFM (please, prove me wrong and send me links !!). I compiled all my code into a toolbox so that others do not have to reimplement everything. I apologize in advance if you do not find what you need (like algorithms using the L-infinity beauty) but I only implemented what I needed and optimized what I needed most:

  * data generation (random rigid/non-rigid scene, popular non-rigid examples, different camera models, random or smooth random camera positions (with circle splines for Sequin), noise addition ...)
  * 2D geometry (homography computations ...)
  * 3D geometry (quaternion, globally optimal pose estimation, absolute orientation, triangulation ...)
  * rigid 3D reconstruction: orthographic (Tomasi-Kanade with or without orthonormality constraints) or projective (eight point, Sturm - Triggs 96, Oliensis - Hartley 07, globally optimal affine/metric upgrades from Chandraker 09 (with bug fixes to the original papers)), bundle adjustment ...
  * non-rigid 3D reconstruction: interface to the available Torresani 08, optimized Xiao - Kanade 04 (with bug fixes to the original papers), CSFM from Rabaud 09
  * visualization for rigid/non-rigid (sequences , multiple-views at once, comparisons ...) 

This toolbox provides several Structure from Motion algorithms I developed until 2009.

Quickstart
##########

Follow the install instructions and then just launch demoSfm

Introduction
############

Installation
************

This toolbox was developed with the following configurations:
  * Ubuntu 9.04 32 bits Matlab 2008b
  * Ubuntu 9.04 64 bits Matlab 2009b, Octave 3.2
  * Windows XP 32 bits, Matlab 2007b

But as I do not have those configurations anymore, I am maintaining this code on Ubuntu with the latest Octave.

Simply unzip, then add all directories to the Matlab path:

.. code-block:: matlab

  addpath(genpath('c:\toolboxSFM')); savepath;

Now, the mex files need to be created. Just run:

.. code-block:: matlab 

  toolboxSfmCompile()

Compiling on Windows
====================

I still provide SBA binaries for Windows 32 as it can be a pain to compile everything there. If you want to compile your own SBA libraries, or if you are on Windows 64 bits, launch matlab from the Visual Studio command line and then run:

.. code-block:: matlab  

  toolboxSfmCompile(true)

You might need to modify some Makefile though.

Compiling on Linux
==================

If that crashes, you don't have one of the following: make, gcc, g++, lapack or blas. For example, on Ubuntu, you need the following packages: build-essential, liblapack-dev, liblapack (or liblapack_pic on 64bits), liblapack3gf, libblas3gf, libblas-dev.

.. code-block:: matlab

  toolboxSfmCompile(true)

but you will need a PIC Blas (on 64-bits) and you will need to change the SBA Makefile too.

If you get some warnings about your gcc, just impose the version to use this way:

.. code-block:: matlab 

  toolboxSfmCompile(false, 4.2)

Dependencies
************

This toolbox depends on an excellent toolbox that I helped reorganize and improve a lot: Piotr's Image & Video Toolbox for Matlab ( http://vision.ucsd.edu/~pdollar/toolbox/doc/index.html).

For certain parts of the code, the following matlab toolboxes are also required:
  * Yalmip : http://users.isy.liu.se/johanl/yalmip/
  * GloptiPoly 3 : http://www.laas.fr/~henrion/software/gloptipoly3/
  * an SDP solver : I recommend SDPA (http://sdpa.indsys.chuo-u.ac.jp/sdpa/ but not on matlab 64-bit yet) or CSDP (https://projects.coin-or.org/Csdp/). You can also use Sedumi (http://sedumi.mcmaster.ca/) or whatever Yalmip supports.

For Octave, you need to install octave-optim

Misc
****


I tried to keep a very good and detailed documentation with references to the algorithms I implemented (most of them have a reference to HZ2 standing for the second edition of the Hartley and Zisserman : Multiple View Geometry) but if you have a question, suggestion, or if you find a bug, please email me.

If you use this toolbox in a paper, I would be grateful if you could cite it using the following !BibTex entry:

.. code-block:: latex

  @misc{vincentsSFMToolbox,  Author = {Vincent Rabaud}, Title = {Vincent's {S}tructure from {M}otion {T}oolbox}, howpublished = {\url{http://github.com/vrabaud/sfm_toolbox}}}


The Animation object
####################

An Animation object is the main kind of object passed between the different methods of the toolbox. It contains the 2D/3D positions of the different points (W,S,mask), the camera info (isProj,P,K,R,t), some info about the data (type,conn) and some info about NRSFM if needed (l,SBasis). It has the following members:

  * nPoint: the number of points.
  * nFrame: the number of frames in the sequence.
  * nBasis: the number of bases used in NRSFM (if applicable).
  * S: [ 3 x nPoint x nFrame ] matrix: the position of the 3D points (X,Y,Z).
  * W: [ 2 x nPoint x nFrame ] matrix: the projected 2D points (x,y).
  * mask: [ nPoint x nFrame ] boolean matrix: (i,j) contains true if the point i appears in the frame j.
  * isProj: boolean indicating if the camera is projective or not (in which case it is assumed to be affine).
  * P: [ 3 x 4 x nFrame ] matrix: the camera projection matrix for every frame
  * K: [ 3 x nFrame ] (affine) or [ 5 x nFrame ] (projective) matrix: contains the intrinsic camera parameters for every frame. The coefficients are stored as follows:

    * Affine
    
    +----+----+----+
    | K1 | K2 | K0 |
    +----+----+----+
    | 0  | K3 | 0  |
    +----+----+----+
    | 0  | 0  | 1  |
    +----+----+----+

    * Projective

    +----+----+----+
    | K1 | K2 | K4 |
    +----+----+----+
    | 0  | K3 | K5 |
    +----+----+----+
    | 0  | 0  | 1  |
    +----+----+----+

  *  KFull: the full version of the above [ 3 x 3 nFrame ] or [ 3 x 3 ]
  * R: [ 3 x 3 x nFrame ] matrix: the rotation matrix.
  * t: [ 3 x nFrame ] matrix: the translation matrix.
  * type: the type according to generateToyAnimation (-1 if none).
  * conn: cell of arrays indicating connectivities: each array is a list of the indices of the points forming a broken line. E.g. [1 2 3] means there will be a line from 1 to 2 and one from 2 to 3. This is only used for display purposes.
  * SBasis: [ nBasis x nPoint x nFrame ] matrix: the bases used in linear NRSFM
  * l: [ nBasis x nPoint x nFrame ] matrix: the coefficients used in linear NRSFM. If the matrix is [ (nBasis-1) x nPoint x nFrame ], the first basis will always have a coefficient of 1 (assumption of Torresani et al).
  * misc: whatever you want :)

The Animation member functions
******************************

The Animation object comes with a few very useful member functions. Their input/output is detailed in the functions themselves:
  * addNoise: add different kinds of noise to the Animation.
  * alignTo: aligns an Animation to another or to 2D data by applying a rigid transformation or homography.
  * centerW: centers the projection matrix W.
  * computeError: compute several errors (Bregler CVPR 2000, Torresani CVPR 2001, Torresani PAMI 2008).
  * generateCamFromRt: generate the positions of the optical center based on R and t in Animation.
  * generateKRt: generate K,R,t from the projection matrices.
  * generateP: generate the projection matrices from K,R,t.
  * generateSAbsolute: generate S transformed by R and t.
  * generateW: generate the projected point coordinates.
  * sampleFrame: sample an Animation object with respect to the number of frames.
  * setFirstRToId: set the first rotation/translation to id for an Animation.

The Animation rules
*******************

To simplify the use of an Animation object, certain members of an Animation are automatically updated when others are too.
  * nPoint, nBasis, nFrame are automatically filled/updated when certain matrices are filled/updated.
  * when any element in l or SBasis is modified, S is geenrated automatically and cannot be modified.
  * if R and t are not empty and filled/updated, P is generated automatically and cannot be modified.
  * if K or KFull is modified, so is the other one

This is very convenient but costly. I just believe that it is very little compared to some sfm/nrsfm computations.

Using it in SFM/NRSFM
*********************

Well, you can now look into the doc folder to see how to use the toolbox in 2d Geometry, SfM or NRSFM.
