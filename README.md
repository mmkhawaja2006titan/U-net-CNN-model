# U-net-CNN-model
U NET MODEL




📌 Overview

This project implements an end-to-end computer vision pipeline:

Input Image
     ↓
Image Preprocessing
     ↓
U-Net Semantic Segmentation
     ↓
Predicted Segmentation Mask
     ↓
Class Selection
     ↓
Binary Mask
     ↓
Connected Component Analysis
     ↓
Object Centroid
     ↓
(X, Y) Coordinates

The system is designed to recognize three object classes:

Class ID	Class
0	Background
1	Red
2	Blue
3	Green

The final centroid coordinates can be used as a foundation for directing a robot toward a detected object.

🚀 Key Features
COCO-format annotation processing
Conversion of bounding boxes into training masks
U-Net semantic segmentation
Multi-class pixel-level classification
Binary mask generation
Connected-component analysis
Object centroid calculation
Image-coordinate localization
PyTorch-based deep learning pipeline
OpenCV-based image processing
🧠 U-Net Architecture

The project uses a U-Net encoder-decoder architecture.

The input image is resized to 256 × 256 and processed through the encoder:

256 × 256
     ↓
128 × 128
     ↓
64 × 64
     ↓
32 × 32
     ↓
16 × 16

At the bottleneck, the network contains a compressed feature representation of the image.

The decoder then reconstructs the spatial resolution:

16 × 16
     ↓
32 × 32
     ↓
64 × 64
     ↓
128 × 128
     ↓
256 × 256

U-Net skip connections transfer spatial information from the encoder to the corresponding decoder layers, helping the model recover object boundaries and spatial details.

The final layer produces class scores for every pixel.

For example, one pixel may have:

Background = 0.05
Red        = 0.90
Blue       = 0.03
Green      = 0.02

The class with the highest score is selected using argmax.

📂 Dataset Preparation

The original dataset uses COCO-style annotations containing bounding boxes rather than pixel-perfect segmentation masks.

A preprocessing script:

coco_boxes_to_masks.py

converts the bounding-box annotations into class-based masks.

Example

A COCO annotation may contain:

[x, y, width, height]

The preprocessing pipeline creates a mask and fills the corresponding bounding-box region with its class ID.

COCO Annotation
       ↓
Bounding Box
       ↓
Class Mapping
       ↓
Pixel Mask
       ↓
PNG Mask

This produces paired training data:

image_001.png
mask_001.png

image_002.png
mask_002.png

image_003.png
mask_003.png
⚠️ Dataset Limitation

Because the original annotations are bounding boxes rather than true segmentation polygons, the generated masks represent objects as rectangular regions.

For example:

Ground-truth approximation

┌───────────────┐
│    OBJECT     │
│               │
└───────────────┘

rather than following the exact object boundary.

As a result, the model may produce blocky segmentation boundaries.

This is primarily a limitation of the available training annotations.

Future Improvement

Using datasets with pixel-level segmentation annotations would allow the model to learn more accurate object boundaries.

🔬 Training Process

During training, the model receives both an image and its corresponding ground-truth mask.

                 ┌─────────────────┐
                 │   Actual Image  │
                 └────────┬────────┘
                          ↓
                       U-Net
                          ↓
                 Predicted Mask
                          │
                          ↓
                    ┌───────────┐
                    │   Loss    │
                    └─────┬─────┘
                          ↑
                 Ground Truth Mask

The predicted mask is compared against the ground-truth mask using a loss function.

The loss is used during backpropagation to update the network's weights.

This process is repeated over many training examples and epochs so that the network learns to associate visual features with the correct pixel classes.

🔎 Inference

After training, the model can process a new image that it has not previously seen.

Unlike training, a ground-truth mask is not required during inference.

New Image
    ↓
Preprocessing
    ↓
Trained U-Net
    ↓
Class Scores
    ↓
Argmax
    ↓
Predicted Mask

The model generates its own segmentation mask.

🎭 Binary Mask Extraction

Once the predicted mask is generated, a specific class can be isolated.

For example, to extract the red object:

