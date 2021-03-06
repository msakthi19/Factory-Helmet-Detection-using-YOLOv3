# Factory Helmet Detection using YOLO3 (Detection, Training, and Evaluation)

## Required Files:

Grab the pretrained weights of YOLOv3 from:  https://drive.google.com/open?id=1tqh3UMkL2q1aOmoofFNXqa1CBcRpB70n

Download pretrained weights for backend at:  https://drive.google.com/file/d/1m94x13Sk1ajaVszPi1s5CJoF1V_DYMWR/view?usp=sharing

**This weights must be put in the root folder of the repository. They are the pretrained weights for the backend only and will be loaded during model creation. The code does not work without this weights.**

## Training

### 1. Data preparation 

Organize the dataset into 4 folders:

+ train_image_folder <= the folder that contains the train images.

+ train_annot_folder <= the folder that contains the train annotations in VOC format.

+ valid_image_folder <= the folder that contains the validation images.

+ valid_annot_folder <= the folder that contains the validation annotations in VOC format.
    
There is a one-to-one correspondence by file name between images and annotations. If the validation set is empty, the training set will be automatically splitted into the training set and validation set using the ratio of 0.8.

### 2. Edit the configuration file
The configuration file is a json file, which looks like this:

```python
{
    "model" : {
        "min_input_size":       352,
        "max_input_size":       448,
        "anchors":              [10,13,  16,30,  33,23,  30,61,  62,45,  59,119,  116,90,  156,198,  373,326],
        "labels":               ["helmets", "person_without_helmet", "person_with_helmet"]
    },

    "train": {
        "train_image_folder":   "/home/data/train_image_folder/",
        "train_annot_folder":   "/home/data/train_annot_folder/",      
          
        "train_times":          10,             # the number of time to cycle through the training set, useful for small datasets
        "pretrained_weights":   "",             # specify the path of the pretrained weights, but it's fine to start from scratch
        "batch_size":           16,             # the number of images to read in each batch
        "learning_rate":        1e-4,           # the base learning rate of the default Adam rate scheduler
        "nb_epoch":             50,             # number of epoches
        "warmup_epochs":        3,              # the number of initial epochs during which the sizes of the 5 boxes in each cell is forced to match the sizes of the 5 anchors, this trick seems to improve precision emperically
        "ignore_thresh":        0.5,
        "gpus":                 "0,1",

        "saved_weights_name":   "helmets.h5",
        "debug":                true            # turn on/off the line that prints current confidence, position, size, class losses and recall
    },

    "valid": {
        "valid_image_folder":   "/home/data/valid_image_folder/",
        "valid_annot_folder":   "/home/data/valid_annot_folder/",

        "valid_times":          1
    }
}

```

The ```labels``` setting lists the labels to be trained on. Only images, which has labels being listed, are fed to the network. The rest images are simply ignored. 

### 3. Generate anchors for your dataset (optional)

`python gen_anchors.py -c config.json`

Copy the generated anchors printed on the terminal to the ```anchors``` setting in ```config.json```.

### 4. Start the training process

`python train.py -c config.json`

By the end of this process, the code will write the weights of the best model to file helmets.h5 (or whatever name specified in the setting "saved_weights_name" in the config.json file). The training process stops when the loss on the validation set is not improved in 3 consecutive epoches.

or download a trained model to directly use for detection from : https://drive.google.com/open?id=1hAjLC-zCTmx6D8Bbahw3vtYExAtfuNWA

### 5. Perform detection using trained weights on image, set of images, video
`python predict.py -c config.json -i /path/to/image/or/video`

It carries out detection on the image and write the image with detected bounding boxes to the same folder.

Perform Real-time detection using Webcam

`python predict.py -c config.json -i webcam`

## Evaluation

`python evaluate.py -c config.json`

Compute the mAP performance of the model defined in `saved_weights_name` on the validation dataset defined in `valid_image_folder` and `valid_annot_folder`.
