# Project: Search and Sample Return
---

## Introduction 
The aim of the project was to map an unknown environment of a rover using camera mounted on it. The goal was to map 40% of the environment with 60% minimum fidelity, and locate minimum one rock

## Notebook Analysis
### Image Processing

#### Perspective Transform
The rover camera faces forward. Perspective transform is used to convert this to a top view. 
Also as the field of vision does not cover the area behind the camera, it cannot be assumed obstacles.
A mask is created to take this into account

![Perspective_Transform](https://github.com/mahajanrevant/Search-Sample-and-Return/blob/master/Pictures/Perspective_Transform.PNG)

#### Threshold 
The navigable terrain, obstacles, rocks are identified using simple rgb threshold interpreted as `[red, blue, green]`
* All pixels below `[160, 160, 160]` classify under navigable terrain - `color_thresh(img, rgb_thresh=(160, 160, 160))`
* All pixels above `[160, 160, 160]` classify under obstacles - `obstacle_thresh(img, rgb_thresh=(160, 160, 160))`
* All pixels below `[160, 160, 160]` classify under rocks - `rock_thresh(img, rock_thresh=(130, 180, 100,170,0,30))`

![Obstacle](https://github.com/mahajanrevant/Search-Sample-and-Return/blob/master/Pictures/Obstacle.PNG)
__Obstacle__

![Path](https://github.com/mahajanrevant/Search-Sample-and-Return/blob/master/Pictures/Path.PNG)
__Navigable Terrain__

![Rock](https://github.com/mahajanrevant/Search-Sample-and-Return/blob/master/Pictures/Rock.PNG)
__Rock__

#### World Coordinates 
The above processes give results in terms of rover centric coordinates. These coodrinates are then converted into world-centric coordinates. The mean of all the navigable terrain is taken and is represented by the line. The angle of this line with respect to the x-axis is the navigation angles. 

![Coordinate_Transform]https://github.com/mahajanrevant/Search-Sample-and-Return/blob/master/Pictures/Coordinate_Transform.PNG)

### Mapping 
* Red Channel - Obstacles
* Blue Channel - Navigable Terrain 
* All Channel - Rocks

```
    data.worldmap[y_world, x_world,2] = 255
    data.worldmap[obsy_world, obsx_world,0] = 255
```
To sort out the conflict between navigable terrain and obstacle, a trivial methodolgy is used,
Every pixel having a navigable terrain cannot be in the obstacle channel

```
    nav_pix = data.worldmap[:,:,2] > 0
    data.worldmap[nav_pix , 0] = 0
```
Rocks are mapped as below

```
    if rock_threshed.any():
        rockxpix,rockypix = rover_coords(threshed)
        rock_x_world, rock_y_world =  pix_to_world(rockxpix, rockypix, xpos, ypos, yaw, world_size, scale)
        data.worldmap[rock_y_world, rock_x_world,:] = 255
```
## Autonomous Navigation and Mapping 

### Perception
This is handled by the perception_step() method in perception.py. This method is similar to what is performed in notebook analysis.

Perspective transform is performed on the camera image. Thresholding is done and everything is converteed into the world coordinates.
Instead of updating each channel with 255, a small value is added over time.

```
    Rover.worldmap[obsy_world, obsx_world, 0] += 1 # Obstacle map
    Rover.worldmap[y_world, x_world, 2] += 5 # Navigable Terrain
```
Based on the navigable terrain, naigation angles were updates. This is an essential part as the rover will not move without there being valid values of navigable terrrain.

Also, rocks are updated in the yellow channel.
```
        Rover.vision_image[:,:,1] = rock_threshed * 255
```
All the necessary changes are made in the rover object before returning it.

### Autonomous Mapping
The decision step of the Rover is handled in the decision_step() method in decision.py.This contains the code to drive rover based on inferences made by the perception_step() method in perception.py

`if Rover.nav_angles is not None: `

The rover does not move if there is no navigation angles updated in the Rover object.

Case 1: Forward and enough navigable terrain

* If robot is moving forward and is __not__ at max velocity, accelerate.
* If robot is moving forward and is at max velocity, cruize.

Case 2: Forward and not enough navigable terrain

* Go to stop mode

Case 3: Stop mode

* If not stopped, keep braking.
* If stopped, turn to find navigable terrain.
* If found nagiable terrain, go to forward state.

## Improvements 
* Only change Rover object values from perecpet when the robot is stable (Roll is close to zero)
* Rover gets stuck sometimes.Add code to avoid that.
* Add functionality to pick up rocks.
* Map the world with close boundaries.
* Make the rover clever to map efficiently.


