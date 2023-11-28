---
title: 'Rapid Local aws Lambda Testing With cdk and Docker'
date: 2023-11-23T23:06:36+11:00
draft: true
description: Example showing the end to end testing of a lambda function. The function itself uses mangum to glue it to a FastAPI. All python.
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

<!-- <span class="summary">**Summary**: In a sentence... </span> -->

#### Prerequites

Some familiarity with [cdk](https://docs.aws.amazon.com/cdk/v2/guide/home.html), [cloudformation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html), and [lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) would be good. My previous post goes into that more.

**!!!! NB: (BELOW for previous post to this)**

Lambda functions in aws are very cool. The lie dormant in the aws cloud until they're needed. Instead of bothering with an ec2 instance, or even running a small server at home, if you just need an endpoint you can call that does something, saves something, you can use a lambda. For this project I needed an endpoint with an sqlite database that I could send queries to as a POST. I need to do this once or twice a day so a dedicated server would be about 99% wasted.

The lambda [free tier](https://aws.amazon.com/lambda/pricing/) gives 400,000 GB seconds of compute time. You configure the instance's power based on memory alone and I needed a function with 1024 MB of memory.  So even with that relatively large amount:

400,000 x 1,000 =  400,000,00 MB / 1024 MB = 390,625 s = 6510 m = 108.5 hours per month for free. About 50x more than what I needed for this. 

(Add part about fastapi etc)

**!!!! NB: (ABOVE for previous post to this)**



The problem with aws lambda functions is testing and debugging. It's possible to have a cycle where you edit locally and then deploy every time you need to test. This is extremely slow and unwieldly though because of the speed of the tools available to do this (SAM cli and cdk deploy). Even worse is zipping and uploading files manually in the gui. 

There's a better way. Below, stage 1-4 are all done locally and in 5. you deploy to the cloud and test it in the form it'll be used in production. 1-3 are practically **instantaneous** because it's just testing functions. 4., testing with docker, takes a little longer but it's second vs minutes to deploy with cdk. The idea is that you do 1-3 regularly. 4. on local commits and 5. when integrating etc. All testing is done with pytest and no other external libraries.

**Stage 1:** 

- Non lambda tests. In this example testing that the sqlite querying code works. 

**Stage 2:**

- API testing. Testing fastapi with TestClient: `from fastapi.testclient import TestClient`, etc.

**Stage 3:**

- Test mangum in the lambda function by invoking it directly in code with test event (the json string can be interpolated with data for different testing situtations by importing it and using string formatting). When running sam init a test event will be created in the events folder. This even can be used to fire the lambda by deserializing it and then calling the lambda_handler function with it as the event and an empty dictionary as the context. 

**Stage 4 (docker):**

- Build docker: `docker build -t lambda_api .`
- Assuming the [aws-lambda-rie](https://github.com/aws/aws-lambda-python-runtime-interface-client#local-testing) is in the folder `$HOME\.aws-lambda-rie\` run:

```shell
docker run -v "$HOME\.aws-lambda-rie:/aws-lambda" -p 9000:8080 --env-file ./.env --entrypoint /aws-lambda/aws-lambda-rie lambda_api poetry run python -m awslambdaric lambda.lambda_api.lambda_handler
```

 - Invoke with curl (or python requests). $jsonContent is powershell variable containing an event with the right path / body / headers to trigger the lambda function -> mangum -> fastapi. Rather than accessing the endpoint directly, the rie emulator expects an event formatted a certain way:

```powershell
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d $jsonContent
```

**Stage 5:**

- Given above passing: `cdk deploy`
    - NB: Then delete cdk.out folder
- Get endpoint url (changing stack name to name set in "app.py"): `aws cloudformation describe-stacks --stack-name <YourStackName> --query "Stacks[0].Outputs[?OutputKey=='lamda_url'].OutputValue" --output text`
- Test using real url (doesn't need to be sent special event as in step 4)

  







