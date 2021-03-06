<properties
	pageTitle="Detect Face and Emotion with Azure Media Analytics"
	description="This topic demonstrates how to detect faces and emotions with Azure Media Analytics."
	services="media-services"
	documentationCenter=""
	authors="juliako"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="06/22/2016"   
	ms.author="milanga;juliako;"/>
 
#Detect Face and Emotion with Azure Media Analytics

##Overview

The **Azure Media Face Detector** media processor (MP) enables you to count, track movements, and even gauge audience participation and reaction via facial expressions. This service contains two features: 

- **Face detection**

	Face detection finds and tracks human faces within a video. Multiple faces can be detected and subsequently be tracked as they move around, with the time and location metadata returned in a JSON file. During tracking, it will attempt to give a consistent ID to the same face while the person is moving around on screen, even if they are obstructed or briefly leave the frame.

	>[AZURE.NOTE]This services does not perform facial recognition. An individual who leaves the frame or becomes obstructed for too long will be given a new ID when they return.

- **Emotion detection**
	
	Emotion Detection is an optional component of the Face Detection Media Processor that returns analysis on multiple emotional attributes from the faces detected, including happiness, sadness, fear, anger, and more. 

The **Azure Media Face Detector** MP is currently in Preview.

This topic gives details about  **Azure Media Face Detector** and shows how to use it with Media Services SDK for .NET.

##Face Detector input files

Video files. Currently, the following formats are supported: MP4, MOV, and WMV.

##Face Detector output files

The face detection and tracking API provides high precision face location detection and tracking that can detect up to 64 human faces in a video. Frontal faces provide the best results, while side faces and small faces (less than or equal to 24x24 pixels) might not be as accurate.

The detected and tracked faces are returned with coordinates (left, top, width, and height) indicating the location of faces in the image in pixels, as well as a face ID number indicating the tracking of that individual. Face ID numbers are prone to reset under circumstances when the frontal face is lost or overlapped in the frame, resulting in some individuals getting assigned multiple IDs.

###<a id="output_elements"></a>Elements of the output JSON file

For the face detection and tracking operation, the output result contains the metadata from the faces within the given file in JSON format.

The face detection and tracking JSON includes the following attributes:

Element|Description
---|---
Version|This refers to the version of the Video API.
Timescale|"Ticks" per second of the video.
Offset|This is the time offset for timestamps. In version 1.0 of Video APIs, this will always be 0. In future scenarios we support, this value may change.
Framerate|Frames per second of the video.
Fragments|The metadata is chunked up into different segments called fragments. Each fragment contains a start, duration, interval number, and event(s).
Start|The start time of the first event in ???ticks???.
Duration|The length of the fragment, in ???ticks???.
Interval|The interval of each event entry within the fragment, in ???ticks???.
Events|Each event contains the faces detected and tracked within that time duration. It is an array of array of events. The outer array represents one interval of time. The inner array consists of 0 or more events that happened at that point in time. An empty bracket [] means no faces were detected.
ID| The ID of the face that is being tracked. This number may inadvertently change if a face becomes undetected. A given individual should have the same ID throughout the overall video, but this cannot be guaranteed due to limitations in the detection algorithm (occlusion, etc.)
X, Y|The upper left X and Y coordinates of the face bounding box in a normalized scale of 0.0 to 1.0. <br/>-X and Y coordinates are relative to landscape always, so if you have a portrait video (or upside-down, in the case of iOS), you'll have to transpose the coordinates accordingly.
Width, Height|The width and height of the face bounding box in a normalized scale of 0.0 to 1.0.
facesDetected|This is found at the end of the JSON results and summarizes the number of faces that the algorithm detected during the video. Because the IDs can be reset inadvertently if a face becomes undetected (e.g. face goes off screen, looks away), this number may not always equal the true number of faces in the video.

Face Detector uses techniques of fragmentation (where the metadata can be broken up in time-based chunks and you can download only what you need), and segmentation (where the events are broken up in case they get too large). Some simple calculations can help you transform the data. For example, if an event started at 6300 (ticks), with a timescale of 2997 (ticks/sec) and framerate of 29.97 (frames/sec), then:

- Start/Timescale = 2.1 seconds
- Seconds x (Framerate/Timescale) = 63 frames

Below is a simple example of extracting the JSON into a per frame format for face detection and tracking:
	
	var faceDetectionResultJsonString = operationResult.ProcessingResult;
	var faceDetecionTracking = 
	     JsonConvert.DeserializeObject<FaceDetectionResult>(faceDetectionResultJsonString, settings);


