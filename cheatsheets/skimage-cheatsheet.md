---
layout: page
title: Scikit-Image Cheatsheet
background: '/img/posts/ml.jpg'
---

## Image Processing Basics

### Components
* Rows and columns of pixels
* Each pixel measures colour intensity
* Grayscale vs. RGB:
    * Grayscale images only has one channel
    * RGB images have red, blue, and green channels

### Converting Between RGB and Gray

```py
from skimage import color
grayscale = color.rgb2gray(rgb_image)
rgb = color.gray2rgb(grayscale_image)
```

### Plotting Images in Matplotlib

```py
plt.imshow(image, cmap='gray')
plt.axis('off')
plt.show()
```

### Load Images

```py
import matplotlib.pyplot as plt

# Load image - numpy.ndarray object
image = plt.imread('image_file.jpg')
```

### Colour Channels

```py
red = image[:, :, 0]
green = image[:, :, 1]
blue = image[:, :, 2]
```

### Image Shape
```py
image.shape     # (height, width, channels)
```

### Flipping Images
```py
# Vertical flip
flipped = np.flidud(image)

# Horizontal flip
flipped = np.fliplr(image)
```

### Histograms
* Can plot the histogram of a channel (gray / R/G/B)
* Used to:
    * Do thresholding
    * Alter brightness and contrast
    * Equalise an image

```py
red = image[:, :, 0]

# Ravel flattens the array into 1D
# 256 bins is for 1 pixel each
plt.hist(red.ravel(), bins=256)
```

### Thresholding
* A technique of image segmentation
* Partitions image into a background and a foreground
* Set:
    * 255 (white) if pixel is above a threshold value
    * 0 (black) if pixel is below a threshold value
* Works best for high-contrast grayscale images
* Categories:
    * Global / histogram-based: Good for uniform backgrounds
    * Local or adaptive: For uneven background illumination

```py
# Set threshold
thresh = 127

# Apply threshold
binary = image > thresh
```

#### Testing Different Algorithms
* Can test out different thresholding techniques:

```py
from skimage.filters import try_all_threshold

# Obtain all the resulting images
fig, ax = try_all_threshold(image, verbose=False)
```

#### Optimal Thresh Value

##### For Global Thresholding
Calculate optimal value:

```py
from skimage.filters import threshold_otsu

# Obtain optimal threshold value
thresh = threshold_otsu(image)

# Apply thresholding to the image
binary_global = image > thresh
```

##### For Local Thresholding:

```py
from skimage.filters import threshold_local

# Block size to surround each pixel (a.k.a. local neighbourhoods)
block_size = 35

# Compute optimal threshold
# Offset is to subtract from the mean of ???
local_thresh = threshold_local(text_image, block_size, offset=10)

# Apply threshold to image
binary_local = text_image > local_thresh
```

## Filtering
* Enhancing an image
* Used to emphasise or remove features
* Includes smoothing, sharpening or edge detection
* Is a neighbourhood operation

### Edge Detection
```py
from skimage.filters import sobel

# Apply edge detection filter
edge_sobel = sobel(image)
```

### Smoothing
* Blur edges and reduces contrast

```py
from skimage.filters import gaussian

# Apply filter
gaussian_image = gaussian(image, multichannel=True)
```

## Contrast
* Makes details more visible
* Difference between maximum and minimum intensity
* Enhance contrast using **contrast stretching** and **histogram equalisation**
* Types of histogram equalisation:
    * Histogram equalisation
    * Adaptive histogram equalisation
    * Constrast Limited Adaptive Histogram Equalisation (CLAHE):
* ADaptive equalisation computes several histograms, each corresponding to a different part of the image

```py
from skimage import exposure

# Histogram equalisation
image_eq = exposure.equalize_hist(image)

# CLAHE
# Higher clip limit = higher contrast
image_adapteq = exposure.equalize_adapthist(image, clip_limit=0.03)
```

## Transformations
* Optimisation and compression of image
* Save images with same proportion
* Rotation

### Rotation

```py
from skimage.transform import rotate

# Rotate 90 degrees clockwise
image_rotated = rotate(image, -90)

# Rotate 90 degrees anti-clockwise
image_rotated = rotate(image, 90)
```

### Rescale
```py
from skimage.transform import rescale

# Rescale 4 times smaller
image_rescaled = rescale(image, 1/4, anti_aliasing=True, multichannel=True)
```

### Resizing
```py
from skimage.transform import resize

# Height and width
height = 400
width = 500

# Resize image
image_resized = resize(image, (height,width), anti_aliasing=True)

# To resize proportionally:
height = image.shape[0]/4
width = image.shape[1]/4
image_resized = resize(image, (height,width), anti_aliasing=True)
```

