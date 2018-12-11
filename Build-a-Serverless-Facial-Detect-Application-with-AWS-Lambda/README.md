Build a Serverless Facial Detect Application with AWS Lambda
============================================================

## About this lab
### Scenario
 In this lab, you will host a static facial detecting website in [Amazon S3](https://aws.amazon.com/tw/s3/), and build public backend API using [Lambda](https://aws.amazon.com/tw/lambda/) and [API Gateway](https://aws.amazon.com/tw/api-gateway/) to receives data that identify a person in photo with [Amazon Rekognition](https://aws.amazon.com/tw/rekognition/).

![Framework.png](./images/Framework.png)

## Prerequisites
  -  Make sure you are in __US East (N. Virginia)__, which short name is __us-east-1__.
  -  Download __index.html__ which is in this repository.
  - Complete Lab : [Create-Customize-Face-Collection-with-Rekognition](../Create-Customize-Face-Collection-with-Rekognition/) 

## Lab tutorial
### Create S3 bucket to store image
- On the service menu, click __S3__.

- Click __Create Bucket__.

- For Bucket Name, type a __Unique Name__.

- For Region, choose __US East(N.Virginia)__.

- Click __Create__.
![CreateBucket.jpg](./images/CreateBucket.jpg)

- On the __Permissions__ Tab, click __Public Acesses settings__ .

- Disable the following options.
![PublicAccessSetting.png](./images/PublicAccessSetting.png)

- Click __Save__.

- On the __Permissions__ Tab, click __CORS configuration__ .
![CORSSetting.jpg](./images/CORSSetting.jpg)

- Paste the following code to editor.

      <?xml version="1.0" encoding="UTF-8"?>
      <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
      <CORSRule>
          <AllowedOrigin>*</AllowedOrigin>
          <AllowedMethod>GET</AllowedMethod>
          <AllowedMethod>PUT</AllowedMethod>
          <AllowedHeader>*</AllowedHeader>
      </CORSRule>
      </CORSConfiguration>

- Click __Save__.

- Note your bucket name and replace **`your bucket name`** with it at __index.html__  line 210.

<img width="400" alt="ChangeBucketName.png" src="./images/ChangeBucketName.png">


### Creating an Identity Pool

- On the service menu, click __Cognito__.

- Choose __Manage Federated Identities__.

- Choose __Create new identity pool__.

- In the __Identity pool name__ filed, type __rekognition web__.

- Select __Enable access to unauthenticated identities__ from the __Unauthenticated identities__ collapsible section.
![CognitoCreate.png](./images/CognitoCreate.png)

- Click __Create__.

- Click __View Details__ to check IAM Role data.

- In the Unauthenticated Role, Click __View Policy Document__.

<img width="500" alt="ViewUnauthRolePolicy.png" src="./images/ViewUnauthRolePolicy.png">

- Click __Edit__ and it will show dialog to check edit policy, choose __Ok__.

<img width="400" alt="EditPolicy.png" src="./images/EditPolicy.png">

- Paste the following policy to editor, replaced **`<your-bucket-name>`** with the bucket name you create to store image.

      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Effect": "Allow",
                  "Action": [
                      "mobileanalytics:PutEvents",
                      "cognito-sync:*"
                  ],
                  "Resource": [
                      "*"
                  ]
              },
              {
                  "Effect": "Allow",
                  "Action": [
                      "s3:PutObject",
                      "s3:PutObjectAcl"
                  ],
                  "Resource": [
                      "arn:aws:s3:::<your-bucket-name>",
                      "arn:aws:s3:::<your-bucket-name>/*"
                  ]
              }
          ]
      }

- Click __Allow__.

- On the reight navigation bar choose __Sample code__.

- Select JavaScript from the __Platform__ section.

- Copy the sample code below __Get AWS Credentials__ section.

<img width="500" alt="CopySampleCode.jpg" src="./images/CopySampleCode.jpg">

- Paste the __Amazon Cognito credentials__ to __index.html__ line 135 to line 137.

<img width="500" alt="ChangeCognitoCredential.png" src="./images/ChangeCognitoCredential.png">

### Create a S3 Bucket to Host static website

- On the service menu, click __S3__.

- Click __Create Bucket__.

- For Bucket Name, type a __Unique Name__.

- For Region, choose __US East(N.Virginia)__.

- Click __Create__.
![CreateBucket.jpg](./images/CreateBucket.jpg)

- On the __Permissions__ Tab, click __Public Acesses settings__ .

- Disable the following options.
![PublicAccessSetting.png](./images/PublicAccessSetting.png)

- Click __Save__.

- On the __Permissions__ Tab, click __Bucket policy__ .
![BucketPolicy.jpg](./images/BucketPolicy.jpg)

- Paste the following code : 

      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "PublicReadGetObject",
                  "Effect": "Allow",
                  "Principal": "*",
                  "Action": "s3:GetObject",
                  "Resource": "arn:aws:s3:::<your-bucket-name>/*"
              }
          ]
      }

- Click __Save__.

- Download the __index.html__ file in this Github repository first then click __Add files__.

- Select and add the __index.html__ file which you downloaded.

- Click __Upload__ button on lower left side without any setting.

### Enable Static website hosting through S3

- Select __Properties__ tab.

- Click __Static website hosting__.

- Select __Use this bucket to host a website__.

-  Type __index.html__ for the index document.

<img width="500" alt="SetIndexHtml.jpg" src="./images/SetIndexHtml.jpg">

- Click the __Endpoint__ on the top of window.

- You will see the website as below.

![web.png](./images/web.png)

### Build a Lambda Function to identify face

