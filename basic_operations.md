# Image Basics, Drawing, Basic image processing and Morphological Operations

We start image the numpy array 'image' (unsigned 8 bit)


### Loading an image
`cv2.imread(image)`

### Display an image
cv2.imshow('Image', image)

### Save a numpy array to an image file
`cv2.imwrite('newimage.jpg', image)`

### Shape of image (height and width)
```python
(h,w) = image.shape[:2]
(cX, cY) = (w // 2, h // 2)
```

### Crop an image
`tl = image[0:cY, 0:cX]`

### Initializing an empty image (canvas)
`canvas = np.zeros((300,300,3), dtype='uint8')`

### Drawing shapes on image
```python
# last parameter = -1 will fill the shape with color
blue = (255,0,0)
cv2.line(canvas, (0,0), (300,300), blue, 3)
cv2.rectangle(canvas, (0,0), (300,300), blue, 2)
cv2.circle(canvas, (centerX, centerY), radius, blue, 3)
```
### Translation
```python
M = np.float32([[1,0,x], [0,1,y]]) # x,y = shift in x and y
shifted = cv2.warpAffine(image, M, (image.shape[1], image.shape[0]))
```
### Rotation
```python
M = cv2.getRotationMatrix2D ((cX, cY), 45, 1.0)
# cX, cY = rotation center, 45 = angle of rotation, 1.0 = scaling
rotated = cv2.warpAffine(image, M, (w,h))
```
### Resizing
```python
r = 150/image.shape[1] # calculate ratio of scaling to 150 pixels in width
dim = (150, int(image.shape[0]*r)) # new dimensions of resized image
resized = cv2.resize (image, dim, interpolation=cv2.INTER_AREA)
```
`cv2.INTER_NEAREST` : Simply finds "nearest" neighboring pixel and assumes the intensity value
`cv2.INTER_LINEAR`  : Bilinear interpolation (similar to y = mx+b)
`cv2.INTER_AREA`    : Similar to nearest neighbour interpolation
`cv2.INTER_CUBIC`   : bicubic interpolation over 4x4 pixel neighbourhood
`cv2.INTER_LANCZOS4`: bicubic interpolation over 8x8 pixel neighbourhood

### Flipping
```python
flipped_x = cv2.flip(image, 1) # Horizontal flip
flipped_y = cv2.flip(image, 0) # Vertical flip
flipped_xy = cv2.flip(image, -1) # Horizontal and vertical flipped_x
```

### Bitwise operations

```python
bitwiseAnd = cv2.bitwise_and(image1, image2) # sets pixel to 255 if both pixels are >0
bitwiseOr  = cv2.bitwise_or(image1, image2) # sets pixel to 255 if either one pixel >0
bitwiseXor = cv2.bitwise_xor(image1, image2) # sets pixel to 255 if either pixel is >0 but not both
bitwiseNot = cv2.bitwise_not(image1) # inverts the pixel (255 to 0 and 0 to 255)
```
### Masking
```python
mask = np.zeros(image.shape[:2], dtype='uint8') # create an empty canvas mask
cv2.circle(mask, (cX, cY), radius, 255, -1) # make some pixels white, fill with -1
masked = cv2.bitwise_and(image, image, mask=mask)

# This means in pixels which are equal in the two images (image, image) and > 0 from mask, the pixels from image are shown

```
### Splitting and Merging channels
```python
(B, G, R) = cv2.split(image)

merged = cv2.merge([B, G, R])

# To see in respective channels
zeros = np.zeros(image.shape[:2], dtype='uint8')
cv2.imshow('Red', cv2.merge([zeros, zeros, R])) # similarly for G and B
```
### Morphological operations

We use a kernel (structuring element) for Morphological operations but only for comparison tests and not for convolution operations (like in blurring, sharpening, etc)
`kernel = cv2.getStructuringElement`

#### Erosion

A foreground pixel is kept only if all pixels inside the structuring element are > 0. Otherwise, the pixels are set to 0 (background)

Erosion removes small blobs in an image or disconnecting two connected objects

`eroded = cv2.erode(image, None, iterations = i) # None uses 3x3 structuring element`

#### Dilation

The center pixel is set to white if any pixel in the structuring element is > 0

Dilation increase the size of foreground and used for joining broken parts of an image
`diated = cv2.dilate(image, None, iterations = i)`

#### Opening

Erosion followed by dilation: Used to remove small blobs

```python
kernelSize = (3,3)
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, kernelSize)
opening = cv2.morphologyEx(image, cv2.MORPH_OPEN, kernel)
```

#### Closing

Dilation followed by erosion: Used to close holes inside of objects or for connecting components together

```python
kernelSize = (3,3)
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, kernelSize)
closing = cv2.morphologyEx(image, cv2.MORPH_CLOSE kernel)
```

#### Morphological Gradient

Difference between dilation and erosion: Used for determining the outline of an object in an image

```python
kernelSize = (3,3)
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, kernelSize)
gradient = cv2.morphologyEx(image, cv2.MORPH_GRADIENT, kernel)
```

#### Top Hat/White Hat and Black Hat

More suited for grayscale images

##### Top Hat/White Hat
Difference between original (grayscale/single channel) input image and opening: Used to reveal bright regions of an image on dark backgrounds

##### Black Hat
Difference between closing of input image and input image itself: Used to reveal darker regions on brighter backgrounds

```python
kernelSize = (13,5)
rectkernel = cv2.getStructuringElement(cv2.MORPH_RECT, kernelSize)
tophat = cv2.morphologyEx(image, cv2.MORPH_TOPHAT, rectkernel)
blackhat = cv2.morphologyEx(image, cv2.MORPH_BLACKHAT, rectkernel)
```
