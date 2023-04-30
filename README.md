# Detection of Edited Images

## Team: Neha Barde (neha.barde@duke.edu) ; Iqra Imtiaz(iqra.imtiaz@duke.edu)

The goal of this project is to create a web application that can distinguish between real and edited images. For the edited images it needs to further differentiate between the image being a deepfake or it being manipulated using facetune.

## Problem

As generative AI becomes more powerful, it can lead to misuse that goes undetected. In the near past we have seen multiple instances of "fun" videos being made using faceswap or similar deepfake technologies. While recreationally it opens up a new avenue for entertainment, these deepfakes have been used for various nefarious purposes.
Take the incident of the persona of Oliver Taylor , where long story short, a seemingly normal human persona was used to share terrorist propaganda. Or if we take the incident of Female Activists in the arab countries who lost credibility after altered images were circulated in the media by their opposition.
There has also been an instance of a political figure shifting the blame of one of their comments by claiming that the comments had been altered by deepfake technologies. In this case eventhough the deepfake technology gets the short end of the stick, an app that can verify this claim will help hold people accountable for their actions.

Added to this beautifying face manipulation technologies have become lighter, less complicated and more freely available. People no longer have to pay any sum as they would have to do with s/w like photoshop. With the current influencer culture, it becomes important to understand if an image influencing you is actually unaltered. The story of young kids being conscious of their bodies and chasing unhealthy standards, is old but still a very valid concern.

That is why I think this app would be very helpful atleast as a starting step.
Moreover clearly classifying the s/w involved in the manipulation is essential because based on the software the general riskiness of the data consumption changes.

## Approach

