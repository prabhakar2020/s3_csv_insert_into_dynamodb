# Insert S3 csv file content into DynamoDB

The following code snippet we can use it inside AWS lambda for fetching csv file content on S3 and store those data/values into DynamoDB
```
import json
import boto3

s3_cient = boto3.client('s3')
dynamo_db = boto3.resource('dynamodb')
table = dynamo_db.Table('example_table')  # DynamoDB table name

def lambda_handler(event, context):
    # TODO implement
    bucket_name = event["Records"][0]["s3"]["bucket"]["name"]
    s3_file_name = event["Records"][0]["s3"]["object"]["key"]
    resp = s3_cient.get_object(Bucket=bucket_name, Key=s3_file_name)
    
    data = resp['Body'].read().decode('utf-8')
    print (data) # This print statement will be displayed on Lambda console as well as on CloudWatch. You can also remove this if not required
    employees = data.split("\n")
    for emp in employees:
        emp = emp.split(",")
        # Exception handling done for skipping errors. When we are dealing with CSV file content, there may be a chance to get last row as empty row. So I have used try-catch for avoiding the runtime issues.
        try:                
            table.put_item(
                Item = {
                    "id": str(emp[0]),
                    "Name": emp[1],
                    "Location": emp[2]
                })
        except Exception as err:
            print (">>>>>>>>"+str(err))
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```
