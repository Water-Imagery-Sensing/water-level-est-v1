---
title: Water Level Estimation
layout: single
toc: true
toc_sticky: true
---

## Motivation

For this project, we aimed to develop a vision system that can accurately estimate water level data from camera images of rivers, streams, and lakes. The United States Geological Survey (USGS) monitors the water level of different bodies of water at numerous locations across the United States using different gauges and cameras. Rather than relying on costly gauges to record the time varying water level of a site, we seek to create a system that can accept images of these sites and predict their corresponding water level. While this is often accomplished with water level markers or gauges being present in the image, we hope to leverage information in images and in the entire scene to determine water level variations. 

## Approach

<img src="assets/approach_overview.png" alt="Approach Overview" width="80%">

1. Capture new image
2. Segment water mask
3. Extract water level
    - Compare water mask with elevation map

## Implementation

### Elevation Map

### Water Level Extraction
<img width="700" height="551" alt="Basic Extraction Idea" src="assets/Basic_extraction_Idea.png">

To extract the water level from an image, three inputs are needed: the image itself, an elevation map of the scene for each pixel, and a relative depth map for the image indicating the distances from the camera to each pixel. The elevation map and depth map can be extracted from the scene and then used for all future predicitons. With these three inputs, the algorithm identifies where the water meets the shore and extracts the elevations corresponding to that specific shoreline, since each water level corresponds to a unique shoreline location in the image.

With these inputs the following steps occur:
1. Mask Extraction - A mask of the water in the image is extracted. This can be done with a Neural network such as the segment anything model (SAM).
2. Mask Cleaning - The mask is often full of unwanted holes and artifacts. To fix this a series of dialations and errosions that expand and contract the mask are used to filled the empty space.
3. Edge Extraction - To extract the edges of the mask, first diatliton is applied to the mask. Is shirnks the mask by 1 pixel. Then the smaller mask is subtracted from the original to leave just the edge pixels as a mask.

<img src="assets/extraction_steps1-3.png" alt="Extraction Steps" width="80%">


## Results

## Discussion

### Real world data is challenging

### The Future
- 3D Elevation Map Generation
- Full Prediction based on Neural Network
- Implement in the field
