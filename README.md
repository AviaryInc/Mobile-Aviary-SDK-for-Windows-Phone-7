Aviary Windows Phone 7 SDK Setup Guide
====================================

Contents
--------

* [How to integrate the Aviary SDK into your WP7 project](#howto)
	* [Aviary Photo Genius](#photogenius)
	* [Technical notes](#notes)

<a name="howto"></a>
How to integrate the Aviary SDK into your WP7 project
-------------------------------------------------

### 1. Copy Aviary SDK All files to Project root folder.

### 2. Add reference to the Aviary SDK related DLLs, see screenshot reference below:

### 3. Copy folder “icons” to the root folder of the App, see screenshot below:
                      
  *Note: The “Build Action” of the images files contained in “icons” folder should be changed to “Content” from the default value of “Resource”.*
                           
### 4. Add the following code to App.xaml:
 
	  	xalns:resources="clr-namespace:AviarySDK.Resources;assembly=AviarySDK"
		
		<!--Application Resources-->
		<Application.Resources>
			<ResourceDictionary>
				<ResourceDictionary.MergedDictionaries>
					<ResourceDictionary Source="/AviarySDK;component/Themes/generic.xaml"/>
				</ResourceDictionary.MergedDictionaries>
				<resources:Images x:Key="ImageResources"/>
				<aviary:LocalizedStrings xmlns:aviary="clr-namespace:AviarySDK;assembly=AviarySDK" x:Key="LocalizedStrings" />
			</ResourceDictionary>
		</Application.Resources>

To execute the Aviary Editor in the SDK, you need to do the following inside the source code.

#### 4.1 For any source code calling any API in the SDK, you need to first reference the SDK:

		using AviarySDK;
		
#### 4.2 To invoke the Editor in the SDK, you need to include the following parameters:
		
		stream (required):  
			Stream object describing the input image.  (required)
		ThemeColor (optional): 
			you can choose a custom color for change the theme color of the UI inside the editor. The default value is "32a9ff".
		features (optional):
			features controls the visibility of tools when editor is launched.  The default is to show all
		For example:
	         List<AviaryFeature> features=new List<AviaryFeature>();
	         features.Add(AviaryFeature.Enhance);
	         features.Add(AviaryFeature.Effects);
	         features.Add(AviaryFeature.Orientation);

		originalFileName (optional):
			If the stream is from an existing file, specifying this will allow the editor to extract any EXIF information to determine its orientation.
			If the file came from Choose Photo or Take photo.  The original file could be extracted as:

             //e: PhotoResult of Choose Photo,Take Photo
			 //
			Stream stream =e.ChosenPhoto;
			String originalFile= e.OriginalFileName

		Usage:
            AviaryTask aviaryTask = new AviaryTask(stream, features,themeColor: "FF0000", originalFileName: originalFile); 
            // aviaryTask_Completed is the call back function after the editor completed the photo edit.
        aviaryTask.Completed += new EventHandler<AviaryTaskResultArgs>(aviaryTask_Completed);
        aviaryTask.Show();
		
#### 4.3 To obtain the final image from the SDK

	// finishedBitmap - the final processed image that is passed back from the SDK
	 void aviaryTask_Completed(object sender, AviaryTaskResultArgs e)
	 {
	   		if (e.AviaryResult== AviaryResult.OK)
       {
			   // / Deal with the return image here.
			   ///e.PhotoResult is AviarySDK handled image
			     // BTW, we decide to pass WritebaleBitmap for ease of saving to photo Album
			}
			else
			{
				//#4.3 Public method 1. 
				aviaryTask_Error(e.Exception);
			}
	 }


	Public method 1..
		void aviaryTask_Error(Exception ex)
		{
		   if (ex == null)
		      return;

		   if (ex.Message == AviaryError.StreamNull)
		   {
		      // Input stream can't be null
		   }
		   else if (ex.Message == AviaryError.FeaturesEmpty)
		   {
		       // Features list determines which tools are exposed in the Aviary editor and cannot be null or empty
		   }
		   else if (ex.Message == AviaryError.ImageBig)
		   {
		       // The image cannot exceed 8 mega pixels
		   }
		   else if (ex.Message == AviaryError.AdjustmentsEmpty)
		   {
		       // The adjustment array passed into Photo Genius Apply is not valid.
		       // The array must be of 4 float values and the array can't be empty or null
		   }
		   else
		   {
		      // This is to handle any error thrown by the system
		   }
		 }
 
 <a name="photogenius"></a>
### 5. About AviaryPhotoGeniusScores API

To execute the `AviaryPhotoGeniusScores` in the SDK, you need to do the following inside the source code.

#### 5.1 For any source code calling any API in the SDK, you need to first reference the SDK:
 
		using AviarySDK;

#### 5.2 To invoke the AviaryPhotoGeniusScores in the SDK, you need to do the following:

	AviaryPhotoGeniusScores aviaryPhotoGenius = new AviaryPhotoGeniusScores(stream);
	aviaryPhotoGenius.Completed+=new EventHandler<AviaryPhotoGeniusScoresResultArgs>(aviaryPhotoGenius_Completed);
	aviaryPhotoGenius.Execute();

#### 5.3 To obtain the final image from the SDK

	// finishedBitmap - the final processed image that is passed back from the SDK
	 void aviaryPhotoGenius_Completed(object sender, AviaryPhotoGeniusScoresResultArgs e)
	 {
	   	if (e.AviaryResult== AviaryResult.OK)
	   {
     e.Predicts is the predicted data 
	          e.Scores is the score on each parameters
	          e.TotalScoreis the total score of the image
		}
		else
		{
			//#4.3 Public method 1. 
			aviaryTask_Error(e.Exception);
		}
	 }

#### 5.4 Notes: 

	Parameters returned by prediction:
		Predicts[0] = Contrast prediction [-100, 100]
		Predicts[1] = Brightness prediction [-50, 50]
		Predicts[2] = Saturation prediction [0, 2]
		Predicts[3] = Sharpness prediction [-100, 100]

	Scores returns how close it is to prediction...100 is perfection
		Scores[0] = Contrast score [0, 100]  
		Scores[0] = Brightness score [0, 100]
		Scores[0] = Saturation score [0, 100] 
		Scores[0] = Sharpness score [0, 100] 
		TotalScore = Overall score vs prediction [0, 100]
	
### 6. About AviaryPhotoGeniusApply API

To execute the `AviaryPhotoGeniusApply` in the SDK, you need to do the following inside the source code.
 
#### 6.1 For any source code calling any API in the SDK, you need to first reference the SDK:
 
		using AviarySDK;
		
To invoke the `AviaryPhotoGeniusApply` in the SDK, you need to do the following:

	AviaryPhotoGeniusApply aviaryPhotoGenius = new AviaryPhotoGeniusApply(stream, new float[4]);
	aviaryPhotoGenius.Completed += new EventHandler<AviaryPhotoGeniusApplyResultArgs>(aviaryPhotoGenius_Completed);
	aviaryPhotoGenius.Execute();;
	
#### 6.2 To obtain the final image from the SDK

    void aviaryPhotoGenius_Completed(object sender, AviaryPhotoGeniusApplyResultArgs e)
		 {
		   	if (e.AviaryResult== AviaryResult.OK)
		       {                        
				    ///e.PhotoResult is AviarySDK handled image
				}
				else
				{
					//#4.3 Public method 1. 
					aviaryTask_Error(e.Exception);
				}
		  }
		  
#### 6.3 Notes:

		Parameters used for photo genius correction:
			Predicts[0] = Contrast value [-100, 100]
			Predicts[1] = Brightness value [-50, 50]
			Predicts[2] = Saturation value [0, 2]
			Predicts[3] = Sharpness value [-100, 100]

<a name="notes"></a>
### 7. Technical Notes:

Inside the AviarySDK editor, the GestureService from the Silverlight Toolkit is being used, described in [Windows Phone Toolkit - Nov 2011 (7.1 SDK)](http://silverlight.codeplex.com/releases/view/75888).  However, there is a known issue that this service will conflict with another system control, Slider. Please read more about this issue and the resolution at [http://forums.create.msdn.com/forums/p/82897/501068.aspx](http://forums.create.msdn.com/forums/p/82897/501068.aspx). Although the conflict inside the AviarySDK is resolved, it is still a potential conflict with Slider used in the host application. If the host application is using the Slider control, it is recommended that the developer resolve the conflict inside the host application as per the link article above.