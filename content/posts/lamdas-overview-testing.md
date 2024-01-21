---
title: 'Lamdas: Overview, routing and fast local test and deploy'
date: 2023-11-23T23:06:36+11:00
draft: false
description: (Python) An overview of AWS Lambda functions and how to do end to end testing. The function itself uses mangum to glue it to FastAPI.
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: rapid-local-aws-lambda-testing
keywords:
   - aws
   - lambda
   - local testing
   - cdk
   - mangum
   - fastapi
# lastmod: 
series:
   - python
   - lambda
   - aws
# tags: 
#   -
---



**Topics:** [cdk](https://docs.aws.amazon.com/cdk/v2/guide/home.html), [cloudformation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html), and [lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html).

## Lambda functions

Lambda functions in aws are very cool. They lie dormant on aws' server until they're needed. No need to bother with an ec2 instance, or even running a small server at home. For a project I needed an endpoint with an sqlite database that I could send queries to as a POST. I need to do this once or twice a day so a dedicated server would be 99% wasted. 

The lambda [free tier](https://aws.amazon.com/lambda/pricing/) gives 400,000 GB seconds of compute time. You configure the instance's power based on memory alone and I needed a function with 1024 MB of memory.  So even with that relatively large amount:

400,000 x 1,000 =  400,000,00 MB / 1024 MB = 390,625 s = 6510 m = **108.5 hours per month for free.** About 50x more than what I needed for this. 

This post only covers python lambdas but they support javascript and rust also (which is **extremely** cool—the idea of uploading a single small rust binary). 

### Events

For a lambda function to operate you need at least one other aws service. To have it as an endpoint you connect an api gateway ([exact docs page for this section](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html#apigateway-add)) that sends the function an "event" (a json file). [This](https://github.com/awsdocs/aws-lambda-developer-guide/blob/main/sample-apps/nodejs-apig/event-v2.json) ([a](https://archive.is/hwDeR)) is what the event looks like (http api gateway). Your function takes this and then does something with it. For example an endpoint that queries a database might take query params in the POST body. AWS will send your function these in the event file. It's more convoluted than it sounds but basically lambdas can be summarized as: "a function that takes a json and returns something (or nothing)". That's it.

### Package size

You can send aws your code in two ways:

1. Zip your whole python package and upload it. (Or separate zip files as "layers"—This layers concept is not very useful and is dated. Advise ignoring.) 

    Zip packages have a hard limit of 250 MB **unzipped**

2. Docker image: 500 MB free storage and then you pay per GB a month ([pricing](https://aws.amazon.com/ecr/pricing/)). A lot more convenient and allows local testing as your code will run in the same environment locally as on aws' server.

### Routing

It's possible to do routing by analyzing the json event file aws gives your function and calling another function. [Mangum](https://mangum.io/) does this and is compatible with FastAPI which means you can very quickly get an openapi compatible endpoint.  

## Testing

The big issue with aws lambda functions is testing and debugging. You can end up with a cycle where you edit locally and then deploy each time you need to test. This is extremely slow and unwieldly because of the speed of the tools available to do this (SAM cli and cdk deploy). Even worse is zipping and uploading files manually in the gui.

SAM supposedly supports local testing but it's bug ridden, undersupported and seemingly deprecated already by AWS as a bad idea.

There is a better way. Below, stages `1`-`4` are all done locally and in `5`. you deploy to the cloud and test it in the form it'll be used in production. Test stages `1` to`3` are practically **instantaneous** and will pick up the majority of bugs. You get a very accurate idea quickly whether the lambda will work in the cloud without having to deploy until much later.

Here's an example setup (for a fastAPI endpoint that queries an sqlite database and uses mangum to interface with the lambda):

(All testing is done with pytest and no other external libraries)

**Stage 1:** 

- Non lambda tests. E.g. Testing that the code to query an sqlite database code works. 

**Stage 2:**

- Purely API testing. Testing fastapi with [TestClient](https://fastapi.tiangolo.com/tutorial/testing/#using-testclient): `from fastapi.testclient import TestClient`, etc.

**Stage 3:**

- Test mangum in the lambda function by invoking it directly in code with a test event (a json string can be interpolated with data for different testing situations by importing it and using string formatting. See the link in the "Events" paragraph above.) 

**Stage 4 (docker):**

- See [these](https://docs.aws.amazon.com/lambda/latest/dg/python-image.html#python-image-instructions) docs for background.
- Docker build: `docker build -t lambda_api .`
- Assuming the [aws-lambda-rie](https://github.com/aws/aws-lambda-python-runtime-interface-client#local-testing) is in the folder `$HOME\.aws-lambda-rie`[^1], and that you're using poetry[^2], run:

```shell
docker run -v "$HOME\.aws-lambda-rie:/aws-lambda" -p 9000:8080 --env-file ./.env --entrypoint /aws-lambda/aws-lambda-rie lambda_api poetry run python -m awslambdaric lambda.lambda_api.lambda_handler
```

 - Then invoke with curl (or python requests). $jsonContent is powershell variable containing an event with the right path / body / headers to trigger the lambda function -> mangum -> fastapi. Rather than accessing the endpoint directly, the rie emulator expects an event formatted a certain way:

```powershell
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d $jsonContent
```

**Stage 5:**

- Given above passing: `cdk deploy`
    - (Then delete cdk.out folder)
- Get the stack url if you missed it or above didn't output it (may need to change `Stacks[0]` index etc):

    - Change `<YourStackName>` to the name set in "app.py"): `aws cloudformation describe-stacks --stack-name <YourStackName> --query "Stacks[0].Outputs[?OutputKey=='lamda_url'].OutputValue" --output text`

- Test using real url (doesn't need to be sent special event as in stage 4, can treat it now as any other http endpoint).

    







[^1]: NB: Use forward slashes on non-windows.  
[^2]: Recommend poetry vs pip. Speed and organization improvements. `poetry run` here, runs the command in the local virtual env. Very nice.
