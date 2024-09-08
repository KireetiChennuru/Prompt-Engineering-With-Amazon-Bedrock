
# **Prompt Engineering with Amazon Bedrock**

## **Overview**

This project demonstrates a serverless single-page web application (SPA) designed to provide personalized prompt recommendations powered by Generative AI. The architecture utilizes various AWS services to ensure scalability, low latency, and secure content delivery while maintaining a serverless infrastructure.

### **Platform and Resources**
- **AWS Training and Certification**

### **Skills Demonstrated**
- Amazon Simple Storage Service (S3)
- Amazon CloudFront
- Amazon S3
- Amazon API Gateway
- AWS Lambda
- Amazon Bedrock
- Amazon SageMaker Studio
- Amazon SageMaker Notebook

## **Architecture Design**

### **Architecture Type:**  
The architecture is designed as a serverless single-page application (SPA) leveraging multiple AWS managed services for efficient content delivery, secure API management, and dynamic content generation. The solution is highly scalable, cost-effective, and optimized for low latency.

### **Core Components:**

1. **Amazon CloudFront**:  
   A highly distributed, fast, and secure content delivery network that serves as the entry point for customers accessing the application. CloudFront caches content to reduce latency and offload traffic from the origin S3 bucket, enhancing performance and reducing costs.

2. **Amazon S3**:  
   Stores static content for the SPA, which is delivered to customers via CloudFront. S3 provides durable storage for website assets and offloads traffic from the origin server.

3. **Amazon API Gateway**:  
   Acts as the "front door" for the application, managing and securing API requests. It enables developers to create, publish, maintain, and monitor APIs at any scale.

4. **AWS Lambda**:  
   Facilitates serverless execution of dynamic content generation and prompt enhancement processes. It interfaces with Amazon Bedrock to optimize and fine-tune customer prompts.

5. **Amazon Bedrock**:  
   Provides access to foundational models (FMs) from leading AI companies such as Anthropic, Cohere, Stability AI, and Amazon. It offers a serverless experience and access to cutting-edge generative AI capabilities.

6. **Amazon SageMaker Studio & Notebook**:  
   Used by data scientists to experiment with and refine prompt templates. The Studio and Notebook environments provide a fully managed, collaborative space for building, training, and deploying ML models.


## Prompt Engineering Techniques

### Word Choice
Selecting the most appropriate and specific words to make the prompt more precise and contextually relevant.

### Phrasing
Structuring the prompt in a way that aligns with the desired output and model capabilities.

### Providing Additional Information
Including necessary details or context in the prompt to guide the model toward generating accurate responses.

### Providing Examples
Giving examples within the prompt to set a pattern or context for the model to follow.

### Generate Recommendations Based on Metadata
Creating recommendations or outputs that are specifically tailored based on provided metadata or structured data.

## Code Implementation

### Import Libraries and Create an Amazon Bedrock Client

This step initializes the necessary libraries and creates an Amazon Bedrock client to interact with various foundation models.

```python
## Code Cell 1 ##

import boto3
import json
import csv
from datetime import datetime

bedrock = boto3.client('bedrock')
bedrock_runtime = boto3.client('bedrock-runtime')
bedrock.list_foundation_models()



### Create a Help Function for Calling Amazon Bedrock
- A helper function is defined to facilitate calling Amazon Bedrock models with different parameters depending on the model type.

## Code Cell 2 

def call_bedrock(modelId, prompt_data): 
    if 'amazon' in modelId:
        body = json.dumps({
            "inputText": prompt_data,
            "textGenerationConfig":
            {
                "maxTokenCount": 1024,
                "stopSequences":[],
                "temperature":0.7,
                "topP":0.9
            }
        })
    elif 'meta' in modelId:
        body = json.dumps({
            "prompt": prompt_data,
            "max_tokens_to_sample": 4096,
            "stop_sequences":[],
            "temperature":0,
            "top_p":0.9
        })
    elif 'mistral' in modelId:
        body = json.dumps({
            "prompt": prompt_data,
            "max_tokens_to_sample": 4096,
            "stop_sequences":[],
            "temperature":0,
            "top_p":0.9
        })
        print('Parameter model must be one of Titan, Lama, or Mixtral')
        return
    accept = 'application/json'
    contentType = 'application/json'

    before = datetime.now()
    response = bedrock_runtime.invoke_model(body=body, modelId=modelId, accept=accept, contentType=contentType)
    latency = (datetime.now() - before)
    response_body = json.loads(response.get('body').read())

    if 'amazon' in modelId:
        response = response_body.get('results')[0].get('outputText')
    elif 'meta' in modelId:
        response = response_body.get('completion')
    elif 'mistral' in modelId:
        response = response_body.get('completions')[0].get('data').get('text')
        
    return response, latency
##

### Use Cases with Code Examples
1. Generate Recommendations Based on Metadata

- Task: Text generation
- Prompt technique: Zero-shot
- Model: Amazon Titan























  

## **Testing and Validation**

1. **Unit Testing**:  
   - Verify that individual components, such as Lambda functions and API Gateway endpoints, work as expected.
   - Use AWS Lambda testing tools and API Gateway testing capabilities for initial validation.

2. **Integration Testing**:  
   - Ensure seamless integration between all components, including CloudFront, S3, API Gateway, Lambda, Bedrock, and SageMaker Studio.
   - Validate end-to-end functionality by simulating customer requests and monitoring data flow through the architecture.

3. **Performance Testing**:  
   - Test the application's response times and scalability under different load conditions.
   - Utilize Amazon CloudFront metrics and AWS X-Ray to analyze performance bottlenecks and optimize resource allocation.

4. **Security Testing**:  
   - Conduct a comprehensive security assessment to ensure that all components adhere to AWS security best practices.
   - Validate API Gateway security configurations and access controls for Lambda functions and S3 buckets.

5. **User Acceptance Testing (UAT)**:  
   - Perform testing with actual end-users to gather feedback and ensure that the application meets customer requirements and provides an optimal user experience.

## **Project Repository Structure**

```plaintext
/Prompt-Engineering-With-Bedrock
│
├── README.md                  # Project overview and documentation
├── architecture-diagram.png   # AWS Architecture Diagram
├── src/
│   ├── api/                   # API Gateway definitions and configurations
│   ├── lambda/                # Lambda function code for prompt processing
│   ├── s3/                    # Static content for the SPA
│   └── notebooks/             # SageMaker Studio Notebook files
│
├── test/
│   ├── unit/                  # Unit test scripts and cases
│   └── integration/           # Integration test scripts and cases
│
├── scripts/
│   └── deployment/            # Scripts for deploying resources to AWS
└── LICENSE                    # License information
