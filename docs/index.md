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
<img src="assets/basic_extraction_idea.png" alt="Basic Extraction Idea" width="80%">

To extract the water level from an image, three inputs are needed: the image itself, an elevation map of the scene for each pixel, and a relative depth map for the image indicating the distances from the camera to each pixel. The elevation map and depth map can be extracted from the scene and then used for all future predicitons. With these three inputs, the algorithm identifies where the water meets the shore and extracts the elevations corresponding to that specific shoreline, since each water level corresponds to a unique shoreline location in the image.

With these inputs the following steps occur:
1. Mask Extraction - A mask of the water in the image is extracted. This can be done with a Neural network such as the segment anything model (SAM).
2. Mask Cleaning - The mask is often full of unwanted holes and artifacts. To fix this a series of dialations and errosions that expand and contract the mask are used to filled the empty space.
3. Edge Extraction - To extract the edges of the mask, first diatliton is applied to the mask. Is shirnks the mask by 1 pixel. Then the smaller mask is subtracted from the original to leave just the edge pixels as a mask.

<img src="assets/extraction_steps1-3.png" alt="Extraction Steps 1 to 3" width="100%">
4. Edge Trimming - Since short edges are likely to only be artifcats, the edges are labeled using 4-connectivity and edges that are too short are removed. The result is a clean edge mask that can be used for extracting pixel values.
<img src="assets/extraction_steps4.png" alt="Extraction Step 4" width="100%">
5. Compute Weighted Median - Use the edge mask to extract the corresponding elevation and depth values. Using these values, computed the weighted median for each river bank using the distance from the camera as a weight (closer = larger weight).
6. Combine Banks - Finally, combine the estimates from each bank using the variance in banks water level predictions. The result is the predicted water level.
<img src="assets/extraction_steps5-6.png" alt="Extraction Steps 5 to 6" width="100%">

## Results
We tested our algorithm on a total of 8 different sites, each with a variety of setups and characteristics. For all sites we used the masks fpr the images to generate our elevation map and then tested our extraction using these same masks. For three sites, we additionally tested on completely different images of the site not used to generate the depth map. Even when tested on unseen images, our method performed very well matching the general trends of the true data. 

<p align="center">
    <b>1. WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet</b>: Example Image, Elevation Map, and Relative Depth Map <br>
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/Image.jpg" width="33%" />
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/elevation_map_with_background.png" width="33%" />  
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/relative_depth.png" width="33%" />   <br>
    Predicitons Vs. Ground Truth (Left: Seen, Right Unseen) <br>
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/Water Level Predicitons Using weighted median.png" width="40%" />    
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/Water Level Predicitons Using weighted median Test.png" width="40%" />    
</p>
<br>
<p align="center">
    <b>2. CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton</b>: Example Image, Elevation Map, and Relative Depth Map <br>
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/Image.jpg" width="33%" />
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/elevation_map_with_background.png" width="33%" />  
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/relative_depth.png" width="33%" />   <br>
    Predicitons Vs. Ground Truth (Left: Seen, Right Unseen) <br>
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/Water Level Predicitons Using weighted median.png" width="40%" />    
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/Water Level Predicitons Using weighted median Test.png" width="40%" />    
</p>
<br>
<p align="center">
    <b>3. OK_Illinois_River_near_Moodys</b>: Example Image, Elevation Map, and Relative Depth Map <br>
    <img src="assets/OK_Illinois_River_near_Moodys/Image.jpg" width="30%" />
    <img src="assets/OK_Illinois_River_near_Moodys/elevation_map_with_background.png" width="30%" />  
    <img src="assets/OK_Illinois_River_near_Moodys/relative_depth.png" width="30%" />   <br>
    Predicitons Vs. Ground Truth (Left: Seen, Right Unseen) <br>
    <img src="assets/OK_Illinois_River_near_Moodys/Water Level Predicitons Using weighted median.png" width="40%" />    
    <img src="assets/OK_Illinois_River_near_Moodys/Water Level Predicitons Using weighted median Test.png" width="40%" />    
</p>
<br>
<p align="center">
    <b>4. OR_Breitenbush_River_above_French_Creek_near_Detroit</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/OR_Breitenbush_River_above_French_Creek_near_Detroit/Image.jpg" width="33%" />
    <img src="assets/OR_Breitenbush_River_above_French_Creek_near_Detroit/elevation_map_with_background.png" width="33%" />   
    <img src="assets/OR_Breitenbush_River_above_French_Creek_near_Detroit/Water Level Predicitons Using weighted median.png" width="33%" />    
</p>
<br>
<p align="center">
    <b>5. SC_Waccamaw_River_at_SC_22_below_Longs</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/SC_Waccamaw_River_at_SC_22_below_Longs/Image.jpg" width="33%" />
    <img src="assets/SC_Waccamaw_River_at_SC_22_below_Longs/elevation_map_with_background.png" width="33%" />   
    <img src="assets/SC_Waccamaw_River_at_SC_22_below_Longs/Water Level Predicitons Using weighted median.png" width="33%" />    
</p>
<br>
<p align="center">
    <b>6. VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX/Image.jpg" width="33%" />
    <img src="assets/VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX/elevation_map_with_background.png" width="33%" />   
    <img src="assets/VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX/Water Level Predicitons Using weighted median.png" width="33%" />    
</p>
<br>
<p align="center">
    <b>7. WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV/Image.jpg" width="33%" />
    <img src="assets/WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV/elevation_map_with_background.png" width="33%" />   
    <img src="assets/WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV/Water Level Predicitons Using weighted median.png" width="33%" />    
</p>
<br>
<p align="center">
    <b>7. WI_Silver_Creek_at_State_Highway_21_near_Angelo</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/WI_Silver_Creek_at_State_Highway_21_near_Angelo/Image.jpg" width="33%" />
    <img src="assets/WI_Silver_Creek_at_State_Highway_21_near_Angelo/elevation_map_with_background.png" width="33%" />   
    <img src="assets/WI_Silver_Creek_at_State_Highway_21_near_Angelo/Water Level Predicitons Using weighted median.png" width="33%" />    
</p>
<br>

## Discussion

### Real world data is challenging

### The Future
- 3D Elevation Map Generation
- Full Prediction based on Neural Network
- Implement in the field
