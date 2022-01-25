## **Deploying a support platform that delivers exceptional customer experience**
The Genesys Cloud Developer Blueprint provides a simple example of deploying Limitless GigCX on Genesys Cloud to manage and deliver expert customer support.

This blueprint demonstrates how to:

- Connect To Limitless SmartCrowd
- Open messaging
- Chat (This flow is compatible with web chat v1.1 & web chat v2)
- Connect Limitless to your email channels
- Additional Considerations

## **Limitless GigCX**
Via Limitless GigCX your organization can quickly build a qualified crowd of expert product users to resolve customer support tickets. Experts are available 24/7, in any language, to deliver amazing support at a significantly lower cost to serve. You can find out more about Limitless and GigCX at: <https://www.limitlesstech.com/>

This blueprint brings the power of Limitless and GigCX to Genesys Cloud architect flows. Using these flows, open messaging, chat, and email channels in Genesys Cloud can be connected to Limitless APIs to bring GigCX Experts into the conversation via the SmartCrowd platform. 

The flow performs the following functions:

- Routing of conversations to Expert Crowd
- Integration between Genesys Cloud and the Limitless SmartCrowd platform
- Facilitating the conversation between Customer and GigCX Expert
- Manages the lifecycle/status events coming from the SmartCrowd platform and routes accordingly within Genesys
- Brings a Genesys Agent into the conversation if the Expert Crowd is unable to help the Customer

A high-level representation of the implementation can be seen in the diagram.

![](images/001.png) 

You can learn more about Limitless and Genesys at: <https://www.limitlesstech.com/genesys/>

# **Implementation steps**

## **Connecting to Limitless SmartCrowd**

The first step. Without connectivity to Limitless SmartCrowd, you cannot bring Experts into the conversation. The Expert Crowd uses the SmartCrowd platform to receive conversations and respond to them. Experts can do this via Web or Mobile App, and they receive rewards on a per task basis. Limitless operates a fully managed service for the Expert Crowd, so that all required is a Limitless Client to connect to the platform and route tasks. This blueprint shows you how to achieve that for Genesys Cloud.

Steps to connectivity:

1. Contact Limitless and our friendly team to request a demo: <https://www.limitlesstech.com/request-a-demo/>
1. The Limitless Team will provide the following for inclusion in your Genesys organization:
   1. Your API Key (x-api-key)
   1. The Group Name for your implementation
   1. User Credentials so you can play the role of an Expert in your Group on the SmartCrowd platform

The API key and Group Name are passed through to SmartCrowd via Data Actions, and these values enable you to submit questions and customer dialogues to Limitless.

## **The data actions**
We will use Data Actions to call the Limitless APIs. There are twelve data actions saved in this blueprint under the "Data Actions" subdirectory. Before importing the Data Actions, you should be sure to have both the "Web Services Data Actions" integration and the “Genesys Cloud Data Actions” installed and made active. You can learn more about installing and activating this integration here: <https://help.mypurecloud.com/articles/about-web-services-data-actions-integration/>

Now we import each of the twelve Data Actions. You can learn how to import Data Actions here: <https://help.mypurecloud.com/articles/import-or-export-a-data-action-for-integrations/>

Each data action connects to an API - some are Genesys Cloud APIs, and some are Limitless Web Services, as shown in the table below. You can read more about Genesys APIs here: <https://developer.genesys.cloud/api/>

The Data Actions to import are as follows:

