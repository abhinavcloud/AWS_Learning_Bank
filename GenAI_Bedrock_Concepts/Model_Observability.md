# Model Observability

## GenAi Model Observability

For Generative AI Observability we can leverage AWS existing observability tools like
- Cloudwatch
- X-Ray

We can monitor the bedrock endpoints and APIs.

## For monitoring bedrock-runtime endpoint

> bedrock-mantle endpoints supports APIs such as ChatCompletions, Resposnes (Open AI SDK) or Messages(Anthropic) SDK.

> We can use CloudWatch, CloudTrail and Model Invocation Logging

> Below are the steps to set up Model Invocation Logging and get metrics from CloudWatch.
- Set up S3 destination for logging Amazon Bedrock
    1. Create a S3 bucket were logs will be deliverd.
    2. Add a bucket policy to allow Bedrock to execute PutObject action on the bucket resource.
        - Principal: bedrock.amazonaws.com
        - Action: s3:PutObject
        - Resource: Bucket ARN
        - Condition ARN Like: Bedrock ARN
    3. Alternatively you can set up CloudWatch Logs destination.
    4. For this first create a Trust Policy for AWS Bedrock to assume the role.
        - Principal: bedrock.amazonaws.com
        - Action: sts:AssumeRole
        - Condition ARN Like: Bedrock Infernece Model ARN
    5. Create an Execution Policy on the AWS Bedrock role.
        - Principal: bedrock.amazonaws.com
        - Actions: logs:CreateLogStream, logs:PutLogEvents
        - Resource: Model Invocation ARN

    6. Create a Model Invocation Logging from Terraform or Enable it via Bedrock Settings
    7. Select or attach the Destination S3 bucket or Cloudwatch Log Groups and then add the role created for bedrock.
    8. Select the modalities for which the logs should be published
        - Text
        - Image
        - Embedding
        - Video

> Cloudwatch Metrics for bedrock-runtime endpoint
There are several metrics published by cloudwatch for model invocations, like:
- Invocations
- InvocationsLatency -  Time of an inference request, from when the request is received to when the last output token is produced.
- InputTokenCount
- OutputTokenCount
- CacheReadInputTokens
- CacheWriteInputTokens

**Apart from the above we can also monitor a very important metric which is OTPS (Output Tokens Per Second). We can check if the model is generating tokens more slowly (performance) or generating more tokens per second (cost)**

> CloudTrail bedrock-runtime endpoint invocation logs for APIs such as
- InvokeModel
- InvokeModelWithResponseStream
- Converse
- ConverseStream
- ListAsyncInvokes

## For monitoring bedrock-mantle endpoint

### CloudWatch Metrics

>bedrock-mantle publishes metrics in four levels of granuality. Each level uses a different combination of CloudWatch dimensions.

#### Dimensions
1. Inference Metrics
    - Inferences
    - InferenceClientErrors

2. Token Metrics
    - TotalInputTokens
    - TotalOutputTokens
    - InputTokens
    - OutputTokens

3. Dimensions
    - Project
    - Model

#### Granuality Level
1. Account Level
2. Prokect Level
3. Model Level
4. Project + Model Level



    


