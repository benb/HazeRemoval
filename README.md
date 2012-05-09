HazeRemoval
===========

Overview
====

Haze removal (or dehazing) is the process in computer vision that attempts to remove haze from images. The process implemented for this project utilizes the dark channel of images.  The dark channel for an area within an image, is the color channel of the area that has the lowest intensity.  

**I**(**x**) = **J**(**x**)_t_(**x**) + **A**(1 − _t_(**x**)), is the formula largely used to describe haze images.

#Haze Removal

---

### Dark Channel

To calculate the dark channel I iterated over every 15x15 patch in the image. For each patch we find the color channel that has the minimum value and used that patch in the final dark channel combination.

![dark channel](https://github.com/KnownSubset/HazeRemoval/raw/master/dark_channel.png "dark channel") 
######Pseudo-Code

    ```matlab
    %image is already available
    cols = size(image, 2);
    rows = size(image, 1);
    dark_channel = zeros(rows, cols);
    for ix = 8:rows-8
        for iy = 8:cols-8
            dark_channel(ix-7:ix+7, iy-7:iy+7) = find_minimum(image(ix-7:ix+7, iy-7:iy+7));
        end
    end
    ```

### Atmospheric Light

The atmospheric light is calculated using the dark channel and the original image.  To ensure that we select the brightest pixels that are not objects within the image, you must look at the pixels in the dark channel.  The simple approach used in the paper finds the .1% brightest pixels of the dark channel, then selects the maximum value from those pixels in the original image.

######Pseudo-Code

    ```matlab
    %image & dark_channel are already available
    [cols rows] = size(dark_channel);
    [vector index] = sort(reshape(dark_channel, cols * rows, []), 1, 'descend');
    % take the brightest .1% of the dark channel
    limit = round(cols * rows /1000);
    for ix = 1:limit
        vector(ix) = max(image(floor(index(ix)/rows)+1, mod(index(ix),rows)+1,:));
    end
    atmospheric_light= max(vector(1:limit));
    ```

### Transmission

The transmission is the medium transmission describing the portion of the light that is not scattered and reaches the camera.

![transmission](https://github.com/KnownSubset/HazeRemoval/raw/master/transmission.png "transmission") 

######Pseudo-Code

    ```matlab
    cols = size(image, 2);
    rows = size(image, 1);
    t = zeros(rows, cols);
    w = .95;
    for ix = 8:rows-8
        for iy = 8:cols-8
            t(ix-7:ix+7, iy-7:iy+7) = calculate_transmission(image(ix-7:ix+7, iy-7:iy+7), atmospheric_light, w);
        end
    end
    ```
    
The paper mentions that ω is parameter for each 15x15 patch of the image, that can be fined tuned to keep more haze for the distant objects. I did the same thing that the paper did and fixed it to 0.95 for all results reported.  As an afterthought since the paper's process is able to generate a depth map, it should be able to reconfigure ω.  After generating the first of the depth map, you could reprocess the image utilizing the depth map info for ω.

### Scene Radiance

This is the final image that will have the hazyiness removed.  

![scene radiance](https://github.com/KnownSubset/HazeRemoval/raw/master/radiance.png "scene radiance") 

######Pseudo-Code

    ```matlab
    cols = size(image, 2);
    rows = size(image, 1);
    r = zeros(rows, cols, 3);
    t(:,:,1) = transmission;
    for ic = 1:3
        for ix = 1:rows
            for iy = 1:cols
                r(ix, iy, ic) = (image(ix, iy, ic) - atmospheric_light) / (max(t(ix, iy), .1)) + atmospheric_light;
            end
        end
    end
    ```

## Results

*Dark Channel
![ny 12 dc](https://github.com/KnownSubset/HazeRemoval/raw/master/ny12_dc.jpg "ny 12 dc") 

*Transmission
![ny 12 t](https://github.com/KnownSubset/HazeRemoval/raw/master/ny12_t.jpg "ny 12 t")   

*Haze
![ny 12 haze](https://github.com/KnownSubset/HazeRemoval/raw/master/ny12_haze.jpg "ny 12 haze")

*Final
![ny 12 dehaze](https://github.com/KnownSubset/HazeRemoval/raw/master/ny12_dehaze.jpg "ny 12 dehaze") 

 = = =

*Dark Channel
![ny 17 dc](https://github.com/KnownSubset/HazeRemoval/raw/master/ny17_dc.jpg "ny 17 dc") 

*Transmission
![ny 17 t](https://github.com/KnownSubset/HazeRemoval/raw/master/ny17_t.jpg "ny 17 t")   

*Haze
![ny 17 haze](https://github.com/KnownSubset/HazeRemoval/raw/master/ny17_haze.jpg "ny 17 haze")

*Final
![ny 17 dehaze](https://github.com/KnownSubset/HazeRemoval/raw/master/ny17_dehaze.jpg "ny 17 dehaze") 

 = = =

*Dark Channel
![cones dc](https://github.com/KnownSubset/HazeRemoval/raw/master/cones_dc.jpg "cones dc") 

*Transmission
![cones t](https://github.com/KnownSubset/HazeRemoval/raw/master/cones_t.jpg "cones t")   

*Haze
![cones haze](https://github.com/KnownSubset/HazeRemoval/raw/master/cones_haze.jpg "cones haze")

*Final
![cones dehaze](https://github.com/KnownSubset/HazeRemoval/raw/master/cones_dehaze.jpg "cones dehaze") 

 = = =

*Dark Channel
![stadium dc](https://github.com/KnownSubset/HazeRemoval/raw/master/stadium_dc.jpg "stadium dc") 

*Transmission
![stadium t](https://github.com/KnownSubset/HazeRemoval/raw/master/stadium_t.jpg "stadium t")   

*Haze
![stadium haze](https://github.com/KnownSubset/HazeRemoval/raw/master/stadium_haze.jpg "stadium haze")

*Final
![stadium dehaze](https://github.com/KnownSubset/HazeRemoval/raw/master/stadium_dehaze.jpg "stadium dehaze") 

 = = =

*Dark Channel
![toys dc](https://github.com/KnownSubset/HazeRemoval/raw/master/toys_dc.jpg "toys dc") 

*Transmission
![toys t](https://github.com/KnownSubset/HazeRemoval/raw/master/toys_t.jpg "toys t")   

*Haze
![toys haze](https://github.com/KnownSubset/HazeRemoval/raw/master/toys_haze.jpg "toys haze")

*Final
![toys dehaze](https://github.com/KnownSubset/HazeRemoval/raw/master/toys_dehaze.jpg "toys dehaze") 

 = = =

    operation       |	toys 		|  stadium	  	|   cones  		|   ny17 			|  ny12 		|  ny night scene 
    dark channel    |	0:2.970111 	|   0:5.439195  |   0:2.899942  |   0:13.287957 	|  1:-52.578556 |  3:28.651487 
    atmospheric     |	0:0.017480	|   0:0.023223  |   0:0.011458  |   0:0.056524		|  0:0.031109	|  0:1.098448  
    transmission    |	0:4.333701	|   0:7.725875  |   0:4.136798  |   0:18.308125		|  0:10.956740	|  5:-7.965656 
    radiance        |	0:0.016281  |   0:0.033103  |   0:0.015385  |   0:0.139510		|  0:0.048617	|  0:1.840210  
    total           |	0:7.337573  |   0:13.221396 |   0:7.063583  |   0:32.026976 	|  1:-41.542090 |  8:23.624489 