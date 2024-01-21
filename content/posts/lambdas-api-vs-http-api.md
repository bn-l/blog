---
title: 'Lambdas and aws: The clear difference between API and HTTP API' 
date: 2023-11-23T23:06:36+11:00
draft: false
description: It is extremely hard to differentiate between these two API gateway types from AWS. The documentation is extremely opaque and convoluted. After doing a lot of detective work, I try to delineate the two services clearly.
images: 
    - ./images/default-cover.png
slug: lambdas-api-vs-http-api
keywords:
   - aws
   - lambda
   - api
   - http
   - REST
   - ApiGateway
   - HttpApi
   - ServerlessRestApi
   - Difference
# lastmod: 
series:
   - python
   - lambda
   - aws
# tags: 
#   -


---

<span class="summary">**Summary**: Use HttpApi. </span> 



## Taxonomy

Probably the most painful aspect of this is that at some point AWS completely gave up on taxonomically separating these two services and basically throughout the documentation they're referred to using many different conflicting and confusing names. To make sense of this you need to completely ignore the normal (sane) use of the words "http", "api" or "rest" as they're purely arbitrary classifiers for amazon. That said, here is the best taxonomic separation I can arrive at:

Type 1: `Api`

Type 2: `HttpApi`

## Official documentation

- AWS' own [comparison](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html) (warning: basically useless).
- Authorization methods [comparison](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-controlling-access-to-apis.html) (e.g. HttpApi does not support IAM but does support OAuth 2.0 where Api does not).

- See [here](https://github.com/aws/serverless-application-model/blob/master/docs/internals/generated_resources.rst#api) for clues on the categorization of "Api" and "HttpApi" using the help docs for ~~Cloudformation~~ SAM templates.

## Side by side

| Api                                                          | HttpApi                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| AKA: **Api**, REST API,  AWS::ApiGateway::RestApi, ServerlessRestApi (Logical Id), | AKA: RESTful Api, HTTP API, AWS::Serverless::HttpApi, ServerlessHttpApi (Logical Id), AWS::ApiGatewayV2::Api |
| Lambda return value: Must return a specific format ([source](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)) | Will infer correct formatting so can return a string or a Dict, etc ([source](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html#http-api-develop-integrations-lambda.v2)) |
| More features, more complicated, harder                      | Fewer features, easier, lower cost                           |
| Lambda specific [docs](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html) | Lambda specific [docs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html) |
| Template resource properties [source](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-api.html) | Template resource properties ([source](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-httpapi.html)) |
| Managed with ApiGateway (v1) [source](https://stackoverflow.com/a/72409509/2083958) | Managed with **ApiGatewayV2** (which can also create a websocket api) [source](https://stackoverflow.com/a/72409509/2083958) |



