
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

## Code Cell 2 ##

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



### Use Cases with Code Examples
1. Generate Recommendations Based on Metadata
- Task: Text generation
- Prompt technique: Zero-shot
- Model: Amazon Titan

## Code Cell 3 ##

prompt_data ="""
Human:
Generate a list of 10 recommended TV shows to watch, considering the information in the <metadata></metadata> XML tags. Include a very brief description of each recommendation.

<metadata>
Country is UK
Age range between 20-30
Shows must be about sports
</metadata>

Assistant:
"""

response, latency = call_bedrock('amazon.titan-text-premier-v1:0', prompt_data)
print(response, "\n\n", "Inference time:", latency)


2. Estimate Audience for TV Shows Based on Historical Data

- Task: Complex reasoning
- Prompt technique: Chain-of-thought (CoT)
- Model: Amazon Titan

## Code Cell 4 ##

prompt_data ="""
Human: Last week, three television channels had the following viewership data:
- Monday: SportsTV 6500, NewsTV 3200, EntertainmentTV 4150
- Tuesday: SportsTV 6400, NewsTV 3300, EntertainmentTV 4100
- Wednesday: SportsTV 6300, NewsTV 3400, EntertainmentTV 4250

Question: How many viewers can we expect next Friday on SportsTV?
Answer: According to the numbers given, and without having more information, there is a daily decrease of 100 viewers for SportsTV.
If we assume that this trend will continue for the next few days, we can expect 6200 viewers for the next day, which is Thursday, and
therefore 6100 viewers for Friday.

Question: How many viewers can we expect on Saturday for each of the three channels? Think step-by-step, and provide recommendations for increasing the viewers.
Assistant:
Answer:
"""

response, latency = call_bedrock('amazon.titan-text-premier-v1:0', prompt_data)
print(response, "\n\n", "Inference time:", latency)


3. Create a Question-Answering Assistant
- Task: Question answering with a dialogue Assistant (no memory)
- Prompt technique: Few-shot
- Model: Amazon Titan

## Code Cell 5 ##

prompt_data ="""
Context: The shows available are as follows
1. Circus, showing at the Plaza venue, assigned seating, live at 8pm on weekends
2. Concert, showing at the Main Theater, assigned seating, live at 10pm everyday
3. Basketball tricks, showing at the Sports venue, standing seating, live at 5pm on weekdays

Instruction: Answer any questions about the available shows. If you don't know the answer, say 'Apologies, I don't have the answer for that. Please contact our team by phone.'

Assistant: Welcome to Entertainment Tonight, how can I help you?
Human: Hi, I would like to know what shows are available please.
Assistant: Of course. Right now, we have the Circus, the Concert, and the Basketball tricks shows.
Human: Thank you. I would like to know when and where those shows are available please.
Assistant:
"""

response, latency = call_bedrock('amazon.titan-text-premier-v1:0', prompt_data)
print(response, "\n\n", "Inference time:", latency)


4. Create Splash Pages that Describe Upcoming Events
- Task: Code generation
- Prompt technique: Zero-shot
- Model: Amazon Titan

## Code Cell 6 ##

prompt_data ="""
An upcoming music concert is presented by the company, Music Promotions.
The event targets a young audience, age range between 18 and 40.
The event will occur in the Royal Music Theater.
Seating is assigned and tickets can be purchased through the Music Promotions website.
The event is a music concert performed by the band, Super Rockers.
The event will occur on June 30, 2023, and doors will open at 20:00.

Based on the information provided above, generate the HTML code for an attractive splash page to promote the event.
"""

response, latency = call_bedrock('amazon.titan-text-premier-v1:0', prompt_data)
print(response, "\n\n", "Inference time:", latency)
from IPython.display import display, HTML
display(HTML(response))



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
