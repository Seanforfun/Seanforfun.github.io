---
layout: page
title: Deep Learning
permalink: /DeepLearning/
---
## GMAN(Jan, 2018 - June, 2018, Dec, 2018)
### Introduction
GMAN is a awesome Convolutional neural network purposed on haze removal. It is a completely end-to-end dehaze system so the input to the system is hazed rgb images and the output of the system is the clear images that processed by the system. The results can be found in the [Results](https://github.com/Seanforfun/GMAN_Net_Haze_Removal/tree/master/Results) column, where lists our evaluation results on SOTS evaluation dataset. There lists 500 images for indoor test and 500 images for outdoor test saparately, in each file, we can also find a log.txt file showing the psnr and ssim for each images.

| Dateset | PSNR(dB) | SSIM | Model Path | Link |
| :-: | :-: | :-: |  :-: | :-: |
| SOTS outdoor | 28.474217 | 0.944434 | [click me](https://github.com/Seanforfun/GMAN_Net_Haze_Removal/tree/master/Results/models/outdoor) | [click me](https://github.com/Seanforfun/GMAN_Net_Haze_Removal/tree/master/Results/Outdoor) |
| SOTS indoor | 27.934801 | 0.896512 | [click me](https://github.com/Seanforfun/GMAN_Net_Haze_Removal/tree/master/Results/models/indoor) | [click me](https://github.com/Seanforfun/GMAN_Net_Haze_Removal/tree/master/Results/Indoor) |

![Imgur](https://i.imgur.com/HfPpj6Q.png)

## How to use?
1. Step 1:
    * You need to install all packages required by the program, such as tensorflow(GPU version), numpy etc.
    * You need to download current repository into your local environment and extract it.
2. Step 2:
    * Go to your local file and enter the setup.sh file. Change the line 14, "PROGRAM_ROOT_PATH=''" to the path you want.
    * Run ```sh setup.sh```, it will automatically create the directories for running the program

### Evaluate your own image
3. Step 3:
    * Move the hazed images to (PROGRAM_ROOT_PATH)/HazeImages/TestImages.
    * If you have ground truth images to compare with the pictures processed by the net, move them to (PROGRAM_ROOT_PATH)/ClearImages/TestImages.
4. Step 4:
    * Copy the model from (DOWNLOAD FILE)/Results/models/(outdoor or indoor) to (PROGRAM_ROOT_PATH)/DeHazeNetModel
5. Step 5:
    * If you want to compare the result with groundtruth, you just need to run the program by using ```python3 gman_eval.py```.
    * If you don't have ground truth, you can can run the program by using ```python3 gman_eval.py --eval_only_haze=True```.
6. Step 6:
    * You results are in (PROGRAM_ROOT_PATH)/ClearResultImages.

## Demonstration
Only listed several examples, more results can be found in my [github](https://github.com/Seanforfun/GMAN_Net_Haze_Removal/tree/master/Results).
#### Outdoor
<table>
	<tr>
		<th>Hazy</th>
		<th>Groundtruth</th>
		<th>Our result</th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/8S6cpRe.jpg"/></th>		
		<th><img src="https://i.imgur.com/fUhQuld.png"/></th>
		<th><img src="https://i.imgur.com/jOLhygU.jpg"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/deNcv4O.jpg"/></th>		
		<th><img src="https://i.imgur.com/L66qRbw.png"/></th>
		<th><img src="https://i.imgur.com/miDdMgk.jpg"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/rFTGcVD.jpg"/></th>		
		<th><img src="https://i.imgur.com/aSmMOJE.png"/></th>
		<th><img src="https://i.imgur.com/r1YPHym.jpg"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/iBE5sGw.jpg"/></th>		
		<th><img src="https://i.imgur.com/u6HY6qE.png"/></th>
		<th><img src="https://i.imgur.com/x2Uu3Tc.jpg"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/cVtaJnm.jpg"/></th>		
		<th><img src="https://i.imgur.com/4QKZdHa.png"/></th>
		<th><img src="https://i.imgur.com/wQ6SmiQ.jpg"/></th>
	</tr>
</table>

#### Indoor
<table>
	<tr>
		<th>Hazy</th>
		<th>Groundtruth</th>
		<th>Our result</th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/81MUWBh.png"/></th>		
		<th><img src="https://i.imgur.com/bsqSWNC.png"/></th>
		<th><img src="https://i.imgur.com/pBhsVG8.jpg"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/UrDTN2G.png"/></th>		
		<th><img src="https://i.imgur.com/75yuyRw.png"/></th>
		<th><img src="https://i.imgur.com/Y7TbUOR.jpg"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/rx5jrpd.png"/></th>		
		<th><img src="https://i.imgur.com/7cPB8Wg.png"/></th>
		<th><img src="https://i.imgur.com/fFIwaMG.jpg"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/9bWE6zj.png"/></th>		
		<th><img src="https://i.imgur.com/fbAWMTg.png"/></th>
		<th><img src="https://i.imgur.com/r6GiyXj.jpg"/></th>
	</tr>
</table>

### Citation
Please cite the paper if you are using this project.

```
@article{liu2019single,
  title={Single Image Dehazing with a Generic Model-Agnostic Convolutional Neural Network},
  author={Liu, Zheng and Xiao, Botao and Alrabeiah, Muhammad and Wang, Keyan and Chen, Jun},
  journal={IEEE Signal Processing Letters},
  volume={26},
  number={6},
  pages={833--837},
  year={2019},
  publisher={IEEE}
}
```


## Car oriented Deep image matting (July, 2018 - Nov, 2018)
### Introduction
Car oriented Deep image matting is a project I worked together with CarMedia2.0 company in Burlington. The purpose of the project is we can create a matting image using the original rgb images. The projects contains two parts, one is using MaskRcnn neural network to locate the position of car in the image, the second step is create a image matting by which we can extract the car from the original RGB image.
![Imgur](https://i.imgur.com/1h8wd0M.png)

* Step 1: [MASK-RCNN](https://arxiv.org/pdf/1703.06870.pdf) from Facebook AI group
![Imgur](https://i.imgur.com/4hOnfrJ.png)

* Step 2: [Deep Image Matting](https://arxiv.org/pdf/1703.03872.pdf) from Adobe research group
![Imgur](https://i.imgur.com/lPmd2QJ.png)

Since the codes are now the property of the company, I cannot ditribute them right now and I will show some demos. Improvement is easily to be achieved by some future work, however my research must be stopped by some reasons.

### Demonstration
<table>
	<tr>
		<th>RGB</th>
		<th>Result</th>
		<th>Synthetic images</th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/NkjvE1R.png"/></th>		
		<th><img src="https://i.imgur.com/AsIfjDY.png"/></th>
		<th><img src="https://i.imgur.com/Gt0g1BY.png"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/6pwUJ7e.png"/></th>		
		<th><img src="https://i.imgur.com/1c80v6C.png"/></th>
		<th><img src="https://i.imgur.com/eYHD8iu.png"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/o8dZuh7.png"/></th>		
		<th><img src="https://i.imgur.com/SJb4oTI.png"/></th>
		<th><img src="https://i.imgur.com/cNaFwB5.png"/></th>
	</tr>
	<tr>
		<th><img src="https://i.imgur.com/Hjgy4Ey.png"/></th>		
		<th><img src="https://i.imgur.com/4ZkOAdB.png"/></th>
		<th><img src="https://i.imgur.com/Vp0Je1W.png"/></th>
	</tr>
</table>

### Reference
1. [Car media 2.0](http://www.carpics2p0.com/)

## Distributed Tensorflow Framework(Dec, 2018 - Now)
This is a distributed Framework used for distributed training and evaluation. I am still working on this Framework.

Other than the tutorials in Tensorflow/Model, I wrote this framework in the way that EE in Java, removing the strong coupling between the project and the system. I used a lot of reflaction to remove the initialization of instance(Just like IOC), and create a lot of function handler for user's convinience so that can create their own aspects and cutpoint has already previoded in the framework by hard coding, that's because I didn't find aspect in python, and readlly don't want to spend to much time on creating proxy XP.

Some instruction will come soon once I finished testing the framework in different occasions.

### Link
1. [Source code of this framework](https://github.com/Seanforfun/Distributed-Tensorflow-Framework)
2. [My explaination to this framework: Tensorflow集群式处理思想](https://seanforfun.github.io/deeplearning/2018/12/03/DistributeTensorflowFramework.html)