|**Data Action Name**|**Integration**|**Genesys API**|**Used In**|**Purpose**|
| :-: | :-: | :-: | :-: | :-: |
|Limitless - Get Contact By Phone from Genesys v1|Genesys Cloud|/api/v2/externalcontacts/contacts?q=${input.phone}|- Open messaging flow|Looks up the Customer in the Genesys External Contacts using the phone number. Returns the Customers' First Name and first initial of their Last Name (this is used for personalisation of responses) Note: In your contact center you may be using a CRM for this information and should use your CRM API to get the customer details) |
|Limitless - Get Message Id Count from Genesys v1|Genesys Cloud|/api/v2/conversations/messages/${input.conversationId}|- Open messaging flow- Chat flow|Counts the number of messages authored by the Customer for this Genesys Conversation. Used in logic to send additional customer dialogues to Limitless|
|Limitless - Get Last N Message Ids from Genesys v1|Genesys Cloud|/api/v2/conversations/messages/${input.conversationId}|- Open messaging flow|Returns the last N message ids of messages authored by the Customer on this Genesys Conversation. Used in logic to send additional customer dialogues to Limitless|
|Limitless - Get Last Message from Genesys v1|Genesys Cloud|/api/v2/conversations/messages/${input.conversationId}/messages/${input.messageId}|- Open messaging flow|Returns the Customer authored dialogue (e.g. message body) for a specific message so it can be sent to Limitless|
|Limitless - Get Message Ids From Chat Conv By Id from Genesys v1|Genesys Cloud|/api/v2/conversations/chats/${input.conversationId}/messages?sortOrder=${input.sortOrder}|- Chat flow|Get any new message ids from the chat, oldest first|
|Limitless - Get Message Ids From Chat Conv By Id And MsgId from Genesys v1|Genesys Cloud|/api/v2/conversations/chats/${input.conversationId}/messages?after=${input.messageId}&sortOrder=${input.sortOrder}|- Chat flow|Additionally searches with messageId/api/v2/conversations/chats/${input.conversationId}/messages?sortOrder=${input.sortOrder}vs./api/v2/conversations/chats/${input.conversationId}/messages?after=${input.messageId}&sortOrder=${input.sortOrder}
|Limitless - Push Question to SmartCrowd v1|Web Services|N/A| - Open messaging flow - Email flow- Chat flow |Submits the question/conversation to Limitless.|
|Limitless - Submit FollowUp Dialogue to SmartCrowd v1|Web Services|N/A| - Open messaging flow - Email Flow - Chat flow |Submits new Customer authored dialogues to Limitless.|
|Limitless - Return from SmartCrowd v1|Web Services|N/A| - Open messaging flow - Chat flow |Tells SmartCrowd to close its message lifecycle as Genesys has brought an Agent into the conversation.|
|Limitless - Get Full Event from SmartCrowd v1|Web Services|N/A| - Open messaging flow - Email flow - Chat flow</p>|Retrieves Expert Dialogue and SmartCrowd Message Status events from the Limitless event queue.|
|Limitless Send Email Reply v1|Web Services|N/A|- Email flow|Send the Expert response via email using an SMTP server.|
|Limitless - Shorten Link v1|Web Services|N/A| - Open messaging flow - Email flow - Chat flow |Used to call out to a 3rd Party url shortener to reduce the size of the CSAT link. The example url here is for Bitly: <https://bitly.com/> and to use that service you will need a Bitly account and then to input your account specific token into the header. You can also replace Bitly with your url shortner of choice.|
After importing all the data actions, you will need to publish them to be used in our Architect flows.

## **The architect flows**

An Architect Flow is needed to send a customer question (from an incoming message) to Limitless. This blueprint contains the following flows:

- **Open messaging** - to connect Limitless to your open messaging channels
- **Chat** - Connect Limitless to your web chat channels (note: This flow is compatible with web chat v1.1 & web chat v2)
- **Email** - Connect Limitless to your email channels 

You can import these flows from the "Architect Flows" GUI. Learn more about importing Architect Flows here: <https://help.mypurecloud.com/articles/import-export-call-flow/>
### **Flow Variables**
For all flows you will need to set the following Flow Variables. You can set these in the ***Resources>Data*** area of each flow. 

![](images/003.png) 

More information about managing variables can be found here: <https://help.mypurecloud.com/articles/manage-a-variable/>

|**Variable**|**Set to**|
| :-: | :-: |
|Flow.apikey|Your API key supplied by Limitless|
|Flow.stage|The target Limitless environment being used by this flow. This value will be supplied by Limitless|
|Flow.EnterpriseName|Set this to the name of your Enterprise. This value will appear in some of the Send Response blocks within the flow.|
|Flow.GroupName|Set this to the Group Name value that Limitless provided to you|
|Flow.LoopMax|Set to the maximum number of loops you want the Flow to go through (See ‘Enter The Loop’ section). The default value is 90 and will see you through most implementations - consider changing this in only consultation with the Limitless team.|
|Flow.WaitSeconds|Set to the wait time for the Loop that you require (See ‘Enter The Loop’ section). The default value is 8 seconds and will see you through most implementations - consider changing this in only consultation with the Limitless team.|

