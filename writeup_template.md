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

#### Threshold 
The navigable terrain, obstacles, rocks are identified using simple rgb threshold interpreted as `[red, blue, green]`
All pixels below `[160, 160, 160]` classify under navigable terrain - `color_thresh(img, rgb_thresh=(160, 160, 160))`
All pixels above `[160, 160, 160]` classify under obstacles - `obstacle_thresh(img, rgb_thresh=(160, 160, 160))`
All pixels below `[160, 160, 160]` classify under rocks - `rock_thresh(img, rock_thresh=(130, 180, 100,170,0,30))`

#### World Coordinates 
The above processes give results in terms of rover centric coordinates. These coodrinates are then converted into world-centric coordinates. The mean of all the navigable terrain is taken and is represented by the line. The angle of this line with respect to the x-axis is the navigation angles. 

### Mapping 
Red Channel - Obstacles
Blue Channel - Navigable Terrain 
ALl Channel - Rocks

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
