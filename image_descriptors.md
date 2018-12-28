# Image Descriptors

A brief cheatsheet on functions for image descriptors, feature descriptors and feature vectors


### Image Descriptors
* Works on the whole image and quantifies it globally
* Not robust to translation and rotation
* Cannot quantify parts/regions of image
* Output is only one feature vector per image
* Example descriptors - color channel statistics, color histograms, local binary patterns, etc.

### Feature Descriptors
* Works on selective parts/regions of image
* Output is N feature vectors for N regions of interest
* Robust to translation, rotation, orientation, etc.
* Example descriptors - SIFT, SURF, ORB, BRISK, BRIEF, FREAK, etc.

#### Color channel statistics (Image descriptor)
* Uses image statistics such as mean, standard deviation, skew and kurtosis as feature vectors

* We can then use Euclidean distance as a measure to compare these feature vectors

```python
[means, stds] = cv2.meanStdDev(image)
featurevec = np.concatenate([means, stds]).flatten()

from scipy.spatial import distance as dist
d = dist.euclidean(featurevec1, featurevec2)
```

#### Color histograms (Image descriptor)
* Uses histograms of color channels or pixel intensities to define the feature vector
* We can use clustering algorithm such as K-Means clustering to group images based on the feature vectors obtained from histograms
* K-Means utilizes Euclidean space to compare two points. It is preferred to use L*a*b color space to generate the feature vector as L*a*b has better perceptual meaning of an image compared to other color spaces like RGB or HSV

```python
lab = cv2.cvtColor(image, cv2.COLOR_BGR2LAB)
hist = cv2.calcHist([lab], [0,1,2], mask, bins, [0,256, 0,256, 0,256])
featurevec = cv2.normalize(hist,hist).flatten() # opencv 3+
featurevec = cv2.normalize(hist).flatten() # opencv 2.4

from sklearn.cluster import KMeans
clt = KMeans(n_clusters)
labels = clt.fit_predict(featurevectors)
```

#### Hu Moments (Image Descriptor)

* Characterizes shape of the image and captures area, centroid and orientation of image
* Invariant to changes in rotation, translation, scale, reflection etc but is relies heavily on proper centroid calculation
* Segmentation is preferred as centroid calculation is sensitive to noise
* Hu calculates 7 moments which forms the feature vector
* We can use this algorithm to find outliers from a population (for example select one square shape from a population of circles)


moments = [M1, M2, M3, M4, M5, M6, M7]

```python
moments = cv2.HuMoments(cv2.moments(roi)).flatten()
```

#### Zernike Moments (Image Descriptor)

* Work similar to Hu Moments, but perform better
* They work on theory of orthogonal functions like sine/cosine (rather than statistical derivators like Hu Moments)
* There is no redundancy between the moments calculated -- So, they have more discriminatory power
* Two parameters to tune:
  * Radius of circle enclosing the shape
  * Degree of polynomial (default=8)
* Better to create a shape mask and calculate Zernike moments on the extracted mask
* The radius should be carefully selected so that it encloses the shape completely. Any part of object outside the circle will be ignored

```python
import mahotas
moments = mahotas.features.zernike_moments(image, 21, degree=8)
```

#### Haralick Texture (Image descriptor)
* Describes texture present in the image
* Quantifies feel, appearance, consistency, surface
* Differentiates between 'rough' and 'soft' surfaces
* Feature vector is calculated by GLCM - (Gray Level Co-Concurrency Matrix)
* GLCM indicates how often pairs of adjacent pixels with a specific intensity occur
* We can also specify directions of adjacency - 4 Directions
  * Left-to-Right
  * Top-to-Bottom
  * TopLeft-to-BottomRight
  * TopRight-to-BottomLeft
* 4 feature vectors - one for each direction
  * Each vector is either 13-d or 14-d
* After averaging over directions, we get one feature vector with 13-d/14-d
* Averaging in directions improves the robustness in rotation

```python
import mahotas
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
featurevec = mahotas.features.haralick(gray).mean(axis=0)
```
#### Local Binary Patterns (LBPs) - (Image descriptor)
* Used for local representation of texture
* Compares each pixel with its surrounding pixel values (ex: in 3x3 neighborhood)
* Powerful than Haralick descriptor but computationally more expensive due to large size of feature vector
* Popular for face recognition
* Original implementation:
  * Create a binary map in a 3x3 neighborhood by threshold against the central pixel value
  * Assign a decimal value by calculating the bit-wise addition in clockwise direction
  * output is the histogram of all calculated LBP decimals
* Current Implementation:
  * Neighbourhood is defined by radius 'r' and number of points in a circle 'p'
  * points will be again 0s' or 1s which are thresholded against the central pixel
  * Used to calculate uniformity - number of 0-1/1-0 transitions
  * output is again histogram of all calculated LBPs

```python
from skimage import feature
lbp = feature.local_binary_pattern(gray, numPoints, radius, method='uniform')
(hist, _) = np.histogram(lbp.ravel(), bins=range(0, numPoints + 3),
	range=(0, numPoints + 2))
  # optionally normalize the histogram
eps = 1e-7
hist = hist.astype("float")
hist /= (hist.sum() + eps)
```

#### Histogram of Oriented Gradients, HOG (Image descriptor)
* quantifies both shape and texture
* Done by calculating the gradients and magnitudes of pixels in cells
* important parameters: orientations, pixels_per_cell and cells_per_block
* HOG is done in 5 steps
  1. Normalization (usually squareroot normalization (sqrt(p)))
  2. Gradient computation (magnitude & direction)
  3. Weighted votes in each cell
    * Calculate histogram of gradients with directions
    * number of bins = number of orientations
    * Concatenate histograms of every cell to form final feature vector
  4. Contrast normalization over blocks
    * Blocks are defined with a parameter 'cells_per_block' (2x2 or 3x3)
    * Normalization (L1 or L2) is done inside block and block move by overlapping
    * So featurevector of cell contributes multiple times (better for accuracy)
  5. Concatenate all normalized feature vectors to final feature vector

```python
from skimage import feature
(H, hogImage) = feature.hog(logo, orientations=9, pixels_per_cell=(8, 8),
	cells_per_block=(2, 2), transform_sqrt=True, , block_norm="L1",
	visualise=True)
hogImage = exposure.rescale_intensity(hogImage, out_range=(0, 255))
hogImage = hogImage.astype("uint8")
```