- On the service menu, click __Lambda__.

- Click __Create function__.

- Choose __Author from scratch__.

- Enter the following information :
  - Name : __face_identify__
  - Runtime : __Python 3.6__
  - Role : __Create a custom role__

<img width="600" alt="SetupLambda.png" src="./images/SetupLambda.png">

- Select __Create a new IAM Role__ as IAM Role.

- Type __lambda_identify_face__ as Role name.

<img width="600" alt="LamdaRoleName.png" src="./images/LamdaRoleName.png">

- Click __View Policy Document__ and click __Edit__ and it will show dialog to check edit policy, choose __Ok__.

<img width="400" alt="EditPolicy.png" src="./images/EditPolicy.png">

- Paste following code in the console. And replace **`<your-bucket-name>`** you created to store images.

      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Effect": "Allow",
                  "Action": [
                      "logs:CreateLogGroup",
                      "logs:CreateLogStream",
                      "logs:PutLogEvents"
                  ],
                  "Resource": "arn:aws:logs:*:*:*"
              },
              {
                  "Effect": "Allow",
                  "Action": [
                      "s3:HeadBucket",
                      "s3:ListAllMyBuckets",
                      "s3:ListBucket",
                      "s3:GetObject"
                  ],
                  "Resource": [
                      "arn:aws:s3:::<your-bucket-name>",
                      "arn:aws:s3:::<your-bucket-name>/*"
                  ]
              },
              {
                  "Effect": "Allow",
                  "Action": [
                      "rekognition:SearchFacesByImage"
                  ],
                  "Resource": "*"
              }
          ]
      }

- Click __Allow__.

- Back to lambda function, then click __Create Function__.

- After creating the lambda function, copy the following code and paste into the Lambda code field, then replace **`<your face collection id>`** with your face collection id.

      import json
      import boto3
      rekognition_client = boto3.client('rekognition')

      def lambda_handler(event, context):
          
          response = rekognition_client.search_faces_by_image(
              CollectionId='<your face collection id>',
              Image={
                  'S3Object': {
                      'Bucket': event['Bucket'],
                      'Name': event['ObjectName'],
                  
                  }
              },
              MaxFaces=2,
              FaceMatchThreshold=60
          )
          
          if len(response['FaceMatches']) == 0:
              print("No Matches!")
              rekognition_face = "This face is not in collection"
          else :
              rekognition_face = str(response['FaceMatches'][0]['Face']['ExternalImageId'])
              print("Rekognition result:"+rekognition_face)
          
          return (rekognition_face)

- Click __Save__.

### Set up API Gateway
- On the service menu, click __API Gateway__.

- Click __Get Started__ or __Create API__.

- Select __New API__.

- Type-in API name with __Serverless-rekognition-web__, then click __Create API__.
![CreateAPI.png](./images/CreateAPI.png)

- In the __Resources__ tab, choose the root __/__, click __Actions__ and select __Create Resource__.

- Type __get-identify-result__ in the __Resource Name__.

- Make sure __Enable API Gateway CORS is enabled__, then click Create Resource.
![CreateAPIResource.png](./images/CreateAPIResource.png)

- After resource being created, click __Actions__ and select __Create Method__ to add method.

- In the drop-down list, select __POST__ method and click __yes__.

- In the setup page as below:

  - Integration type : Select __Lambda Function__  
  - Lambda Region : Choose __us-east-1__  
  - Lambda function : Type the  __face_identify__ created in previous chapter in the Lambda Function

![PostMethodSetup.png](./images/PostMethodSetup.png)

- Click __Save__.

- Click __OK__ to give API Gateway permission to invoke Lambda function.

- Click __Actions__ within Resource layer, and select __Enable CORS__.

- Click __Enable CORS and replace existing CORS headers__.

- Review and click __Yes, replace existing values__.

- Waiting for the steps all be checked.
![EnableCORS.png](./images/EnableCORS.png)

### Deploy APIs
- Click __Actions__ and select __Deploy API__.

- In the prompt console,
  - Deployment stage : Select __[New Stage]__ 
  - Stage name : __prod__

<img width="500" alt="DeployAPI.png" src="./images/DeployAPI.png">

- Click __Deploy__.

- The console would jump out as below, select the __SDK Generation__ tab.

- In the __SDK Generation__ tab, select __JavaScript__ in the Platform field and click __Generate SDK__.

- The browser would ask you to confirm download a zip file named as __javascript_TIMESTAMP.zip__.

- Save and unzip the zip file, after entering the folder, there would be files show as below:

<img width="500" alt="SDKFolder.png" src="./images/SDKFolder.png">

- Back to the S3 bucket you created in previous section and upload these files by dragging them into the upload window. These files must be on the same layer of index.html file. It should be like as below:
![S3Folder.png](./images/S3Folder.png)

- Reload the web page and click the __Borowse__ button to choose image.

![PreviewImage.png](./images/PreviewImage.png)

- Click upload button to upload image to S3.

![UploadImage.png](./images/UploadImage.png)

- If you success to upload, it will show the message.

![SuccessUpload.png](./images/SuccessUpload.png)

- The result of identify face will show below the button.

![Result.png](./images/Result.png)

## Conclusion
Congratulations! We now have learned how to:
- Create an Lambda application that uses Amazon Rekognition to identify face
- Build API through API Gateway and Lambda
- Hosting a static website through S3
- Use Cognito to supports identity and access management

## Clean Resources
- The API you created
- S3 bucket to store image
- S3 bucket to host static website
- The Lambda function you create to identify face
- The Lambda function you create to build customize face collection
- Cognito identity pool