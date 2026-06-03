# Amazon Lex Chatbot
## Project Overview
#### A conversational Pizza Ordering Chatbot built with Amazon V2 Lex and AWS Lambda. Customers can place pizza orderd, choose sizes, and confirm delivery all through natural conversation!
### Features

* Natural Language Understanding via Amazon Lex V2
* Full pizza customization - size, quantity
* Delivery address collection via slot elicitation
* Multi-turn conversation with context carry-over
* Serverless backend with AWS Lambda
* Monitoring via Amazon CloudWatch
* Deployable to Web, Slack, Facebook Messenger, or Twillio SMS
  
  ### Architecture
  Customer Chat Input
        │
        ▼
┌──────────────────────┐
│    Amazon Lex V2     │  ◄── Intents, Slots, Utterances
│  Pizza Ordering Bot  │
└────────┬─────────────┘
         │ Lambda Fulfillment
         ▼
┌──────────────────────┐
│    AWS Lambda        │  ◄── Order Processing Logic
│  (Python 3.9)        │
└────────┬─────────────┘
         │
         ▼
┌──────────────────────┐
│   Amazon DynamoDB    │  ◄── Orders Table (store & track)
└──────────────────────┘
         │
         ▼
  Confirmation to Customer

  ### Prerequisites

  * An AWS Account with appropriate permissions
  * AWS CLI installed and configured
  * Python 3.9+
  * Basic lnowledge of Amazon Lex V2, Lambda, and DynamoDB
    
  


  ## AWS Account Setup

  ### Step 1 : Logging In to the Amazon Web Service Console
  login to your AWS account using your credentials.
  
  ### Step 2 : Open Amazon Lex 
  1 Click Amazon Lex
  2 Click "Create bot"
  ### Step 3 : Configure the Bot
  * Bot name : GreetingIntent
  * Description : A chatbot to order pizza 
  * IAM permissions : Select "Create a role with basic Amazon Lex permissions"
  * COPPA : No
  * Session timeout : 5 minutes
  * Click Next
  ### Step 4 : Set Language
  * Language : English (US)
  * Voice : Choose any (e.g., Joanna) or None for text only
  * Click Done
  ### Step 5 : Create PizzaSize Slot Type
  ### (Custom Options)
  1 In the left menu -> Click "Slot types" ->
  "Add slot type" -> "Add blank slot type"
  
  2 Name : PizzaSize
  
  3 Add values :

* Small

* Medium

* Large

* Extra Large

### Step 6 : Create CrustType Slot Type 
1 Add another slot type -> Name : CrustType

2 Add values :

* Thin Crust 
* Thick Crust
* Stuffed Crust
* Gluten Free

3 Click Save slot type

### Step 7 :  Create OrderPizza Intent
1 Left menu -> Intent -> "Add intent" -> "Add empty intent"

2 Name : OrderPizza
### Step 8 : Add Sample Utterances
Click "Add utterances" and add these : 

I want to order a pizza


Order a pizza


I'd like to buy a pizza


Get me a pizza


Can I order a pizza


I want a {PizzaSize} pizza


Give me a {PizzaSize} {CrustType} pizza

### Step 9 : Add Slots to OrderPizza 
Scroll down to Slot -> Add these one by one

## Pizza Order Slots

| Slot Name | Slot Type | Prompt |
|-----------|-----------|---------|
| PizzaSize | PizzaSize | What size pizza would you like? Small, Medium, Large, or Extra Large? |
| CrustType | CrustType | What crust type would you like? Thin, Thick, Stuffed, or Gluten Free? | |
| Quantity | AMAZON.Number | How many pizzas would you like to order? |
| DeliveryType | AMAZON.AlphaNumeric | Would you like Delivery or Pickup? |
| Address | AMAZON.StreetAddress | What is the delivery address? |

Mark all slota as Required 

### Step 10 : Add Confirmation Prompt

Scroll to Confirmation and add : 

* Confirm : Your Order : {Quantity} x {PizzaSize}  {CrustType} . Delivered to {Address}.
Shall I confirm ?

