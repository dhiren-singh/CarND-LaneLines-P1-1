#**Finding Lane Lines on the Road** 

##Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

## Reflection

###1. Description of pipeline
Steps in pipeline.
####1.1. Convert the image to Gray scale for processing.
```python
gray_image = grayscale(image)
```
####1.2. Smoothening the image by Gaussian Blur.
Here kernel size was kept at 3.

```python
kernel_size = 3
blur_image = gaussian_blur(gray_image,kernel_size)
```
####1.3. Detecting edges using Canny Edge detection.
Threshold for edge detection was kept as 50 to 150.

```python
low_threshold = 50
high_threshold = 150
edge_image = canny(blur_image,low_threshold,high_threshold)
```

####1.4. Region extraction.
Region was extracted in form of a Quadrilateral. Size of Quadrilateral was in proportion of image size.

```python
imshape = edge_image.shape
height, width = imshape[0], imshape[1]
vertices = np.array([[(0+100, height), (width/2-20, height/2+50),
           (width/2+20, height/2+50), (width-50, height)]],dtype=np.int32)
region_image = region_of_interest(edge_image,vertices)
```

####1.5. Lines detection using Hough Line detection.

```python
rho = 2  
theta = np.pi/180
threshold = 10 
min_line_length = 5 
max_line_gap = 1  
lines = hough_lines(region_image, rho, theta, threshold, min_line_length,
                    max_line_gap)
```
####1.6. Draw Lines
#####1.6.1. Segregate lines as Left and Right Lane. Left lane would have Negative slopne while Right lane line would have positive slope. Extrapolate the lines.
```python
for line in lines:
    for x1, y1, x2, y2 in line:
        slope = find_slope(x1, y1, x2, y2)
        if slope == -2 or abs(slope) < 0.5: continue
        max_y = min(max_y, y1, y2)
        if slope < 0: 
            left_lane_lines.append(line)
            left_lane_slopes.append(slope)
        else: 
             right_lane_lines.append(line)
             right_lane_slopes.append(slope)
```
#####1.6.2. Merge segragated lines.
```python
for index, line in enumerate(lines):
    length = line_length(line)
    line_length_sum = line_length_sum + length
    line_count = line_count + 1
    for x1,y1,x2,y2 in line:
        weighted_sum_x = weighted_sum_x + (x1 * length) + (x2 * length)
        weighted_sum_y = weighted_sum_y + (y1 * length) + (y2 * length)
        weighted_sum_slope = weighted_sum_slope + (slopes[index] * length)

wavg_x = int(weighted_sum_x / (line_length_sum*2))
wavg_y = int(weighted_sum_y / (line_length_sum*2))
wslope = weighted_sum_slope / line_length_sum
```
#####1.6.4. Draw lane lines on a blank image.
```python
min_y = img.shape[0] - 1
# X value for lane lines
lleft_x, lright_x = find_x(lslope, lx, ly, [min_y, max_y])
rleft_x, rright_x = find_x(rslope, rx, ry, [max_y, min_y])
final_lines = [[[lleft_x, min_y, lright_x, max_y]], [[rleft_x, max_y, 	rright_x, min_y]]]
# Draw lines
draw_lines(img, final_lines, color, thickness)
```
#####1.6.5. Merge original image and the lane line image.
```python
weighted_image = weighted_img(line_img, image)
```


###2. Shortcomings with current pipeline

####2.1. Consider only straight line. Due to this on curved roads lane detection would be limited to very small section.
####2.2. How do we detect lanes on a rainy day? Or  heavy traffic?


###3. Possible improvements to pipeline

####3.1. Find better way to modify Segregation of lines and Extrapolating them.