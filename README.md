### Step 1: Set Up DynamoDB for Data Storage

	1.	Create a DynamoDB Table:
	•	Go to the DynamoDB Console.
	•	Click “Create table.”
	•	Enter a table name, e.g., SurveyResponses.
	•	Set the primary key as ResponseID with type String.
	•	Configure additional settings as needed and click “Create.”
	2.	Define Attributes:
	•	The SurveyResponses table can have attributes like ResponseID, SurveyID, UserID, Answers (a map or list), and Timestamp.

### Step 2: Set Up Lambda for Backend Logic

	1.	Create a Lambda Function:
	•	Go to the AWS Lambda Console.
	•	Click “Create function.”
	•	Choose “Author from scratch.”
	•	Enter a function name, e.g., SaveSurveyResponse.
	•	Choose a runtime, e.g., Node.js, Python, or any other supported language.
	•	Choose an execution role with necessary permissions (or create one).
	2.	Write the Function Code:
	•	In the Lambda editor, write code that will receive survey responses via API, validate the input, and save it to DynamoDB.
	•	Here’s an example in Node.js:

const AWS = require('aws-sdk');
const dynamoDb = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
    const requestBody = JSON.parse(event.body);
    const { SurveyID, UserID, Answers } = requestBody;

    const params = {
        TableName: 'SurveyResponses',
        Item: {
            ResponseID: ${SurveyID}-${UserID}-${Date.now()},
            SurveyID: SurveyID,
            UserID: UserID,
            Answers: Answers,
            Timestamp: new Date().toISOString()
        }
    };

    try {
        await dynamoDb.put(params).promise();
        return {
            statusCode: 200,
            body: JSON.stringify({ message: 'Survey response saved successfully!' }),
        };
    } catch (error) {
        return {
            statusCode: 500,
            body: JSON.stringify({ message: 'Failed to save survey response.', error }),
        };
    }
};


### 3.	Deploy the Lambda Function:
	•	Once your code is ready, deploy the Lambda function.

Step 3: Set Up API Gateway to Expose Lambda as REST API

	1.	Create an API:
	•	Go to the API Gateway Console.
	•	Click “Create API.”
	•	Choose “REST API” and click “Build.”
	•	Name your API, e.g., SurveyAPI, and click “Create API.”
	2.	Create a Resource and Method:
	•	Click “Actions” and then “Create Resource.”
	•	Name the resource, e.g., surveys.
	•	Under this resource, create a POST method.
	•	Set the integration type as “Lambda Function” and select the function you created earlier (SaveSurveyResponse).
	•	Click “Save.”
	3.	Deploy the API:
	•	Click “Actions” and choose “Deploy API.”
	•	Create a new stage, e.g., prod, and click “Deploy.”
	•	Note the endpoint URL provided; this is the URL where your API is accessible.

### Step 4: Frontend for Collecting Survey Responses

	1.	Create a Simple HTML Form:
	
	•	Replace your-api-id.execute-api.region.amazonaws.com/prod/surveys with your actual API Gateway endpoint.

	2.	Host the Form:
	•	You can host this HTML form on S3 as a static website following the steps provided in the earlier guide.

### Step 5: Security and Permissions

	1.	CORS Configuration:
	•	In API Gateway, enable CORS on the POST method for your resource to allow cross-origin requests from your hosted survey form.
	2.	IAM Role and Policies:
	•	Ensure the Lambda function has the correct IAM role with policies that allow it to interact with DynamoDB.

### Step 6: Testing

	•	Submit a Test Response:
	•	Open your survey form in a browser, fill it out, and submit it.
	•	Check DynamoDB to see if the response was saved successfully.

