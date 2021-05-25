# Image classification steps
I used GDAL Merge.py to combine and clip all 1993 bands into a single multiband image and did the same for 2016. Next I merged the 1993 and 2016 multiband images into a single multi-date image that will be used to create the classified forest cover change map. To get the clip extent I projected the study_area.geojson file to match the UTM 37N used for the Landsat images then used QGIS to read the coordinates of the extent. The GDAL code I used is below.

Since these images are precision and terrain corrected (L1TP) I didn’t do further geometric correction and since they were already converted to surface reflectance (sr) no additional radiometric processing was done.

In QGIS I displayed the single-date images to familiarize myself with the area. This could have been done using the 13-band multi-date image but I find that more cumbersome to work with. For the 1993 image I visualized bands 4,5,3 as RGB and for the 2016 image I used 5,6,4. To enhance the visual images I used “Stretch to MinMax" contrast enhancement with mean +/- standard deviation times 2. I applied the stretch to the 1993 image and saved the style in QGIS. Since the images are both surface reflectance I loaded the saved style and applied it to the 2016 image to ensure the enhancements were similar. 

Tanzania is a very challenging area for forest cover change analysis in large part due to the wide coverage of sparse woodland forest which is very sensitive to changes in season and rainfall. Although the images were acquired with near-anniversary dates some of the apparent change is likely due to weather and not actual land cover change. 

I flickered the two dates by turning the 2016 image, displayed on top of the 1993 image, on and off. I panned around to get a feel for the area. There is definitely a lot going on with a lot of agriculture including swidden and plantations. To get a better sense of change and to help me delineate change classes for training I used the multi-date image to create a change composite using bands 10, 3, 3 for RGB (the red bands from each image). This enhancement displays cut forest as red and reforestation as cyan. In this case, however, nearly all of the areas that appear to be reforested are almost certainly not forest but likely a mix of young trees, shrubs and grass given the very smooth texture. This area produces a lot of cashew nuts and I expect that is much of the agriculture northeast of the town of Liwale. Some of these areas may have been reforested but would not typically be classified as forest. Without a definition of forest given in the challenge I defined forest as trees with >30% crown closure and over 5m in height, a common definition for forest. I will also note that I am considering forest as a land cover class and not a land use class. Much of the tree covered land use in this area is likely cashew agriculture. 

For training data, I created a polygon layer in QGIS and defined four classes:
* 11: Forest - Forest
* 12: Forest - Nonforest
* 21: Nonforest - Forest
* 22: Nonforest - Nonforest

I manually digitized several polygons with the intent of representing the spectral variability throughout the image. 

For the classification I used an R script that I wrote several years ago that uses the random forest algorithm. A machine learning algorithm like random forest is idea for this sort of complex classification task. To train the random forest model I selected 5000 pixels for each class from my training polygons. The script output a classified map as well as a map with the probability of correct classification assigned to each pixel. 

# Overall comments
I do not have high confidence in the accuracy of this classification. This is an incredibly complex area since tree density varies greatly making if difficult to delineate forest vs non-forest and most of the change is likely due to cashew agriculture. I found very little non-forest to forest change that went from bare ground to forest. Most of the non-forest to forest change was in areas that had vegetation in 1993 and was likely undisturbed, perhaps abandoned agriculture. Also, a good bit of the forest to non-forest area in the northeast portion of the image was possibly older cashew that was cut and perhaps replanted. I expect some of my forest to nonforest class is actually nonforest to nonforest.

If I had more time and resources I would invest in ground reference data or expert opinion from someone very familiar with this area. I would also incorporate some derived layers to add to the multi-date image stack. Texture metrics such as standard deviation and some of the Haralick texture features would help separate some of the younger plantations (smooth texture) from the more mature plantations that would qualify as forest under my definition. Depending on the intended use of the classified map I might also run a median filter to reduce speckle. Lastly, I would run several more iterations of the R script to refine the classification. 
 
It appears as if this exercise was from the NASA ARSET Change Detection for Land Cover Mapping webinar training from 2018. In the spirit of full disclosure, I provided input to that training including authoring the R script that was used. I used a newer version of that R script with changes made by a colleague to speed up processing through the use of multiple CPU cores. 

# GDAL code
Create multiband 1993 image
```
gdal_merge.py -separate -n -9999 -a_nodata -9999 -ot Int16 -of GTiff -ul_lr 341316 -1068490 396692 -1112106 -o LT05_L1TP_166067_19930705_20170118_01_T1_sr_clipped.tif LT05_L1TP_166067_19930705_20170118_01_T1_sr_band1.tif LT05_L1TP_166067_19930705_20170118_01_T1_sr_band2.tif LT05_L1TP_166067_19930705_20170118_01_T1_sr_band3.tif LT05_L1TP_166067_19930705_20170118_01_T1_sr_band4.tif LT05_L1TP_166067_19930705_20170118_01_T1_sr_band5.tif LT05_L1TP_166067_19930705_20170118_01_T1_sr_band7.tif
```

Create multiband  2016 image
```
gdal_merge.py -separate -n -9999 -a_nodata -9999 -ot Int16 -of GTiff -ul_lr 341316 -1068490 396692 -1112106 -o LC08_L1TP_166067_20160704_20180526_01_T1_sr_clipped.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_band1.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_band2.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_band3.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_band4.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_band5.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_band6.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_band7.tif
```

Create multi-date image
```
gdal_merge.py -separate -n -9999 -a_nodata -9999 -ot Int16 -of GTiff -o Tanzania1993_2016_clipped.tif LT05_L1TP_166067_19930705_20170118_01_T1_sr_clipped.tif LC08_L1TP_166067_20160704_20180526_01_T1_sr_clipped.tif
```

# Repository contents
* README.md: This file
* rf_classification.R: R script for classification
* QGIS_Files: Directory with images for QGIS project
	* Tanzania.qgz: QGIS project file
	* StudyArea_UTM37N.geojson: Study region rectangle projected to UTM37N 

	* ### Landsat image stacks
		* LT05_L1TP_166067_19930705_20170118_01_T1_sr_clipped.tif - Landsat 5 bands 1,2,3,4,5,7
		* LC08_L1TP_166067_20160704_20180526_01_T1_sr_clipped.tif -  Landsat 8 bands 1,2,3,4,5,6,7
		* Tanzania1993_2016_clipped.tif – 13 band image stack with 1993 image (bands 1-5 and 7) and 2016 image bands (1-7)

	* ### Classified image output
		* Tanzania1993_2016_clipped_Class.tif – classified image
		* Tanzania1993_2016_clipped_Class.tif.aux.xml
		* Tanzania1993_2016_clipped_Prob.tif – Probability of correct classification

	* ### QGIS style files
		* training.qml - Style for training polygons
		* class.qml - Style for classified image
		* L5Lut.qml - Style for landsat images

	* ### Shapefile files
		* TzTraining.cpg
		* TzTraining.dbf
		* TzTraining.prj
		* TzTraining.shp
		* TzTraining.shx