binary_mask = (mask == 1).astype(np.uint8)

This converts the multi-class mask into a binary mask:

0 = Not red
1 = Red

Example:

0 0 0 1 1
0 0 1 1 1
0 0 0 0 0
📍 Object Localization

The binary mask identifies the pixels belonging to the selected object.

The next step is to determine the object's location.

Connected-component analysis is used to identify connected regions within the binary mask.

The relevant object region can then be used to calculate its centroid.

Conceptually:

        Object
      █████████
     ███████████
      █████████
          ●
       Centroid

The centroid produces an image coordinate:

(X, Y)

For example:

(320, 240)
📐 Image Coordinate System

Image coordinates start from the top-left corner:

(0,0)
  ┌────────────────────────→ X
  │
  │
  │
  ↓
  Y

Therefore:

X increases from left to right.
Y increases from top to bottom.
(0, 0) represents the top-left pixel.

For an image with resolution 640 × 480, the bottom-right pixel is approximately:

(639, 479)
🤖 Robotics Application

The ultimate purpose of calculating the centroid is to convert visual information into a location that can potentially be used by a robot.

Camera
   ↓
Image
   ↓
U-Net
   ↓
Object Segmentation
   ↓
Object Detection
   ↓
Centroid
   ↓
(X, Y)
   ↓
Robot Positioning

The current system provides image-plane coordinates. Converting these coordinates into real-world robot coordinates would require additional calibration, camera geometry, and/or depth information.

🛠️ Technologies Used
Python
PyTorch
OpenCV
NumPy
U-Net
Semantic Segmentation
COCO Annotations
Connected Component Analysis
Computer Vision
Deep Learning
📁 Project Structure

A typical project structure is:

project/
│
├── coco_boxes_to_masks.py
│
├── dataset/
│   ├── images/
│   │   ├── image_001.png
│   │   ├── image_002.png
│   │   └── ...
│   │
│   └── masks/
│       ├── mask_001.png
│       ├── mask_002.png
│       └── ...
│
├── model/
│   └── unet.py
│
├── train.py
├── predict.py
│
└── README.md
📊 End-to-End Pipeline
                 DATA PREPARATION
                       │
                       ↓
              COCO Annotations
                       │
                       ↓
             Bounding Box → Mask
                       │
                       ↓
                 Training Data
                       │
                       ↓
                 ┌───────────┐
                 │   U-Net   │
                 └───────────┘
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
          Encoder             Decoder
             │                   │
  256 → 128 → 64 → 32 → 16 → 32 → 64 → 128 → 256
                       │
                       ↓
               Pixel Predictions
                       │
                       ↓
                    Argmax
                       │
                       ↓
              Predicted Mask
                       │
                       ↓
                Binary Mask
                       │
                       ↓
          Connected Components
                       │
                       ↓
                  Centroid
                       │
                       ↓
                  (X, Y)
                       │
                       ↓
             Robot Localization
🎯 Project Outcome

This project demonstrates an end-to-end implementation of a deep-learning computer vision pipeline, progressing from raw image data and imperfect annotations to pixel-level segmentation and object localization.

The project also highlights an important practical machine-learning lesson: the quality of training data and annotations directly affects the quality of model predictions.

The resulting system provides a foundation for future development in:

🤖 Autonomous robotics
📷 Vision-based navigation
🎯 Object localization
🦾 Robotic manipulation
🧠 AI-powered perception systems
🔮 Future Improvements

Potential improvements include:

Replace bounding-box-derived masks with true segmentation annotations.
Improve segmentation boundary accuracy.
Experiment with Dice Loss / Cross-Entropy / combined losses.
Add data augmentation.
Improve detection of multiple objects of the same class.
Convert image coordinates into real-world coordinates.
Integrate the vision system directly with a robot or robotic arm.
Add depth information using a depth camera.
Evaluate the model using metrics such as IoU, Dice Score, precision, and recall.
👨‍💻 Project Focus

This project combines Deep Learning + Computer Vision + Image Processing + Robotics, with the goal of turning visual perception into actionable spatial information.

From pixels → to objects → to coordinates → to robotic action
