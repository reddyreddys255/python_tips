# Python Tips
A collection of python tips in my everyday coding

# Table of Contents
1. [Python](#python)
2. [Numpy](#numpy)
3. [Pandas](#pandas)
4. [Keras](#keras)
5. [Pytorch](#pytorch)
6. [Tensorflow](#tensorflow)
7. [Other](#other)

## Python

#### 1. parallel computing
I found that the joblib package is very good for a loop version of multiprocessing computing. For example, the following code is a simple for loop.
```python
from math import sqrt
[sqrt(i ** 2) for i in range(10)]
```
With joblib, we can do it in a multiprocess fashion.
```python
from math import sqrt
from joblib import Parallel, delayed
Parallel(n_jobs=2)(delayed(sqrt)(i ** 2) for i in range(10))
```
Just wrap the function name in the delayed function as delayed(function_name). The returned values will be a list and each element is the output of each loop iteration. For more complicated usage, check [here](https://pythonhosted.org/joblib/parallel.html)

#### 2. use pickle to save data
Pickle can be used to save any kind of data.
```python
import pickle

with open('train_test_id.pickle', 'wb') as f:
    pickle.dump(train_test_id, f) 
    
with open('train_test_id.pickle', 'rb') as f: 
    train_test_id = pickle.load(f) 
```

#### 3. Use h5py to save data
```python
import h5py
hdf5_file = h5py.File('test.h5', 'w')
hdf5_file.create_dataset('some_keys', data=some_numpy_array, dtype=np.uint8)
hdf5_file.close()
```

#### 4. Choose conda env in Jupyter Notebook
Need to install the following package in conda
```python
conda install nb_conda
```

#### 5. Python 2 and Python3 code compatibility
```
from __future__ import absolute_import, division, print_function
```

#### 6. Plot numpy data in grid
```python
## assume numpy_data has dimension: W, H, C
n_rows = 5
n_cols = 5
plt.figure(figsize=(n_cols*4, n_rows * 4))
for row in range(n_rows):
    for col in range(n_cols):
        index = n_cols * row + col
        if index >= numpy_data.shape[2]: break
        plt.subplot(n_rows, n_cols, index + 1)
        plt.imshow(numpy_data[:,:,index])
        plt.axis('off')
        plt.title('Fig %s' % (index + 1))
plt.show()
```

#### 7. Shutdown Jupyter Notebook
```python
## first list the running notebook
jupyter notebook list
## then shutdown the jupyter notebook with the specific port
jupyter notebook stop 8888
```

#### 8. Install Jupyter Notebook extensions
```python
pip install jupyter_contrib_nbextensions
```

## Numpy

#### 1. Save numpy array

```python
### without compression
np.save(file_name,array_name)
### with compression
np.savez_compressed(file_name, array_name1=array1, array_name2=array2)
### load saved array
np.load(file_name)
```



## Pandas

#### 1. groupby excludes NA

```python
df = pd.DataFrame({'col1':[1,2,np.NaN,1,np.NaN,2],'col2':[0,0,1,2,2,2]})
df.GroupBy(['col1']).size()
```

It will exclude the NA in col1. We can fill the Nan with string "NA".

```python
df = pd.DataFrame({'col1':[1,2,np.NaN,1,np.NaN,2],'col2':[0,0,1,2,2,2]})
df.fillna('NA').GroupBy(['col1']).size()
```
#### 2. rename columns

We can rename the entire column names by 
```python
df.columns = ['a', 'b']
```

Or we can rename specific columns by
```python
df.rename(columns = {'a':'b'}, inplace = True)
```

#### 3. select data by dates
Make sure the date variable is in datetime format
```python
df['date_var'] = pd.to_datetime(df['date_var'])  
mask = (df['date_var'] >= '2006-01-01') & (df['date_var'] <= '2006-12-30')
df.loc[mask]
```
#### 4. show all columns
Change the pandas settings to show all columns.
```
import pandas as pd
pd.set_option('display.max_columns', 100)
pd.set_option('display.width', 500)

pd.set_option('display.max_rows', 100)
pd.set_option('display.height', 500)
```

## Keras

#### 1\. Check GPU 
Sometimes, we want to know if the GPU has been recognized. The following code assume Tensorflow as the backend.
```python
from tensorflow.python.client import device_lib
print(device_lib.list_local_devices())
```
#### 2\. Plot the fitting curve
```python
## save the fitting history
history = model.fit(...)

## list all variables in history
print(history.history.keys())

## plot accuracy
plt.plot(history.history['acc'])
plt.plot(history.history['val_acc'])
plt.title('model accuracy')
plt.ylabel('accuracy')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()

## plot loss
plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.title('model loss')
plt.ylabel('loss')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()
```

#### 3\. Preprocess images for pre-trained models
When use the pre-trained models in Keras, the images need to be preprocessed differently. Keras provides different preprocess functions for different pre-trained models. Just load the preprocess_input function from the pre-trained model that you want to use. For example, for inception_v3
```python
from keras.applications.inception_v3 import InceptionV3,preprocess_input
x = preprocess_input(x)
```
For VGG16, load the preprocess_input from keras.applications.vgg16 instead
```python
from keras.applications.vgg16 import VGG16,preprocess_input
x = preprocess_input(x)
```

The keras pre-trained models can be found [here](https://keras.io/applications/)


## Pytorch

### 1. Use multiple GPUs
```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = ... # define your model here
if torch.cuda.device_count() > 1:
    print('--'*10)
    print("Use", torch.cuda.device_count(), "GPUs")
    print('--' * 10)
    model = nn.DataParallel(model)
model.to(device)
```

### 2. Define the Dataset for dataloader
```python
class ImageDataset(Dataset):
    def __init__(self, some_parameters):
        super(ImageDataset, self).__init__()     

    def __len__(self):
        return len_of_the_dataset

    def __getitem__(self, index):   
```


### 3. Change channel order
Pytorch uses BCWH (batch, channel, width, height) instead of BWHC, which is different from Tensorflow and Keras. Use the permute function to change the channel last order to channel first order.
```python
image.permute(0, 3, 1, 2)
```

### 4. Save and load Pytorch model
The following code will load and save the whole model
```python
torch.save(model, 'model.pt')
model = torch.load('model.pt')
```
The following code will only load and save the weights
```python
torch.save(model.state_dict(), file_path)
model.load_state_dict(torch.load(file_path))
```

### 5. Use tensorboard with Pytorch
Use [tensorboard-pytorch](https://github.com/lanpa/tensorboard-pytorch), first install tensorboardX
```
pip install tensorboardX
```
You may also need to install tensorflow and tensorboard. In the Pytorch code
```python
import torch
from tensorboardX import SummaryWriter
writer = SummaryWriter()
for iter in range(100):
    ## train your model
    saved_images = torchvision.utils.make_grid(train_image, nrow=6)
    writer.add_image('Image', saved_images, iter)
    ## be carefull, not feed the Pytorch tensor
    writer.add_scalar('loss', loss, iter)
    writer.add_scalar('acc', acc, iter)
    ## if you want to show train and test loss in the same figure
    writer.add_scalars('loss', {'train': train_loss,
                                'test': test_loss),
                                }, iter)
    ## check the learned weights
    for name, param in model.named_parameters():
        writer.add_histogram(name, param.clone().cpu().data.numpy(), iter)
writer.off()
```
Here is [an exmaple](https://github.com/lanpa/tensorboard-pytorch) of other outputs you can visualize with tensorboard. The outputs will be written into the runs folder.

Then launch the tensorboard 
```
tensorboard --logdir runs
```
Go to localhost:6006


### 6. Change learning rate during training
```python
from torch.optim.lr_scheduler import ReduceLROnPlateau
optimizer = torch.optim.SGD(model.parameters(), lr=0.1, momentum=0.9)
scheduler = ReduceLROnPlateau(optim, 'min', factor=0.2, patience=5, verbose=True)
for epoch in range(10):
    ## train your model
    ## calclulate val_loss
    scheduler.step(val_loss)
```
Check [this page](https://pytorch.org/docs/stable/optim.html) for details

### 7. Print model details (layer name, number of parameters, trainable)
```python
for name, para in model.named_parameters():
    print(name, para.numel(), para.requires_grad)
```

### 8. Match weight names
A great headache for me when using Pytorch is sometimes the weight name mismatch. This occurs when I train a model with multi-GPUs and use the trained model with single GPU. Somehow the weight names are different with multi-GPU and single GPU. I happen to run into the densenet source code and find the following code.
```python
def densenet121(pretrained=False, **kwargs):
    r"""Densenet-121 model from
    `"Densely Connected Convolutional Networks" <https://arxiv.org/pdf/1608.06993.pdf>`_

    Args:
        pretrained (bool): If True, returns a model pre-trained on ImageNet
    """
    model = DenseNet(num_init_features=64, growth_rate=32, block_config=(6, 12, 24, 16),
                     **kwargs)
    if pretrained:
        # '.'s are no longer allowed in module names, but pervious _DenseLayer
        # has keys 'norm.1', 'relu.1', 'conv.1', 'norm.2', 'relu.2', 'conv.2'.
        # They are also in the checkpoints in model_urls. This pattern is used
        # to find such keys.
        pattern = re.compile(
            r'^(.*denselayer\d+\.(?:norm|relu|conv))\.((?:[12])\.(?:weight|bias|running_mean|running_var))$')
        state_dict = model_zoo.load_url(model_urls['densenet121'])
        for key in list(state_dict.keys()):
            res = pattern.match(key)
            if res:
                new_key = res.group(1) + res.group(2)
                state_dict[new_key] = state_dict[key]
                del state_dict[key]
        model.load_state_dict(state_dict)
    return model
```

## Tensorflow

### 1. Unit test in tensorflow
```python
import tensorflow as tf

class EvalUtilTest(tf.test.TestCase):

  def test_function(self):
    some_code_here
    
if __name__ == '__main__':
  tf.test.main()
```

### 2. Set the verbosity level
```python
import tensorflow as tf

tf.logging.set_verbosity(tf.logging.DEBUG)
#tf.logging.set_verbosity(tf.logging.ERROR) 
#tf.logging.set_verbosity(tf.logging.INFO)
#tf.logging.set_verbosity(tf.logging.FATAL)
#tf.logging.set_verbosity(tf.logging.WARN)
```

### 3. Use tensorboard in notebook
```
%load_ext tensorboard.notebook
```

### 4. Install cuda and cudnn by conda
```python
## if you get the following error, try install cuda and cudnn
## ImportError: libcublas.so.10.0: cannot open shared object file: No such file or directory
conda install cudatoolkit
conda install cudnn
```

## Other

### 1. Use R in Jupyter Notebook
Install some R packages in R console first.
```R
install.packages(c('repr', 'IRdisplay', 'evaluate', 'crayon', 'pbdZMQ', 'devtools', 'uuid', 'digest'))
devtools::install_github('IRkernel/IRkernel')
IRkernel::installspec()
### if you want to install other R packages 
### make sure to add the new package to the anaconda R library path
##install.packages("newpackage", "/home/user/anaconda3/lib/R/library")
```

Use conda to install some necessary R packages in your Python env.
```
### if you want to run R in Jupyter Notebook
conda install -c r r-essentials
### if you want to run both R and Python in the same notebook
conda install -c r rpy2
```

Run both R and Python code in the same notebook. Note that the Python code and R code need to be in separated cells.
```python
%load_ext rpy2.ipython
```
```python
%R require(ggplot2)
```
```python
import pandas as pd 
df = pd.DataFrame({'group': ['a', 'b', 'c', 'd','e', 'f', 'g', 'h','i'],
                   'A': [1, 2, 5, 5, 1, 6, 7, 5, 9],
                   'B': [0, 2, 3, 6, 7, 6, 5, 9, 13],
                   'C': [3, 2, 3, 1, 3, 3, 4, 2, 1]})
```
```python
%%R -i df
ggplot(data=df) + geom_point(aes(x=A, y=B, color=C))
```

I found that there are also magic functions can run R code within notebook. Check [this link](https://blog.dominodatalab.com/lesser-known-ways-of-using-notebooks/)

### 2. Profile code
I found that the %prun and %lprun functions in the Jupyter notebook are quite usefel to profile the code and make it more efficient. First, install the follow package
```python
pip install line-profiler
```
Then, in the notebook, load the package
```python
import line_profiler
%load_ext line_profiler
```
Assume we want to profile the function called function_to_be_profile
```python
%lprun -f function_to_be_profile function_to_be_profile(para)
```
The above code will generate a line-by-line profiling result. You can find which part of your code is the bottleneck.

### 3. Create env
```
conda create -n env_name python=3.6
```

### 4. Use SSH tunneling for Jupyter Notebook
```
ssh -f -N -L 1234:localhost:8788 user@server.com
```
1234 is the local port and 8788 is the remote port used by Jupyter Notebook. Now you can connect to Jupyter Notebook by `localhost:1234` in the local browser


### 5. Add SSH public keys to the remote server
```
ssh-copy-id -i ~/.ssh/id_rsa user@remote_host
```

### 6. Git add manually deleted files
```
git add -u .
```

### 7. Reload bashrc without logging out
```
source ~/.bashrc
```

### 8. unrar with wildcard
Somehow the unrar does not support wildcard. The following solution works for me.
```
unrar x '*.rar' 
```
