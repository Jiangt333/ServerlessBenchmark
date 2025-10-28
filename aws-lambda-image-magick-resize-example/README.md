# AWS Lambda Image Resize Example (with ImageMagick)

This repository deploys an AWS Lambda Function that resizes images using ImageMagick.

**Blog post:** https://www.bytescale.com/blog/aws-lambda-image-resize/

## Quick Start

**Important:** 

- Replace `my-lambda-function-code` and `my-images` with unique S3 bucket names!

### Deploying the Lambda Function

1. Checkout the repository:
 
   ```bash
   git clone git@github.com:bytescale/aws-lambda-image-magick-resize-example.git
   
   cd aws-lambda-image-magick-resize-example
   ```

2. Install NPM dependencies:

   ```bash
   npm install
   ```

3. Create the S3 bucket stack (remember: change the bucket names in `ParameterValue` below):

   ```bash
      aws cloudformation create-stack `
      --stack-name my-bucket-stack `
      --parameters ParameterKey=LambdaCodeBucketName,ParameterValue=serverless-s3-bucket-ting `
                     ParameterKey=ImageBucketName,ParameterValue=test-image `
      --template-body file://buckets-cloudformation.yml `
      --region us-east-1
   ```

4. Update `function.js` > `imageBucketName` to your image bucket's name.

5. Upload the Lambda Function's code:
   
   ```bash
   zip -r function-dist.zip .
   aws s3 cp function-dist.zip s3://serverless-s3-bucket-ting/AwsLambdaImageResize.zip
   ```

6. Deploy the Lambda Function (remember: change the bucket names in `ParameterValue` below):

   ```bash
      aws cloudformation deploy `
         --stack-name my-image-resize-lambda `
         --template-file function-cloudformation.yml `
         --parameter-overrides LambdaCodeBucketName=serverless-s3-bucket-ting ImageBucketName=serverless-s3-bucket-ting `
         --region us-east-1 `
         --capabilities CAPABILITY_NAMED_IAM
   ```

7. Wait for the Lambda Function's stack to complete:

   ```bash
   aws cloudformation describe-stack-events `
     --stack-name my-image-resize-lambda `
     --region us-east-1
   ```
   
   **Note:** the latest event should read `"ResourceStatus": "CREATE_COMPLETE"`.
   
### Invoking the function

1. Upload the image to resize:

   ```bash
   aws s3 cp test-image.jpg s3://serverless-s3-bucket-ting/original-image.jpg
   ```

2. Invoke the function:

   ```bash
   aws lambda invoke `
     --function-name AwsLambdaImageResizeExample `
     function-result.json
   ```

3. "See" the result:

   ```bash
   cat function-result.json
   ```
   
   Outputs:

   ```json
   {
     "isBase64Encoded": true,
     "statusCode": 200,
     "headers": {
       "content-type": "image/jpg"
     },
     "body": "/9j/4AAQSkZJRg...9k="
   }
   ```
   
   **Note:** AWS Lambda only supports JSON responses. `body` contains the resized image as a base64-encoded string: to return the raw image, you'll need to put API Gateway in front of the Lambda function.


If you just want to update code:

   ```bash
   # 1. Repackage and upload
   zip -r function-dist.zip . `
   aws s3 cp function-dist.zip s3://serverless-s3-bucket-ting/AwsLambdaImageResize.zip
   # 2. Only update code
    aws lambda update-function-code `
     --function-name AwsLambdaImageResizeExample ` 
     --s3-bucket serverless-s3-bucket-ting `
     --s3-key AwsLambdaImageResize.zip `
     --region us-east-1
   ```
