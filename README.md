# Aws-T4-Automated-Image-Resizer

## 🚀 Building a Serverless Image Resizer on AWS: From Console Configuration to Production (Using Lambda Layers & ARNs!)

I just finished building an automated serverless image-resizing pipeline using AWS S3, Lambda, and Python, managing everything directly through the AWS console using Lambda Layer ARNs instead of local ZIP files.


<br>

## what happens under the hood every single time an image is uploaded and processed by the function:

🔹Upload: An image hits your source S3 bucket.

🔹Trigger: S3 fires an event notification payload directly to Lambda.

🔹Container Spin-up: AWS Lambda provisions a secure, isolated execution environment running your Python 3.12 runtime.

🔹Layer Injection: The Lambda Layer ARN mounts the pre-compiled Pillow library into the runtime's execution path, making from PIL import Image work.

🔹Event Parsing & Download: Your script reads the file metadata from the event payload and uses boto3 to download the raw image file from S3 into Lambda's local /tmp directory.

🔹Execution (Pillow): Pillow opens the file locally from /tmp, performs the thumbnail/resize math, and writes the modified image back to /tmp.

🔹Upload & Cleanup: The script uploads the newly resized image from /tmp into your destination S3 bucket, and the Lambda container eventually shuts down, wiping the temporary storage clean. 

<br>

<img width="1024" height="559" alt="048a9198-7ee4-4ff6-9950-98ccb07283ee" src="https://github.com/user-attachments/assets/375baa1c-4d02-4f29-bcc9-dc9184dbc47f" />

fig:1.1- Architecture Diagram


graph TD

    subgraph AWS Cloud Environment
        A["User / Client"] -->|"1. Uploads Image"| B["S3 Input Bucket"]
        B -->|"2. Triggers S3 Event"| C["AWS Lambda Function"]
        C -->|"3. Downloads to /tmp"| D["Ephemeral Storage /tmp"]
        D -->|"4. Processes & Resizes"| D
        C -->|"5. Uploads Resized Image"| E["S3 Output Bucket"]
    end




<br>

##  🛑 The Technical Hurdles & How I Solved Them
✨The Missing Module (No module named 'PIL'):

🔹The Issue: AWS Lambda environments are lean and don't include image-processing libraries out of the box.

🔹The Fix: Instead of dealing with local .zip file packages or terminal builds, I bypassed local packaging entirely by attaching a pre-compiled Lambda Layer ARN (Klayers-p312-Pillow) directly to the function.

✨Runtime Compatibility:

🔹The Issue: Python C-extensions are notoriously strict; a binary built for one version won't run on another.

🔹The Fix: Ensured the Lambda runtime matched the layer's specifications precisely at Python 3.12, making the integration seamless.

✨The Silent Memory Trap (CloudWatch Debugging):

🔹The Issue: Smaller test images processed instantly, but a larger third image caused the execution to hang silently.

🔹The Fix: Checked CloudWatch logs and discovered max memory usage hitting 218 MB against a 128 MB ceiling. Bumping the memory configuration up to 256 MB gave the function the exact RAM and CPU headroom needed to process larger files without choking.


<br>




   
## Why These Particular Steps?
Every step in this architecture was chosen to satisfy the core requirements of being Serverless, Event-Driven, and Cost-Effective.


### Architectural Choices

| Component | Why it was chosen |
| :--- | :--- |
| **Event-Driven (S3 Trigger)** | Eliminates the need for a server running 24/7. Lambda remains idle (costing $0) until S3 pushes a notification, representing the core efficiency of serverless computing. |
| **AWS Lambda** | Provides a scalable, managed compute environment. It removes the need to manage servers, OS, or patching, scaling automatically from one to thousands of concurrent requests. |
| **Separate Buckets** | Using distinct buckets for Input and Output ensures a clean separation of concerns and prevents infinite recursion loops during processing. |
| **Ephemeral Storage (/tmp)** | Image manipulation requires local disk access. Lambda provides temporary scratch space in `/tmp`, perfect for staging images during the brief processing lifecycle. |
| **Lambda Layers (Pillow)** | Used to manage the critical Pillow (PIL) dependency. This keeps function code clean, modular, and reusable across multiple serverless functions. |
| **Python 3.12 Runtime** | Selected for its scripting maturity, developer ease-of-use, and extensive support for pre-compiled binary layers, ensuring seamless compatibility for dependencies. |


<br>
<img width="1652" height="520" alt="Screenshot 2026-08-13 112444" src="https://github.com/user-attachments/assets/7ff3f097-c175-44e1-af0d-9af69aa73027" />

fig:1.2- Uploaded original size image in s3 bucket (input bucket).


<br>

 
<img width="1654" height="520" alt="Screenshot 2026-08-13 112426" src="https://github.com/user-attachments/assets/31ebe57d-45af-4e5f-a6ec-df68e7dba938" />

fig:1.3- Resized output image in s3 bucket (output bucket).


<br>

## 💡 Key Takeaway
Serverless development isn't just about writing code—it's about understanding how runtime environments, resource limits, and cloud configurations talk to each other. By leveraging inline editing, Layer ARNs, and CloudWatch metrics, we can spin up powerful production-ready pipelines entirely from the cloud console! ☁️✨
