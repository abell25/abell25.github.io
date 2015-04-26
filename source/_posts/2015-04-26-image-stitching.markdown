---
layout: post
title: "image stitching"
date: 2015-04-26 11:08:44 -0500
comments: true
categories: 
---

<div class="container">

<blockquote>
  <p>We’ll show how to stitch images together using octave/matlab.</p>
</blockquote>

<p>The source code can be found <a href="https://github.com/abell25/image_stitching">here</a></p>

<p><div class="toc">
<ul>
<li><a href="#image-stitching">Image Stitching</a><ul>
<li><a href="#1-getting-correspondance-points-between-images">1. Getting correspondance points between images</a></li>
<li><a href="#2-calculating-the-homography-matrix">2. Calculating the homography matrix</a></li>
<li><a href="#3-warping-between-image-planes">3. Warping between image planes</a></li>
<li><a href="#4-applying-warping-to-images">4. Applying warping to images</a></li>
<li><a href="#5-applying-warping-to-pictures-taken">5. Applying warping to pictures taken</a></li>
<li><a href="#6-project-on-a-rectangle-in-an-image">6. Project on a rectangle in an image</a></li>
<li><a href="#conclusion">Conclusion.</a></li>
</ul>
</li>
</ul>
</div>
</p>



<h2 id="1-getting-correspondance-points-between-images">1. Getting correspondance points between images</h2>

<p>Before we start we need points to map between the 2 images.  We could use some kind of feature detection to actimate finding these points but for now we’ll retrieve them manually. <br>
In octave/matlab, we can create a function that uses <code>ginput()</code> to retrieve input clicks:</p>



<pre class="prettyprint"><code class=" hljs scilab"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">points</span> = <span class="hljs-title">getPoints</span><span class="hljs-params">(image)</span></span>
  done = false;
  while !done,
      imshow(image);
      title(<span class="hljs-string">'Click on correspondence points, press ENTER when complete'</span>);
      hold on;
      <span class="hljs-matrix">[xs ys]</span> = ginput();
      points = <span class="hljs-matrix">[xs ys]'</span>;
      plot(xs, ys, <span class="hljs-string">'rx'</span>, <span class="hljs-string">'linewidth'</span>, <span class="hljs-number">2</span>);
      <span class="hljs-keyword">for</span> k=<span class="hljs-number">1</span>:rows(xs),
          text(xs(k), ys(k)-<span class="hljs-number">10</span>, num2str(k))
      <span class="hljs-keyword">end</span>
      title(<span class="hljs-string">'do these look ok? Type yes or no'</span>);
      done = yes_or_no(<span class="hljs-string">'do these look ok?'</span>)
      title(<span class="hljs-string">''</span>);
      hold off;
  <span class="hljs-keyword">end</span>
<span class="hljs-function"><span class="hljs-keyword">endfunction</span></span>
</code></pre>

<p>For this function the user clicks points and when the user presses enter, it shows the user the points and asks if the user wants to accept the points.  If they type <code>no</code> that the image clears and they can select new points.  The function returns the points when the user types <code>yes</code>.</p>



<h2 id="2-calculating-the-homography-matrix">2. Calculating the homography matrix</h2>

<p>We are using the projective transform, so we are using homogenuous coordinates: <br>
<script type="math/tex; mode=display" id="MathJax-Element-1"> \left( \begin{array}{c} u \\ v \\ w \end{array} \right) =
  \left( \begin{array}{ccc}
h_{00} & h_{01} & h_{02} \\
h_{10} & h_{11} & h_{12} \\
h_{20} & h_{21} & 1 \end{array} \right)
\left( \begin{array}{c} x \\ y \\ 1 \end{array} \right) </script> <br>
The projective transform has 8 degrees of freedom so we can set <script type="math/tex" id="MathJax-Element-2">h_{22} = 1</script>, giving us the transformed points: <br>
<script type="math/tex; mode=display" id="MathJax-Element-3">\begin{align*}
x' &= u/w = \frac{h_{00}x + h_{01}y + h_{02}}{h_{20}x + h_{21}y + 1} &
y' &= v/w = \frac{h_{10}x + h_{11}y + h_{12}}{h_{20}x + h_{21}y + 1} 
\end{align*}
</script></p>

<p>We end up with a system of equations of the form:</p>



<p><script type="math/tex; mode=display" id="MathJax-Element-4">
\begin{align}
\left( \begin{array}{cccccccc}
&&&&&\vdots&& \\
x_i & y_i & 1 & 0 & 0 & 0 & -x_ix_i' & -y_ix_i' \\
0 & 0 & 0 & x_i & y_i & 1 & -x_iy_i' & -y_iy_i' \\
&&&&&\vdots&& \end{array} \right)
\left( \begin{array}{c} h_{00} \\ h_{01} \\ h_{02} \\ h_{10} \\ h_{11} \\ h_{12} \\ h_{20} \\ h_{21} \end{array} \right) &=
\left( \begin{array}{c} \vdots \\ x_i' \\ y_i' \\ \vdots \end{array} \right)
\\
A \quad\quad\quad h\quad\; &= \quad b
\end{align}
</script></p>

