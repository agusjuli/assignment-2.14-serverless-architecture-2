# Assignment-2.14-serverless-architecture-2

### Question:     
Does SNS guarantee exactly once delivery to subscribers?     

### Answer:     
No, **AWS Simple Notification Service (SNS)** does **not guarantee exactly-once delivery** to subscribers. It guarantees **at-least-once delivery**, which means a message will be delivered to subscribers **one or more times**. This can lead to **duplicate messages** in certain scenarios.     

### Why Duplicates Can Occur   
1. **Network Failures:** If an acknowledgment is not received due to a network glitch, SNS may retry delivery, causing duplicates.    
2. **Subscriber Unavailability:** If a subscriber is temporarily unavailable, SNS may retry delivery.    
3. **Internal Errors:** Rare internal errors can also cause duplicate message deliveries.   

### How to Handle Duplicates   
To handle duplicates effectively:   
- **Idempotency:** Design subscribers to process messages idempotently. For example, include a unique message ID and track processed messages.   
- **Deduplication Logic:** Use a caching mechanism (like Redis or DynamoDB) to store processed message IDs temporarily.    

If we need **exactly-once processing**, consider using **AWS SQS with FIFO queues** combined with SNS, which helps maintain order and avoid duplicates at the cost of higher complexity.    
<br>   

### Question:   
What is the purpose of the Dead-letter Queue (DLQ)? This is a feature available to SQS/SNS/EventBridge.    

### Answer:   
The purpose of a Dead-Letter Queue (DLQ) in services like Amazon SQS, SNS, and EventBridge is to handle messages or events that cannot be processed successfully. Essentially, it acts as a safety net for failed deliveries.    

### Here's a breakdown:    
Core Function:    
🎯 A. Error Handling:   
DLQs capture messages that fail to be processed by consumer applications or target services. This prevents those failed messages from being lost and allows for later analysis and reprocessing.      
📧 B. Failure Isolation:  
By moving failed messages to a separate queue, DLQs isolate them from the main message flow, preventing them from repeatedly causing failures and disrupting normal operations.      
⚙️ C. Debugging and Analysis:       
DLQs provide a repository of failed messages, which can be examined to identify the root causes of errors, such as: Code bugs, Network issues, Invalid message formats, Target service unavailability.    
📊 D. Reprocessing:          
Once the underlying issues are resolved, the messages in the DLQ can be reprocessed, ensuring that no data is lost.         

### How it Applies to Specific Services:   
🎯 A. Amazon SQS:    
In SQS, a DLQ is used to store messages that cannot be processed after a specified number of retries. This is particularly useful for handling situations where a consumer application repeatedly fails to process a message.    
📧 B. Amazon SNS:    
In SNS, a DLQ can be attached to a subscription to capture messages that cannot be delivered to the subscriber. This can happen due to issues with the subscriber's endpoint, such as unavailability or permission problems.   
⚙️ C. Amazon EventBridge:    
In EventBridge, a DLQ is used to store events that cannot be delivered to a target. This allows you to handle situations where a target service is unavailable or encounters errors while processing events.  
      
***In essence***, DLQs enhance the reliability and robustness of message-driven and event-driven architectures by providing a mechanism for handling failures and preventing data loss.   
<br>    

### Question:    
How would you enable a notification to your email when messages are added to the DLQ?

### Answer:  
To enable **email notifications** when messages are added to a **Dead Letter Queue (DLQ)** in AWS, we can follow these steps:

🎯 **A. Create an SNS Topic for Notifications**   
1. Go to the **SNS Console** in AWS.   
2. Click **"Create topic"**.   
   - **Type:** Standard (default).  
   - **Name:** Choose a descriptive name (e.g., `DLQ-Alerts`).   
3. Click **"Create topic"**.    

📧 **B. Subscribe Your Email to the SNS Topic**   
1. In the SNS Console, select your new topic (`DLQ-Alerts`).   
2. Go to the **"Subscriptions"** tab and click **"Create subscription"**.   
   - **Protocol:** Email     
   - **Endpoint:** Your email address     
3. Check your email for a **confirmation link** and confirm the subscription.   

⚙️ **C. Create a CloudWatch Alarm for DLQ Monitoring**   
1. Go to the **CloudWatch Console**.   
2. In the left menu, click **"Alarms" > "All alarms" > "Create alarm"**.   
3. **Select metric:**  
   - Choose **SQS metrics** > **By queue name** > Select your **DLQ**.  
   - Click on the **"ApproximateNumberOfMessagesVisible"** metric (indicates messages waiting in the DLQ).    
4. Click **"Select metric"**.   

📊 **D. Configure Alarm Settings**  
1. **Specify the threshold:**   
   - **Threshold type:** Static     
   - **Condition:** Greater than 0 (or another threshold if you want).     
2. Click **"Next"**.  

🔔 **E. Set Notification for the Alarm**  
1. **Send a notification to:**   
   - Choose **"In an existing SNS topic"**.    
   - Select the **`DLQ-Alerts`** SNS topic you created earlier.        
2. Click **"Next"**.   

🏷 **F. Name and Create the Alarm**   
1. Give your alarm a descriptive **name** (e.g., `DLQ-Message-Alert`).   
2. Click **"Create alarm"**.    

✅ **G. Test the Setup**     
1. Manually move a message to the DLQ (or wait for a real one).    
2. Check if you receive an **email notification**.    

That’s it! 🎉 You’ll now get an email notification whenever a message lands in the DLQ.       

    