Flow Diagram:
![image](https://user-images.githubusercontent.com/110474064/234205320-2a44c4e6-e8f9-46c9-9a6f-9750d39a1527.png)

Here we have 2 separate models that are trained using 2 separate datasets. The first model is trained using a dataset with real and altered images (altered images are a combination of deepfake images and factuned images). The second model is trained using a dataset of deepfake and factune images as the 2 classes.

The original datasets were randomly sampled to get an equal number of real and altered images to pass through the first model. If the image is edited only then will it get passed to the 2nd model which determines the technology used to create it.

## Data Sourcing and Manipulations:

Deepfakes dataset sourced from - https://www.kaggle.com/datasets/manjilkarki/deepfake-and-real-images?resource=download

Facetune Dataset sourced from - https://github.com/JordanMLee/Identifying-Human-Edited-Images

For the real vs edited dataset, equal number of images were sampled from both of these inidividual datasets for both of the classes.
For the deepfake vs facetune dataset, equal number of images were sampled from these same datasets.

In both cases the files were randomly split into train, test and val sets. Any files that threw "UnidentifiedImage" Exception were deleted as these were likely corrupted and could not be read by the PIL library.

## Modeling Approaches:

### DL approach:
Here I chose to use the Resnet 152 pretrained model, that would later be fine tuned using 2 separate datasets to generate 2 separate models.

Real VS Edited:

The Model finds it fairly difficult to tell the difference between a real and an edited image. I hypothesize that the model is finding it hard to differentiate between a signature of the app being present and absent in the images. This model underperforms from the current benchmark of 67% as published in various papers. It should be noted though, that these articles seldom consider different kinds of edited images as is the case with this dataset.

One thing that i will do to confirm this hypothesis is to look do a multi-class training and verify the results instead of a binary training. This might clear up on which "edited" class is actually causing the issue.

![image](https://user-images.githubusercontent.com/110474064/234380131-86a8f8ba-94ef-4e8d-b664-56aa32657609.png)![image](https://user-images.githubusercontent.com/110474064/234380148-6f2ca755-1ddc-4be0-aed2-c3518fbb584d.png)

Deepfake VS Facetune:

This Model works very well with a high accuracy, which makes me believe that the model finds it easier differentiating between the 2 signatures of the deepfake vs facetune editing as opposed to the Real VS Edited usecase. 

![image](https://user-images.githubusercontent.com/110474064/234381970-ab87208a-e6a8-4f53-b4a7-bc2ce2adda27.png)![image](https://user-images.githubusercontent.com/110474064/234381999-b6da28b6-0ba4-4d3d-91d2-3d025df97081.png)

## Deployment Details:

### Simple web app:
1. Dockerized image uploaded to Azure Container Registry
2. Azure web app created using ACR image.
3. App can be accessed here : https://editdetect.azurewebsites.net/? 

### Kubernetes Deployment:
1. Using Azure Kubernetes Service used to deploy and manage the same container based application.
2. Self scaling with 2 nodes and a basic 2vCPU VM and 7GB RAM.
3. Each pod uses 0.5 CPU and 2GB RAM
4. AKS_generated_YAML.txt provides the YAML file that AKS generated when trying to deploy the app using a single-container image. This is a load balanced service : http://20.242.216.135:7000/ 

### Load Testing :
1. Python Locust library was used for testing with locustfile.py being the starting point.
2. Configuration at run time and results for a heavy load (2000 users; 500 spawned in a second):
![image](https://user-images.githubusercontent.com/110474064/235369485-a62bc1d0-9aa0-4c54-9b0d-9f69ec65f8ba.png)

![image](https://user-images.githubusercontent.com/110474064/235369189-7392fdcb-1ca4-4eb2-b9e9-eadadd854c34.png)
![image](https://user-images.githubusercontent.com/110474064/235369217-010d836f-20ad-407e-95e2-675b5fcbbddf.png)
![image](https://user-images.githubusercontent.com/110474064/235369262-8e0c239d-5cba-432a-b8d6-5373edf3921a.png)
![image](https://user-images.githubusercontent.com/110474064/235369288-b8dc48a0-9467-41f0-beb7-610580dbccbd.png)

3. Configuration at run time and results for a heavy load (500 users; 100 spawned in a second):
![image](https://user-images.githubusercontent.com/110474064/235370268-3d0217a0-2832-459c-a79e-1482e58a666f.png)

![image](https://user-images.githubusercontent.com/110474064/235369717-33b2a62d-2af2-4da0-a78c-21a87f6b53d7.png)
![image](https://user-images.githubusercontent.com/110474064/235369785-211d2712-782c-40c5-beee-833b8868df45.png)
![image](https://user-images.githubusercontent.com/110474064/235369811-885fd2ec-93d1-4613-a052-22e56d8ac036.png)
![image](https://user-images.githubusercontent.com/110474064/235369828-c6223b07-7be4-46dc-b137-5ae1aa2e6e16.png)

### Our take on the results:
This is because of there being just 2 nodes and 3 pods created as part of the configuration of the kubernetes cluster. As soon as the number of users starts to climb past 1000, we start seeing failures with connection timeout errors. For Heavy load ,we also start seeing "OS Error: Too many files open", and this points directly to the app.py file which is the entry point for the app. This indicates that our system had exceeded the maximum number of files that can be open simultaneously and is likely because a large number of network connections and open files were created, which caused the system to reach its limit.

## User Guide: 

This is a web app deployed using Azure and created using gradio. Click on the below link to explore!

[editdetect.azurewebsites.net ](https://editdetect.azurewebsites.net/?)

If you have images of your own, you can upload them to the UI or you can use the list of examples randomly sampled from the test data (the models havent seen this data yet).

Demo Video: https://duke.box.com/s/ksm6uhdasmenijbb4o5mrklpn3jfl07y 

## Replication:

If you want to replicate this, use the above mentioned data sources and you can find the fine tuned models here - https://duke.box.com/s/ca2usi4foz9ama3pkp2rc5sm9ue9lv9q 

## Future Enhancements:

1. Recreate the dataset using same original images ,passing them through the DGAN architecture to create facetuned datasets and through the faceswap project to create the deepfake datasets. Retrain the model.

2. Create a web API so that this functionality can be integrated with other social media image sharing apps where images can be flagged as being “edited” as a discretionary viewer warning.

3. Integrate this with engagement rates to define impact of the edited image for better flagging with appropriate warning.

## References:

1. Data and data creation process flow for facetune : Identifying Human Edited Images using a CNN by Jordan Lee et. al. 

 https://arxiv.org/ftp/arxiv/papers/2101/2101.03275.pdf
 
2. Deepfakes creation project using Faceswap : https://github.com/deepfakes/faceswap 

3. Deployment ref - https://datasciencedojo.com/blog/web_app_for_gradio/

4. Azure Ref- https://learn.microsoft.com/en-us/azure/aks/cluster-container-registry-integration?tabs=azure-cli



 