### **Flow states**
Each flow has two states:

- **Starting state** - The Starting State represents a very simple flow that engages Limitless GigCX by calling the Limitless Expert state. Other than demos, you probably won’t use this “Starting State” as you will have your own flows that you wish to extend to the Expert Crowd.
- **Limitless Expert** - This state contains the connectivity and conversation lifecycle management when the conversation is handled by Expert Crowd via the SmartCrowd platform. The logic contained in this state is described in more detail below, but you should only make changes here in consultation with the Limitless team. This state has been designed to drop it into your existing Genesys flows and bring the Expert Crowd into conversations where you wish.
### **The open messaging flow**
Use this to connect Limitless to your open messaging channels.
#### **Starting state**
This state contains:

- ***A Send Response Block*** - Debugging information that confirms you are in the Limitless Demo and the Genesys Message ID
- ***Switch Block***:
  - Case 1 - A simple example shows routing straight to an Agent rather than the GigCX Crowd
  - Case 2 - A simple example shows where automation/chatbot could be used
  - Case 3 - Shows routing to Limitless via the ‘Limitless Expert’ state

![](images/004.png)
#### **Limitless expert state**
This state contains the connectivity and conversation lifecycle management when the conversation is handled by Expert Crowd via the SmartCrowd platform.
##### **Look up the customer:**
The first step in our "Limitless Expert" flow is a Call Data Action using the ‘Get Contact By Phone' action.

![](images/005.png) 

This block uses the Customer’s phone number to retrieve the first N]name and first initial of their last name to include in the post of the question to Limitless. The customer's first name and first Initial of their last name are shared and can be seen by the Expert Crowd to aid the personalization of the Expert response. In your contact center, you may be using a Customer Relationship Management (CRM) solution, and you can replace this Data Action with a call to your CRM to retrieve the required customer information. 

***An important note here on ‘Transfer to ACD’ blocks in the flow*** - You can see exception handling throughout the flow sends the conversation to an ‘Escalated’ queue via a ‘Transfer to ACD’ block. This ‘Escalated’ queue should be replaced with the Agent queue you want conversations to go to if the ‘Limitless Expert' state hits a problem or if the Expert Crowd cannot help the customer.

![](images/006.png) 
##### **Submit the question to Limitless:**
Next, we have another Call Data Action using the ‘Limitless Push Question’ action. This submits the Customer’s question to the Limitless SmartCrowd platform, and it becomes visible to Experts in the GigCX Crowd. In the integration, the Genesys Conversation ID and Limitless Message ID are exchanged.

![](images/007.png) 
##### **Enter the loop:**
After successfully submitting the customer question to Limitless, the ‘Limitless Expert’ flow prepares to enter a loop. The Loop itself performs two functions:

1. Monitor the customer side of the conversation - detecting and submitting additional Customer dialogues to Limitless.
1. Monitor the Limitless side of the conversation - detecting and displaying Expert dialogues to the Customer and monitoring the Lifecycle state of the Limitless message within the SmartCrowd platform, and taking appropriate action based on these statuses.

By default, the Loop will execute 90 times with a wait time of 8 seconds. These values can be changed to adjust the customer experience in different messaging channels and scenarios - consider changing these values only in consultation with the Limitless Team.

The flow does the following to prepare for entering the loop:

1. Attaches the Limitless Message ID to the Conversation - This is used to signify a conversation is ‘with Limitless’ and this association is removed later in the flow when the Limitless portion of the conversation is over.
1. Sets the Customer Message Count to 1, signifying the question has been submitted to Limitless. This value is used within the loop to decide if additional customer dialogues need to be sent to Limitless.

![](images/008.png) 
##### **The loop**
At the start of each loop, the number of customer dialogues on the conversation is compared with the previous number at the start of the last loop. This is done by performing the Action: Get Message ID Count and the subsequent Decision block.

![](images/009.png) 

If the number is now greater than the saved number, the loop moves to the Customer Dialogue part to pick up and send  the additional customer dialogues to Limitless.


If the number is not greater than the saved number, the loop moves to the Limitless part to pick up and send Expert dialogues and monitor the status of the message within SmartCrowd so appropriate actions can be taken within the conversation.

