# two-stream-action-recognition-ucf101
We use a spatial and motion stream cnn with ResNet101 as baseline for modeling video information in UCF101 dataset.
## Reference Paper
*  [[1] Two-stream convolutional networks for action recognition in videos](http://papers.nips.cc/paper/5353-two-stream-convolutional)
*  [[2] Temporal Segment Networks: Towards Good Practices for Deep Action Recognition](https://link.springer.com/chapter/10.1007/978-3-319-46484-8_2)
* [[3] TS-LSTM and Temporal-Inception: Exploiting Spatiotemporal Dynamics for Activity Recognition](https://arxiv.org/abs/1703.10667)

## 1. Data Preprocessing
  ### 1.1 Spatial input data -> rgb frames
  * We extract RGB frames from each video in UCF101 dataset with sampling rate: 10 and save as .jpg image in disk which cost about 5.9G.
  ### 1.2 Motion input data -> stacked optical flow images
  In motion stream, we use two methods to get desire optical flow data. 
  1. Download the preprocessed tvl1 optical flow dataset directly from https://github.com/feichtenhofer/twostreamfusion. 
  2. Using [flownet2.0 method](https://github.com/lmb-freiburg/flownet2-docker) to generate 2-channel optical flow image and save its x, y channel as .jpg image in disk respectively which cost about 56G. 

## 2. Model
  ### 2.1 Spatial cnn
  * As mention before, we use ResNet101 first pre-trained with ImageNet then fine-tuning on our UCF101 spatial rgb image dataset. 
  ### 2.2 Motion cnn
  * Input data of motion cnn is a stack of optical flow images which contained 10 x-channel and 10 y-channel images, So it's input shape is (20, 224, 224) which can be considered as a 20-channel image. 
  * In order to utilize ImageNet pre-trained weight on our model, we have to modify the weights of the first convolution layer pre-trained  with ImageNet from (64, 3, 7, 7) to (64, 20, 7, 7). 
  * In [2] Wang provide a method called **Cross modality pre-training** to do such weights shape transform. He first average the weight value across the RGB channels and replicate this average by the channel number of motion stream input( which is 20 is this case)
  
## 3. Training stategies
  ###  3.1 Spatial cnn
  * In every mini-batch, we randomly select 64 frames from 186351 training frames.
  ### 3.2 Motion cnn
  * In every mini-batch, we randomly select 64 (batch size) videos from 9537 training videos and futher randomly select 1 stacked optical flow in each video. 
  ### 3.3 Data augmentation
  * Both stream apply the same data augmentation technique such as random cropping.
## 4. Testing method
  * For every 3783 testing videos, we uniformly sample 25 frames in each video and the video level prediction is the voting result of all 25 frame level predictions.
## 5. Performace
   
 network      | top1  | top5 |
--------------|:-----:|:----:| 
Spatial cnn   | 82.4% | 94.8% | 
Motion cnn    | 78.2% | 94.4% | 
Average fusion| 89.9% | 98.3% |     
   
## 6. Result
<img src="https://github.com/jeffreyhuang1/pytorch-two-stream-cnn-ucf101/blob/master/result/spatial_cnn.png" width = "400" height = "250" alt="Spatial" align=left />
<img src="https://github.com/jeffreyhuang1/pytorch-two-stream-cnn-ucf101/blob/master/result/motion_cnn.png" width = "400" height = "250" alt="Motion" align=left />

