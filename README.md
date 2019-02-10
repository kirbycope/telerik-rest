# telerik-rest
Testing REST API Using Test Studio

## PROJECT DESCRIPTION 
This project makes use of the new
[HttpClient Class](http://msdn.microsoft.com/en-us/library/system.net.http.httpclient(v=vs.110).aspx) in the [.Net Framework 4.5](http://www.microsoft.com/en-us/download/details.aspx?id=30653).
This is much easier to use than the [WebRequest/WebResponse Class](https://docs.telerik.com/teststudio/advanced-topics/coded-samples/general/invoke-web-service-call) in older .Net frameworks.
I have included an example project, but will detail the steps of making your own.

## Add the Assembly Reference
After installing the .Net Framework 4.5, follow the [Telerik Article](https://docs.telerik.com/teststudio/features/coded-steps/add-assembly-reference) on adding the assemblies to your project. The .dll files are likely in these locations:
   * C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\v4.5\System.Net.Http.dll
   * C:\Program Files (x86)\Microsoft ASP.NET\ASP.NET MVC 4\Assemblies\System.Net.Http.Formatting.dll

## Add the Using Directives to your .CS Files
For every .cs file (created when you make a coded step with C# as your language), add the following using directives:
   * using System.Net.Http;
   * using System.Net.Http.Formatting;
   
## Create Your HttpClient Container
The HttpClient/session will be disposed after the test(s) is ran. To prevent this, and to allow multiple tests with the same session, you must contain the HttpClient in its own class. This will allow you to have multiple tests like; login, get a record, delete a record, and then logout.
To make an HttpClient container, make a new web test. I like to make a folder for reused modules called, "Modules". In that folder I make a web test called, "HttpClient". Add a coded step to generate you .cs file. Delete everything in the namespace, you won't need it. Now, add your using directives (see previous section).
The Class you need to make needs to allow the HttpClient to be created only once and read-only. It's a bit odd, so just copy+paste my code from the example. You will want to change the BaseAddress property of _client to suit your testing. Think of this property like the [BaseUrl](https://docs.telerik.com/teststudio/user-guide/knowledge-base/test-execution/using-baseurl.aspx) project setting.
Here is what my container class looks like:
````csharp
using Telerik.TestingFramework.Controls.KendoUI;
using Telerik.WebAii.Controls.Html;
using Telerik.WebAii.Controls.Xaml;
using System;
using System.Collections.Generic;
using System.Text;
using System.Linq;
 
using ArtOfTest.Common.UnitTesting;
using ArtOfTest.WebAii.Core;
using ArtOfTest.WebAii.Controls.HtmlControls;
using ArtOfTest.WebAii.Controls.HtmlControls.HtmlAsserts;
using ArtOfTest.WebAii.Design;
using ArtOfTest.WebAii.Design.Execution;
using ArtOfTest.WebAii.ObjectModel;
using ArtOfTest.WebAii.Silverlight;
using ArtOfTest.WebAii.Silverlight.UI;
 
using System.Net.Http;
using System.Net.Http.Formatting;
 
namespace TestRestAPI
{
    public class RestClient
    {
        private static HttpClient _client = null;
 
        public static HttpClient client
        {
            get
            {
                if (_client == null)
                {
                    // Write action to console
                    Console.WriteLine("No active HttpClient, creating a new one.");
                     
                    // Create new Http Client Handler
                    HttpClientHandler handler = new HttpClientHandler
                    {
                        // Do not use a proxy
                        UseProxy = false
                    };
                     
                    // Create new Http Client using the Handler
                    _client = new HttpClient(handler);
                     
                    // Set BaseAddress
                    _client.BaseAddress = new Uri("
http://httpbin.org/
");
                     
                    // Write action to console
                    Console.WriteLine("Set Base Address to " + _client.BaseAddress.ToString());
                }
                return _client;
             }
         }
    }
}
````

## Create Your Test(s)
Now that you have an HttpClient that can be used by multiple tests, you can call that client in your test(s). Create a web test and add a coded step, I like to delete all the #region code in the class as well as the giant comment block in the namespace. Also, I like to close the browser window that opens when a web test is ran, because it is not needed for REST test(s). To close the browser window, use the ActiveBrowser.Close(); function.

## Sending a GET
In your test's class perform a get using [GetAsync()](http://msdn.microsoft.com/en-us/library/hh158912(v=vs.110).aspx). This takes a Uri parameter, which appends the BaseAddress to the value you pass in. In my example, comes out to "http://httpbin.org/get".
Here is what my GET test looks like:
````csharp
using Telerik.TestingFramework.Controls.KendoUI;
using Telerik.WebAii.Controls.Html;
using Telerik.WebAii.Controls.Xaml;
using System;
using System.Collections.Generic;
using System.Text;
using System.Linq;
 
using ArtOfTest.Common.UnitTesting;
using ArtOfTest.WebAii.Core;
using ArtOfTest.WebAii.Controls.HtmlControls;
using ArtOfTest.WebAii.Controls.HtmlControls.HtmlAsserts;
using ArtOfTest.WebAii.Design;
using ArtOfTest.WebAii.Design.Execution;
using ArtOfTest.WebAii.ObjectModel;
using ArtOfTest.WebAii.Silverlight;
using ArtOfTest.WebAii.Silverlight.UI;
 
using System.Net.Http;
using System.Net.Http.Formatting;
 
namespace TestRestAPI
{
    public class Get : BaseWebAiiTest
    {
        [CodedStep(@"GET")]
        public void Get_CodedStep()
        {
            // Close the web browser window (not needed)
            ActiveBrowser.Close();
             
            // Write action to console
            Console.WriteLine("Sending GET...");
             
            // Create and define the result of the GET. Note that GetAsync uses the BaseAddress set in the Module, "HttpClient"
            HttpResponseMessage result = RestClient.client.GetAsync("get").Result;
             
            // Handle HttpClient Result
            if ((int) result.StatusCode == 200)
            {
                // Response was OK
                Console.WriteLine("Response == 'OK'");
                 
                // Write result to log
                Log.WriteLine(result.ToString());
            }
            else
            {
                // Response was not expected
                Console.WriteLine("Response != 'OK'");
                throw new Exception(result.ToString());
            }
        }
    }
}
````

## Sending a POST
In your test's class perform a get using [PostAsync()](http://msdn.microsoft.com/en-us/library/hh138190(v=vs.110).aspx). This also takes the Uri parameter, but also requires an HttpContent parameter. What that parameter contains and it's format is completely up to your specific API. For my example, I know that httpbin.org uses JSON. So I made a simple JSON formatted object from a String Array. You will need to check what parameter your API takes from POST requests and of what type. This is expressed in my example in the HttpContent content = ... line.
Here is what my example looks like:
````csharp
using Telerik.TestingFramework.Controls.KendoUI;
using Telerik.WebAii.Controls.Html;
using Telerik.WebAii.Controls.Xaml;
using System;
using System.Collections.Generic;
using System.Text;
using System.Linq;
 
using ArtOfTest.Common.UnitTesting;
using ArtOfTest.WebAii.Core;
using ArtOfTest.WebAii.Controls.HtmlControls;
using ArtOfTest.WebAii.Controls.HtmlControls.HtmlAsserts;
using ArtOfTest.WebAii.Design;
using ArtOfTest.WebAii.Design.Execution;
using ArtOfTest.WebAii.ObjectModel;
using ArtOfTest.WebAii.Silverlight;
using ArtOfTest.WebAii.Silverlight.UI;
 
using System.Net.Http;
using System.Net.Http.Formatting;
 
namespace TestRestAPI
{
    public class Post : BaseWebAiiTest
    {
        [CodedStep(@"Post")]
        public void Post_CodedStep()
        {
            // Close the web browser window (not needed)
            ActiveBrowser.Close();
             
            // Create the user login data
            String [] user = {"username", "password"};
             
            // Create and define the HttpContent object to be passed in client.PostAsync function
            HttpContent content = new ObjectContent<String[]>(user, new JsonMediaTypeFormatter());
             
            // Write action to console
            Console.WriteLine("Sending POST...");
                 
            // Create and define the result of the POST
            HttpResponseMessage result = RestClient.client.PostAsync("post", content).Result;
             
            // Handle HttpClient Result
            if ((int) result.StatusCode == 200)
            {
                // Response was OK
                Console.WriteLine("Response == 'OK'");
                 
                // Write result to log
                Log.WriteLine(result.ToString());
            }
            else
            {
                // Response was not expected
                Console.WriteLine("Response != 'OK'");
                throw new Exception(result.ToString());
            }
        }
    }
}
````

## Test Validation
The responses of the GET/POST/etc. are of type HttpResponseMessage, which liekly contain a JSON. You can convert the result to your object type and do asserts on those fields. Also, the get/post can return a 200 status code and have as empty JSON. Or, it can return a 409, 204, etc. Be sure to account for all expected outcomes or your test will break or pass when it ought to have been failed.
I will use a made up data type called MyDataType to show an example:
````csharp
// Convert the result to your data type
MyDataType getResult = result.Content.ReadAsAsync<MyDataType>().Result;
 
// Handle the result as the object could be null and the http response code was a 200
if (getResult.ResultEntries.Count() > 0)
{
    //some action on the object, like...
    Log.WriteLine("Name: " + getResult.ResultEntries.ElementAt(0).Name.ToString());
}
else
{
    // some action in response to the object being empty
}
````
