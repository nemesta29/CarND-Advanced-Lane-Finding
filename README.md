# CarND-Advanced-Lane-Finding
Files for advanced lane finding project submission
-Advanced_Lane_Finding.ipynb : The jupyter notebook containing all the code using for the submission
-Advanced_Lane_Finding_Writeup.md : The project writeup describing the intuition and thought process
-Project_video_out.mp4 : The output video with the lanes identified
-output_images : A folder containing the various output images

Output_images description :
- Images prefixed 'calib' are chessboard images with the corners marked
- Image prefixed 'undist' is a chess board image undistorted
- Images prefixed 'warped' have been threshold to show gradient changes and identify lane lines with minimal noise
- Images prefixed 'perspect' have undergone perspective transformation for bird's eye view
- Images prefixed 'lane' have their lane lines identified
- Images prefixed 'final' have been reverted back to normal with the lane region marked
