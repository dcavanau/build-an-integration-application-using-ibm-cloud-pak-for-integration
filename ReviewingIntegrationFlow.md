# Review API Integration Flow

Login to CP4I and navigate to App Connect Designer. Click on the `Car Insurance Cognitive API Lab Short` tile on the dashboard.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616161203051.png" alt="image-20200616161203051" style="zoom:50%;" align="left"/>

What you can see  first is our API model.

App Connect Designer builds your API for you – you don’t need to worry about OpenAPI specs or Swagger editors – it’s all built in. To create your API, you just type in the names of the fields you want to use in plain English. If you want, you can use objects for complex structures but we won’t here

These are the fields we are going to use for our API – we’ve imported them to save you time. You can rename them if you wish but if you do, our test scripts for the APIs won’t match – or work, so leave them as they are for now.

![image-20200616135210029](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616135210029.png)

Note that we tell our API which field is the key – in our case, CaseReference. When creating RESTful APIs, they should be resource based and each resource should have a unique key.

ICP4i Designer bakes in good REST API creation right into the tooling so you don’t need to worry too much about it.

Note that the `PhotoOfCar` property is a string – our consumers will pass the photo data in as a string of base64 encoded text. This is one way of passing a binary image to an API.

Now that we’ve told the API what data to use, we need to define what actions to perform on that data.

For this lab, we’ve defined our `CarRepairClaim` data model. We have data fields – what do we want to do with them?

Now we want to do a `Create Car Repair Claim` operation.

Click `Operations` – operations are the actions that the API exposes with the data.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616135340633.png" alt="image-20200616135340633" style="zoom:60%;" align="left" />

We can have multiple operations in one API – such as `Create` `Retrieve` `Update` etc. The tooling auto-generates good REST for you, translating into HTTP verbs like GET and POST automatically. You don’t need to know REST to build APIs with ICP4i – the knowledge you need is built in.

For example, look how a pull-down menu auto generates the HTTP ‘POST’ and the path of /carrepairclaim.

In this exercise, we’re going to build just one operation – you can add more if you wish.

We’re going to go into the flow logic – click `Edit Flow`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616135559162.png" alt="image-20200616135559162" style="zoom:50%;" align="left" />

You’ll now see the flow in the designer flow editor here:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616135644866.png" alt="image-20200616135644866" style="zoom:67%;" align="left"/>

See the `App Connect Trial` account name on the IBM Watson Visual Recognition? That’s the reason we had to get the account name correct so it matched.

Scroll through all of the connectors in the flow (use the scroll bar at the bottom) and make sure there are no red dots anywhere.

App Connect Designer connects to the endpoint service every time you open the flow to see if there is updated metadata (this is why you can see spinners when you open the flow). This means the services need to be connected correctly. If there is an issue, there will be a red dot on the connector node.

The most likely reason for a red dot is that your connector account name does not match the name of the account used in the flow. To fix this, go back and rename it in the connector (click the cogwheel at the top right and click ‘Catalog’ to get back to it).

You could rename the accounts in the flow if you wish, but that might make the lab harder to follow: App Connect doesn’t really mind what the accounts are called as long as the references all match.

Using the zoom in/out bar, we’ve shown you the entire flow below:

![image-20200616135920177](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616135920177.png)

Or two readable chunks!

![image-20200616135943139](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616135943139.png)

![image-20200616135955838](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616135955838.png)

You can see that the flow visually matches the logic we defined at the start let’s step through.

### 1. Receive the Customer’s car repair request with photograph via an API

Designer automatically creates an API “Request” and “Response” node for your API flow.

Click on the ‘Request’ node.

![image-20200616171718268](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616171718268.png)

Note how the request body is created from the model – and sample data is automatically generated. When building there is literally nothing to do here – it’s done for you.

### 2. Use IBM Watson Image Recognition to analyse the photo

If it is not a valid picture, Watson will return an error immediately to the user calling the API.

![image-20200616171813202](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616171813202.png)

We use the built-in Watson Visual Recognition connector that we configured earlier. Note that we selected `App Connect Trial` account here. If you had it named incorrectly (e.g. `Account 1`) then you would have an error. To fix it, change to `App Connect Trial` in the pull down.

The `Classifier ID` is to tell Watson which image training data set, or classifier it should use. You can train Watson with your own image data e.g. for products your company sells or assets it uses. If you create a custom classifier, the connector will go and find it and offer it to you in a drop-down. We will use the default ‘out of the box’ classifier.

We only need one ‘target field’ to populate – our images file. This is our bas64 string with the photo in it.