If neither the customer nor the expert presents dialogue, the Wait block is executed. This Wait keeps the loop from “running away” and reaching the LoopMax count too quickly. A smaller value in the WaitSeconds flow variable can make the whole experience more responsive. However, too small of a WaitSeconds value will cause the entire loop to complete too soon. Again, please consult with the Limitless team before altering these default values.

##### **Customer dialogue side of the loop**
This part of the Loop picks up additional Customer dialogues to send to Limitless. It picks up the number of Customer dialogues authored since the last send to Limitless and then loops through them, submitting each one to Limitless via the ‘Limitless Submit Follow Up Dialogue’ Data Action.

If the customer includes the word “Agent” then the loop uses a ‘Transfer to ACD” block to immediately bring in an Agent. You would consider the logic you want here, if any. If this ‘Transfer to ACD’ block is triggered, and the Agent is brought into the conversation, the Flow informs Limitless SmartCrowd via the 'Limitless Return’ Data Action. This ensures the conversation gets closed on the Limitless side and removed from Experts.

![](images/010.png) 
##### **Limitless side of the loop**
This part of the Loop monitors a Limitless Event queue for Expert dialogues and Limitless status events that need to trigger specific actions in the conversation. This part of the Loop starts with retrieving the events from the queue via the ‘Limitless Get Full Event Prod’ Data Action. 

![](images/011.png) 

The subsequent Event Type block then evaluates the following cases:

- ***Case 1*** - An Expert dialogue event is detected via the Limitless update type = ‘dlg’. The Expert dialogue is sent to the Customer via a Send Response block

![](images/012.png)

- ***Case 2*** - The Expert has expressed an opinion that they believe the conversation to have been completed and this event is detected via the Limitless update type = ‘cust\_confirmation’. If a CSAT hasn’t already been sent, then this event triggers a CSAT link (shortened via a Data Action) to be sent to the Customer. 

![](images/013.png)

- ***Case 3*** - A Limitless SmartCrowd status change event is detected via the Limitless update type = ‘state’. These status changes are further evaluated, and the following actions are performed:
  - ***‘Escalated’*** - The Expert Crowd has decided they cannot resolve this question. When this state is detected, the Customer receives a message stating that an Agent will be brought into the conversation, and a ‘Transfer to ACD’ block is used to bring in the Agent.
  - ***‘TimedOut’*** - The Expert Crowd did not respond in time. When this state is detected, the Customer receives a message stating that an Agent will be brought into the conversation, and a ‘Transfer to ACD’ block is used to bring in the Agent.
  - ***‘Open’ & Not ‘TimedOut’*** - The Limitless lifecycle is in a state where a CSAT, if not already issued, should be sent to the Customer. The flow actions this.
  - ***‘Resolved’*** - The Limitless lifecycle was completed, and the question resolved. This is an end state, and the flow disconnects. 

![](images/014.png)
### **The chat flow**
Use this to connect Limitless to your web chat channels (note: This flow is compatible with web chat v1.1 & web chat v2)
#### **Starting state**
This state begins by welcoming the customer and checking to ensure there is a customer message to respond to. 

![](images/015.png) 

Once this has been established, then the customers chat is routed to Limitless via a switch block:

- Case 1 - Shows example routing straight to an Agent rather than the GigCX Crowd
- Case 2 - Shows an example where automation/chatbot could be used
- Case 3 - Shows example routing to Limitless via the ‘Limitless Expert’ state

![](images/016.png)
#### **Limitless expert state**
This state contains the connectivity and conversation lifecycle management when the conversation is handled by Expert Crowd via the SmartCrowd platform.
#### **Submit the question to Limitless:**
First, we have a Call Data Action using the ‘Limitless Push Question’ action. This submits the Customer’s question to the Limitless SmartCrowd platform, and it becomes visible to Experts in the GigCX Crowd. The Genesys Conversation ID and Limitless Message ID are exchanged in the integration.

![](images/017.png) 
#### **Enter The loop:**
After successfully submitting the customer question to Limitless the ‘Limitless Expert’ flow prepares to enter a loop. The Loop itself performs two functions:

1. Monitor the customer side of the conversation - detecting and submitting additional Customer dialogues to Limitless.
1. Monitor the Limitless side of the conversation - detecting and displaying Expert dialogues to the Customer and monitoring the Lifecycle state of the Limitless message within the SmartCrowd platform, and taking appropriate action based on these statuses. 

![](images/018.png)

By default, the Loop will execute 90 times with a wait time of 8 seconds. These values can be changed to adjust the customer experience in different messaging channels and scenarios - consider changing these values only in consultation with the Limitless Team.

The flow does the following to prepare for entering the loop:

1. Attaches the Limitless Message ID to the Conversation - This signifies that a conversation is ‘with Limitless’, and this association is removed later in the flow when the Limitless portion of the conversation is over.
1. Sets the Customer Message Count to 1, signifying that the question has been submitted to Limitless. This value is used within the loop to decide if additional customer dialogues need to be sent to Limitless.

#### **The loop**
At the start of each loop the number of customer dialogues on the conversation is compared to the previous number at the start of the last loop. This is done by performing the Action: Get Message ID Count and the subsequent Decision block.

If the number is now greater than the saved number, the loop moves to the Customer Dialogue part to pick up and send the additional customer dialogues to Limitless.

If the number is not greater than the saved number, the loop moves to the Limitless part to pick up and send Expert dialogues and monitor the status of the message within SmartCrowd so appropriate actions can be taken within the conversation.

If neither the customer nor the expert presents dialogue, then the Wait block is executed.  This Wait keeps the loop from “running away” and reaching the LoopMax count too quickly. A smaller value in the WaitSeconds flow variable can make the whole experience more responsive.  However, too small of a WaitSeconds value will cause the entire loop to complete too soon.  Again, please consult with the Limitless team before altering these default values.

![](images/019.png) 
##### **Customer dialogue side of the loop**
This part of the Loop picks up additional Customer dialogues to send to Limitless. It picks up the number of Customer dialogues authored since the last send to Limitless. It then loops through them, submitting each one to Limitless via the ‘Limitless Submit Follow Up Dialogue’ Data Action.

If the customer includes the word “Agent” then the loop uses a ‘Transfer to ACD” block to immediately bring in an Agent. You would consider the logic you want here, if any. If this ‘Transfer to ACD’ block is triggered, and the Agent is brought into the conversation, then the Flow informs Limitless SmartCrowd via the 'Limitless Return’ Data Action. This is to ensure the conversation gets closed on the Limitless side and removed from Experts.

![](images/020.png) 
##### **Limitless side of the loop**
This part of the Loop monitors a Limitless Event queue for Expert dialogues and Limitless status events that need to trigger specific actions in the conversation. This part of the Loop starts with retrieving the events from the queue via the ‘Limitless Get Full Event Prod’ Data Action. 

![](images/021.png) 

The subsequent Event Type block then evaluates the following cases:

- ***Case 1*** - An Expert dialogue event is detected via the Limitless update type = ‘dlg’. The Expert dialogue is sent to the Customer via a Send Response block

![](images/022.png)

- ***Case 2*** - The Expert has expressed an opinion that they believe the conversation has been completed and this event is detected via the Limitless update type = ‘cust\_confirmation’. If a CSAT has not been sent, this event triggers a CSAT link (shortened via a Data Action) to send to the Customer.

![](images/023.png)

- ***Case 3*** - A Limitless SmartCrowd status change event is detected via the Limitless update type = ‘state’. These status changes are further evaluated, and the following actions are performed:
  - ***‘Escalated’*** - The Expert Crowd has decided they cannot resolve this question. When this state is detected, the Customer receives a message stating that an Agent will be brought into the conversation, and a ‘Transfer to ACD’ block is used to bring in the Agent.
  - ***‘TimedOut’*** - The Expert Crowd did not respond in time. When this state is detected, the Customer receives a message stating an Agent will be brought into the conversation, and a ‘Transfer to ACD’ block is used to bring in the Agent.
  - ***‘Open’ & Not ‘TimedOut’*** - The Limitless lifecycle is in a state where a CSAT, if not already issued, should be sent to the Customer. The flow actions this.
  - ***‘Resolved’*** - The Limitless lifecycle was completed, and the question resolved. This is an end state, and the flow disconnects.

![](images/024.png)
### **The email flow**
Use this to connect Limitless to your email channels 
#### **Starting state**
This state contains:

