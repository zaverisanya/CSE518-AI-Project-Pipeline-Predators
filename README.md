# CSE518-AI-Project-Pipeline-Predators

We have implemented hand pose detection using RGB images via a CNN-based approach. As an application, we have implemented music generation based on hand pose.

## Table of Contents


***

## Introduction

Contact-less control has become a fundamental feature of recent and emerging human-interaction-based technology. Especially in AR and VR, where true physical interaction in the virtual or projected environment is impossible, various modalities like hand gestures and speech are utilized to bridge the gap for natural intuitive interaction. In this project, our objective is to design a system that detects the hand pose from a live camera feed, and performs the application of generating music dynamically based on the detected hand pose, much like the musical instrument theremin.

* * *

## Approach

### Dataset

After considering several datasets, we decided to use the Large-scale Multiview 3D Hand Pose Dataset, because it satisfied a few requirements we had.

- It contains RGB images.
- It is publicly available and free to use.
- It is a very large dataset.
- It has 22 keypoints including the palm normal.
- It has both 2D and 3D coordinates for the keypoints.
- It has the views from 4 different angles for each pose, hence there is good variation in the data.
- The authors of the dataset have included physical variation such as the presence and absence of rings and gloves, different peoples’ hands (skin color, shape, etc.), and left and right sides.
- Moreover, they have artificially augmented data as well, with many virtual backgrounds, so the CNN can be trained for a variety of user backgrounds.

### Pre-processing

From the 3D coordinates provided for the dataset, such that for every set of 4 images describing the same pose from different cameras at different angles, the coordinates of the keypoints remain the same, and in the frame of the Leap Motion controller. So, the first thing we needed to do is perform projection transitions to get the 2D coordinates for each image in their own frame of reference.

![Dataset Generation](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/Dataset%20Gen.jpg)

Then we had to generate a Pandas DataFrame consisting of image paths and corresponding newly generated x and y coordinates for each keypoint. We then take a random sample of it to use, to make training computationally feasible. Finally, we split it into train-validation data and make respective Keras ImageDataGenerators to pass to the model for training. ImageDataGenerators were indispensable because otherwise the input array consisting of the image arrays was extremely large and was computationally impossible to make with the available resources. We did not use the dataset to do the testing. Rather, we tested on the live webcam feed.

### CNN Architecture

After hyperparameter optimization, we fixed on a suitable model architecture that balanced the bias against the variance. We chose to use our custom model instead of standard models. However, VGG-19 was the basis of our inspiration.

Layers:

- Input (RGB Images, 3 channels, pixels rescaled to [0,1])
- 2 × Convolution (64 filters, 3×3 kernel, ReLU)
- MaxPooling (/2)
- 3 × Convolution (128 filters, 3×3 kernel, ReLU)
- MaxPooling (/2)
- 5 × Convolution (256 filters, 3×3 kernel, ReLU)
- MaxPooling (/2)
- 5 × Convolution (512 filters, 3×3 kernel, ReLU)
- MaxPooling (/2)
- 5 × Convolution (512 filters, 3×3 kernel, ReLU)
- MaxPooling (/2)
- Flatten [32768 units]
- Dense (4000, ReLU)
- Droupout(0.2)
- Dense (1000, ReLU)
- Droupout(0.2)
- Dense (500, ReLU)
- Dense (100, ReLU)
- Dense (42, Relu, Output)


### Training (Hand Pose)

We train the CNN to estimate keypoint locations from hand
images.

![Training Process](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/Training.jpg)

### Testing (Hand Pose)

![Testing Process](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/Testing.jpg)

### Music Generation

We used a simple mathematical function to map the coordinates to a note.
- Root mean square of all 21 x coordinates was calculated.
- It was squared.
- Then it was multiplied once by 2000 and once by 4000 to yield two different frequencies (say, frequency 1 and frequency 2).
- Frequency 1 was used to create a sine wave note and frequency 2 was used to create a cosine wave note.

Similar exercise was done with the y coordinates. All in all, we had 4 notes (waves). We added all of them to get the final wave note to play. If R_X is the root mean square of all x coordinates and R_Y is the root mean square of all y coordinates, the equation of the wave describing the note played is:
![Equation](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/equation.png)

