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

![Fig. 1. Dataset Generation](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/Dataset%20Gen.jpg)

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

![Fig. 2. Training Process](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/Training.jpg)

### Testing (Hand Pose)

![Fig. 3. Testing Process](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/Testing.jpg)

### Music Generation

We used a simple mathematical function to map the coordinates to a note.
- Root mean square of all 21 x coordinates was calculated.
- It was squared.
- Then it was multiplied once by 2000 and once by 4000 to yield two different frequencies (say, frequency 1 and frequency 2).
- Frequency 1 was used to create a sine wave note and frequency 2 was used to create a cosine wave note.

Similar exercise was done with the y coordinates. All in all, we had 4 notes (waves). We added all of them to get the final wave note to play. If R_X is the root mean square of all x coordinates and R_Y is the root mean square of all y coordinates, the equation of the wave describing the note played is:

![Fig. 4. Equation](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/equation.jpg)

Our objective was to create a relatively complex and pleasant wave (for quality) rather than a simple sine wave that produced an unpleasant beep, while maintaining simplicity for the user to exercise control over their music generation. Apart from squaring the root mean square, the complexity of the function also helps increase the range of producible music. 

### Issues

The primary issues we faced are related to computing power limitations. The RAM on Google Colab and Kaggle Notebooks runs out. Using alternative approaches, the speed of execution becomes painfully slow. The GPU time allowed is also insufficient. However, sampling of the datast and using ImageDataGenerator from Keras, alleviated most of the computational restrictions.

***

## Results

### Hand Pose Results

Here are a few results of testing with the webcam. The hand pose skeleton was plotted over the frame images using the coordinates of keypoints.

![Fig. 5. Inner side of hand](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/result1.jpg)
![Fig. 6. Outer side of hand](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/result2.jpg)
![Fig. 7. Curled hand and twisted skeleton](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/result3.jpg)
![Fig. 8.  Curled fingers and prediction on hidden part of fingers](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/result4.jpg)
![Fig. 9. Prediction on partial hand
](https://github.com/kkalaria16/CSE518-AI-Project-Pipeline-Predators/blob/main/images/result5.jpg)

As we can see, the results are mostly satisfactory. From Fig. 7, Fig. 8, and Fig. 9, we can see that despite some slightly off-target predictions, the model predicts keypoints reasonably well even in difficult cases. However, one of our limitations is that currently, we have designed the pipeline to work only on a single hand. With some work, it can be extended to work on multiple hands. Moreover, sometimes there is are time lags but usually the performance is satisfactory.

### Music Generation

The music generation part works well and the note frequencies are responsive to changes in the hand pose or location within the frame. The results can not be displayed here on
paper.

From the point of view of the program as an instrument, a very slight change in the hand pose or location of hand changed the frequency slightly but noticeably, but did not
changed the waveform much. However, a larger change in the location changed both the waveform and the frequency. The same happens with the hand pose but it depends on how the
hand pose changes the root mean square changes. Overall, the music generation depends explicitly on the mathematical function used to produce it from the hand pose and thus is an independent application based on but not dependant on the hand pose detection system.

***

## Installation guidelines and platform details 
