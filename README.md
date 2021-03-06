# Relativistic GAN

Code to replicate all analyses from the paper [The relativistic discriminator: a key element missing from standard GAN](https://arxiv.org/abs/1807.00734)
**Discussion at https://ajolicoeur.wordpress.com/RelativisticGAN.**

**Notice:**
There was an error in the code used to construct RaSGAN and RaLSGAN. I used "torch.mean(y_pred_fake) - y_pred" instead of "y_pred_fake - torch.mean(y_pred)" in the second terms of the equation with the expectation over fake data. RaHingeGAN was correctly coded. This probably won't change much, I'll be rerunning the analyses with the correct coding, it might takes a few weeks since I only have a Geforce 1060. The correct versions were added as Loss_D = 16 and 17 for RaSGAN and RaLSGAN respectively.

**To add Relativism to your own GANs in PyTorch, you can use pieces of code from this:**

```python
### Assuming this gets you real and fake data

# Real data
x.data.resize_as_(images).copy_(images)
y_pred = D(x)
y.data.resize_(current_batch_size).fill_(1)

# Fake data
z.data.resize_(current_batch_size, param.z_size, 1, 1).normal_(0, 1)
fake = G(z)
x_fake.data.resize_(fake.data.size()).copy_(fake.data)
y_pred_fake = D(x_fake.detach()) # For generator step do not detach
y2.data.resize_(current_batch_size).fill_(0)


### Standard GAN (non-saturating)

# Use torch.nn.Sigmoid() as last layer in generator

criterion = torch.nn.BCELoss()

# Real data Discriminator loss
errD_real = criterion(y_pred, y)
errD_real.backward()

# Fake data Discriminator loss
errD_fake = criterion(y_pred_fake, y2)
errD_fake.backward()

# Generator loss
errG = criterion(y_pred_fake, y)
errG.backward()


### Relativistic Standard GAN

# No sigmoid activation in last layer of generator because BCEWithLogitsLoss() already adds it

BCE_stable = torch.nn.BCEWithLogitsLoss()

# Discriminator loss
errD = BCE_stable(y_pred - y_pred_fake, y)
errD.backward()

# Generator loss (You may want to resample again from real and fake data)
errG = BCE_stable(y_pred_fake - y_pred, y)
errG.backward()


### Relativistic average Standard GAN

# No sigmoid activation in last layer of generator because BCEWithLogitsLoss() already adds it

BCE_stable = torch.nn.BCEWithLogitsLoss()

# Discriminator loss
errD = ((BCE_stable(y_pred - torch.mean(y_pred_fake), y) + BCE_stable(y_pred_fake - torch.mean(y_pred), y2))/2
errD.backward()

# Generator loss (You may want to resample again from real and fake data)
errG = ((BCE_stable(y_pred - torch.mean(y_pred_fake), y2) + BCE_stable(y_pred_fake - torch.mean(y_pred), y))/2
errG.backward()


### Relativistic average LSGAN

# No activation in generator

# Discriminator loss
errD = (torch.mean((y_pred - torch.mean(y_pred_fake) - y) ** 2) + torch.mean((y_pred_fake - torch.mean(y_pred) + y) ** 2))/2
errD.backward()

# Generator loss (You may want to resample again from real and fake data)
errG = (torch.mean((y_pred - torch.mean(y_pred_fake) + y) ** 2) + torch.mean((y_pred_fake - torch.mean(y_pred) - y) ** 2))/2
errG.backward()


### Relativistic average HingeGAN

# No activation in generator

# Discriminator loss
errD = (torch.mean(torch.nn.ReLU()(1.0 - (y_pred - torch.mean(y_pred_fake)))) + torch.mean(torch.nn.ReLU()(1.0 + (y_pred_fake - torch.mean(y_pred)))))/2
errD.backward()
 
# Generator loss  (You may want to resample again from real and fake data)
errG = (torch.mean(torch.nn.ReLU()(1.0 + (y_pred - torch.mean(y_pred_fake)))) + torch.mean(torch.nn.ReLU()(1.0 - (y_pred_fake - torch.mean(y_pred)))))/2
errG.backward()
```

# To replicate analyses from the paper

**Needed**

* Python 3.6
* Pytorch (Latest from source)
* Tensorflow (Latest from source, needed to get FID)
* Cat Dataset (http://academictorrents.com/details/c501571c29d16d7f41d159d699d0e7fb37092cbd)

**To do beforehand**

* Change all folders locations in fid_script.sh, stable.sh, unstable.sh, GAN_losses_iter.py, GAN_losses_iter_PAC.py
* Make sure that there are existing folders at the locations you used
* If you want to use the CAT dataset
  * Run setting_up_script.sh in same folder as preprocess_cat_dataset.py and your CAT dataset (open and run manually)
  * Move cats folder to your favorite place
  * mv cats_bigger_than_64x64 "your_input_folder_64x64"
  * mv cats_bigger_than_128x128 "your_input_folder_128x128"
  * mv cats_bigger_than_256x256 "your_input_folder_256x256"

**To run**
* To run models
  * Use GAN_losses_iter.py (Version you probably will want to use)
  * Use GAN_losses_iter_PAC.py (Version with PacGAN-2, to use when there is severe mode collapse only)
* To Replicate
  * Open stable.sh and run the lines you want
  * Open unstable.sh and run the lines you want

**Notes**

If you just want to generate pictures and you do not care about the Fréchet Inception Distance (FID), you do not need to download Tensorflow.

If you don't want to generate cat, nor get the FID, you can skip ahead and focus entirely on "GAN_losses_iter.py".

Although I always used the same seed (seed = 1), keep in mind that your results may be sightly different. Neural networks are notoriously difficult to perfectly replicate. You never know if something that I changed while working on the paper could have affected the randomness in some of my earlier experiments (I did stable experiments first, then unstable experiments for CIFAR-10, and most recently the unstable experiments for CAT). CUDNN is also said to introduce some randomness and I use it. I forgot to put a seed for the tensorflow FID scripts so FIDs may vary by a 2-3 points, but hopefully they shouldn't change much since I used a very large amount of images for the calculations.

# Results

**64x64 cats with RaLSGAN (FID = 11.97)**

![](/images/best_64x64_crop.png)

**128x128 cats with RaLSGAN (FID = 15.85)**

![](/images/best_128x128_crop.png)

**256x256 cats with SGAN (5k iterations)**

![](/images/GAN.jpeg)

**256x256 cats with LSGAN (5k iterations)**

![](/images/LSGAN.jpeg)

**256x256 cats with RaSGAN (FID = 32.11)**

![](/images/RaSGAN.jpeg)

**256x256 cats with RaLSGAN (FID = 35.21)**

![](/images/RaLSGAN.jpeg)

**256x256 cats with SpectralSGAN (FID = 54.73)**

![](/images/SpectralSGAN.jpeg)

**256x256 cats with WGAN-GP (FID > 100)**

![](/images/WGAN-GP.jpeg)