Designer doesn’t have ‘Mapping Nodes’ – it’s inspired by spreadsheets where you concentrate on what data you want to put in a cell, rather than where source data needs to be mapped to. All of the fields that have been populated in the flow from variables, requests or connectors are automatically stored and are available for you to use at any time.

You can see that we’ve mapped out `PhotoOfCar` field from our request. You can tell it’s from the request because it has the same icon as the request node next to it!

Click the hamburger (three lines) pull down next to the field:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616172001428.png" alt="image-20200616172001428" style="zoom:50%;" align="left"/>

All of the fields that are in the flow so far are available – just click on the one you want. This is how ‘mapping’ is done in designer – it’s like filling in cells in a spreadsheet.

Make sure `PhotoOfCar` is selected (or don’t change it) before you move on.

### 3. Check that Watson can `see` a car in the picture

Watson will return a list of what it thinks it can see in a picture, each with a confidence rating.

For example, for one of our test pictures, we will use a picture of a Subaru SUV. If we ask Watson to look at this, we see the following – it’s .87 (87%) confident it can see a car in the picture.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616172209776.png" alt="image-20200616172209776" style="zoom:50%;" align="left"/>

(This screenshot is from Watson Studio – Available for free in the IBM Cloud – search for it and you can try it yourself with your cloud account and your Visual Recognition Service instance)

We’re going to set variables to check for three things:

- Is there a car in the image? `ImageCar`
- Is there a person in the image? `imagePerson`
- Is there a roadster (convertible) car in the image – this is for the extension lab, but we have the logic here anyway.

![image-20200616172414907](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616172414907.png)

Let’s look at ‘is there a car?’

Click on the menu on the right, then expand ‘IBM Watson Visual Recognition’

*(in the lab environment, you might need to zoom out on the browser to about 67% to see the menu – use the Firefox hamburger menu on the top right and choose ‘zoom’ as below)*

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174222627.png" alt="image-20200616174222627" style="zoom:50%;" align="left" />

The ‘Available Inputs’ menu appears. You can see we now have fields from both the request and IBM Watson Visual Recognition.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174252860.png" alt="image-20200616174252860" style="zoom:50%;" align="left"/>

If you scroll down, you’ll see we get down to Image->Images[]->Classifiers[]->Classes[]->Class name (together with the Score)

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174326095.png" alt="image-20200616174326095" style="zoom:50%;" align="left"/>

What does this mean?

It means Watson has returned an Image object.

--In the Image object is a list (array) of images (we denote this using [])

----In each image (they may be more than one) there is a list(array) of Classifiers (Watson training)

------In each Classifier, there is a list(array) of Classes. This is what Watson sees. e.g. ‘Car’

--------In the Class, there is a class name and a score – amongst other things.

In all of that, we need to say ‘Hey Watson – thanks for the data: Can you see a car?’ Normally we’d end up coding loops around arrays and if/thens to find it…

App connect does it by using a formula – just like a spreadsheet.

Close the pull-down and click on the ‘Classes’ bubble in the ‘imageCar’ field. Click ‘Edit expression’

![image-20200616174412565](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174412565.png)

![image-20200616174426679](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174426679.png)

What we did is click on the hierarchy pull down to build a query that looks like this:

$IBMWatsonVisualRecognitionClassifyimages.classify_images.classifiers.classes[class=’car’]

(we had to manually add the [class=’car’] part at the end)

All this does is go down the hierarchy, each level separated by dots and then has a select query at the end [class=’car’] to give us all the entries where the class is a car.

App Connect automatically scans through the entire hierarchy, sorting out things like arrays/lists and objects and lets us get straight to the data we want.

We use the same approach to populate imagePerson using [class=’car’] and imageRoadster [class=’roadster’].

If you want to more easily see the query expression, then hover over the ‘classes’ bubble e.g. here:

![image-20200616174504163](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174504163.png)

All mapping is done the same way -for example, we want a string that joins (concatenates) all of the classes (things that Watson can see) together, separated by commas so it’s Human Readable. For this we use ‘apply a function’ and select ‘Join’ from String functions, just like building a spreadsheet formula.

To get the ‘dot hierarchy’, just use the pull down variable explorer. Pick the field you want and the choose ‘Apply a function’

$join($IBMWatsonVisualRecognitionClassifyimages.classify_images.classifiers.classes.class, ', ')

Note that App Connect joins all the classes together in one go – no for loops, building up strings etc. It’s a different way of looking at mapping.

All of this mapping uses a language called JSONata – more details here https://jsonata.org

### 4. If not, we will immediately respond back with an error saying `There is no car in this picture`

We now know if there is a car or not in our image…if there is no car image (i.e. if ‘imageCar’ is empty) we want to send back an error.

