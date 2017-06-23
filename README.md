[![awshosting.jpg](https://s23.postimg.org/r8tp1no5n/awshosting.jpg)](https://postimg.org/image/l7w04l1jb/)

Part 1 of this tutorial is meant to be used as a guide to take an already developed MVC 5 application with a LocalDb MSSQL server and present the steps of re-hosting in on the AWS cloud using Elastic Beanstalk and an RDS MSSQL Express server. Part 2 goes into automating your service run time using AWS Lambda. Most of this material is aggregated from other online resources which I have credited below. You will need Visual Studio 2017, Microsoft Server Management Studio, and Windows 10 for this tutorial to work. You can either follow along with the project provided, or use your own. Enjoy!

## Libraries and Resources Used 

- [Sample Project](https://github.com/benmpeterson/NET-TO-AWS) - Clone this repo to start with the tutorial MVC project
- [Matt Perderc](https://www.codeproject.com/Articles/889059/Amazon-Web-Services-part-How-to-deploy-a-load-bala#sourcebundle) - Great Tutorial that expands on AWS. Part 1 and 2 are excellent.
- [AWS - EC2 Automation](https://aws.amazon.com/premiumsupport/knowledge-center/start-stop-lambda-cloudwatch/) - Article used for Part 2 of the tutorial.


## Creating an AWS account

1. Before beginning to alter your MVC project you have to first sign up for an aws account [Here](https://www.amazon.com/ap/signin?openid.assoc_handle=aws&openid.return_to=https%3A%2F%2Fsignin.aws.amazon.com%2Foauth%3Fresponse_type%3Dcode%26client_id%3Darn%253Aaws%253Aiam%253A%253A015428540659%253Auser%252Fawssignupportal%26redirect_uri%3Dhttps%253A%252F%252Fportal.aws.amazon.com%252Fbilling%252Fsignup%253Fredirect_url%253Dhttps%25253A%25252F%25252Faws.amazon.com%25252Fregistration-confirmation%2526state%253DhashArgs%252523%2526isauthcode%253Dtrue%26noAuthCookie%3Dtrue&openid.mode=checkid_setup&openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&openid.identity=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select&openid.claimed_id=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select&action=&disableCorpSignUp=&clientContext=&marketPlaceId=&poolName=&authCookies=&pageId=aws.ssop&siteState=registered%2Cen_US&accountStatusPolicy=P1&sso=&openid.pape.preferred_auth_policies=MultifactorPhysical&openid.pape.max_auth_age=120&openid.ns.pape=http%3A%2F%2Fspecs.openid.net%2Fextensions%2Fpape%2F1.0&server=%2Fap%2Fsignin%3Fie%3DUTF8&accountPoolAlias=&forceMobileApp=0&language=en_US&forceMobileLayout=0). This signup does require you to input a CC, but my goal here is to only use the supplied free tier services for this tutorial. 

2. Next you need to generate a Key Pair which will be used during deployment. Select services on the top left then under Compute select EC2. You will then see a new horizontal dashboard where you need to select Key Pairs. Once selected click Create Key Pair. Name this key the same as your MVC application for clarity.  You should now see the Key Pair and associated Fingerprint name it will also automatically download the key,you can disregard it for the time being.

## Creating an RDS Server

1. Select services again and under Database select RDS, then select Get Started Now. Select the last Microsoft SQL Server option then SQL Server Express. Tick the box that says only show options that are eligible for RDS Free Tier and leave the Instance Specs as they are. Fill in the settings as appropriate for your project. The Username and Password will be used to log into the database. Select through the next instance, leave the default options and finally select Launch DB instance. Select View Instance and wait for the status to complete. Can take a view minutes. 

2. Now that AWS is telling us the server has been created, lets make sure. Copy the endpoint URL and Open Microsoft SQL Server Management Studio. Paste the endpoint URL and delete the :1433 at the end of it. Select SQL Server Authentication, and login with the same credentials you created when making the Db instance. If it connected successfully, the database was successfully created.

## Changing the connection string

1. In the example project provided or your own MVC project, navigate to the Web.Config file and find the connection string

```cs
  <connectionStrings>
    <add name="DefaultConnection" connectionString="Data Source=(LocalDb)\MSSQLLocalDB;Initial Catalog=exampledb;Integrated Security=True"
      providerName="System.Data.SqlClient" />
  </connectionStrings>
```

2. We now want to change these settings to instead use the new Database we just created. If you would like to keep the local for production you can just comment it out and make sure the connection string is looking at the amazon Db before deploying. I had to fight this connection string quite a bit, and I'm not sure if this is the most efficient string, but it works! Yours will look different then mine, make sure your Data Source is your Db endpoint, and the User ID and Password are the same you set when creating the Db. Make sure the end of you Data Source url is .com as before, if there is a port number at the end of it, delete it. Your new connection string should look like this, but with your information

```cs

  <connectionStrings>    
    <add name="DefaultConnection" connectionString="Data Source=tutorial.cdajhybxx6x0.us-east-1.rds.amazonaws.com;Initial Catalog=ANYDATABASENAMEHERE;Integrated Security=False;User ID=YOURIDHERE;Password=YOURPASSWORDHERE;Connect Timeout=15;Encrypt=False;TrustServerCertificate=True;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" providerName="System.Data.SqlClient" />
  </connectionStrings>

```

3. Save the new connection string and run the project in Google Chrome. Try to create a new user, if it works you have successfully linked the project to the Database. Hopefully the connection string didn't give you as much trouble as it gave me!


## Building and Hosting the Project

1. Great! Now that the Database is connected we need to save the project and create a build of it that AWS can utilize. To do this open the Developer Command Prompt for VS 2017 and cd to the folder where your csproj file is located. In the tutorial project, it is located in the MVC5App folder. Once there type the following replacing the template csproj file name with your own.


        msbuild MVC5App.csproj /t:Package /p:Configuration=Release /p:PackageLocation=.
        /p:AutoParameterizationWebConfigConnectionStrings=False


2. Once completed it will created a zipped build that we will now use to deploy.

## Hosting our app using Elastic BeanStalk

1. Using Elastic BeanStalk is a great all in one tool that creates virtual machines for your application and hosts it all through one wizard. To create an Elastic Beanstalk application select Services, Compute, Elastic Beanstalk. Then on the top right select Create New Application. 

2. Select create new Environment, then select Web server environment. For platform select .NET and for application code select upload your code and direct it to the zip build we created in your root project folder. Now select configure more options. Most of these options can stay as defaults but to keep this completely free select modify on the instances box and change the instance type to t2.micro and the size to 30GB, and in the scaling box change from load balanced to signle instance. Finally in the security box add in the key pair you created earlier. Now create the environment. This will take you to a progress window and the deployment takes a few minutes. 

3. Now that it is deployed click the URL link to see your webpage! you can keep it hosted or terminate the environment. Make sure you look into how the service is billed [Here](https://aws.amazon.com/free/)


## Part 2 - Automating activity hours

Amazon Web Services have extensive tooling when it comes to automating specific functions of your web site including scripting checks to make sure the EC2 instance is running correctly, or changing what times of the day the EC2 instance is operational. For this example we will go through the steps to keep a single instance web page functional from 9 am - 5 pm daily, while turning off the service during off hours. 

1. When using the Elastic Beanstalk wizard to deploy your site it automatically configures auto scaling, which in most cases is beneficial. However, for our purposes we need to turn that feature off so AWS does not try to automatically create a new RC2 instance when we stop the running one. From the Services tab, select EC2. Then from the side menu scroll down and select Auto Scaling Groups. You should see a group that AWS has created for you. 

2. First select Edit on the Details tab and change the Min number of instances running to 0, leaving the max at 1. Next Select the Instances tab and highlight your instance. Once highlighted, select Actions and choose Detach. This will detach the RC2 instance that is hosting your Web Page from the auto scale group, eliminating the possibilities of AWS automatically scaling or creating a new instance for you. Again, this exercise is not something you would do on a official production product, but works great for static resume type webpages. A detach instance warning will pop up, do not check any of the boxes, and finally select detach instance.    

## Using AWS Lambda to automate server run times

The AWS article on this is presented very well. I will copy that over to this document and clarify things as needed. 


You can configure a Lambda function to start and stop instances when triggered by this CloudWatch event.

For this example, you’ll create Lambda functions to start and stop EC2 instances and then create CloudWatch Events that trigger your instances to start in the morning and stop at night.

1.    Open the AWS Lambda console and select Create a Lambda function (First time Lambda users may need to choose “Get Started Now,” which will direct you to the “function create” screen). When prompted to select a blueprint, choose Blank Function.

2.    Choose Configure triggers if it is not already selected, and then choose Next. You will configure a Lambda trigger later.

3.    Enter the following information to configure your Lambda function:

        - For Name, enter "StopEC2Instances" or another name - that’s meaningful for you.
        - For Description, add a meaningful description; for - example, “stops EC2 instances every day at night”.
        - For Runtime, select Python 2.7.

4.    To stop your instances, enter the following sample code

            import boto3
            # Enter the region your instances are in, e.g. 'us-east-1'
            region = 'XX-XXXXX-X'
            # Enter your instances here: ex. ['X-XXXXXXXX', 'X-XXXXXXXX']
            instances = ['X-XXXXXXXX']
            def lambda_handler(event, context):
            ec2 = boto3.client('ec2', region_name=region)
            ec2.stop_instances(InstanceIds=instances)
            print 'stopped your instances: ' + str(instances)

        -To retrieve your instance id and region select Services, EC2, then select Running Instances. Under the          Description tab you will see the Instance ID and the availability zone, which is your region. One thing to note here is to delete the last letter of the zone if you have one. Mine displays as us-east-1b, but I had to truncate that to us-east-1 for the script to run. 

5.    Expand the Role drop-down menu and choose Create a custom role. This should open a new tab or window in your browser.

6.    Enter the following information to create a role for Lambda to use:
        - Under IAM Role, choose Create a new IAM Role.
        - For Role Name, enter “lambda_start_stop_ec2” or  another name that’s meaningful for you.

7.    Choose View Policy Document, Edit, and then edit the policy as follows:

    {
    "Version": "2012-10-17",
    "Statement": [
        {
        "Effect": "Allow",
        "Action": [
            "logs:CreateLogGroup",
            "logs:CreateLogStream",
            "logs:PutLogEvents"
        ],
        "Resource": "arn:aws:logs:*:*:*"
        },
        {
        "Effect": "Allow",
        "Action": [
            "ec2:Start*",
            "ec2:Stop*"
        ],
        "Resource": "*"
        }
        ]
    }

8.    Choose Allow.

9.    From Advanced settings, input 10 seconds for the function timeout.
Note: Environment variables, dead letter queues, and VPC are not necessary for this example; however, if you wish to use these features, you will need to add additional permissions. See the AWS Lambda documentation for more details.

10. Choose Next to review your function configuration, and then choose Create function.

11. Repeat steps 1-4 and 9 to create another function that will start your instances again, using code similar to the following:

            import boto3

            # Enter the region your instances are in, e.g. 'us-east-1'
            region = 'XX-XXXXX-X'
            # Enter your instances here: ex. ['X-XXXXXXXX', 'X-XXXXXXXX']
            instances = ['X-XXXXXXXX']        
            def lambda_handler(event, context):
                ec2 = boto3.client('ec2', region_name=region)
                ec2.start_instances(InstanceIds=instances)
                print 'started your instances: ' + str(instances)


## Testing Your Functions

To test your newly created functions:

1.    From the Lambda console, choose Functions, select your function, and then choose Test.

2.    Your function doesn’t use the test event, so from the Input test event editor just choose Save and test.

Create a CloudWatch event that will trigger your Lambda function at night:

1.    Open the CloudWatch console.

2.    Choose Events, and then choose Create rule.

3.    Select Schedule under Event Selector.

4.    Enter an interval of time or cron expression that will tell Lambda when to stop your instances; for more information on the correct syntax, see Schedule Expression Syntax for Rules.
Note: Cron expressions are evaluated in GMT. Make sure to adjust for your preferred time zone. For my example I created two crons expressions that turned off the service at 6 pm and on at 9 am. Formatting is very important with these.

        - 9am start = cron(0 13 * * ? *)
        - 6pm stop = cron(0 22 * * ? *)

5.    Choose Add target.

6.    Under Targets, choose Lambda function.

7.    For Function, choose the Lambda function that stops your instances.

8.    Choose Configure details.

9.    Enter the following information in the provided fields:

        - For Name, enter "StopEC2Instances", or another name -that’s meaningful for you.
        - For Description, add a meaningful description; for - example, “stops EC2 instances every day at night”.
        - For State, check Enabled.

10.    Choose Create rule. To restart your instances in the morning, repeat these steps using your preferred time.


## Conclusion


I hope you found this helpful, you can email me with any questions at ben.micah.peterson@gmail.com
