# **Prompt Engineering with Amazon Bedrock**

## **Overview**

This project demonstrates a serverless single-page web application (SPA) designed to provide personalized prompt recommendations powered by Generative AI. The architecture utilizes various AWS services to ensure scalability, low latency, and secure content delivery while maintaining a serverless infrastructure.

### Architecture Diagram
Refer to the architecture diagram visual representation of the setup and illustrates the various AWS services involved in this project

![Architecture Diagram](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/Cloud%20Architecture.png?raw=true)


## **Architecture Design**

### **Architecture Type:**  
The architecture is designed as a serverless single-page application (SPA) leveraging multiple AWS managed services for efficient content delivery, secure API management, and dynamic content generation. The solution is highly scalable, cost-effective, and optimized for low latency.

## Architecture Diagram Core Aspects:

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

### **Services Used**
- Amazon Simple Storage Service (S3)
- Amazon CloudFront
- Amazon S3
- Amazon API Gateway
- AWS Lambda
- Amazon Bedrock
- Amazon SageMaker Studio
- Amazon SageMaker Notebook

# Project Setup Guide

## 1. Request Access to Amazon Titan FM

1. **Log in to AWS Management Console**
   - Go to the [AWS Management Console](https://aws.amazon.com/console/).

2. **Request Acces**
   - Search for and select Amazon Bedrock from the services list and request access to Amazon Titan FM.
  
![Request Access to Amazon Titan FM](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/Amazon%20Bedrock%20-%20FMs.png?raw=true)


## 2. Explore Bedrock Text Playground

1. **Open Amazon Bedrock**
   - Go to the Amazon Bedrock console.

2. **Access Text Playground**
   - Locate and open the Bedrock Text Playground.

3. **Select Model**
   - Choose "Amazon Titan Express" from the list of available models.

4. **Use Sample Prompt**
   - Enter a sample prompt to generate output and evaluate the results.

![Explore Bedrock Text Playground](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/Bedrock%20-%20Text%20Playground%20.png?raw=true)


## 3. Setup in SageMaker Studio

1. **Open SageMaker Studio**
   - Navigate to SageMaker Studio from the AWS Management Console.
  
![Setup in SageMaker Studio](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/SageMaker%20Studio.png?raw=true)

2. **Open a Terminal**
   - Launch a terminal in SageMaker Studio.

   ```bash
   # Create S3 Bucket
   aws s3 mb s3://your-bucket-name

   # Upload HTML Files
   aws s3 cp /path/to/your/html-files s3://your-bucket-name --recursive

![Setup in SageMaker Studio](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/SageMaker%20Studio%20-%20Terminal.png?raw=true)

![Setup in SageMaker Studio](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/SageMaker%20Studio%20-%20Terminal%202.png?raw=true)

## 4. Create Lambda Function

1. **Write Lambda Function Code: Create a Lambda function to invoke Amazon Bedrock FM**
   - Deploy Lambda Function: Use the AWS CLI or console to deploy the function.
   - Grant Permissions: Ensure the Lambda function has the necessary permissions to access Amazon Bedrock.

![Create Lambda Function](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/Lambda%20Function%20-%20Invoke%20Bedrock.png?raw=true)


## 5. Configure CloudFront for SPA Access
1. **Create CloudFront Distribution**
   - Open CloudFront: Go to the CloudFront console.
   - Create Distribution: Set up a new distribution.
   - Configure Settings: Point to the S3 bucket where your HTML files are stored.

2. **Access SPA**
   - Obtain CloudFront Domain: Retrieve the CloudFront distribution domain name.
   - Provide Prompt Inputs: Access your Single Page Application (SPA) using the CloudFront domain and enter prompt inputs.

![Configure CloudFront](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/CloudFront-Distributions%20Domain%20.png?raw=true)


## **Prompt Engineering Techniques**

- **One-shot**: One-shot learning is when you provide an AI with just one example or instruction to help it understand the task.
- **Few-shot**: Few-shot learning involves giving the AI a few examples (more than one but not too many) to help it understand the task better.
- **Chain of Thought (CoT)**: Chain of Thought (CoT) is a technique where you encourage the AI to explain its thinking step-by-step..
- **Tree of Thought (ToT)**: Tree of Thought (ToT) is a technique where the AI explores multiple possible paths or solutions before deciding on the best one. 

## **Code Implementation**

### **Import Libraries and Create an Amazon Bedrock Client**

- This step initializes the necessary libraries and creates an Amazon Bedrock client to interact with various foundation models.

```
import boto3
import json
import csv
from datetime import datetime

bedrock = boto3.client('bedrock')
bedrock_runtime = boto3.client('bedrock-runtime')
bedrock.list_foundation_models()
```

### **Create a Helper Function for Calling Amazon Bedrock**
- A helper function is defined to facilitate calling Amazon Bedrock models with different parameters depending on the model type.

```
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
```

### Use Cases with Code Examples

#### 1. Generate Recommendations Based on Metadata
   - Task: Text generation
   - Prompt technique: Zero-shot
   - Model: Amazon Titan

```
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
Estimate Audience for TV Shows Based on Historical Data

```


#### 2. Estimate Audience for TV Shows Based on Historical Data
   - Task: Complex reasoning
   - Prompt technique: Chain-of-thought (CoT)
   - Model: Amazon Titan

```
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
```



#### 3. Create a Question-Answering Assistant
   - Task: Question answering with a dialogue Assistant (no memory)
   - Prompt technique: Few-shot
   - Model: Amazon Titan
```
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

```

#### 4. Create Splash Pages that Describe Upcoming Events
   - Task: Code generation
   - Prompt technique: Zero-shot
   - Model: Amazon Titan

```
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

```

# **Testing and Validation**

1. **Unit Testing**:  
   - Verify that individual components, such as Lambda functions and API Gateway endpoints, work as expected.
   - Use AWS Lambda testing tools and API Gateway testing capabilities for initial validation.

2. **Integration Testing**:  
   - Ensure seamless integration between all components, including CloudFront, S3, API Gateway, Lambda, Bedrock, and SageMaker Studio.
   - Validate end-to-end functionality by simulating customer requests and monitoring data flow through the architecture.

3. **User Acceptance Testing (UAT)**:  
   - Perform testing with actual end-users to gather feedback and ensure that the application meets customer requirements and provides an optimal user experience.


# **Output**

![Output](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/Prompt%20Enginnering%20WIth%20Bedrock%20-%20Output%20.png?raw=true)

![Output](https://github.com/KireetiChennuru/Prompt-Engineering-With-Amazon-Bedrock/blob/main/Project_Files/Prompt%20Enginnering%20WIth%20Bedrock%20-%20Output%202.png?raw=true)



## Author

- Kireeti Chennuru | www.linkedin.com/in/kireeti-chennuru
