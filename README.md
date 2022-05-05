# **RudderStack 101 Workshop**

Today we’ll be experimenting how to send event via a javascript SDK to a Data Warehouse and another Downstream application like Google Analytics 4. 

Prerequisites:

- A website hosted with a Domain name (you can create a free one using www.w3schools.com)
- A RudderStack free trial account
- A GCP free trial account
- A Google analytics account

The first pipeline we’ll set up is going to be an Event Stream pipeline, this type of pipeline enables us to use the RudderStack SDK to send events from a website, web app or mobile app to either a Cloud warehouse or a downstream application, while optionally applying transformations. 

**Step 1: Create a Source from the RudderStack dashboard:** 

1. Once you login to rudderstack.com, sign in with your company email you will start seeing the RudderStack homepage. 
2. Note the Sources, Destinations as well as the data plane details in the top right corner of the page. The Data Plane URL is something you will need to copy somewhere to save as you will be using it in later steps, it should look something like this **https://yourcompanyname.dataplane.rudderstack.com**
3. Click on “+ Add Source”, you can either have search for Javascript in the search field, or select the Javascript source from the **Popular** section.
4. Name your source, make sure to use meaningful names if you plan to add more  javascript sources later on. 
5. Once that is created, a page will appear with all your source details. On the right hand side, you should be able to find your **write key**, take note of that because you will be needing that along side your **data plane URL**. 

**Step 2:** **Installing the RudderStack Javascript SDK:**

Now that we have set up our source in RudderStack, we’ll have to add the piece of code to our website that lets us create a connection to RudderStack and also start sending events to our destinations. 

1. Copy the following snippet, then paste it into the head node of your .html code in your website. 
```html
  
<script>
  rudderanalytics = window.rudderanalytics = [];
  var methods = [
    "load",
    "page",
    "track",
    "identify",
    "alias",
    "group",
    "ready",
    "reset",
    "getAnonymousId",
    "setAnonymousId",
  ];
  for (var i = 0; i < methods.length; i++) {
    var method = methods[i];
    rudderanalytics[method] = (function (methodName) {
      return function () {
        rudderanalytics.push(
          [methodName].concat(Array.prototype.slice.call(arguments))
        );
      };
    })(method);
  }

  rudderanalytics.load(<WRITE_KEY>, <DATA_PLANE_URL>);
  rudderanalytics.page();
</script>

<script src="https://cdn.rudderlabs.com/v1.1/rudder-analytics.min.js"></script>

 ```
b. Note where it says <WRITE_KEY> and <DATA_PLANE_URL>, you’ll need to replace those with the details you copied in Steps 1.b and 1.e. You will notice here that we are calling two methods:

1. The **load()** method will set up and establish a connection to the Source you’ve configured in RudderStack earlier.
2. The **page()** method will capture page details, you have the option of removing this or adding different methods to capture different types of user information. We will see examples of that later on.

Now, we have set up our connection. Make sure to apply those changes to your website.

**Step 3. Verifying your Source connection:**

Now that we have set up our connection, we want to verify that RudderStack is able to receive events. 

1. Go back to RudderStack, on the same page that our Source details, click on “Live Events” in the upper right corner of the page. 
2. Open up a different tab on your browser, navigate to your website’s URL and refresh the page. 
3. In you RudderStack Live Events page, you should start seeing events coming in. Since we only made a page call, we should see an event called Page along with the contents of the event or the Payload on the right. This will have all the information capture by the RudderStack page call from your website.

**Step 4. Set up the destination:**

1. **Cloud Warehouse (Google BigQuery):** 

Now that we set up our source and verified that we can receive events, we need to set up our destination that we want the events to be sent to. The first destination we’ll set up is to a Cloud Warehouse which will be Google BigQuery for this tutorial.

1. In the same Source page that we created, click on “Add Destination” within the blue button, then select create new destination.
2. Choose BigQuery from the list of destinations, then click continue.
3. Name your destination.
4. Follow the steps in this doc to set up your BigQuery destination https://www.rudderstack.com/docs/data-warehouse-integrations/google-bigquery/
5. After creating the BigQuery destination, similarly you should be able to select the destination from the Destinations tab in RudderStack and from there check the “Live Events” to see if the stream has been successful. 
6. Once you start seeing events, head to your BigQuery instance in GCP and take a look at the tables. You should be able to see a new dataset was created with the same name of the Source, or whatever name you specified for you Namespace in the Destination settings. 
7. Collapse the dataset and you should be able to see a table called pages, which will have all the page view data captured by the RudderStack SDK from your website. 

1. **Google Analytics 4**

Now that we have seen how to send the data to a Cloud Warehouse to visualize the data into table, run queries on the data and create customer profiles, let’s look at how we can stream data into different tool like GA4 to extract key marketing insights:

1. In your browser, enter [analytics.google.com](http://analytics.google.com) and sign in with your work account.
2. In the bottom left corner, you’ll find a gear symbol, click there to open up the admin portal. 
3. If you haven’t already created an account, create one now.
4. If you have created an account, click on “+ Click Property”. Name your property, select your time zone then answer questions about your business. When done, click on “Create”
5. Now that the property is created, within the Property column you’ll see a setting for Data streams, click on that.
6. Now click on “Add stream”, select Web.
7. To Set up your web Stream, add your website URL and provide a name for your Data stream.
8. Once done, click on “Create Stream”.
9. Once your stream is created, you’ll get a Measurement ID, make note of that since you will be using it to set up your destination connection in RudderStack
10. Switch back to RudderStack in your browser. On the Destinations tab on the left select the GA4 option, and name your destination. 
11. In the Destination configuration settings, you’ll need to enter your measurement ID.
12. For the rest of the seetings, you can keep the defaults. Under Native SDK, make sure you enable the option to send User ID to GA, and also to extend page view property.
13. Go back to Google Analytics 4, Click on report under home in the Admin panel then click on real-time. Here, you should start seeing the page events coming in along with any events GA4 will capture. 
