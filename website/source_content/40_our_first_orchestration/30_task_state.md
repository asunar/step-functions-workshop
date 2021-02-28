+++
title = "Introducing the Task State type"
chapter = false
weight = 30
+++

The Task State causes the interpreter to execute work identified by the state’s Resource field. Below, we’ll use Task states to invoke our Data Checking service Lambda function, passing appropriate name and address attributes from our application in to the Lambda function invocations.

{{% notice info %}}
In the Task states below, in addition to specifying the ARN of the Lambda function we want the Task state to use, we also include a Parameters section. This is how we pass data from the state’s input into the target Lambda function’s input. You’ll notice that we’re using a syntax that includes dollar signs at the end of each property name and the beginning of each property value. This is so that Step Functions knows we want to inject state data into these parameters instead of actually sending a raw string of `$.application.name` for example.  
{{% /notice %}}

The state machine description we use below assumes that the state machine will receive an initial input object with a single property called `application` that has `name` and `address` properties on it. We’ll specify this input for execution in a moment after we update our state machine definition.

### Make these changes

➡️ Step 1. Back in the Step Functions web console, click ‘Edit state machine’

➡️ Step 2. Next, we’re going to update our state machine definition. Note that after you paste the content below,  you will see a few lines with error indicators because our new state machine definition has some placeholder strings called ‘REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN’.  We’ll fix this in the next step. Replace our existing definition with the following updated state machine definition:

```
{
    "StartAt": "Check Name",
    "States": {
        "Check Name": {
            "Type": "Task",
            "Parameters": {
                "command": "CHECK_NAME",
                "data": {
                    "name.$": "$.application.name"
                }
            },
            "Resource": "REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN",
            "Next": "Check Address"
        },
        "Check Address": {
            "Type": "Task",
            "Parameters": {
                "command": "CHECK_ADDRESS",
                "data": {
                    "address.$": "$.application.address"
                }
            },
            "Resource": "REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN",
            "Next": "Approve Application"
        },
        "Approve Application": {
            "Type": "Pass",
            "End": true
        }
    }
}
```

➡️ Step 3. In the state machine definition, delete Resource property value between the quotes and type
```arn:aws:lambda```
You should see a list of lambda arns.
![Data Checking Arn](/images/data-checking-arn.png)

Pick data checking arn from the list and hit enter.
   
➡️ Step 4. Repeat for the other REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN Resource value.

➡️ Step 5. Click ‘Save’

➡️ Step 6. Notice how we receive a warning that our IAM role may need to change in order to allow our updated state machine to execute. This is a helpful reminder. In fact, we *have* changed our state machine in way that will require permissions changes. Now, we require the ability to invoke our Data Checking Lambda function in order to execute this state machine. We’ll address this next. Click ‘Save anyway’ to continue.

### Try it out

The warning we saw just now when we updated our state machine definition was correct. We *will* need to update our IAM role permissions in order for this to work. But let’s try another execution anyway just to see what an insufficient permission failure looks like.

➡️ Step 1. Click ‘Start execution’

➡️ Step 2. Paste the following JSON into the input field:

```
{
    "application": { 
        "name": "Spock", 
        "address": "123 Enterprise Street" 
    }
}
```

➡️ Step 3. Click ‘Start execution’. 
    
After a moment, you should see the results of this failed execution. The ‘Execution Status’ label shows ‘Failed’ underneath it, and you’ll see a big red background in the visualization section, highlighting the state that experienced a failure. 

➡️ Step 4. Click the failed state, then expand the Exception area on the right-hand side to see more details about the failure. You should see something like the screenshot below.

![Check name failure](/images/simplified-workflow-vis-name-fail.png)

This failure isn’t surprising. When this state machine executes, it assumes an IAM role in order to determine which sorts of actions it’s allowed to take inside the AWS cloud. And, of course, we haven’t yet added any explicit permissions to allow this role to invoke our Data Checking Lambda so, to keep things secure, we get a failure when this state machine tries to run.

Let’s fix this by adding the appropriate permissions to the role that our Step Function assumes during execution. 