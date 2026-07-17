# Amazon Bedrock Guardrails

    Amazon Bedrock Guardrails provides configurable safeguards to help you build safe generative AI applications. With comprehensive safety and privacy controls across foundation models (FMs), Amazon Bedrock Guardrails offers a consistent user experience to help detect and filter undesirable content and protect sensitive information that might be present in user inputs or model responses (excluding reasoning content blocks).


# Guardrails Filters

1. Content Filters
    - Hate
    - Profaminty
    - Sexual
    - Violence
    - Insult
    - Misconduct 
    - Prompt Attack

    Can control input and output actions and the input and output modalities also

2. Topic Filters
    - Define what are the topics which are prohibited to generate response. For example, health advice or investement advice. Configure a standard response for such content.

3. Sensitive Information Filters
    - Define the PIIs like Name, SSN, Phone Number, Email ID, Address, Race etc

4. Word Filters
    - Define which words are prohibited in input and output context

5. Context Grounding 
    - To prevent hallucinations. Make sure that the model does not invent or give out of context responses

6. Automated Reasoning Checks 
    - Validate the accuracy of foundation model responses against a set of logical rules.


# Safeguard tiers
    - Classic 
        Provides guardrailes across English, French and Spansih

    - Standard
        Provides more robust performances compared to classic model and has more comprehensive language and code related prompt support


**If the model is being invoked by an AWS resource and guardrails need to be applied on top of the model invocation, then the respective AWS resource must have the policy to bedrock:ApplyGuardrail actions along with bedrock:InvokeModel actions. The ApplyGuardrail action must be applied on the Guardrail resouurce while the InvokeModel action must be applied on the Model.**

```json
    {
        "Version":"2012-10-17",
        "Statement": [
            {
                "Sid": "InvokeFoundationModel",
                "Effect": "Allow",
                "Action": [
                    "bedrock:InvokeModel",
                    "bedrock:InvokeModelWithResponseStream"
                ],
                "Resource": [
                    "arn:aws:bedrock:us-east-1::foundation-model/*"
                ]
            },
            {
                "Sid": "ApplyGuardrail",
                "Effect": "Allow",
                "Action": [
                    "bedrock:ApplyGuardrail"
                ],
                "Resource": [
                    "arn:aws:bedrock:us-east-1:123456789012:guardrail/guardrail-id"
                ]
            }
        ]
    }
```

For example a lambda resource is holding the Bedrock Converse/Invoke API to call the model and the guardrails then the lambda resource execution role must have the abovev permissions.

**Similarly if an AWS Resource like lambda is used to create the Guardrails then it must have the following policies attached to it in its Execution Policy**

```json
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Sid": "CreateAndManageGuardrails",
            "Effect": "Allow",
            "Action": [  
                "bedrock:CreateGuardrail",
                "bedrock:CreateGuardrailVersion",
                "bedrock:DeleteGuardrail", 
                "bedrock:GetGuardrail", 
                "bedrock:ListGuardrails", 
                "bedrock:UpdateGuardrail"
            ],
            "Resource": "*"
        }
    ]   
}
```


