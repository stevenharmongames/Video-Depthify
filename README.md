# Video-Depthify
Collection of scripts for generating depth maps for videos using machine learning.

## Running on Google Colab
You can run the whole process directly from your browser without setting anything up locally, thanks to Google Colab.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1eB8Q2MtYSucttwv0XojojMGkdpfh1CMq?usp=sharing)

_**June 2021 update**_

_Use the following Colab notebook to use [BoostingMonocularDepth](https://github.com/compphoto/BoostingMonocularDepth/) instead of MiDaS for even more detailed (and slower) depth maps._

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1-c7xSMqbdiykjyNWRvn-s-l4Hirvez5c?usp=sharing)

## Running locally

### Requirements
- Python/Conda 3 environment (virtualenv recommended)

```pip3 install torch torchvision opencv-python timm Pillow numpy```
- ffmpeg
- Unix system recommended (WSL on Windows)

### Setup
  Step 1: Install Anaconda
  Download Anaconda: Go to the Anaconda website and download the Anaconda installer for Windows.
  https://www.anaconda.com/download
  
  Install Anaconda: Run the installer and follow the on-screen instructions to install Anaconda. Make sure to check the option to add Anaconda to your PATH environment variable.
  
  Step 2: Set Up a New Conda Environment
  Open Anaconda Prompt: Search for "Anaconda Prompt" in your Start menu and open it.
  Create a New Environment: Run the following command to create a new environment named depthify:
  
  `conda create --name depthify python=3.8`
  
  Activate the Environment: Activate your new environment with:
  
  `conda activate depthify`
  
  Step 3: Install Dependencies
  Install Required Packages: Run the following commands to install the necessary dependencies:
  
  `pip install torch torchvision opencv-python timm Pillow numpy ffmpeg-python`
  
  Step 4: Navigate to the Project Directory
  
  still inside the Anaconda Prompt type
  
  `cd C:\Users\yourname\Documents\thefolderwiththisproject`
  
  Copy `depth.py`, `average.py` and `merge.py` into a folder with the video you wish to depthify. In our case we are using a short, royalty-free [video](https://pixabay.com/videos/elephant-pachyderm-tanzania-6447/) of an elephant.
  In the same folder, create empty folders `rgb`, `depth`, `averaged`, and `merged`.
  
  
  Your folder structure should look like this:
  ```
  .
  ├── averaged
  ├── average.py
  ├── depth
  ├── depth.py
  ├── Elephant.mp4
  ├── merged
  ├── merge.py
  └── rgb
  ```
  
  Step 5: Run the scripts!

### Steps
1) Get the FPS of the source video. We will need this to put the video back together.

`ffmpeg -i Elephant.mp4 2>&1 | sed -n "s/.*, \(.*\) fp.*/\1/p"`

2) Split the video into individual frames (this will populate the `rgb` folder).

`ffmpeg -i Elephant.mp4 -qmin 1 -qscale:v 1 ./rgb/%06d.jpg`

3) Run the depth inferrence (this will populate the `depth` folder).

For faster (but lower quality) depth map generation you can switch to a lighter model by changing [line 13](https://github.com/jankais3r/Video-Depthify/blob/main/depth.py#L13) to `False`.

`python3 depth.py`

4) [**Optional step**] Run the frame average to reduce the flicker betweeen individual frames (this will populate the `averaged` folder).

See [Elephant_depth_averaged_sound.mp4](https://github.com/jankais3r/Video-Depthify/blob/main/Elephant_depth_averaged_sound.mp4) vs. [Elephant_depth_sound.mp4](https://github.com/jankais3r/Video-Depthify/blob/main/Elephant_depth_sound.mp4) to see the effect of this step.

`python3 average.py`

5) Merge the depthmaps with the original frames (this will populate the `merged` folder).

`python3 merge.py`

6) Combine the merged frames into a video file. (substitute `25` fps with the number you got from step 1).

`ffmpeg -framerate 25 -i ./merged/%06d.jpg -vcodec libx264 -pix_fmt yuv420p Elephant_depth.mp4`

7) Copy the audio track from the original video.

`ffmpeg -i Elephant_depth.mp4 -i ./Elephant.mp4 -c copy -map 0:0 -map 1:1 -shortest Elephant_depth_sound.mp4`

8) Voilà

**MiDaS result**:

![Demo MiDaS](https://github.com/jankais3r/Video-Depthify/blob/main/demo.gif)


**BoostingMonocularDepth result**:

![Demo Boosting](https://github.com/jankais3r/Video-Depthify/blob/main/demo_boosting.gif)

BoostingMonocularDepth does not always provide better results compared to vanilla MiDaS, as you can see on the following example.

**MiDaS result**:

![Demo MiDaS](https://github.com/jankais3r/Video-Depthify/blob/main/demo2.gif)


**BoostingMonocularDepth result**:

![Demo Boosting](https://github.com/jankais3r/Video-Depthify/blob/main/demo2_boosting.gif)
