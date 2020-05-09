# **YOLO object detection on  Ubuntu 16.04/18.04** 

## **1. Installation**
Download darknet-master Github repository from [https://github.com/AlexeyAB/darknet.git](https://github.com/AlexeyAB/darknet.git)

**Compile on Linux (using** **cmake**)
The CMakeLists.txt will attempt to find installed optional dependencies like CUDA, cudnn, ZED and build against those. It will also create a shared object library file to use darknet for code development.

Do inside the cloned repository:

    mkdir build-release
    cd build-release
    cmake ..
    make
    make install

Just do `make` in the darknet directory. Before make, you can set such options in the `Makefile`
GPU=1
CUDNN=1
CUDNN_HALF=0
OPENCV=1


**Modify Some Code**

Look at darknet/examples/detector.c line 138  
`if(i%10000==0 || (i < 1000 && i%100 == 0)){`

There is one change that we should make to /examples/detector.c before training. When the network is training, it will save the weights to "backup" every 100 iterations until 900. After 900 iterations, the default setting is to save every 10,000 iterations. We would like to save more often than that with this small dataset. To change this setting, change the following line in examples/detector.c  
`if(i%10000==0 || (i < 1000 && i%100 == 0)){  `
to  
`if(i%100==0 || (i < 1000 && i%100 == 0)){`

run **make** to re-compile.

## **2. Image Annotation**
Label your data in Darknet format use image annotation tool from [https://github.com/tzutalin/labelImg.git](https://github.com/tzutalin/labelImg.git)

**Install LabelImg**  **FOR LINUX:**

Install LabelImg via PIP

     pip3 install labelImg

Launch LabelImg via the command below

    labelImg

**Step (YOLO)**

1.  Click 'Change default saved annotation folder' in Menu/File
2.  Click 'Open Dir'
3.  click "PascalVOC" button to switch to YOLO format.
4.  Click 'Create RectBox'
5.  Click and release left mouse to select a region to annotate the rect box
6.  You can use right mouse to drag the rect box to copy or move it

> *Save image files and label files in directory /build/darknet/x64/data/obj/*

## **3. Download Pre-trained model**

You can download Pre-trained model from [https://pjreddie.com/darknet/imagenet/](https://pjreddie.com/darknet/imagenet/)

## **4. Create train and test files**
Create allImg.txt that each row contains a path to an image (.jpg) in folder and save file as `build/darknet/x64/data` 
![image](https://user-images.githubusercontent.com/59696434/81464664-cc5ac980-91ed-11ea-9f5d-f339ceba4ff1.png)

 - Split datasets into test and train by using
$ `python split.py`

It generates text.txt and train.txt (with 80% / 20% of training and validation data respectively)

> *You can change the ratio by change `num_train = 0.8` in line 18 (file `split.py`) to another value.*

## **5. Create new** ***.names** **file**
*.names file contains the names for the categories we want to detect. The line number should match the category number in the .txt label files we created  earlier

![image](https://user-images.githubusercontent.com/59696434/81464670-d2e94100-91ed-11ea-9c27-eb2e16e2473f.png)

## **6. Create new** ***.data** **file**
*Create new *.data file save file as build/darknet/x64/data*


 - classes  needs the number of classes

- train and valid needs absolute paths of the file train.txt and test.txt generated earlier

- names needs path to your *.names file.

 - backup is where we can store the weights files as the training progresses.
 ![image](https://user-images.githubusercontent.com/59696434/81464676-e0063000-91ed-11ea-95da-d3ad50ec97e7.png)

## **7. Edit yolo-obj.cfg file**

  -  Create file yolo-obj.cfg with the same content as in yolov3-voc.cfg (or copy yolov3-voc.cfg to yolo-obj.cfg)
-   change line batch to `batch=64`
-   change line subdivisions to `subdivisions=16`
-   change line max_batches to (`classes*2000` but not less than number of training images, and not less than `6000`), f.e. `max_batches=6000` if you train for 3 classes
-   change line steps to 80% and 90% of max_batches, f.e. `steps=4800,5400`
-   set network size `width=416 height=416` or any value multiple of 32  _line 8,9_
-   change line `classes=80` to your number of objects in each of 3 `[yolo]`-layers:  line 610,696,783
-   change [`filters=255`] in line  603, 689, 776 to filters=(classes + 5)x3 in the 3 `[convolutional]` before each `[yolo]` layer, keep in mind that it only has to be the last `[convolutional]` before each of the `[yolo]` layers.
-   when using `[Gaussian_yolo]` layers, change [`filters=57`] filters=(classes + 9)x3 in the 3 `[convolutional]` before each `[Gaussian_yolo]` layer
<![endif]-->

So if `classes=1` then should be `filters=18`. If `classes=2` then write `filters=21`.

> **Do not write in the cfg-file: filters=(classes + 5)x3**

## **8. Train**


Start training:  

    ./darknet detector train <path to .data> <path to .cfg> <path to .weights> -map

**Example**

    ./darknet detector train build/darknet/x64/data/obj.data build/darknet/x64/data/ yolov3-voc.cfg .cfg darknet53.conv.74 -map