## Morphology
* Remove imperfections by accounting for form and structure of images
* Suited to binary images, but some techniques may be applied for grayscale
* Main techniques:
    * Dilation: Add pixels to boundaries of objects in an image
    * Erosion: Remove pixels from object boundaries
* The structuring element affects the number pixels added (moving box shape)

```py
from skimage import morphology

# Square structuring element
square = morphology.square(4)

# Rectangle structuring element
rectangle = morphology.rectangle(4,2)
```

### Erosion
```py
from skimage import morphology

# Set structuring element
selem = rectangle(12,6)

# Obtain eroded image
# Function uses cross-shaped structuring elem if selem is not specified
eroded_image = morphology.binary_erosion(image, selem=selem)
```

### Dilation
```py
from skimage import morphology

# Obtain dilated image
dilated_image = morphology.binary_dilation(image)
```

## Image Reconstruction

### Inpainting
* Reconstructing lost parts of images
* Looking at non-damaged regions

```py
from skimage.restoration import inpaint

# Mask
mask = get_mask(image)

# Apply inpainting
restored_image = inpaint.inpaint_biharmonic(defect_image, mask, multichannel=True)
```

## Noise
* Denoising types:
    * Total variation filter: Minimise total variation of image
    * Bilateral filtering: Smooths images by preserving images and replaces intensity of each pixel by average of neighbouring pixels
    * Wavelet denoising
    * Non-local means denoising

```py
from skimage.restoration import denoise_tv_chambolle, denoise_bilateral
from skimage.util import random_noise

# Add noise to image
noisy_image = random_noise(dog_image)

# Total variation filter denoising
# The greater the weight, the more denoising and higher smoothing
denoised_image = denoise_tv_chambolle(noisy_image, weight=0.1, multichannel=True)

# Bilateral filter denoising
denoised_image = denoise_tv_chambolle(noisy_image, multichannel=True)
```

## Segmentation
* Partition images into segments to change the representation into something easier to analyse
* Superpixel: Group of connected pixels with a similar colour or gray level
    * More meaningful regions
    * Computation efficiency
* Types of segmentation
    * Supervised (e.g. thresholding)
    * Unsupervised
        * Simple Linear Iterative Clustering (SLIC)

```py
from skimage.segmentation import slic
from skimage.color import label2rgb

# Obtain the segments
segments = segmentation.slic(image, n_segments=300)

# Put segments on top of original image
# Avg = average colour of superpixel
segmented_image = label2rgb(segments, image, kind='avg')
```

## Contours
* Closed shape of points or line segments representing the boundaries of objects
* Purpose:
    * Measure size
    * Classify shapes
    * Determine the number of objects
* Should input a threshold image

```py
# Make image grayscale
image = color.rgb2gray(image)

# Obtain threshold value
thresh = threshold_otsu(image)

# Get binary image
thresholded_image = image > thresh

from skimage import measure

# Find contours
# Closer to 1 = higher sensitivity to contours
contours = measure.find_contours(thresholded_image, 0.8)
```

## Advanced Operations

### Edge Detection with Canny
Canny:

* Standard algorithm for edge detection
* Requires grayscale image
* Includes a gaussian filter (use `sigma` parameter to increase intensity of filter)
    * Higher = more noise removed and less edgy

```py
from skimage.feature import canny

# Convert to grayscale
image = color.rgb2gray(image)

# Apply canny detector
canny_edges = canny(coins, sigma=0.5)
```

### Corner Detection
* Points of interest are invariant to rotation, translation, intensity, and scale changes, and are therefore robust and reliable
* Corners are just one type of point of interest
* Harris algorithm is typically used
    * Needs grayscale image

```py
from skimage.feature import corner_harris

# Convert to grayscale
image = rgbs2gray(image)

# Apply Harris corner detector
measure_image = corner_harris(image)

# Find coordinates of corners, separated from each other by a minimum distance
coords = corner_peaks(corner_harris(image), min_distance=5)
```

### Face Detection
* Uses a window that moves through the image until it finds something similar to a face
* Searching on multiple scale: windows of small size to detect faraway faces, and bigger windows for nearer faces

```py
from skimage.feature import Cascade

# Initialise the cascade detector
detector = Cascade(trained_file)

# Apply detector on image
detected = detector.detect_multi_scale(
    img=image,
    scale=1.2,  # factor by which the searching window is multiplied in each step
    step_ratio=1, # 1 to 1.5 is good
    min_size=(10,10),
    max_size=(200,200)
)

# Result has:
#   - r: row position of top left corner
#   - c: column position of top left corner
#   - width: width
#   - height: height
```