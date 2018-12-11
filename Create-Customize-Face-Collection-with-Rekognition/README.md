Create Customize Face Collection with Rekognition
==============================================

[Amazon Rekognition](https://aws.amazon.com/tw/rekognition/) makes it easy to add image and video analysis to your applications. You just provide an image or video to the Rekognition API, and the service can identify the objects, people, text, scenes, and activities, as well as detect any inappropriate content. Amazon Rekognition also provides highly accurate facial analysis and facial recognition on images and video that you provide. You can detect, analyze, and compare faces for a wide variety of user verification, people counting, and public safety use cases.

## About this lab
### Scenario
This lab use [Amazon Rekognition](https://aws.amazon.com/tw/rekognition/) to build your own face collection service by combining the capabilities of Amazon Rekognition and other AWS services, like AWS Lambda.This enables you to build a solution to create, maintain, and query your own collections of faces, be it for the automated detection of people within an image library, building access control, or any other use case you can think of.

![Framework.png](./images/Framework.png)

## Prerequisites
  -  Make sure you are in __US East (N. Virginia)__, which short name is __us-east-1__.
  -  Installed and configured the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html).

## Lab tutorial
### Upload the image for the Face Collection
Create a S3 Bucket that can be store face images for identfy.

1. On the __Service__ menu, click __S3__, Click __Create Bucket__.

2. For Bucket Name, type __Unique Name__.

3. For Region, choose __US East (N. Virginia)__, Click __Create__.

![CreateBucket.jpg](./images/CreateBucket.jpg)

4. Select the bucket which you created before, and Upload __Cristine__ Folder.

## Build your own Customize Face Collection
Create a Lambda function to build face collection and index the faces of our existing image.
1. On the __Services__ menu, click __Lambda__.

2. Click __Create function__.

3. Choose __Author from scratch__.

4. Enter the following information : 
- Name : __build_face_collection__
- Runtime : __Python 3.6__
- Role : __Create a custom role__

![LambdaConfig.png](./images/LambdaConfig.png)

5. Select __Create a new IAM Role__ as __IAM Role__.

6. Type __lambda_rekognition__ as Role name.

![CreateIAMCustomRole.png](./images/CreateIAMCustomRole.png)

7. Click __Hide Policy Document__ and paste following code in the console. And replace the __bucket name__ you created before in __S3__ section.

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
                        "arn:aws:s3:::<your bucket name>",
                        "arn:aws:s3:::<your bucket name>/*"
                    ]
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "rekognition:IndexFaces",
                        "rekognition:CreateCollection",
                        "rekognition:ListFaces",
                        "rekognition:ListCollections"
                    ],
                    "Resource": "*"
                }
            ]
        }

8. Back to lambda function, then click __Create Function__.

9. After creating the lambda function, copy the [build-face-collection.py](build-face-collection.py) code and paste into the Lambda code field, then __Save__.

10. Replace the following parameter in [build-face-collection.py](build-face-collection.py):
- __bucket_name__ : your bucket name created before
- __collection_name__ : your collection name


11. Change __Basic settings__ and set __Timeout__ to __5 min__.

![SetTimeout.png](./images/SetTimeout.png)

12. Click __Test__ button on the top.

13. Type __Exec__ as __Event name__.

![ExecLambda.png](./images/ExecLambda.png)

14. Click __Save__.

15. Then you click __Test__ again.

16. You will see the success message like this.

![SuccessMessage.png](./images/SuccessMessage.png)

## Conclusion
Congratulations! You now have learned how to:
- Build your own face collection.

## Next Level:
- [Build-a-Serverless-Facial-Detect-Application-with-AWS-Lambda](../Build-a-Serverless-Facial-Detect-Application-with-AWS-Lambda)
