---
title: River and Stream Water Level Estimation from Images
excerpt: "CS 766 Final Project <br>Forrest Peterson & Keegan Johnson"
layout: single
author_profile: false
classes: wide
header:
  overlay_image: /assets/splash_page.jpg
  overlay_filter: 0.5
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

### Site Selection 
A key component of this project was the data. We identified 8 sites from across the country as good candidates for our project. When choosing these sites we considered several different factors: 
1. Visibility of the Shoreline - We chose sites that had a very distinct shoreline in the images. Some sites have the camera directed at large bodies of water so the shoreline is very far away and small. Others have lots of growth in the images that make the water edge hard to see. Since our method relies upon clearly identifying the water level on the shore, choosing sites that met this criteria was vital. 
2. Obvious Visual Changes with Water Level - Another important factor was choosing sites with clear changes in the images with changes in water level. For sites with larger bodies of water or cameras far away, a change in the water level is not always visible. To avoid this, we chose sites where the bank of the river changes drastically with water level. 
3. Water Level Variation - To allow for better estimates of the water level, we chose sites that had large changes in water level.
4. Site Variation - While we aren’t focusing on radically different sites, we still wanted some variation in the size of the rivers, orientation of the cameras, and distance from the water in our sites to test the robustness of our method. Because of this we chose a variety of locations of different types.
5. Data Availability - While many sites are listed on the HVIS website, not all of the sites had corresponding river level readings and time matched images. We specifically chose sites with a variety of images and their corresponding water levels.
6. Weather - Since our method relies on the shoreline, it will completely fail when the river freezes. For this reason, we chose a variety of sites that are in a southern climate and didn’t show any signs of freezing in the images. For the sites that did have winter, we marked them in a table.
7. Night Time Images - Some cameras in the HVIS database have IR night vision cameras while others don’t record night events at all. Some night modes perform very poorly, so we avoided sites with bad data and marked which sites had good or no night time data. 

## Implementation

### Elevation Map

### Water Level Extraction
<p align="center">
    <img src="assets/basic_extraction_idea.png" alt="Basic Extraction Idea" width="60%">
</p>
To extract the water level from an image, three inputs are needed: the image itself, an elevation map of the scene for each pixel, and a relative depth map for the image indicating the distances from the camera to each pixel. The elevation map and depth map can be extracted from the scene and then used for all future predicitons. With these three inputs, the algorithm identifies where the water meets the shore and extracts the elevations corresponding to that specific shoreline, since each water level corresponds to a unique shoreline location in the image. <br> <br>

With these inputs the following steps occur:
1. Mask Extraction - A mask of the water in the image is extracted. This can be done with a Neural network such as the segment anything model (SAM).
2. Mask Cleaning - The mask is often full of unwanted holes and artifacts. To fix this a series of dialations and errosions that expand and contract the mask are used to filled the empty space.
3. Edge Extraction - To extract the edges of the mask, first diatliton is applied to the mask. Is shirnks the mask by 1 pixel. Then the smaller mask is subtracted from the original to leave just the edge pixels as a mask.

<img src="assets/extraction_steps1-3.png" alt="Extraction Steps 1 to 3" width="100%">

4. Edge Trimming - Since short edges are likely to only be artifcats, the edges are labeled using 4-connectivity and edges that are too short are removed. The result is a clean edge mask that can be used for extracting pixel values.
<img src="assets/extraction_steps4.png" alt="Extraction Step 4" width="100%">
5. Compute Weighted Median - Use the edge mask to extract the corresponding elevation and depth values. Using these values, computed the weighted median for each river bank using the distance from the camera as a weight (closer = larger weight). To account for instances where very small changes in the pixel locations result in major elevation changes, a confidence score is also computed based on the gradient of elevation map and pixels with a very large gradient are given a lower confidence score and weight.
6. Combine Banks - Finally, combine the estimates from each bank using the variance in banks water level predictions. The result is the predicted water level.
<img src="assets/extraction_steps5-6.png" alt="Extraction Steps 5 to 6" width="100%">

## Results
We tested our algorithm on a total of 8 different sites, each with a variety of setups and characteristics. For all sites we used the masks fpr the images to generate our elevation map and then tested our extraction using these same masks. For three sites, we additionally tested on completely different images of the site not used to generate the depth map. Even when tested on unseen images, our method performed very well matching the general trends of the true data. 

| Number | Site                                                   | Seen/Unseen/Both?| MEA (ft.) |
|-|-------------------------------------------------------------- | ---- | -----       |
|1| WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet    | Both | 0.106/0.069 |
|2| CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton           | Both | 0.276/0.398 |
|3| OK_Illinois_River_near_Moodys                                 | Both | 0.190/0.26  |
|4| OR_Breitenbush_River_above_French_Creek_near_Detroit          | Seen | 0.338       |
|5| SC_Waccamaw_River_at_SC_22_below_Longs                        | Seen | 0.181       |
|6| VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX                  | Seen | 0.132       | 
|7| WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV        | Seen | 0.073       |
|8| WI_Silver_Creek_at_State_Highway_21_near_Angelo               | Seen | 0.243       |