##Face detection input and output example

###Input video

[Input Video](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

###Task configuration (preset)

When creating a task with **Azure Media Face Detector**, you must specify a configuration preset. The following configuration preset is just for face detection.

	{"version":"1.0"}

###JSON output

The following example of JSON output was truncated.

	{
	"version": 1,
	"timescale": 30000,
	"offset": 0,
	"framerate": 29.97,
	"width": 1280,
	"height": 720,
	"fragments": [
	    {
	    "start": 0,
	    "duration": 60060
	    },
	    {
	    "start": 60060,
	    "duration": 60060,
	    "interval": 1001,
	    "events": [
	        [
	        {
	            "id": 0,
	            "x": 0.519531,
	            "y": 0.180556,
	            "width": 0.0867188,
	            "height": 0.154167
	        }
	        ],
	        [
	        {
	            "id": 0,
	            "x": 0.517969,
	            "y": 0.181944,
	            "width": 0.0867188,
	            "height": 0.154167
	        }
	        ],
	        [
	        {
	            "id": 0,
	            "x": 0.517187,
	            "y": 0.183333,
	            "width": 0.0851562,
	            "height": 0.151389
	        }
	        ],

		. . . 

##Emotion detection input and output example


###Input video

[Input Video](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

###Task configuration (preset)

When creating a task with **Azure Media Face Detector**, you must specify a configuration preset. The following configuration preset specifies to create JSON based on the emotion detection.
 	
	{
	  "version": "1.0",
	  "options": {
	    "aggregateEmotionWindowMs": "987",
	    "mode": "aggregateEmotion",
	    "aggregateEmotionIntervalMs": "342"
	  }
	}


####Attribute descriptions

Attribute name|Description
---|---
Mode|Faces: Only face detection <br/>AggregateEmotion: Return average emotion values for all faces in frame.
AggregateEmotionWindowMs|Use if AggregateEmotion mode selected. Specifies the length of video used to produce each aggregate result, in milliseconds.
AggregateEmotionIntervalMs|Use if AggregateEmotion mode selected. Specifies with what frequency to produce aggregate results.

####Aggregate defaults

Below are recommended values for the aggregate window and interval settings. AggregateEmotionWindowMs should be longer than AggregateEmotionIntervalMs.

   |Defaults(s)|Max(s)|Min(s)
---|---|---|---
AggregateEmotionWindowMs|0.5|2|0.25
AggregateEmotionIntervalMs|0.5|1|0.25

###JSON output

JSON output for aggregate emotion (truncated):
 
	
	{
	 "version": 1,
	 "timescale": 30000,
	 "offset": 0,
	 "framerate": 29.97,
	 "width": 1280,
	 "height": 720,
	 "fragments": [
	   {
	     "start": 0,
	     "duration": 60060,
	     "interval": 15015,
	     "events": [
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ]
	     ]
	   },
	   {
	     "start": 60060,
	     "duration": 60060,
	     "interval": 15015,
	     "events": [
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 1,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0.688541,
	             "happiness": 0.0586323,
	             "surprise": 0.227184,
	             "sadness": 0.00945675,
	             "anger": 0.00592107,
	             "disgust": 0.00154993,
	             "fear": 0.00450447,
	             "contempt": 0.0042109
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 1,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,


##Limitations

- The supported input video formats include MP4, MOV, and WMV.
- The detectable face size range is 24x24 to 2048x2048 pixels. The faces out of this range will not be detected.
- For each video, the maximum number of faces returned is 64.
- Some faces may not be detected due to technical challenges; e.g. very large face angles (head-pose), and large occlusion. Frontal and near-frontal faces have the best results.


## Sample code

The following program shows how to:

1. Create an asset and upload a media file into the asset.
1. Creates a job with a face detection task based on a configuration file that contains the following json preset. 
					
		{
		    "version": "1.0"
		}

1. Downloads the output JSON files. 
		 
        using System;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using System.Threading;
		using System.Threading.Tasks;
		
		namespace FaceDetection
		{
		    class Program
		    {
		        // Read values from the App.config file.
		        private static readonly string _mediaServicesAccountName =
		            ConfigurationManager.AppSettings["MediaServicesAccountName"];
		        private static readonly string _mediaServicesAccountKey =
		            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
		
		        // Field for service context.
		        private static CloudMediaContext _context = null;
		        private static MediaServicesCredentials _cachedCredentials = null;
		
		        static void Main(string[] args)
		        {
		
		            // Create and cache the Media Services credentials in a static class variable.
		            _cachedCredentials = new MediaServicesCredentials(
		                            _mediaServicesAccountName,
		                            _mediaServicesAccountKey);
		            // Used the cached credentials to create CloudMediaContext.
		            _context = new CloudMediaContext(_cachedCredentials);
		
		            // Run the FaceDetection job.
		            var asset = RunFaceDetectionJob(@"C:\supportFiles\FaceDetection\BigBuckBunny.mp4",
		                                        @"C:\supportFiles\FaceDetection\config.json");
		
		            // Download the job output asset.
		            DownloadAsset(asset, @"C:\supportFiles\FaceDetection\Output");
		        }
		
		        static IAsset RunFaceDetectionJob(string inputMediaFilePath, string configurationFile)
		        {
		            // Create an asset and upload the input media file to storage.
		            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
		                "My Face Detection Input Asset",
		                AssetCreationOptions.None);
		
		            // Declare a new job.
		            IJob job = _context.Jobs.Create("My Face Detection Job");
		
		            // Get a reference to Azure Media Face Detector.
		            string MediaProcessorName = "Azure Media Face Detector";
		
		            var processor = GetLatestMediaProcessorByName(MediaProcessorName);
		
		            // Read configuration from the specified file.
		            string configuration = File.ReadAllText(configurationFile);
		
		            // Create a task with the encoding details, using a string preset.
		            ITask task = job.Tasks.AddNew("My Face Detection Task",
		                processor,
		                configuration,
		                TaskOptions.None);
		
		            // Specify the input asset.
		            task.InputAssets.Add(asset);
		
		            // Add an output asset to contain the results of the job.
		            task.OutputAssets.AddNew("My Face Detectoion Output Asset", AssetCreationOptions.None);
		
		            // Use the following event handler to check job progress.  
		            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);
		
		            // Launch the job.
		            job.Submit();
		
		            // Check job execution and wait for job to finish.
		            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
		
		            progressJobTask.Wait();
		
		            // If job state is Error, the event handling
		            // method for job progress should log errors.  Here we check
		            // for error state and exit if needed.
		            if (job.State == JobState.Error)
		            {
		                ErrorDetail error = job.Tasks.First().ErrorDetails.First();
		                Console.WriteLine(string.Format("Error: {0}. {1}",
		                                                error.Code,
		                                                error.Message));
		                return null;
		            }
		
		            return job.OutputMediaAssets[0];
		        }
		
		        static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
		        {
		            IAsset asset = _context.Assets.Create(assetName, options);
		
		            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
		            assetFile.Upload(filePath);
		
		            return asset;
		        }
		
		        static void DownloadAsset(IAsset asset, string outputDirectory)
		        {
		            foreach (IAssetFile file in asset.AssetFiles)
		            {
		                file.Download(Path.Combine(outputDirectory, file.Name));
		            }
		        }
		
		        static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
		        {
		            var processor = _context.MediaProcessors
		                .Where(p => p.Name == mediaProcessorName)
		                .ToList()
		                .OrderBy(p => new Version(p.Version))
		                .LastOrDefault();
		
		            if (processor == null)
		                throw new ArgumentException(string.Format("Unknown media processor",
		                                                           mediaProcessorName));
		
		            return processor;
		        }
		
		        static private void StateChanged(object sender, JobStateChangedEventArgs e)
		        {
		            Console.WriteLine("Job state changed event:");
		            Console.WriteLine("  Previous state: " + e.PreviousState);
		            Console.WriteLine("  Current state: " + e.CurrentState);
		
		            switch (e.CurrentState)
		            {
		                case JobState.Finished:
		                    Console.WriteLine();
		                    Console.WriteLine("Job is finished.");
		                    Console.WriteLine();
		                    break;
		                case JobState.Canceling:
		                case JobState.Queued:
		                case JobState.Scheduled:
		                case JobState.Processing:
		                    Console.WriteLine("Please wait...\n");
		                    break;
		                case JobState.Canceled:
		                case JobState.Error:
		                    // Cast sender as a job.
		                    IJob job = (IJob)sender;
		                    // Display or log error details as needed.
		                    // LogJobStop(job.Id);
		                    break;
		                default:
		                    break;
		            }
		        }
		
		    }
        }


##Media Services learning paths

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Provide feedback

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

##Related links

[Azure Media Services Analytics Overview](media-services-analytics-overview.md)

[Azure Media Analytics demos](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)