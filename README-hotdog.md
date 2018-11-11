# Step By Step Instructions for Not Hotdog Demo

You should use Python 3.6 for this demo. Check your current Python version.

```
$ python -V
Python 2.7.15
```

We will setup a Python 3.6 environment with Conda. Check your `conda` version.

```
$ conda --version
zsh: command not found: conda
```

If Conda is not already installed, install for your OS (https://conda.io/docs/user-guide/install/index.html), then check your version again.

```
$ conda --version
conda 4.5.11
```

Setup a Conda environment with Python 3.6. Do NOT use a newer Python version.

```
$ conda create -n py36 python=3.6 anaconda
$ source activate py36
```

Check your current Python version. It should now show Python 3.6.

```
$ python -V
Python 3.6.6 :: Anaconda, Inc.
```

We can now walk through the well written "TensorFlow for Poets" (https://codelabs.developers.google.com/codelabs/tensorflow-for-poets/) guide. Be sure to read their guide for an explanation of each step.

Install TensorFlow 1.7. Do NOT install a newer TensorFlow version.

```
pip install --upgrade "tensorflow==1.7.*"
```

Clone the "TensorFlow for Poets" code repository

```
git clone https://github.com/googlecodelabs/tensorflow-for-poets-2
cd tensorflow-for-poets-2
```

Download the Flower Training Photos
```
curl http://download.tensorflow.org/example_images/flower_photos.tgz \
    | tar xz -C tf_files
```

Retrain Model with Flower Training Photos
```
IMAGE_SIZE=224
ARCHITECTURE="mobilenet_0.50_${IMAGE_SIZE}"
python -m scripts.retrain \
  --bottleneck_dir=tf_files/bottlenecks \
  --how_many_training_steps=500 \
  --model_dir=tf_files/models/ \
  --summaries_dir=tf_files/training_summaries/"${ARCHITECTURE}" \
  --output_graph=tf_files/retrained_graph.pb \
  --output_labels=tf_files/retrained_labels.txt \
  --architecture="${ARCHITECTURE}" \
  --image_dir=tf_files/flower_photos
```

Identify an Image with Retrained Model
```
python -m scripts.label_image \
    --graph=tf_files/retrained_graph.pb  \
    --image=tf_files/flower_photos/daisy/21652746_cc379e0eea_m.jpg
```

Now we will follow the excellent "TensorFlow For Poets Android App" guide (https://codelabs.developers.google.com/codelabs/tensorflow-for-poets-2-tflite/)

Install PILLOW
```
pip install PILLOW
```

Convert Retrained Flower Model to TensorFlow Lite (TFLITE) format
```
IMAGE_SIZE=224
toco \
  --input_file=tf_files/retrained_graph.pb \
  --output_file=tf_files/retrained_graph.lite \
  --input_format=TENSORFLOW_GRAPHDEF \
  --output_format=TFLITE \
  --input_shape=1,${IMAGE_SIZE},${IMAGE_SIZE},3 \
  --input_array=input \
  --output_array=final_result \
  --inference_type=FLOAT \
  --input_data_type=FLOAT
```

If you receive the following error on macOS or Linux, it is likely that the `toco` Python script is hard-coded to use a different Python version.

```
TOCO from pip install is currently not working on command line.
Please use the python TOCO API or use
bazel run tensorflow/contrib/lite:toco -- <args> from a TensorFlow source dir.
```

You can overcome this by locating the `toco` script using `which` on macOS or `whereis` on Linux and substituting the `toco` command with a python command that calls the script.

Locate script
```
$ which toco
/usr/local/bin/toco
```

Updated command
```
python /usr/local/bin/toco \
  --input_file=tf_files/retrained_graph.pb \
  --output_file=tf_files/retrained_graph.lite \
  --input_format=TENSORFLOW_GRAPHDEF \
  --output_format=TFLITE \
  --input_shape=1,${IMAGE_SIZE},${IMAGE_SIZE},3 \
  --input_array=input \
  --output_array=final_result \
  --inference_type=FLOAT \
  --input_data_type=FLOAT
```

Now follow the steps to run the app.

I ran the app without replacing the model files to make sure the built-in image classification was working. The app was identifying non-flower photos and showing non-flower categories.

Then I stopped the app, replaced the files, ran a Gradle sync, and started the app to test the flower classification model. The app was identifying flower categories only.

# Not Hotdog Demo

We are going to use a TensorFlow Lite model to create a "Hotdog or Not Hotdog" app for Android.

Myke Bates provided a 900MB download of photos separated into two folders named "hotdog" and "nothotdog". I downloaded that file, cleaned up problem photos in the "hotdog" folder, and moved the two folders into a "hotdog_photos" folder under "tf_files" (alongside the "flower_photos" folder).

Download the Hotdog Training Photos

```
cd tf_files
curl https://files.jrklein.com/2018/hotdog_photos.zip
unzip hotdog_photos.zip
```

I then repeated the retraining steps that we used for the "flower_photos" above, but instead specified the "hotdog_photos" folder. Here are the actual commands I used.

Retrain Hotdog Images. Note that I removed the "--how_many_training_steps" parameter so that the script would default to 4000 steps.
```
python -m scripts.retrain \
  --bottleneck_dir=tf_files/bottlenecks \
  --model_dir=tf_files/models/ \
  --summaries_dir=tf_files/training_summaries/"${ARCHITECTURE}" \
  --output_graph=tf_files/hotdog_graph.pb \
  --output_labels=tf_files/hotdog_labels.txt \
  --architecture="${ARCHITECTURE}" \
  --image_dir=tf_files/hotdog_photos
```

I converted the model to TensorFlow Lite format
```
IMAGE_SIZE=224
python /usr/local/bin/toco \
  --input_file=tf_files/hotdog_graph.pb \
  --output_file=tf_files/hotdog_graph.lite \
  --input_format=TENSORFLOW_GRAPHDEF \
  --output_format=TFLITE \
  --input_shape=1,${IMAGE_SIZE},${IMAGE_SIZE},3 \
  --input_array=input \
  --output_array=final_result \
  --inference_type=FLOAT \
  --input_data_type=FLOAT
```

I then copied the `tf_files/hotdog_labels.txt` and `tf_files/hotdog_graph.lite` files into the android `assets` folder, overwriting the existing `labels.txt` and `graph.lite` files.