We use an App Connect ‘If’ node to visually show us our logic:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174620180.png" alt="image-20200616174620180" style="zoom:60%;" align="left"/>

We visually create an ‘If imagecar is empty’ check. If it is empty, we send a ‘bad request’ response – note that we don’t need to remember that in REST APIs, ‘Bad Request’ is ‘HTTP 400’ – App Connect knows this – just pull the response from the drop down.

We also add a ‘There is no car in this image, please resubmit’ error.

### 5. Create a `Case` in Salesforce with the data from the API

This Case is where we store the details and progress of our repair.

We’ve already connected to Salesforce, so we can use the Salesforce connector. We want to add a contact for this case, but we need the contact ID from Salesforce, not the name when we create the case. This is a very common integration issue – systems need IDs and not names. No problem, sorting this is simple!

Click on `Retrieve Contacts`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616174808434.png" alt="image-20200616174808434" style="zoom:60%;" align="left"/>

We’re going to use ‘Andy Young’ as our contact – he’s the contact for the insurance company that sends customers. Salesforce Developer Accounts have a pre-populated set of data that you can use to test. ‘Andy Young’ is one of those pre-populated contacts. We will hard-code his name for speed.

But how do we know exactly what ‘Full Name’ means? Does it have ‘Mr?’ in it? Is it ‘Andy’ or ‘Andrew’? Do we need his middle name?

Change the name to ‘Andrew Young’ and we’ll find out. Note that App Connect gives you the field description to help you out. If you add any expressions such as $uppercase(Full Name) then the preview (next to the eye) shows you what your field will look like after your expression.

Now click the ‘Test’ button

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175036839.png" alt="image-20200616175036839" style="zoom:50%;" align="left"/>

We can go straight off to SalesForce to check – we get this:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175107749.png" alt="image-20200616175107749" style="zoom:50%;" align="left"/>

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175137878.png" alt="image-20200616175137878" style="zoom:50%;" align="left"/>

OK, let’s put the field back to ‘Andy Young’ and try clicking ‘Test’ again.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175222971.png" alt="image-20200616175222971" style="zoom:50%;" align="left"/>

Success! Hooray, let’s check our result. Click `View details`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175300249.png" alt="image-20200616175300249" style="zoom:50%;" align="left"/>

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175326523.png" alt="image-20200616175326523" style="zoom:70%;" align="left"/>

There’s our test results, right in the tooling, right from the real system in the cloud. This works with all of the connectors such as Watson in our flows here. It’s a great way of checking your integration calls work the way you want them to without having to test the whole flow.

Now we have the ID that we need, let’s create our Salesforce case. Click on the Salesforce – Create case node. Note that we just re-use the same connector but with a different operation and data.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175408619.png" alt="image-20200616175408619" style="zoom:60%;" align="left"/>

Note we we can see that our contact ID comes from the previous `retrieve contact` Salesforce Call. The Name and email come from the API Request. 

The connector ‘knows’ that fields like ‘Case Type’ have a limited number of values in Salesforce – so it automatically converts them into pull-down lists of values for you to choose from.

Also our subject. It’s like a spreadsheet – we just type in what we want. No Concatenation, no adding strings together!

### 6. Add the photograph to our Salesforce case so we have it stored

To add a photograph, we need to create a salesforce attachement – that’s easy, just use the connector again.

Click on `Create Attachment`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175524660.png" alt="image-20200616175524660" style="zoom:60%;" align="left"/>

Note that we use the Case ID that is a returned value from the ‘Create Case’ connector call – it’s been kept in the flow automatically. We send the PhotoOfCar as a base64 string and we tell Salesforce that the content Type is image/jpeg.

### 7. Analyse the description of the problem as described by the customer using IBM Watson Tone Analysis

We store this in Salesforce for future reference – if the customer is angry or upset, we may wish to take further action or treat them more carefully.

First we’ll use the Watson Tone Analyzer; Click on `Get tone analysis`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175622530.png" alt="image-20200616175622530" style="zoom:60%;" align="left"/>

Then we’ll add a comment to the case with the Salesforce connector and give it the tone name from Watson.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175652190.png" alt="image-20200616175652190" style="zoom:60%;" align="left"/>

### 8. Send a response back to the customer with their Salesforce case reference

For future enquiries and also an estimate of how long it will take to repair and how much it will cost (These are hard coded in this lab)

Click on the Response node – we just fill in the values we want, like all the others.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175830303.png" alt="image-20200616175830303" style="zoom:60%;" align="left"/>

Click ‘Done’ we’ve built the flow – let’s start it!

![image-20200616175858245](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616175858245.png)