<p>This is a overdetermined system of equations that can be solved by multiplying both sides by the transpose of the matrix <script type="math/tex" id="MathJax-Element-5">A</script> solving the resultant system of equations:  <br>
<script type="math/tex; mode=display" id="MathJax-Element-6"> \begin{align*}
Ah &= b \\
A^TAh &= A^Tb \\
h &= (A^TA)^+A^Tb
\end{align*}</script> <br>
We add 1 for the entry <script type="math/tex" id="MathJax-Element-7">h_{22}</script> and reshape h to <script type="math/tex" id="MathJax-Element-8">3\times3</script> to get our homography matrix <script type="math/tex" id="MathJax-Element-9">H</script>. <br>
Writing this function in octave, we construct <script type="math/tex" id="MathJax-Element-10">A</script> 2 rows at a time (there are alternate formulations that avoid a <code>for</code> loop as well).</p>



<pre class="prettyprint"><code class=" hljs matlab"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">[H]</span> = <span class="hljs-title">computeH</span><span class="hljs-params">(t1, t2)</span></span>
  points1 = <span class="hljs-transposed_variable">t1'</span>;
  points2 = <span class="hljs-transposed_variable">t2'</span>;

  A = <span class="hljs-built_in">zeros</span>(<span class="hljs-number">2</span>*N, <span class="hljs-number">8</span>);
  h = <span class="hljs-built_in">zeros</span>(<span class="hljs-number">8</span>,<span class="hljs-number">1</span>);
  b = t2(:);

  <span class="hljs-comment">% x' = h00 x0 + h01 y0 + h02 / h20 x0 + h21 y0 + 1</span>
  <span class="hljs-comment">% y' = h10 x0 + h11 y0 + h12 / h20 x0 + h21 y0 + 1</span>

  <span class="hljs-keyword">for</span> k=<span class="hljs-number">1</span>:N,
    idx = (k-<span class="hljs-number">1</span>)*<span class="hljs-number">2</span> + <span class="hljs-number">1</span>;
    x  = points1(k, <span class="hljs-number">1</span>);
    y  = points1(k, <span class="hljs-number">2</span>);
    xp = points2(k, <span class="hljs-number">1</span>);
    yp = points2(k, <span class="hljs-number">2</span>);
    A(idx,   :) = <span class="hljs-matrix">[x y <span class="hljs-number">1</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> -x*xp -y*xp]</span>;
    A(idx+<span class="hljs-number">1</span>, :) = <span class="hljs-matrix">[<span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> x y <span class="hljs-number">1</span> -x*yp -y*yp]</span>;
  <span class="hljs-keyword">end</span>

  <span class="hljs-comment">% Ah = b  -&gt;  h = A \ b</span>
  h = A \ b;
  h(<span class="hljs-number">9</span>) = <span class="hljs-number">1</span>;

  H = <span class="hljs-built_in">reshape</span>(h,<span class="hljs-number">3</span>,<span class="hljs-number">3</span>)<span class="hljs-string">';
endfunction
</span></code></pre>



<h2 id="3-warping-between-image-planes">3. Warping between image planes</h2>

<p>Now that we have a projective matrix we can transform one image to another’s frame and merge them.  The code to generate the projection of the image and merge it with a reference image is in <code>warpImage.m</code>. </p>

<p>We take the input image, generate a list of all coordinates for the image using <code>meshgrid</code> and apply the projective matrix to the coordinates to obtain the list of coordinates that the original image is projected to in order to get it’s bounding box.  Using the max/max of these coordinates we calculate the needed dimensions for the new image.  We then apply an inverse mapping of H on the integer values for the dimensions of the new image to get the pixel values for the new image.  Since the projective matrix can output negative values, we move all the coordinate values over to make them positive by subtracting by the min of the x and y dimensions.  In order to create the merged image we reuse this same image but superimpose the reference image, adding the same offset that we used to avoid negative indices.  The results are shown in the next section.</p>



<h2 id="4-applying-warping-to-images">4. Applying warping to images</h2>

<p>The first images we merge are of a crop circle taken from different views: <br>
</p><center>First image of the crop circle</center> <br>
<img src="https://lh3.googleusercontent.com/-29wfYgk6yso/VTRFrKF12lI/AAAAAAAAACE/3Pq6ZqDdDa0/s0/crop1.jpg" alt="crop1" title="crop1.jpg"> <br>
<center>Second image of the same crop circle from a different view point</center> <br>
<img src="https://lh3.googleusercontent.com/-HFmGnIZ0euE/VTRFxxi7EiI/AAAAAAAAACQ/ZilV__vES3Y/s0/crop2.jpg" alt="crop2" title="crop2.jpg"><p></p>

