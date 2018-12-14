## Smoothing or Blurring


#### Averaging

All pixels in the kernel matrix contribute equally towards averaging

`blurred = cv2.blur(image, (kX, kY))`

#### Gaussian Blurring

Uses a weighted mean instead of a simple mean. Neighbouring pixels contribute more 'weight' towards average than far out pixels.

`blurred = cv2.GaussianBlur(image, (kX, kY), 0)`

The last parameter is standard deviation (sigma) of the Gaussian kernel. Setting it to '0' will let opencv automatically calculate the value based on kernel size.

#### Median Blurring

Used mainly to remove salt and pepper noise from image and is robust to outliers.

Takes a square kernel as input and replaces the center pixel with the median of the kernel neighbourhood.

`blurred = cv2.medianBlur(image, k)`

#### Bilateral Blurring

This is mainly used to preserve edges while blurring an image.

It is done by using two Gaussians. First Gaussian only considers spatial neighbors and second one models the pixel intensity of the neighborhood. This ensures that only pixels with similar intensity are included in the computation of the blur.


`blurred = cv2.bilateralFilter(image, diameter, sigmaColor, sigmaSpace)`

diameter     =  pixel neighbourhood size (like kernel size)
sigmaColor   =  color standard deviation (larger value will include more colors in neighborhood)
sigmaSpace   =  space standard deviation (larger value means farther pixels have more influence in blurring calculation)
