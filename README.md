This tutorial is meant to be used as a guide to take an already developed MVC 5 application with a LocalDb MySql server and present the steps of re-hosting in on the AWS cloud using ElasticBeanstalk and an RSD MSSQL server. Most of this material is aggregated from other online resources which I have credited below. You will need Visual Studio 2017, Microsoft Server Manager, and Windows 10 for this tutorial to work. You can either follow along with the project provided, or use your own. Enjoy!

## Libraries and Resources Used 

- [Chart.Mvc](https://github.com/martinobordin/Chart.Mvc) - A .NET wrapper to generate charts using the popular Chart.Js V1 library (http://www.chartjs.org).
- [SYEDSHAUNU](https://code.msdn.microsoft.com/ASPNET-MVC-5-Security-And-44cbdb97) - Great Tutorial on setting up an admin user role written by a Microsoft MVP.
- [Scott Allen - LINQ Fundamentals](https://app.pluralsight.com/library/courses/linq-fundamentals-csharp-6/table-of-contents) - Great course on querying a database using LINQ
- [Calorie Equation](https://www.hss.edu/conditions_burning-calories-with-exercise-calculating-estimated-energy-expenditure.asp) - This was the study I used to calculate the calories burned with each workout


## How to use this Application on the web.

1. Launch the [App](http://exercize-env.us-east-1.elasticbeanstalk.com/).
 
2. Click Login. 

3. The application runs differently depending on whether you log in as an admin or user. Login credentials for both are on the right side of login page

4. If logged in as an admin select View Customer Data, if logged in as a user try to create a Workout or view Progress!


## How to run locally

1. Clone the repo 

2. Open the project and set up Home/Index as the start up file in not already

3. Start running the application. 

4. Click Login

5. The application runs differently depending on whether you log in as an admin or user. Login credentials for both are on the right side of login page

6. In order for the administrator to have customer data to view, a customer and their workouts need to be created. 

## Index

* [Setting up an Admin Role](#Admin)
* [Creating a User Workout](#workouts)
* [Creating Charts](#charts)

---

## Creating an AWS account

[demo](admin-role) 

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

cs```

  <connectionStrings>    
    <add name="DefaultConnection" connectionString="Data Source=tutorial.cdajhybxx6x0.us-east-1.rds.amazonaws.com;Initial Catalog=ANYDATABASENAMEHERE;Integrated Security=False;User ID=YOURIDHERE;Password=YOURPASSWORDHERE;Connect Timeout=15;Encrypt=False;TrustServerCertificate=True;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" providerName="System.Data.SqlClient" />
  </connectionStrings>

```

3. Save the new connection string and run the project in Google Chrome. Try to create a new user, if it works you have successfully linked the project to the Database. And hopefully the connection string didn't give you as much trouble as it gave me!





3. Once the ViewBags have been set, implement them in the Home/Index View as shown

```cs
        <div class="banner">
            <div class="bg-color">
                <div class="container">
                    <div class="row">
                        <div class="banner-text text-center col-sm-12">
                            <div class="text-border">
                                <h2 class="text-dec">Exercize</h2>
                            </div>
                            <div class="intro-para text-center quote">
                                
                                    @if (ViewBag.displayMenu == "Yes")
                                {
                                <p>Welcome Admin.</p>
                                
                                    <li>@Html.ActionLink("View Customer Data", "CustomerData", "Admin", new { }, new { @class = "btn" })</li>
                                
                                }
                                    else if (ViewBag.displayMenu == "No")
                                    {
                                        <p>Welcome User</p>
                                        <p>
                                            <li>@Html.ActionLink("Go To Workouts", "Index", "Exercise", new { }, new { @class = "btn" })</li>
                                        </p>
                                    }
                                    else
                                    {
                                        <li>@Html.ActionLink("Login", "Login", "Account", new { }, new { @class = "btn" })</li>
                                                                
                                    } </p>                        
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
```
Now depending on the login information your home view and action options are modified, great!

## Workouts 
[demo](#user-workout)

The next problem of this project was having the ability as a user to create a workout which calculates an estimatation of burned calories. 
I found the calculation for this [here](https://www.hss.edu/conditions_burning-calories-with-exercise-calculating-estimated-energy-expenditure.asp)

**Energy expenditure (calories/minute) = .0175 x Activity (from table) x weight (in kilograms)**

1.To cacluate this for each user we must have them enter in their weight during registration. To do this I added the following to the RegisterViewModel

```cs
        public class RegisterViewModel
        {
            [Required]
            [EmailAddress]
            [Display(Name = "Email")]
            public string Email { get; set; }

            [Required]
            [StringLength(100, ErrorMessage = "The {0} must be at least {2} characters long.", MinimumLength = 6)]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }

            [DataType(DataType.Password)]
            [Display(Name = "Confirm password")]
            [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
            public string ConfirmPassword { get; set; }

            
            public string Sex { get; set; }

           
            public double Age { get; set; } 

            [Required]
            public double Weight { get; set; }
        }
```

2. And this to the Register View

```cs
        @using (Html.BeginForm("Register", "Account", FormMethod.Post, new { @class = "form-horizontal", role = "form" }))
        {
            @Html.AntiForgeryToken()
            <h4>Create a new account.</h4>
            <hr />
            @Html.ValidationSummary("", new { @class = "text-danger" })
            <div class="form-group">
                @Html.LabelFor(m => m.Email, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.TextBoxFor(m => m.Email, new { @class = "form-control" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(m => m.Password, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.PasswordFor(m => m.Password, new { @class = "form-control" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(m => m.ConfirmPassword, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.PasswordFor(m => m.ConfirmPassword, new { @class = "form-control" })
                </div>
            </div>

            <div class="form-group">
                @Html.LabelFor(m => m.Age, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.TextBoxFor(m => m.Age, new { @class = "form-control" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(m => m.Sex, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.TextBoxFor(m => m.Sex, new { @class = "form-control" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(m => m.Weight, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.TextBoxFor(m => m.Weight, new { @class = "form-control" })
                </div>
            </div>
```

3. This ensures that each user will have their assoicated weight in the database. The next thing we must do is retreive that weight each time we are creating a new workout. This is done in the Exercise Controller by creating a new UserManager instance that contains all the database row infomation of the currently logged in user. 

```cs

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create(ExerciseCreate model)
        {
            if (!ModelState.IsValid)
            {
                return View(model);
            }

            var userId = Guid.Parse(User.Identity.GetUserId());

            var manager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(new ApplicationDbContext()));
            var currentUser = manager.FindById(User.Identity.GetUserId());        
            var userWeight = currentUser.Weight;

            var service = new ExerciseService(userId, userWeight);
            service.CreateExercise(model);

            return RedirectToAction("Index");            
        }
```
4. Once the weight is retrieved we pass it into our ExerciseService to create the workout using the constructor that takes the userId and userWeightInPounds.

```cs

        public class ExerciseService
            {
                private readonly Guid _userId;
                private readonly Double _userWeightInPounds;

                public ExerciseService(Guid userId, Double userWeightInPounds)
                {
                    _userId = userId;
                    _userWeightInPounds = userWeightInPounds;
                }

                public ExerciseService(Guid userId)
                {
                    _userId = userId;
                 
                }

                public bool CreateExercise(ExerciseCreate model)
                {
                    var entity =
                        new Workout()
                        {
                            OwnerId = _userId,
                            OwnerWeight = _userWeightInPounds,
                            Type = model.Type,
                            Intensity = model.Intensity,
                            Duration = model.Duration,
                            CaloriesBurned = 0,
                            CreatedUtc = DateTimeOffset.UtcNow,
                        };

                    
                    CalorieCalculator.GetCalories(entity);
           
                    using (var ctx = new ApplicationDbContext())
                    {
                        ctx.Workouts.Add(entity);                
                        return ctx.SaveChanges() == 1;
                    }

                   
                }

``` 
5. Notice here the helper method CalorieCalculator.GetCalorites(entity).

```cs

        namespace Exercise.Services.HelperMethods

        {
            public class CalorieCalculator
            {
                public static double GetCalories(Workout entity)
                {

                    if (entity.Type == "Bicycling" && entity.Intensity == "Low")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 6.0 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Bicycling" && entity.Intensity == "High")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 10 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Dancing" && entity.Intensity == "Low")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 5.0 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Dancing" && entity.Intensity == "High")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 7.0 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Running" && entity.Intensity == "Low")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 10.0 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Running" && entity.Intensity == "High")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 13.0 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Swimming" && entity.Intensity == "Low")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 6 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Swimming" && entity.Intensity == "High")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 10.0 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Walking" && entity.Intensity == "Low")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 3.5 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else if (entity.Type == "Walking" && entity.Intensity == "High")
                    {
                        entity.CaloriesBurned = entity.Duration * .0175 * 5.0 * (entity.OwnerWeight / 2.2);
                        entity.CaloriesBurned = Math.Round(entity.CaloriesBurned, 2);
                        return entity.CaloriesBurned;
                    }
                    else
                    {
                        return entity.CaloriesBurned;
                    }
                }

            }
        }

```
This is taking the available types of activies, along with intensity and user wight to calculate how many calories are burned. Notice it is dividing the OwnerWeight by 2.2. This is convert it to kilograms which is the variable used in the study's formula. 

6. Thats it! Now for each new workout created by any user, a calorie burned calculation is taking place, stored to the database and displayed in their list of work outs. 
            
## Charts
[demo](#charts)

The last challenge of this application was to visually represent data for both the user and admin roles. This was done by implementing Chart.MVC, a great, if a bit outdated, extenstion that makes working with chart.js a bit more C# friendly. Using it I was able to create bar, line and radial graphs that represented different database query results. 

1. The first set of these queries was to show each user a line graph of their workouts with how many calories were burned of each. Notice these are LINQ queries. Here is what I added to my Exercise Controller. 

```cs

        public ActionResult Progress()
                {
                    var userId = Guid.Parse(User.Identity.GetUserId());
                    var service = new ExerciseService(userId);            

                    var manager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(new ApplicationDbContext()));
                    var currentUser = manager.FindById(User.Identity.GetUserId());

                    using (var context = new ApplicationDbContext())
                    {

                        var query = from b in context.Workouts
                                    where b.OwnerId == userId
                                    select b.CaloriesBurned;
                        List<double> plist = query.ToList();
                        ViewBag.caloriesBurned = plist;

                        var query2 = from b in context.Workouts
                                     where b.OwnerId == userId
                                     select b.Type;
                        List<string> elist = query2.ToList();
                        ViewBag.type = elist;
                    }


                    return View();
                }
```

2. With the type of workouts and how many calories were burned in each stored in their respective ViewBags, I next tied that to the Progress View as shown here. Notice the using statements as this is how Chart.Mvc is implemented

```cs
        @model Exercise.Models.ExerciseDetail

        @using Chart.Mvc.ComplexChart;
        @using Chart.Mvc.Extensions
        @Scripts.Render("~/bundles/chart.js")

        <h2>Progress</h2>

        @{
            var barChart = new LineChart();
            barChart.ComplexData.Labels.AddRange(ViewBag.type);
            barChart.ComplexData.Datasets.AddRange(new List<ComplexDataset>
                                     {
                                        
                                        new ComplexDataset
                                            {
                                                Data = ViewBag.caloriesBurned,
                                                Label = "My Second dataset",
                                                FillColor = "rgba(228, 247, 187, 1)",
                                                StrokeColor = "rgba(151,187,205,1)",
                                                PointColor = "rgba(151,187,205,1)",
                                                PointStrokeColor = "#fff",
                                                PointHighlightFill = "#fff",
                                                PointHighlightStroke = "rgba(151,187,205,1)",
                                            }
                                    });
        }

        <canvas id="myCanvas" width="940" height="400"></canvas>
        @Html.CreateChart("myCanvas", barChart)
```

3. You should now see a chart with these values. Here is what a sample one of mine looks like. 
[![UserChart.jpg](https://s17.postimg.org/l9ouz19e7/User_Chart.jpg)](https://postimg.org/image/l9ouz19e3/)

4. The next query was used to show the admin how many times each category of activity was created for all users as a radar chart. Here is what I added to my Admin Controller. Notice that this is a SQL query. 

```cs

        public ActionResult CustomerData()
                {
                    using (var context = new ApplicationDbContext())
                    {                
                                               
                        var ListTypeOfWorkouts = context.Database.SqlQuery<string>("SELECT Type FROM dbo.Workout").ToList();
                        
                        double walkCount = 0;
                        double runCount = 0;
                        double bikeCount = 0;
                        double danceCount = 0;
                        double swimCount = 0;
                        double maleCount = 0;
                        double femaleCount = 0;

```
5. Once the query is complete it stores all the results in a ListTypeOfWorkouts variable. I then created a for each loop to parse through the list and store each category of activity in its own variable. And finally set each category variable to a viewbag.

```cs

        foreach (var item in ListTypeOfWorkouts)
        {
            if (item == "Walking")
            {
                walkCount++;
            }
            else if (item == "Running")
            {
                runCount++;
            }
            else if (item == "Bicycling")
            {
                bikeCount++;
            }
            else if (item == "Dancing")
            {
                danceCount++;
            }
            else if (item == "Swimming")
            {
                swimCount++;
            }
        }

        ViewBag.WalkStat = walkCount;
        ViewBag.RunStat = runCount;
        ViewBag.BikeStat = bikeCount;
        ViewBag.DanceStat = danceCount;
        ViewBag.SwimStat = swimCount;
        ViewBag.MaleCount = maleCount;
        ViewBag.FemaleCount = femaleCount;
        ViewBag.TotalCalories = TotalCaloriesSum.ToString();
    }            
    return View();
```
6. With this information in the ViewBags we can then implement them into the view and construct the radar graph. Injecting the viewbags into the data list field.

```cs

        @model Exercise.Models.ExerciseDetail


        @using Chart.Mvc.ComplexChart
        @using Chart.Mvc.Extensions
        @using Chart.Mvc.SimpleChart


        @Scripts.Render("~/bundles/Chart")
        @Scripts.Render("~/bundles/jquery")
        <link href="/Content/Site.css" rel="stylesheet" type="text/css" />

        
            @{
                const string Canvas = "RadarChart";
                var complexChart = new RadarChart();
                complexChart.ComplexData.Labels.AddRange(new[] { "Walking", "Running", "Bicycling", "Dancing", "Swimming" });
                complexChart.ComplexData.Datasets.AddRange(new List<ComplexDataset>
            {


                  new ComplexDataset
                  {
                      Data = new List<double> { ViewBag.WalkStat, ViewBag.RunStat, ViewBag.RunStat, ViewBag.DanceStat, ViewBag.SwimStat },
                      Label = "My Second dataset",
                      FillColor = "rgba(228, 247, 187, 1)",
                      StrokeColor = "rgba(151,187,205,1)",
                      PointColor = "rgba(151,187,205,1)",
                      PointStrokeColor = "#fff",
                      PointHighlightFill = "#fff",
                      PointHighlightStroke = "rgba(151,187,205,1)",
                  }

            });
            }
            
            <canvas id="@Canvas" width="350" height="400"></canvas>
            @Html.CreateChart(Canvas, complexChart)
            </div>
        </div>
```
7. You should now see a radar chart, here is an example chart of for my admin
[![AdminChart.jpg](https://s22.postimg.org/g43zh8j0x/Admin_Chart.jpg)](https://postimg.org/image/5ha6btavh/)

## Conclusion

I hope you found this helpful, you can email me with any questions at ben.micah.peterson@gmail.com