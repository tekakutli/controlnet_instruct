# controlnet_instruct
disclaimer: I have no idea if this would work  
I myself am unable of following through  

**Following these steps would get you the same dataset with which instruct-pix2pix was trained, format it so that you can train a controlnet, and train it, in the laziest way possible**  

First set the data-location variable, where you will download the dataset
```
data="/tmp/data/"
```
Get the dataset, you can visit the site http://instruct-pix2pix.eecs.berkeley.edu/clip-filtered-dataset
```
cd $data
wget -A zip,json -R "index.html*" -q --show-progress -r --no-parent http://instruct-pix2pix.eecs.berkeley.edu/clip-filtered-dataset -nd -P $data
unzip *.zip .
```

Check if you succedeed, before deleting the zips  
prepare_dataset will create the prompt.json that controlnet uses
```
rm *.zip
cd controlnet_instruct
python prepare_dataset.py $data
```
Get controlnet code
```
git clone https://github.com/lllyasviel/ControlNet
```
Create a new controlnet ckpt
```
cd ControlNet
python tool_add_control.py <where_is_it>/v1-5-pruned.ckpt ./models/control_sd15_ini.ckpt
```
you can either edit the file **tutorial_dataset.py** in the ControlNet repository, changing the three ./training/fill50k/ with the dataset-directory($data)  
**OR** just symlink this way:
```
controlnet_dir=<where_is_ControlNet>

cd controlnet_dir
mkdir -p ./training
ln -s $data ./training/fill50k/
```
# NOTES
Maybe the size of the instruct-pix2pix dataset warrants having **sd_locked = False** at **tutorial_train.py**  
more info at https://github.com/lllyasviel/ControlNet/blob/main/docs/train.md  

There's also the issue of no longer having a CFG-image scale, what I was planning to do was to expand the dataset and train the controlnet with the empty prompt as the identity prompt:
```
{"source": "image-1.jpg", "target": "image-1.jpg", "prompt": ""}
```
notice it's the same image
# Use it
copy gradio_ip2p.py into the ControlNet repository folder
```
cd controlnet_instruct

controlnet_dir=<where_is_ControlNet>
cp gradio_ip2p.py $controlnet_dir
```
I didn't use conda, I only had one issue, something about "block = gr.Blocks().queue()" not being defined
I fixed that dependency with:
```
pip install gradio==3.16.2
```
use with
```
cd $controlnet_dir
python gradio_ip2p.py
```