- ***A Send Response Block*** - Debugging information that confirms you are in the Limitless Demo and the Genesys Message ID
- ***Switch***:
  - ***Case 1*** - Shows routing straight to an Agent rather than the GigCX Crowd
  - ***Case 2*** - Shows where automation/chatbot could be used (Self help)
  - ***Case 3*** - Sends a follow-up message to Limitless via the Data action (customer replies to the thread)

![](images/025.png)

- ***Default*** - Routing the original question to the expert. This part of the flow contains a Call Data Action using the ‘Limitless Push Question’ action. This submits the Customer’s question to the Limitless SmartCrowd platform, and it becomes visible to Experts in the GigCX Crowd. The Genesys Conversation ID and Limitless Message ID are exchanged in the integration.

![](images/026.png)
#### **Limitless expert state**
This state contains the connectivity and conversation lifecycle management when the conversation is handled by Expert Crowd via the SmartCrowd platform.
#### **Enter The loop:**
After successfully submitting the customer question to Limitless, the ‘Limitless Expert’ flow prepares to enter a loop. The Loop performs one function, to monitor the Limitless side of the conversation - detecting and displaying Expert dialogues to the Customer and monitoring the Lifecycle state of the Limitless message within the SmartCrowd platform and taking appropriate action based on these statuses.

![](images/027.png) 

By default, the Loop will execute 90 times with a wait time of 8 seconds. These values can be changed to adjust the customer experience in different Email channels and scenarios - consider changing these values only in consultation with the Limitless Team.

To prepare for entering the loop, the flow attaches the Limitless Message ID to the Conversation - This is used to signify that a conversation is ‘with Limitless’, and this association is removed later in the flow when the Limitless portion of the conversation is over.

#### **The loop**
The loop picks up and sends Expert dialogues and monitors the message's status within SmartCrowd so appropriate actions can be taken within the conversation.

If the expert presents a dialogue, then the Wait block is executed. This Wait keeps the loop from “running away” and reaching the LoopMax count too quickly. A smaller value in the WaitSeconds flow variable can make the whole experience more responsive. However, too small of a WaitSeconds value will cause the entire loop to complete too soon. Again, please consult with the Limitless team before altering these default values.
##### **Monitoring Limitless**
This part of the Loop monitors a Limitless Event queue for Expert dialogues and Limitless status events that need to trigger specific actions in the conversation. This part of the Loop starts with retrieving the events from the queue via the ‘Limitless Get Full Event Prod’ Data Action. The subsequent Event Type block then evaluates the following cases:

- ***Case 1*** - An Expert dialogue event is detected via the Limitless update type = ‘dlg’. The Expert dialogue is sent to the Customer via a Send Response block

![](images/028.png)

- ***Case 2*** - The Expert has expressed an opinion they believe the conversation has been completed and this event is detected via the Limitless update type = ‘cust\_confirmation’. If a CSAT has not been sent, then this event triggers a CSAT link (shortened via a Data Action) to be sent to the Customer.

![](images/029.png)

- ***Case 3*** - A Limitless SmartCrowd status change event is detected via the Limitless update type = ‘state’. These status changes are further evaluated, and the following actions are performed:
  - ***‘Escalated’*** - The Expert Crowd has decided they cannot resolve this question. When this state is detected, the Customer receives a message stating an Agent will be brought into the conversation, and a ‘Transfer to ACD’ block is used to bring in the Agent.
  - ***‘TimedOut’*** - The Expert Crowd did not respond in time. When this state is detected, the Customer receives a message stating an Agent will be brought into the conversation, and a ‘Transfer to ACD’ block is used to bring in the Agent.
  - ***‘Open’ & Not ‘TimedOut’*** - The Limitless lifecycle is in a state where a CSAT, if not already issued, should be sent to the Customer. The flow actions this.
  - ***‘Resolved’*** - The Limitless lifecycle was completed, and the question resolved. This is an end state, and the flow disconnects.

![](images/030.png)
## **Cloud analytics & reporting**
The use of the Limitless flows has no impact on Cloud Analytics or Genesys reporting.

## **Other considerations**
- ***Flow Outcomes*** - The blueprint does not contain Flow Outcomes to avoid validation errors on import. You should consider what Flow Outcomes you want and introduce these to Flow.

