[![awshosting.jpg](https://s23.postimg.org/r8tp1no5n/awshosting.jpg)](https://postimg.org/image/l7w04l1jb/)

This tutorial is meant to be used as a guide to take an already developed MVC 5 application with a LocalDb MSSQL server and present the steps of re-hosting in on the AWS cloud using Elastic Beanstalk and an RSD MSSQL Express server. Most of this material is aggregated from other online resources which I have credited below. You will need Visual Studio 2017, Microsoft Server Management Studio, and Windows 10 for this tutorial to work. You can either follow along with the project provided, or use your own. Enjoy!

## Libraries and Resources Used 

- [Sample Project](https://github.com/benmpeterson/NET-TO-AWS) - Clone this repo to start with the tutorial MVC project
- [MATT Perderc](https://www.codeproject.com/Articles/889059/Amazon-Web-Services-part-How-to-deploy-a-load-bala#sourcebundle) - Great Tutorial that expands on the AWS Part 1 and 2 are excellent.
- [AWS - EC2 Automation](https://aws.amazon.com/premiumsupport/knowledge-center/start-stop-lambda-cloudwatch/) - This will be gone over in part two, but this article outlines how to turn off your EC2 service during the night. 


## Creating an AWS account

1. Before beginning to alter your MVC project you have to first sign up for an aws account [Here](https://www.amazon.com/ap/signin?openid.assoc_handle=aws&openid.return_to=https%3A%2F%2Fsignin.aws.amazon.com%2Foauth%3Fresponse_type%3Dcode%26client_id%3Darn%253Aaws%253Aiam%253A%253A015428540659%253Auser%252Fawssignupportal%26redirect_uri%3Dhttps%253A%252F%252Fportal.aws.amazon.com%252Fbilling%252Fsignup%253Fredirect_url%253Dhttps%25253A%25252F%25252Faws.amazon.com%25252Fregistration-confirmation%2526state%253DhashArgs%252523%2526isauthcode%253Dtrue%26noAuthCookie%3Dtrue&openid.mode=checkid_setup&openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&openid.identity=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select&openid.claimed_id=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select&action=&disableCorpSignUp=&clientContext=&marketPlaceId=&poolName=&authCookies=&pageId=aws.ssop&siteState=registered%2Cen_US&accountStatusPolicy=P1&sso=&openid.pape.preferred_auth_policies=MultifactorPhysical&openid.pape.max_auth_age=120&openid.ns.pape=http%3A%2F%2Fspecs.openid.net%2Fextensions%2Fpape%2F1.0&server=%2Fap%2Fsignin%3Fie%3DUTF8&accountPoolAlias=&forceMobileApp=0&language=en_US&forceMobileLayout=0). This signup does require you to input a CC, but my goal here is to only use the supplied free tier services for this tutorial. 

2. Next you need to generate a Key Pair which will be used during deployment. Select services on the top right then under Compute select EC2. You will then see a new horizontal dashboard where you need to select Key Pairs. Once selected click Create Key Pair. Name this key the same as your MVC application for clarity.  You should now see the Key Pair and associated Fingerprint name it will also automatically download the key,you can disregard it for the time being.

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

```cs
        msbuild MVC5App.csproj /t:Package /p:Configuration=Release /p:PackageLocation=.
         /p:AutoParameterizationWebConfigConnectionStrings=False
```

2. Once completed it will created a zipped build that we will now use to deploy.

## Hosting our app using Elastic BeanStalk

1. Using Elastic BeanStalk is a great all in one tool that creates virtual machines for your application and hosts it all through one wizard. To create an Elastic Beanstalk application select Services, Compute, Elastic Beanstalk. Then on the top right select Create New Application. 

2. Select create new Environment, then select Web server environment. For platform select .NET and for application code select upload your code and direct it to the zip build we created in your root project folder. Now select configure more options. Most of these options can stay as defaults but to keep this completely free select modify on the instances box and change the instance type to t2.micro and the size to 30GB. Also modify the security box adding in the key pair you created earlier. Finally, select create environment. This will then take you to a progress window and the deployment takes a few minutes. 

3. Now that it is deployed click the link to see your webpage! you can keep it hosted or terminate the environment. Make sure you look into how the service is billed so you don't get hit with unexpected charges. [Here](https://aws.amazon.com/free/)

## Conclusion

That's it for Part 1. Part 2 will show you how to implement lambda scripts to keep your site live for only certain times of the day.

I hope you found this helpful, you can email me with any questions at ben.micah.peterson@gmail.com
