# food-not-food-app
Machine Learning powered app to determine whether an image contains food or not.

## Steps to reproduce

To reproduce, the following steps are best run in order.

You will require and installation of Conda, I'd recommend [Miniconda](https://docs.conda.io/en/latest/miniconda.html).

### Clone the repo

```
git clone https://github.com/mrdbourke/food-not-food
cd food-not-food
```

### Environment creation

I use Conda for my environments. You could do similar with [`venv`](https://docs.python.org/3/library/venv.html) and `pip` but I prefer Conda.

This code works with Python 3.8.

```
conda create --prefix ./env python=3.8 -y
conda activate ./env
conda install pip
``` 

### Installing requirements

**Getting TensorFlow + GPU to work**

Follow the install instructions for running [TensorFlow on the GPU](https://www.tensorflow.org/install/gpu).

This will be required for `model_building/train_model.py`.

**Note:** Another option here to skip the installation of TensorFlow is to use your global installation of TensorFlow and just install the `requirements.txt` file below.

**Other requirements**

If you're using your global installation of TensorFlow, you might be able to just run `pip install requirements.txt` in your environment.

Or if you're running in another dedicated environment, you should also be able to just run `pip install -r requirements.txt`.

```
pip install -r requirements.txt
```

### Getting the data

1. Download Food101 data (101,000 images of food).

```
python data_download/download_food101.py
```

2. Download a subset of Open Images data. Use the `-n` flag to indicate how many images from each set (train/valid/test) to randomly download.

For example, running `python data_download/download_open_images.py -n=100` downloads 100 images from the training, validation and test sets of Open Images (300 images in total).

The downloading for Open Images data is powered by [FiftyOne](https://voxel51.com/docs/fiftyone/).

```
python data_download/download_open_images.py -n=100
```

### Processing the data

1. Extract the Food101 data into a "`food`" directory, use the `-n` flag to set how many images of food to extract, for example `-n=10000` extracts 10,000 random food images from Food101.

```
python data_processing/extract_food101.py -n=10000
```

2. Extract the Open Images images into `open_images_extracted` directory. 

The `data_processing/extract_open_images.py` script uses the Open Images labels plus a list of foods and not foods (see `data/food_list.txt` and `data/non_food_list.txt`) to separate the downloaded Open Images.

This is necessary because some of the images from Open Images contain foods (we don't want these in our `not_food` class).

```
python data_processing/extract_open_images.py
```

3. Move the extracted images into "`food`" and "`not_food`" directories.

This is necessary because our model training file will be searching for class names by the title of our directories (`food` and `not_food`).

```
python data_processing/move_images.py 
```

4. Split the data into training and test sets.

This creates a training and test split of `food` and `not_food` images.

This is so we can verify the performance of our model before deploying it.

It'll create the structure:

```
train/
    food/
        image1.jpeg
        image2.jpeg
        ...
    not_food/
        image100.jpeg
        image101.jpeg
        ...
test/
    food/
        image201.jpeg
        image202.jpeg
        ...
    not_food/
        image301.jpeg
        image302.jpeg
        ...
```

To do this, run:

```
python data_processing/data_splitting.py
```

### Modeling the data

**Note:** This will require a working install of TensorFlow.

Running the model training file will produce a TensorFlow Lite model (this is small enough to be deployed in a browser) saved to the `models` directory.

The script will look for the `train` and `test` directories and will create training and testing datasets on each respectively.

It'll print out the progress at each epoch and then evaluate and save the model.

```
python model_building/train_model.py
```

## What data is used?

The current deployed model uses about 40,000 images of food and 25,000 images of not food.

* Food images come from the [Food101 dataset](https://data.vision.ee.ethz.ch/cvl/datasets_extra/food-101/).
* Not food and *some* food images come from [Open Images](https://storage.googleapis.com/openimages/web/index.html).