* Decline : No problem ! Your order has been cancelled.

### Step 11 : Create  AWS Lambda Function
1 Search "Lambda" in AWS Console

2 Click "Create function"

3 Select "Author from scratch"
   * Name : PizzaOrderingHandler
   * Runtime : Python 3.9
   * Click "Create function"
  
  ### Step 12 : Write the Lambda Code 
  In the code editor, replace everything with:
  
import json
import random
import string

def lambda_handler(event, context):
    intent_name = event['sessionState']['intent']['name']
    
    if intent_name == 'OrderPizza':
        return handle_order_pizza(event)
    elif intent_name == 'TrackOrder':
        return handle_track_order(event)
    elif intent_name == 'CancelOrder':
        return handle_cancel_order(event)
    else:
        return fallback(event)

def handle_order_pizza(event):
    slots = event['sessionState']['intent']['slots']
    size     = get_slot(slots, 'PizzaSize')
    crust    = get_slot(slots, 'CrustType')
    toppings = get_slot(slots, 'Toppings')
    quantity = get_slot(slots, 'Quantity')
    address  = get_slot(slots, 'Address')

    order_id = ''.join(random.choices(string.digits, k=4))
    message = (
        f" Order confirmed! Order ID: #PZ{order_id}\n"
        f" {quantity}x {size} {crust} Pizza\n"
        f" Toppings: {toppings}\n"
        f" Delivering to: {address}\n"
        f" Estimated time: 30-40 minutes!"
    )
    return close(event, message)

def handle_track_order(event):
    slots  = event['sessionState']['intent']['slots']
    order_id = get_slot(slots, 'OrderId')
    message = f" Order #{order_id} is on its way! Arriving in about 15 minutes."
    return close(event, message)

def handle_cancel_order(event):
    slots    = event['sessionState']['intent']['slots']
    order_id = get_slot(slots, 'OrderId')
    message  = f"❌ Order #{order_id} has been successfully cancelled."
    return close(event, message)

def fallback(event):
    message = "I didn't understand that. Try saying 'Order a pizza' or 'Track my order'."
    return close(event, message)

def get_slot(slots, name):
    try:
        return slots[name]['value']['interpretedValue']
    except (KeyError, TypeError):
        return 'N/A'

def close(event, message):
    return {
        "sessionState": {
            "dialogAction": {"type": "Close"},
            "intent": {
                "name": event['sessionState']['intent']['name'],
                "state": "Fulfilled"
            }
        },
        "messages": [{
            "contentType": "PlainText",
            "content": message
        }]
    }

Click "Deploy" (Orange Button)

### Step 13 : Connect Lambda to Lex
1 In Lambda -> Click "Configuration" tab -> "Permissions"

2 Click the Role name link -> Opens IAM 

3 Confirm the role exists (auto-created)

### Step 14 : Link Lambda in Lex
1 Go back to Amazon Lex -> Your bot

2 Left menu -> "Aliases" -> Click "TestBotAlias" 

3 Scroll to "Languages" -> Click "English(US)"

4 Source : Select Lambda function -> PizzaOrderingHandker

5 Version " $LATEST

6 Click Save

### Step 15 : Add Lambda Permission Via CLI (or Console)
aws lambda add-permission \
  --function-name PizzaOrderingHandler \
  --statement-id LexInvoke \
  --action lambda:InvokeFunction \
  --principal lexv2.amazonaws.com

  ### Step 16 : Build the Bot

  1 In Lex -> Left menu -> Click "Intents"

  2 Top right -> Click "Build"

  3 Wait for build to complete

  ### Step 17 : Test the Bot

  1 After build -> Click "Test" (top right)

  2 A chat panel opens on the right 

  3 Try these messages :

  "I want to order a pizza"
"Large"

"Stuffed Crust"

"Pepperoni and Cheese"
"2"

"Delivery"

"123 Main Street"

You Should get a full order confirmation with an order ID!

This is the project

   

  
   