<p align="center">
    <b>1. WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet</b>: Example Image, Elevation Map, and Relative Depth Map <br>
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/Image.jpg" width="33%" />
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/elevation_map_with_background.png" width="33%" />  
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/relative_depth.png" width="33%" />   <br>
    Predicitons Vs. Ground Truth (Left: Seen, Right Unseen) <br>
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/Water Level Predicitons Using weighted median.png" width="40%" />    
    <img src="assets/WI_East_Branch_Pecatonica_River_near_Blanchardville_Bullet/Water Level Predicitons Using weighted median Test.png" width="40%" />    <br>
    LEFT - MAE: 0.106 ft, Range: 4.6-10.91ft. <br>
    RIGHT - MAE: 0.069 ft. 
</p>
<br>
<p align="center">
    <b>2. CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton</b>: Example Image, Elevation Map, and Relative Depth Map <br>
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/Image.jpg" width="33%" />
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/elevation_map_with_background.png" width="33%" />  
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/relative_depth.png" width="33%" />   <br>
    Predicitons Vs. Ground Truth (Left: Seen, Right Unseen) <br>
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/Water Level Predicitons Using weighted median.png" width="40%" />    
    <img src="assets/CA_Arroyo_DE_LA_Laguna_A_Corte_Madrid_nr_Pleasanton/Water Level Predicitons Using weighted median Test.png" width="40%" /> <br>
    LEFT - MAE: 0.276 ft, Range: 5.61-14.42ft. <br>
    RIGHT - MAE: 0.398 ft.
</p>
<br>
<p align="center">
    <b>3. OK_Illinois_River_near_Moodys</b>: Example Image, Elevation Map, and Relative Depth Map <br>
    <img src="assets/OK_Illinois_River_near_Moodys/Image.jpg" width="30%" />
    <img src="assets/OK_Illinois_River_near_Moodys/elevation_map_with_background.png" width="30%" />  
    <img src="assets/OK_Illinois_River_near_Moodys/relative_depth.png" width="30%" />   <br>
    Predicitons Vs. Ground Truth (Left: Seen, Right Unseen) <br>
    <img src="assets/OK_Illinois_River_near_Moodys/Water Level Predicitons Using weighted median.png" width="40%" />    
    <img src="assets/OK_Illinois_River_near_Moodys/Water Level Predicitons Using weighted median Test.png" width="40%" />    <br>
    LEFT - MAE: 0.190 ft, Range: 4.07-8.24ft. <br>
    RIGHT - MAE: 0.26 ft.
</p>
<br>
<p align="center">
    <b>4. OR_Breitenbush_River_above_French_Creek_near_Detroit</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/OR_Breitenbush_River_above_French_Creek_near_Detroit/Image.jpg" width="33%" />
    <img src="assets/OR_Breitenbush_River_above_French_Creek_near_Detroit/elevation_map_with_background.png" width="33%" />   
    <img src="assets/OR_Breitenbush_River_above_French_Creek_near_Detroit/Water Level Predicitons Using weighted median.png" width="33%" />   
    <br> MAE: 0.338 ft, Range: 10.78-21.74ft. <br>
</p>
<br>
<p align="center">
    <b>5. SC_Waccamaw_River_at_SC_22_below_Longs</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/SC_Waccamaw_River_at_SC_22_below_Longs/Image.jpg" width="33%" />
    <img src="assets/SC_Waccamaw_River_at_SC_22_below_Longs/elevation_map_with_background.png" width="33%" />   
    <img src="assets/SC_Waccamaw_River_at_SC_22_below_Longs/Water Level Predicitons Using weighted median.png" width="33%" />    
    <br> MAE: 0.181 ft, Range: 4.69-8.03ft. <br>
</p>
<br>
<p align="center">
    <b>6. VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX/Image.jpg" width="33%" />
    <img src="assets/VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX/elevation_map_with_background.png" width="33%" />   
    <img src="assets/VA_DIFFICULT_RUN_ABOVE_FOX_LAKE_NEAR_FAIRFAX/Water Level Predicitons Using weighted median - No Fit.png" width="33%" />    
    <br> MAE: 0.132 ft, Range: 1.1-4.9ft. <br>
</p>
<br>
<p align="center">
    <b>7. WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV/Image.jpg" width="33%" />
    <img src="assets/WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV/elevation_map_with_background.png" width="33%" />   
    <img src="assets/WI_Black_Earth_Creek_nr_Brewery_Rd_at_Cross_Plains_PIV/Water Level Predicitons Using weighted median.png" width="33%" />  
    <br> MAE: 0.073 ft, Range: 2.31-2.76ft. <br>
</p>
<br>
<p align="center">
    <b>8. WI_Silver_Creek_at_State_Highway_21_near_Angelo</b>: Example Image, Elevation Map, and Predictions (Seen) <br>
    <img src="assets/WI_Silver_Creek_at_State_Highway_21_near_Angelo/Image.jpg" width="33%" />
    <img src="assets/WI_Silver_Creek_at_State_Highway_21_near_Angelo/elevation_map_with_background.png" width="33%" />   
    <img src="assets/WI_Silver_Creek_at_State_Highway_21_near_Angelo/Water Level Predicitons Using weighted median.png" width="33%" />    
    <br> MAE: 0.243 ft, Range: 6.03-8.54ft. <br>
</p>
<br>

## Discussion

### Real world data is challenging

### The Future
- 3D Elevation Map Generation
- Full Prediction based on Neural Network
- Implement in the field
