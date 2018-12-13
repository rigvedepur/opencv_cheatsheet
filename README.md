
image: numpy array (8 bit unsigned)

#### Loading an image
`cv2.imread(image)`

#### Display an image
cv2.imshow('Image', image)

#### Save a numpy array to an image file
`cv2.imwrite('newimage.jpg', image)`

#### Shape of image (height and width)
```python
(h,w) = image.shape[:2]
(cX, cY) = (w // 2, h // 2)
```

#### Crop an image
`tl = image[0:cY, 0:cX]`

#### Initializing an empty image (canvas)
`canvas = np.zeros((300,300,3), dtype='uint8')`

#### Drawing shapes on image
```python
# last parameter = -1 will fill the shape with color
blue = (255,0,0)
cv2.line(canvas, (0,0), (300,300), blue, 3)
cv2.rectangle(canvas, (0,0), (300,300), blue, 2)
cv2.circle(canvas, (centerX, centerY), radius, blue, 3)
```
#### Translation
```python
M = np.float32([[1,0,x], [0,1,y]]) # x,y = shift in x and y
shifted = cv2.warpAffine(image, M, (image.shape[1], image.shape[0]))
```
#### Rotation
```python
M = cv2.getRotationMatrix2D ((cX, cY), 45, 1.0)
# cX, cY = rotation center, 45 = angle of rotation, 1.0 = scaling
rotated = cv2.warpAffine(image, M, (w,h))
```
#### Resizing
```python
r = 150/image.shape[1] # calculate ratio of scaling to 150 pixels in width
dim = (150, int(image.shape[0]*r)) # new dimensions of resized image
resized = cv2.resize (image, dim, interpolation=cv2.INTER_AREA)


```