<p>The homography matrix from the first image to the second was calculated for a set of correspondence points.  Below is the result of projecting the image and the merging of the projected image with the second image.</p>

<p></p><center> The warped crop circle image.</center> <br>
<img src="https://lh3.googleusercontent.com/-7WHBDPY_n8g/VTRFSL_J_0I/AAAAAAAAAB4/ZtQEpy5J4kY/s0/crop_warped.jpg" alt="crop_warped" title="crop_warped.jpg"><p></p>

<p></p><center> The merge crop circle images</center> <br>
<img src="https://lh3.googleusercontent.com/-iIBvkTqZ068/VTRAPea92yI/AAAAAAAAABk/zgfn2P9m2-o/s0/crop_merged.jpg" alt="crop_merged" title="crop_merged.jpg"><p></p>

<p>Next we used aerial images of a capital building.  We used a small program to collect correspondence points.  the images and the points collected are shown here: <br>
</p><center>Chosen correspondence points for the building image example</center> <br>
<img src="https://lh3.googleusercontent.com/-5EoD-ezl26U/VTRQExn-FFI/AAAAAAAAAC0/8dFEqE14-oc/s0/building_points.jpg" alt="building_points" title="building_points.jpg"><p></p>

<p>The homography matrix was calculated using the points collected.  The warped and merged images are shown below: <br>
</p><center>The first image of the building projected into the frame of the second</center> <br>
<img src="https://lh3.googleusercontent.com/-Lyl5_066qL8/VTRjI67SdDI/AAAAAAAAADU/DC10TwGEEX4/s0/building_warped.jpg" alt="enter image description here" title="building_warped.jpg"> <p></p>

<p></p><center>The projection of the first image is stitched to the second. </center> <br>
<img src="https://lh3.googleusercontent.com/-bNb9H-7SPQo/VTRjfmHSeqI/AAAAAAAAADk/6zMS9hgwlDY/s0/building_merged.jpg" alt="enter image description here" title="building_merged.jpg"><p></p>



<h2 id="5-applying-warping-to-pictures-taken">5. Applying warping to pictures taken</h2>

<p>We now apply image stitches to some pictures taken.  I took 4 pictures rotating to the right for each image. The first 2 images are shown here with the correspondence points used, the projection, and the merged images in the lower right: <br>
</p><center>Two images stitched together<center> <br>
<img src="https://lh3.googleusercontent.com/-VplpyLiyDOc/VTR13Irh1UI/AAAAAAAAAEE/EiNodYhPFNo/s0/room12_grid.jpg" alt="room12_grid" title="room12_grid.jpg"></center></center><p></p>

<p>The next 2 images with thier correspondence points, projection, and merged images are shown here: <br>
<img src="https://lh3.googleusercontent.com/-TzLlrJ6khfo/VTR2WbDiyII/AAAAAAAAAEQ/-PZjWCudl84/s0/room34_grid.jpg" alt="room34_grid" title="room34_grid.jpg"></p>

<p>The correspondence points for these 2 stitched images were gathered and all four images were stitched together: <br>
</p><center>Room images 3 and 4 stitched together</center> <br>
<img src="https://lh3.googleusercontent.com/-tHnSRjWv1Fs/VTR2pjuZMTI/AAAAAAAAAEc/JQjkD2toRo4/s0/room1234_grid.jpg" alt="room1234_grid" title="room1234_grid.jpg"><p></p>

<p>Stitching 2 images together looks good, but stitching all four starts to cause increased distortion (as expected).  Projecting the left 2 images to the right 2 images shows a similar distortion, but on the other side.</p>

<p></p><center>Projecting the right two stitched images into the left two stitched images.</center> <br>
<img src="https://lh3.googleusercontent.com/-KIGTyninWbM/VTR6pMDngNI/AAAAAAAAAE4/2XsPC8M18pQ/s0/room1234_otherway_grid.jpg" alt=" room1234_grid_other_way" title="room1234_otherway_grid.jpg"><p></p>



<h2 id="6-project-on-a-rectangle-in-an-image">6. Project on a rectangle in an image</h2>

<p>For projecting onto a surface, we just needed to make sure we draw the smaller projection onto the image.  The buildings image is show drawn on the computer screen in the room picture #2: <br>
</p><center>Building image projected onto the computer screen.</center> <br>
<img src="https://lh3.googleusercontent.com/-CVrOCNnhLlY/VTSArLR4HbI/AAAAAAAAAFg/H9N0tC5LJJ8/s0/computer_screen2.jpg" alt="computerScreen" title="computer_screen2.jpg"> <br>
For the coordinates, we just made a square around the projected images border, and another square around the computer screen.<p></p>



<h2 id="conclusion">Conclusion.</h2>

<p>Projecting from 1 image to another is pretty easy, granted we have some correspondence points.  To make this more interesting we could apply a feature detector to find these points automatically (such as sift).  Also some filtering would make the stitched images look nicer.</p></div>
