# Performance Evaluation

## MP.1 Data Buffer Optimization
> Ring buffer is implemented like LIFO stack while comparing with the limit as below:
    
    if (dataBufferSize>dataBuffer.size())
        {
            dataBuffer.push_back(frame);
        }
        else{
            dataBuffer.erase(dataBuffer.begin());
            dataBuffer.push_back(frame);
        }
---
## MP.2 Keypoint Detection
> Harris and Shitomasi detectors are implemented seperatly in individual functions but other detectors are implemented inside single function *detKeypointsModern*, in which each detetor has created with the help of opencv feature2d and xfeature2d files.

> **1. Harris Detector:**   
It was implemented as  __*detKeypointsHarris*__ function in matching2d_student.cpp file at line _142_.  

>**2. ModernDetectors:**     
Other detectors i.e. *FAST, BRISK, ORB, AKAZE, SIFT* are implemented inside of *detKeypointsModern* function in matching2d_student.cpp at line _209_, where it checks which detector is needs to be assinged by string comparison and then that detector gets created and finally detect call is assinged to that detector.
---

## MP.3 Keypoint Removal
> Once we call the detector , it will provide all keypoints in the image but as we are focused only on preceding vehicle we will consider only those keypoints which are present on that vehicle. For that I draw the rectangle with given points around the vehicle and compare each keypoint is lie inside it or not. If keypoint is outside then it is removed from list of keypoints. Below is the code:

    bool bFocusOnVehicle = true;
        cv::Rect vehicleRect(535, 180, 180, 150);
        if (bFocusOnVehicle)
        {  
            cv::rectangle(imgGray,vehicleRect,cv::Scalar(0,255));
            for(auto it=keypoints.begin(); it!=keypoints.end(); )
            {
                if(vehicleRect.x<=(*it).pt.x && vehicleRect.y<=(*it).pt.y
                   && (vehicleRect.x + vehicleRect.width)>=(*it).pt.x 
                   && (vehicleRect.y + vehicleRect.height)>=(*it).pt.y
                    )
                {
                    ++it;
                }
                else{
                    it= keypoints.erase(it);
                }
            }
        }
---
## MP.4 Keypoint Descriptors:
> Keypoint descriptors BRIEF,ORB,FREAK,AKAZE and SIFT are implemented inside the function *descKeypoints* similar to BRISK implemntation with the help of opencv. Firstly string comparsion was done to check which descritor is called and accordingly it gets created and in the end his creator calls to compute function.   


void descKeypoints(vector<cv::KeyPoint> &keypoints, cv::Mat &img, cv::Mat &descriptors, string descriptorType, double &t)
{
    // select appropriate descriptor BRIEF, ORB, FREAK, AKAZE, SIFT
    cv::Ptr<cv::DescriptorExtractor> extractor;
    if (descriptorType.compare("BRISK") == 0)
    {

        int threshold = 30;        // FAST/AGAST detection threshold score.
        int octaves = 3;           // detection octaves (use 0 to do single scale)
        float patternScale = 1.0f; // apply this scale to the pattern used for sampling the neighbourhood of a keypoint.

        extractor = cv::BRISK::create(threshold, octaves, patternScale);
    }
    else if (descriptorType.compare("BRIEF") == 0)
    {
        extractor = cv::xfeatures2d::BriefDescriptorExtractor::create();
    }
    else if (descriptorType.compare("ORB") == 0)
    {

        extractor = cv::ORB::create();
    }
    else if (descriptorType.compare("FREAK") == 0)
    {
        extractor = cv::xfeatures2d::FREAK::create();
    }
    else if (descriptorType.compare("AKAZE") == 0)
    {
        extractor = cv::AKAZE::create();
    }
    else if (descriptorType.compare("SIFT") == 0)
    {
       extractor = cv::xfeatures2d::SiftDescriptorExtractor::create(); 
        
    }
    else
    {
      // ...
    }

    // perform feature description
    t = (double)cv::getTickCount();
    extractor->compute(img, keypoints, descriptors);
    t = ((double)cv::getTickCount() - t) / cv::getTickFrequency();
    cout << descriptorType << " descriptor extraction in " << 1000 * t / 1.0 << " ms" << endl;
}

---
## MP.5 Descriptor Matching and MP.6 Descriptor Distnace Ratio:

> If matcherType is gets compared to check BF or FLANN matching and inside FLANN first it we need to convert the descriptor source from binary into floating as it is a bug in opencv. then we have called the FLANN creater as below:

    else if (matcherType.compare("MAT_FLANN") == 0)
    {
       if (descSource.type() != CV_32F)
        { // OpenCV bug workaround : convert binary descriptors to floating point due to a bug in current OpenCV implementation
            descSource.convertTo(descSource, CV_32F);
            descRef.convertTo(descRef, CV_32F);
        }
      
        matcher = cv::DescriptorMatcher::create(cv::DescriptorMatcher::FLANNBASED);
    }

> After that knn based nearest neighbor is implemented with descriptor distance ratio set to 0.8 as beow:      

    else if (selectorType.compare("SEL_KNN") == 0)
    { 
      vector<vector<cv::DMatch>> goodmatches;
      matcher->knnMatch(descSource, descRef, goodmatches, 2);
       float dist = 0.8;
        for (auto it = goodmatches.begin(); it != goodmatches.end();++it)
        {
            if ((*it)[0].distance < dist*(*it)[1].distance)
            {    
                matches.push_back((*it)[0]);
            }
        }
      
    }
---
## MP.7-9 Performance Evaluation:

> Counting of detected keypoints , mathced keypoints for all combinations of possible detector and descriptors for all 10 images with the timings is logged in **Output.csv** file.  
> It was done by creating a .csv file , and printing the data as we did on terminal.

> Analysis done on logged data:

## Detector analysis:

|Detector|	Avg detection time(ms)|
|:---:|:---:|
|Harris|	24.73126|
|SHITOMASI|	21.15257
FAST	|0.9753441
BRISK	|388.4132
ORB	|10.455803
AKAZE|	93.34803
SIFT|	122.97259999999999

> By looking above table, best detector will be FAST and below is the decriptor combination table with FAST detector:

|Detector|	Descriptor| Avg Descriptor Extraction time (ms)
|:---:|:---:|:---:|
|FAST|BRISK|2.1118|
|FAST|BRIEF|1.1240|
|FAST|ORB|1.2132|
|FAST|FREAK|45.88|

## Conclusion: 
> After doing the analysis of above combinations, below are my top3 recommendations as best choice for detecting keypoints on vehicles based on time taken for keypoint detection and desciptor extraction.

1. *FAST* detector and *BRIEF* descriptor
2. *FAST* detector and *ORB* descriptor
3. *FAST* detector and *BRISK* descriptor




