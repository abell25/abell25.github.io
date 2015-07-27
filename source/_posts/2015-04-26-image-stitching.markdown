---
layout: post
title: "image stitching"
date: 2015-04-26 11:08:44 -0500
comments: true
categories: 
---

# Image Stitching

> We'll show how to stitch images together using octave/matlab.

The source code can be found [here](https://github.com/abell25/image_stitching)

[toc]

## 1. Getting correspondance points between images
Before we start we need points to map between the 2 images.  We could use some kind of feature detection to actimate finding these points but for now we'll retrieve them manually.
In octave/matlab, we can create a function that uses `ginput()` to retrieve input clicks:
```
function points = getPoints(image)
  done = false;
  while !done,
      imshow(image);
      title('Click on correspondence points, press ENTER when complete');
      hold on;
      [xs ys] = ginput();
      points = [xs ys]';
      plot(xs, ys, 'rx', 'linewidth', 2);
      for k=1:rows(xs),
          text(xs(k), ys(k)-10, num2str(k))
      end
      title('do these look ok? Type yes or no');
      done = yes_or_no('do these look ok?')
      title('');
      hold off;
  end
endfunction

```

For this function the user clicks points and when the user presses enter, it shows the user the points and asks if the user wants to accept the points.  If they type `no` that the image clears and they can select new points.  The function returns the points when the user types `yes`.

## 2. Calculating the homography matrix
We are using the projective transform, so we are using homogenuous coordinates:

![enter image description here](https://lh3.googleusercontent.com/-LebFrH8AkpQ/VbWkT3VMe0I/AAAAAAAAALo/WhbBCYeF6hM/s0/Selection_100.png "Selection_100.png")

The projective transform has 8 degrees of freedom so we can set $h_{22} = 1$, giving us the transformed points:

![enter image description here](https://lh3.googleusercontent.com/-kUP_geJBVgg/VbWkk7OIXHI/AAAAAAAAAMA/Zf0xDKlcNWI/s0/Selection_102.png "Selection_102.png")

We end up with a system of equations of the form:

![enter image description here](https://lh3.googleusercontent.com/-WY2S1W2N5QM/VbWkqoNpkaI/AAAAAAAAAMM/wu2v7_CP3q0/s0/Selection_103.png "Selection_103.png")

This is a overdetermined system of equations that can be solved by multiplying both sides by the transpose of the matrix $A$ solving the resultant system of equations: 

![enter image description here](https://lh3.googleusercontent.com/-O115b5x4nu0/VbWky4l8XGI/AAAAAAAAAMY/TR6HJiCOU3k/s0/Selection_104.png "Selection_104.png")

We add 1 for the entry $h_{22}$ and reshape h to $3\times3$ to get our homography matrix $H$.
Writing this function in octave, we construct $A$ 2 rows at a time (there are alternate formulations that avoid a `for` loop as well).

```
function [H] = computeH(t1, t2)
  points1 = t1';
  points2 = t2';

  A = zeros(2*N, 8);
  h = zeros(8,1);
  b = t2(:);

  % x' = h00 x0 + h01 y0 + h02 / h20 x0 + h21 y0 + 1
  % y' = h10 x0 + h11 y0 + h12 / h20 x0 + h21 y0 + 1

  for k=1:N,
    idx = (k-1)*2 + 1;
    x  = points1(k, 1);
    y  = points1(k, 2);
    xp = points2(k, 1);
    yp = points2(k, 2);
    A(idx,   :) = [x y 1 0 0 0 -x*xp -y*xp];
    A(idx+1, :) = [0 0 0 x y 1 -x*yp -y*yp];
  end
  
  % Ah = b  ->  h = A \ b
  h = A \ b;
  h(9) = 1;
  
  H = reshape(h,3,3)';
endfunction

```

## 3. Warping between image planes
Now that we have a projective matrix we can transform one image to another's frame and merge them.  The code to generate the projection of the image and merge it with a reference image is in `warpImage.m`. 

We take the input image, generate a list of all coordinates for the image using `meshgrid` and apply the projective matrix to the coordinates to obtain the list of coordinates that the original image is projected to in order to get it's bounding box.  Using the max/max of these coordinates we calculate the needed dimensions for the new image.  We then apply an inverse mapping of H on the integer values for the dimensions of the new image to get the pixel values for the new image.  Since the projective matrix can output negative values, we move all the coordinate values over to make them positive by subtracting by the min of the x and y dimensions.  In order to create the merged image we reuse this same image but superimpose the reference image, adding the same offset that we used to avoid negative indices.  The results are shown in the next section.
## 4. Applying warping to images
The first images we merge are of a crop circle taken from different views:
<center>First image of the crop circle</center>
![crop1](https://lh3.googleusercontent.com/-29wfYgk6yso/VTRFrKF12lI/AAAAAAAAACE/3Pq6ZqDdDa0/s0/crop1.jpg "crop1.jpg")
<center>Second image of the same crop circle from a different view point</center>
![crop2](https://lh3.googleusercontent.com/-HFmGnIZ0euE/VTRFxxi7EiI/AAAAAAAAACQ/ZilV__vES3Y/s0/crop2.jpg "crop2.jpg")

The homography matrix from the first image to the second was calculated for a set of correspondence points.  Below is the result of projecting the image and the merging of the projected image with the second image.

<center> The warped crop circle image.</center>
![crop_warped](https://lh3.googleusercontent.com/-7WHBDPY_n8g/VTRFSL_J_0I/AAAAAAAAAB4/ZtQEpy5J4kY/s0/crop_warped.jpg "crop_warped.jpg")


<center> The merge crop circle images</center>
![crop_merged](https://lh3.googleusercontent.com/-iIBvkTqZ068/VTRAPea92yI/AAAAAAAAABk/zgfn2P9m2-o/s0/crop_merged.jpg "crop_merged.jpg")

Next we used aerial images of a capital building.  We used a small program to collect correspondence points.  the images and the points collected are shown here:
<center>Chosen correspondence points for the building image example</center>
![building_points](https://lh3.googleusercontent.com/-5EoD-ezl26U/VTRQExn-FFI/AAAAAAAAAC0/8dFEqE14-oc/s0/building_points.jpg "building_points.jpg")

The homography matrix was calculated using the points collected.  The warped and merged images are shown below:
<center>The first image of the building projected into the frame of the second</center>
![enter image description here](https://lh3.googleusercontent.com/-Lyl5_066qL8/VTRjI67SdDI/AAAAAAAAADU/DC10TwGEEX4/s0/building_warped.jpg "building_warped.jpg") 

<center>The projection of the first image is stitched to the second. </center>
![enter image description here](https://lh3.googleusercontent.com/-bNb9H-7SPQo/VTRjfmHSeqI/AAAAAAAAADk/6zMS9hgwlDY/s0/building_merged.jpg "building_merged.jpg")
## 5. Applying warping to pictures taken
We now apply image stitches to some pictures taken.  I took 4 pictures rotating to the right for each image. The first 2 images are shown here with the correspondence points used, the projection, and the merged images in the lower right:
<center>Two images stitched together<center>
![room12_grid](https://lh3.googleusercontent.com/-VplpyLiyDOc/VTR13Irh1UI/AAAAAAAAAEE/EiNodYhPFNo/s0/room12_grid.jpg "room12_grid.jpg")

The next 2 images with thier correspondence points, projection, and merged images are shown here:
![room34_grid](https://lh3.googleusercontent.com/-TzLlrJ6khfo/VTR2WbDiyII/AAAAAAAAAEQ/-PZjWCudl84/s0/room34_grid.jpg "room34_grid.jpg")

The correspondence points for these 2 stitched images were gathered and all four images were stitched together:
<center>Room images 3 and 4 stitched together</center>
![room1234_grid](https://lh3.googleusercontent.com/-tHnSRjWv1Fs/VTR2pjuZMTI/AAAAAAAAAEc/JQjkD2toRo4/s0/room1234_grid.jpg "room1234_grid.jpg")

Stitching 2 images together looks good, but stitching all four starts to cause increased distortion (as expected).  Projecting the left 2 images to the right 2 images shows a similar distortion, but on the other side.

<center>Projecting the right two stitched images into the left two stitched images.</center>
![ room1234_grid_other_way](https://lh3.googleusercontent.com/-KIGTyninWbM/VTR6pMDngNI/AAAAAAAAAE4/2XsPC8M18pQ/s0/room1234_otherway_grid.jpg "room1234_otherway_grid.jpg")

## 6. Project on a rectangle in an image
For projecting onto a surface, we just needed to make sure we draw the smaller projection onto the image.  The buildings image is show drawn on the computer screen in the room picture #2:
<center>Building image projected onto the computer screen.</center>
![computerScreen](https://lh3.googleusercontent.com/-CVrOCNnhLlY/VTSArLR4HbI/AAAAAAAAAFg/H9N0tC5LJJ8/s0/computer_screen2.jpg "computer_screen2.jpg")
For the coordinates, we just made a square around the projected images border, and another square around the computer screen.

## Conclusion.

Projecting from 1 image to another is pretty easy, granted we have some correspondence points.  To make this more interesting we could apply a feature detector to find these points automatically (such as sift).  Also some filtering would make the stitched images look nicer.
