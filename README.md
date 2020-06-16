# Build a Cognitive Car Claim Processing application with CP4I Integration capabilities
### Work-in-progress

Every enterprise in today's markets must offer robust digital products and services, often requiring integration of complex capabilities and systems to deliver one coherent whole. [IBM Cloud Pak for Integration](https://cloud.ibm.com/docs/cloud-pak-integration?topic=cloud-pak-integration-getting-started) offers a simplified solution to this integration challenge, allowing the enterprise to modernize its processes while positioning itself for future innovation. Once installed, IBM Cloud Pak for Integration eases monitoring, maintenance and upgrades, helping the enterprise stay ahead of the innovation curve.

In this Code Pattern will Integration Designer to create an API which takes a car repair claim request, complete with a photograph of the car, and integrates with a SaaS CRM system and IBM’s Watson AI to create a car repair case with all the correct details loaded into the SaaS system. and data routed to the correct location based on the image contents; all in a few seconds before returning a response to the customer.

As an ‘extension’ we check if the car is a convertible/roadster.

 If it is, we translate the request into Spanish for our Spanish-speaking partner and create an incident their ServiceNow SaaS system, complete with car photograph.

Due to time constraints, the ‘extension’ will only be briefly described – we will ensure we build an end-to-end managed API before discussing the extended scenario.

Business Scenario:

- We are a Car Repair company: We take in vehicles with problems and repair them – seems simple but..

  - We want to gain business advantage by allowing multiple car leasing companies to use us to repair their cars – these companies insist that we expose APIs for them to call to do business with them.
  - We want to allow their customers to book their cars in for repair and get an estimate for price and number of days in real time – in seconds. Later we will build more APIs to allow customers to query the status of their repairs, or make updates or add comments to their repair cases.
  - We want to allow them to send photos of their cars so we can check for type, damage etc.
  - We want to check for errors and issues up-front as quickly as possible to feed back to the customer in real time. Photo not valid? No car in the photo? We’ll tell you instantly so you can re-submit.
  - We want to minimize manual processes and have the repair request automatically create a repair case in our CRM system (Salesforce). If a customer wants to book a repair at 3am on a Sunday, they can – it’s their choice.
  - We are wanting to grow our business fast with this new model and expect the use of APIs to really increase the number of requests we get. We need our solution to be scalable and highly available.

  ![image-20200608183714387](./images/image-20200608183714387.png)



# Architecture

![image-20200605145034827](./images/image-20200605145034827.png)



# Flow

1. User sends car repair request to API (API Connect)
2. User request is forwarded to integration flow
3. Use IBM Watson Visual Recognition to analyze the photo. If it is not a valid picture, Watson will return an error immediately to the user calling the API. Check that Watson can ‘see’ a car in the picture – if not, we will immediately respond back with an error saying ‘There is no car in this picture’ so the error can be corrected immediately.
4. Create a `Case` in Salesforce with the data from the API. This Case is where we store the details and progress of our repair. Also, add an attachment of the photograph to Salesforce so that we have the image stored in our system.
5. Analyze the description of the problem as described by the customer using IBM Watson Tone Analysis. We store this in Salesforce for future reference – if the customer is angry or upset, we may wish to take further action or treat them more carefully.
6. This is an extended scenario, wherein if the car is a specialist car (identified by Visual Recognition Service) it is sent to partner for repair. The partner speaks Spanish and and we'll use IBM Watson language translator to translate our request into Spanish before we send it to them
7. This is an extended scenario as well. Partner uses ServiceNow, not Salesforce so we need to create an incident in their ServiceNow system – automatically
8. Send a response back to the customer with their Salesforce case reference for future enquiries and also an estimate of how long it will take to repair and how much it will cost.



# Watch the Video

[![](http://img.youtube.com/vi/TRzO26kawu4/0.jpg)](https://www.youtube.com/watch?v=TRzO26kawu4)



We will achieve the above objectives in two parts as follows:

Part1 - In part1, this Code Pattern, we will build and deploy our integration application using App Connect in CP4I. 

Part2 - In part2, we will expose the integration application APIs using API management feature of CP4I. [Use API Management capabilities of ICP4i to expose APIs to the wider world in a controlled manner]()



# Steps

1. Plan of Work
   1. Set up our integration systems and services endpoints
   2. Create an integration flow for our ‘Car Repair Claim API’
   3. Deploy the API to the Cloud Pak for Integration (ICP4i) runtime
2. List of things we will need:
   1. List of Systems and Services Endpoints
      1. Salesforce:
      2. IBM Watson – Visual Recognition:
      3. IBM Watson – Tone Analysis:
      4. ServiceNow:
      5. IBM Watson – Language Translation:
   2. IBM Cloud Pak for Integration (ICP4i) Capability list:
      1. Application Integration (IBM App Connect):
      2. Smart Connectors
      3. API Management (IBM API Connect)
   3. Some extra things we need for this series:
      1. An email server and client
      2. GitHub
      3. IBM API Connect Toolkit
3. Getting Started – Setting up the endpoints
   1. Setting up IBM Watson Services
      1. Logging in to IBM Cloud
      2. Creating your free Lite-Plan IBM Watson Services:
   2. Setting up SalesForce
4. Getting into the Cloud Pak and Building Your Integration
   1. Getting into SkyTap and the VMs
   2. Accessing ICP4i
   3. Accessing the Designer Integration Tooling
   4. Connecting the tooling to our endpoints
   5. Setting up the Salesforce Connection
   6. Importing the Integration flow into designer
   7. Reviewing our API Integration Flow:
      1. Receive the Customer’s car repair request with photograph via an API
      2. Use IBM Watson Visual Recognition to analyse the photo.
      3. Check that Watson can ‘see’ a car in the picture
      4. If not, we will immediately respond back with an error saying ‘There is no car in this picture’
      5. Create a ‘Case’ in Salesforce with the data from the API.
      6. Add the photograph to our Salesforce case so we have it stored.
      7. Analyse the description of the problem as described by the customer using IBM Watson Tone Analysis.
      8. Send a response back to the customer with their Salesforce case reference
   8. Starting the flow:
5. Testing our API Integration Flow
   1. API Base path
   2. APIKey / Client ID
   3. UserID and Password
6. How we will test the APIs
   1. $cp4ibasepath
   2. $cp4iuser and $cp4ipw
   3. $cp4iclientid
   4. Setting Environment Variables to test in the ACE Designer
7. Running the tests and results:
   1. Test 1: “Chicken Picture” – demotestchicken.sh
   2. Test2: “Subaru SUV Picture” – demotestsubaru.sh
8. Deploying the Integration flow to ICP4i RunTime via the App Connect Dashboard
   1. Exporting the executable bar file:
   2. Navigating to the App Connect dashboard and importing the .bar file
      1. Name:
      2. Which type of image to run:
      3. IBM App Connect Designer flows
      4. Docker images
      5. Integration Server:
   3. Creating the Connector Credentials Secret:
      1. Downloading the Configuration Package
   4. Testing the flow on the ICP4i runtime:



## 1. Plan of Work

This solution has a number of moving parts, so we’ll tackle them in a logical sequence. If you’re familiar with how to do any of the steps, feel free to do them in the way you prefer or are familiar with.

If you’re familiar with any of the tools we’re using, feel free to embellish or change the series – as long as you make sure it all ‘hangs together’. You can build the ‘extension’ version if you’re familiar with the tooling.

We’ll be doing the following:

### 1.1 Set up our integration systems and services endpoints

We are going to integrate with SaaS systems and IBM Watson AI services. 

We will need to have these endpoints created and create credentials for, so that we can integrate to them securely. In the ‘real world’ systems like Salesforce or ServiceNow will be running at customers already.

### 1.2 Create an integration flow for our ‘Car Repair Claim API’

This will create our API and the integrations to all of our endpoints. 

We will create an ‘integration flow’ which takes the API request, calls the endpoints in the correct order, maps the data between them and sends an appropriate API response back to the caller.

### 1.3 Deploy the API to the Cloud Pak for Integration (ICP4i) runtime

Once we have developed our flow and tested it, we will deploy it to ICP4i running on OpenShift. This will create a Kubernetes container/pod deployment. 



## 2. List of things we will need:

As this is an integration application, we will need systems and services to integrate to:

### 2.1 List of Systems and Services Endpoints

The systems and services we will use are as follows: Instructions for these are further on

#### 2.1.1 Salesforce:

Salesforce is a CRM system provided as a SaaS i.e. it is hosted in the cloud.
In this scenario, we as a car repair company will use Salesforce to create and store our car repair claims.

Salesforce allows you to create developer instances/accounts free of charge. You will need a developer account to run these instructions as to how to create them are included and you can set an account up. If you already have a developer Salesforce account, you can use that.

#### 2.1.2 IBM Watson – Visual Recognition:

IBM Watson is available on the IBM Cloud and also in the IBM Cloud Pak for Data. IBM Cloud lets you create non-expiring free instances of the IBM Watson services that you can use for this exercise (or anything else)

The IBM Watson Visual Recognition service lets you send a picture (.jpg, .png) to Watson and returns a list of things that Watson can ‘see’. In our case, we will use Watson to check if there is a car in the picture and, in the extended version,  if it is a convertible/roadster car or not. If it’s a roadster, we’ll send it to our partners. If it’s not, we’ll repair it ourselves. If there’s no car in the photo, we’ll send it back and let our customer know.

#### 2.1.3 IBM Watson – Tone Analysis:

IBM Watson can tell if someone is happy or sad or angry or many other emotions!

If your customer is angry, you want to know so you can make them happy – we’ll use this to look at the customer’s description of the damage/problem and put the tone into our Salesforce case so that when we call them, we know what to expect.

For the ‘extended’ scenario, we will also use:

#### 2.1.4 ServiceNow:

ServiceNow is an incident management system which is provided as a SaaS on the cloud. Our repair partner which repairs convertible/droptop cars uses this system to manage their repairs.

ServiceNow allows you to create free developer instances as well – you will need one if you want to implement the extended scenario.

#### 2.1.5 IBM Watson – Language Translation:

Our convertible car repair partners speak Spanish – and our Spanish isn’t great (yours might be!).

Fortunately Watson speaks Spanish and many other languages better than we do, so we’ll use Watson to translate our ‘please repair this car’ request before we put the request into our partner’s ServiceNow system.

### 2.2 IBM Cloud Pak for Integration (ICP4i) Capability list:

The Cloud Pak for Integration contains components and capabilities to implement multiple integration patterns – we won’t be using all of them in this exercise.

In part2 of the series, we will be using the secure gateway (DataPower) to secure our APIs as part of API Management but we will not be using messaging (MQ), event streaming/Kafka or the high-speed Data Transfer (Aspera) in this exercise. 

We will use the following ICP4i capabilities in this series:

#### 2.2.1 Application Integration (IBM App Connect):

IBM App connect provides a low-code/no code integration capability with a large number of pre-built connectors to connect to a variety of different endpoints.

We will use this to create the API contract with a simple model and then the integration flow API Implementation that is started with our API call and which contains all the integration logic and data transformation.

#### 2.2.2 Smart Connectors

These connectors contain everything needed to connect to the systems and endpoints – we just need to give them the endpoint location and credentials.

We will use one connector each for

- SalesForce
- Watson Visual Recognition
- Watson Tone Analysis.

 For the extended version:

- Watson Language Translation
- ServiceNow

The connectors will take care of things like authentication, session management, retries etc – you just need to give them credentials to connect and they will handle the rest.

#### 2.2.3 API Management (IBM API Connect)

Once we have our integration API built, we need to expose it securely to the outside world via an API gateway.

We also need to be able to create a self-service portal to allow consumers to discover out APIs and sign up to use them.

We also need to create rate plans to limit how many times the API can be called.

We will push our API from the Application integration capability directly into the API Management capability where the API Product and API artefacts we need will be created for us automatically.

We will then add security and rate limiting plans and publish our API to our secure gateway and portal. 

We will use this capability in part2 of our series.



## 3. Getting Started – Setting up the endpoints

Before we can build our API integration, we need to set up the endpoints that we need will integrate to.

We will set up endpoints to connect to:

- IBM Watson Visual Recognition
- Salesforce
- IBM Watson Tone Analysis

Later, for extended scenario, we will connect to the following endpoints.

- IBM Watson Language Translation
- ServiceNow

### 3.1 Setting up IBM Watson Services

We will set up Watson Visual Recogniton and Tone Analysis.

You will need an IBM Cloud account to do this. You can use your existing one if you wish or you can set up a new one. 

IBM cloud access is free and can be provisioned instantly. 

Once you have an account, all of the Watson services have ‘lite’ plans which allow you to use them for free – the only restriction is the number of calls you can make per month. Don’t worry, we won’t be getting anywhere near that number – and you won’t get charged if you hit the limit, it will just stop working until the next month.

#### 3.1.1 Logging in to IBM Cloud

The IBM Cloud can be accessed at https://cloud.ibm.com

<img src="./images/image-20200609174730528.png" alt="image-20200609174730528"/>

If you don’t have an IBM ID (You can use the one you used to register for Think!) then click ‘Create an account’ – all you need is an email, you don’t need a credit card.

When you have an IBM ID, sign in at https://cloud.ibm.com (Depending on your company, e.g. if you’re an IBMer, you may go through a Single Sign-on process).

Once you’re in, you’ll be presented with the cloud dashboard showing which services you have provisioned:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609204306444.png" alt="image-20200609204306444"/>

(You may not see this many services, clusters etc – the lab authors have many things in their IBM Cloud accounts.)

If you already have the IBM Watson services in your account, or you know how to create them then skip to ‘Obtaining your Watson Credentials’

#### 3.1.2 Creating your free Lite-Plan IBM Watson Services:

On the IBM Cloud Dashboard, click `Catalog`. 

You’ll see a list of services (if not, click on `services`).

Check the `AI` filter checkbox on the left to filter for Watson services. (You can also search for them by name)

![image-20200609204553080](./images/image-20200609204553080.png)



Scroll down and click on the `Visual Recognition` tile.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609213928692.png" alt="image-20200609213928692" style="zoom:50%;" />

Inside, you’ll be able create a lite plan (free) instance as shown in the screenshot below (ignore the warning on our screenshot about only being able to have one lite plan per account– that’s because we already had one lite instance set up in the lab authors’ IBM Cloud Account)

![image-20200609214115913](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609214115913.png) 

Select the free `lite` plan and provision the service.

You can change the service name to something more memorable if you wish.

Once you create the service, you’ll be able to see it in your cloud dashboard (to get to the cloud Dashboard, click the `hamburger` menu at the top left of the screen and select `Dashboard`

![image-20200609222149224](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609222149224.png)

 

Look under `Services` and you’ll find your newly created service (you can see a number of services in our screenshot below – we’ve renamed ours to add `Dallas` on the end but yours will have a similar name)     

![image-20200609222405756](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609222405756.png)

Click on your new service and you’ll see the `Manage` tab:

![image-20200609222439728](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609222439728.png)

The API key and URL are what we are going to need to integrate with the service. You can click `Show credentials` and copy/paste them somewhere for later use in this lab or you can click `Download` and they will be downloaded as a text file for you.

You’ll next need to do similar for `Language Translator`. It also has `Lite` plans and are set up in the same place on the IBM Cloud. You may choose any region you want.

Make sure you obtain the URL and API keys (in `credentials`) for all of these Watson services – we’ll be needing them later. You obtain the URL and API keys for both the services in the same way.

![image-20200609223016557](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609223016557.png)



### 3.2 Setting up SalesForce

Salesforce is a CRM system hosted as a SaaS in the cloud.

We will need a **developer** account to use for testing – if you already have a Salesforce developer account, you can use that – if not, you can sign up for a free developer account now.

Go to https://developer.salesforce.com and click on `sign up`

![image-20200609223448761](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609223448761.png)

Not that this is NOT the same as `salesforce.com -> try for free`. **You will need a developer account to use this lab**. You can use a webmail email address to sign up if you wish, rather than your company one.

(we emphasize this a lot but on of the most common reasons for `My integration to Salesforce doesn’t work` is that the account being used is not a developer one).

Once you have a salesforce developer account, log in to check it – you’ll get to something like this:

(Remember to log in at your salesforce developer/instance URL, not just at salesforce.com)

![image-20200609223649581](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200609223649581.png)

You will require admin level access to your Salesforce account.

When you create a free Salesforce account to test, make sure that you create a [Developer account](https://developer.salesforce.com/) rather than a Trial account. If you connect to App Connect with a `Free Trial` account, the Salesforce integrations will not work.

OK, we have our endpoints – we’re ready to integrate!



## 4. Setting up Cloud Pak for Integration instance on IBM Cloud

We will create an instance of Cloud Pak for Integration on IBM Cloud. You can find more about CP4I [here](https://www.ibm.com/cloud/cloud-pak-for-integration1). 

> If you already have a CP4I instance with App Connect and API Connect capabilities added, feel free to use your existing instance. You may skip this section and jump to section 5 (Building your Integration)

Refer to [Provisioning.md](./Provisioning.md) for CP4I set up instructions.



## 5. Building Your Integration

### 5.1 Accessing CP4I

- Login to your IBM Cloud dashboard and click `Schematics workspaces`![image-20200615171009495](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615171009495.png
- Scroll down to the `Schematics workspaces` section and click on the workspace you created

![image-20200615225236641](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615225236641.png)

- Click on the `Offering dashboard` 

![image-20200615225422040](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615225422040.png)

If you see any certificate issue, you can select to proceed to unsafe website/link. You’ll be presented with a login screen to CP4I. Use `admin` as username and the password that you set while following `Provisioning` instructions. Click on `Login`. 

- Click on `Skip Welcome` , if a welcome page is displayed. 



Welcome to ICP4i! You’re now at the home screen showing all the capabilities of the pak, brought together in one place.

We’re going to be using API Connect and App Connect for this exercise.

![image-20200615230600574](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615230600574.png)



Click `View instances` to see the capabilities added in CP4I instance using the option `demoPreparation` while installing CP4I.

![image-20200615231154274](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615231154274.png)



You can see that we have App Connect Designer (the tooling for building integrations), the App Connect Dashboard (this is what manages the integration runtimes) and API Connect (for managing APIs).

(Don’t worry if your instance names are not identical to the screenshots)

At any time, we can use the menu to navigate between these capabilities, as well as using the platform home screen. Use the `hamburger` menu at the top left like so:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615231615577.png" alt="image-20200615231615577" style="zoom:47%;" align="left"/>



> *Occasionally, in the lab environment, you might find that the navigation menu shows ‘0’ instances of the capabilities. Don’t worry, everything is still there!*

*If this happens, click on ‘Platform home’ in the menu, then click ‘View Instances’ tab. After this, the menu will work again!*



### 5.2 Accessing the Designer Integration Tooling

Click on `ace-designer-demo` under `App Connect`. You’ll arrive at the App Connect Designer Dashboard here:

![image-20200615232058774](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615232058774.png)

This is where we can create all of our API integration flows and also manage our connectivity to our services and endpoints. You can create many integration flows and manage them all here.

At the moment, there’s nothing here ye, so let’s build some integration logic.

First, we’re going to connect the designer tooling to our endpoints that we set up earlier.

The Smart Connectors are meta-data driven, so they need to be able to connect whilst we’re using the tooling to ensure that they show us the correct data and functions available from our endpoints.

To connect to our endpoints, we’re going to need the credentials we created earlier.

### 5.3 Connecting the tooling to our endpoints

Let’s go to the connector catalog: click on the cogwheel/sprocket menu and click `Catalog`

![image-20200615232502528](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200615232502528.png)

The connector catalog appears with a list of the cloud pak connectors which are installed locally. There are many more connectors available although not all all of them run ‘locally’. Some of the connectors are currently available in the pak locally, all of them are available on the IBM cloud – you can use the ones that run on the IBM cloud directly from ICP4i designer as well – you just need to link ICP4i to your IBM cloud account, which we won’t be doing in this code pattern.

More connectors are being developed constantly – for a list, look here: https://www.ibm.com/cloud/app-connect/connectors/

You can choose whether you want to run the connectors locally or on the IBM cloud. For this task, we will run them locally:

![image-20200616093457103](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616093457103.png)

Let’s set up our Watson AI endpoints – scroll down until you see the IBM Watson connectors:

![image-20200616093522969](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616093522969.png)



Click on ‘IBM Watson Visual Recognition’. 

You’ll see that the connector expands and shows you the actions available for the connector.

ICP4i connectors are smart connectors and are metadata driven – you don’t need to know what functions and data are in the endpoint – the connectors will usually show them to you.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616093644485.png" alt="image-20200616093644485" style="zoom:50%;" align="left"/>

Click on `Connect`

You may be be asked if you want to run the connector `local` (runs on ICP4i) or on the IBM Cloud. If asked, for this task click `Local` and click `Connect`.

To connect to your Watson Visual Recognition account, you’ll need credentials – otherwise anyone could connect to it. The service is protected by an API key.

You’ll now be asked for the API key that you kept safe from before: Enter it here and click `connect`

Make sure you have the right one – the one for e.g. Tone Analyzer will not work for Visual Recognition.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616094137910.png" alt="image-20200616094137910" style="zoom:50%;" align="left" />

(Hint: you can use the ‘eye’ button to show the API key to check it’s correct)

If you’ve ‘forgotten’ your API key, go back to the service you created in the IBM Cloud – you can view it from there.

If all goes well, (i.e. you’ve entered your key correctly), a connector account will be created for you – that’s it! You’ve added visual recognition capability to your integration

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616095925571.png" alt="image-20200616095925571" style="zoom:50%;" align="left" />

IMPORTANT: DON’T MOVE ON YET! You’ll see `Account 1` as the name of the account.

WE NEED TO RENAME THE ACCOUNT FOR THIS TASK TO WORK SEAMLESSLY (we’ll tell you how to fix it if you don’t later….but it’s easier if you do!)

ICP4i lets you have multiple accounts for connecting to each type of system. For example you could have a DEV account, a TEST account and a PROD account. Or you may have a USA instance and an EU instance. The name is what the integrations use to reference the correct account. You can connect your connectors to as many places as you wish – there’s no extra charge – all connectors are included.

To rename your account, Click the three dots menu and click `rename account`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616100321676.png" alt="image-20200616100321676" style="zoom:50%;" align="left"/>

In the dialog box, name the account `App Connect Trial` (exactly as shown – capitals on the first letter of the words, spaces between the words) and click `Rename Account` as shown below,

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616100427281.png" alt="image-20200616100427281" style="zoom:50%;" align="left"/>

You connector should now look like this:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616100502762.png" alt="image-20200616100502762" style="zoom:50%;" align="left" />

OK, we have our Visual Recognition sorted – let’s do the next two Watson connectors:

Click on `IBM Watson Tone Analyzer` and click `Connect`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616100607006.png" alt="image-20200616100607006" style="zoom:50%;" align="left" />

Select the Local connector (if asked – you may not be) and click Continue

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616100752616.png" alt="image-20200616100752616" style="zoom:50%;" align="left"/>

For this connector, we’ll need the URL and the API key that we got earlier: Enter them in the dialog below – (you won’t need the User name and Password). 

Note: Your URL may be different to our screenshot – it depends in which cloud region your service is running. Click `Connect`

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616100952807.png" alt="image-20200616100952807" style="zoom:50%;" align="left" />

And we’re connected!

IMPORTANT – FOR THE TASK, RENAME THE ACCOUNT to `App Connect Trial`. (use the three dots menu and click `Rename Account` )

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616101114113.png" alt="image-20200616101114113" style="zoom:50%;" align="left"/>

Your connector should look like this:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616101208091.png" alt="image-20200616101208091" style="zoom:50%;" align="left"/>

Finally, let’s connect to the Watson Language Translator – it’s very similar:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616101249851.png" alt="image-20200616101249851" style="zoom:50%;" align="left" />

Click `Connect` and select `Local` for the connector location (if asked) then enter your URL and API key (note your URL may be different from our screenshot depending on the region of your Watson services)

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616103057038.png" alt="image-20200616103057038" style="zoom:50%;" align="left"/>

Click `Connect`

IMPORTANT – FOR THIS CODE PATTERN, RENAME THE ACCOUNT to `App Connect Trial`. (use the three dots menu and click `Rename Account` )

It should look like this:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616103219649.png" alt="image-20200616103219649" style="zoom:50%;" align="left"/>

Why is it important to rename the accounts? We’re going to import an integration flow to save you some typing and clicking. This flow is configured to look for connector accounts named `App Connect Trial`

If you don’t rename your accounts, you’ll need to edit the flow to point to the ones you’ve created and match the names. It’s not hard to do, but it does add extra work.

### 5.4 Setting up the Salesforce Connection

Just one more endpoint to go, then we can look at API flows.

Scroll down to the Salesforce connector. There may be multiple types of salesforce connector shown , pick the first one just called `Salesforce`.

(You may see there are already accounts created – we’ll be creating a new one to connect to your Salesforce account anyway – don’t use the existing accounts – you won’t be able to see where your integrations go..)

Click `Add a new account` if there are existing accounts, or just click `Connect` if this is the first one.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616103600066.png" alt="image-20200616103600066" style="zoom:50%;" align="left" />

Select a Local connector location (if asked – you may not be asked) and click ‘Continue’

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616103813680.png" alt="image-20200616103813680" style="zoom:50%;" align="left"/>

You’ll now be asked for the Salesforce credentials – how do you get these? Follow the steps below. 

Salesforce needs more than just your userid and password – it needs a client Id and Client Secret as well. Also, what you type in the ‘Password’ field in the connector isn’t just your password that you log in with.

The fields we need are shown below

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616103931845.png" alt="image-20200616103931845" style="zoom:50%;" align="left"/>

You will require admin level access to your Salesforce account. 

 When youa free Salesforce account to test, make sure that you created a [Developer account](https://developer.salesforce.com/) rather than a Trial account. If you connect to App Connect with a “Free Trial” account, the Salesforce integrations may not work.

Login to your Salesforce Developer account – you should see the screen like below:

![image-20200616104029898](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616104029898.png)

To get your **loginURL**, click on your user profile. The URL text below your Account Name is your login URL – BUT WITHOUT THE LEADING HTTPS:// .

![image-20200616104103155](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616104103155.png)

 

Insert the **login URL** into the connector account form as shown below:

IMPORTANT: You MUST enter the ‘https://’ part as well – it won’t work if you just copy/paste from the salesforce screen e.g. “um1.salesforce.com” will **not** work. “https://um1.salesforce.com” will!

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616104134388.png" alt="image-20200616104134388" style="zoom:50%;" align="left" />

Next we will need to **retrieve Security Token**. For this click on your user profile and select the Settings option in the profile panel.

![image-20200616104211975](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616104211975.png)

Under Settings, find and click the “Reset Security Token” option

(you may need to go to ‘Switch to lightning experience’ to see this)

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616111047905.png" alt="image-20200616111047905" style="zoom:50%;" align="left"/>(On the top right if you see it)





![image-20200616111152457](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616111152457.png)

Click on Reset Security Token Button and it will send the **newly generated security token to your admin email address**. Use the token for your credentials. 

To populate the Password field on the connector account screen you will need to *concatenate the Password used to log into the Salesforce account with the Security Token received via above step* as shown below:

For example if you Salesforce password is 'myGreatPassword’ and your Salesforce security token is ‘2325jsdhew4312hs534dh’ then you should enter

`myGreatPassword2325jsdhew4312hs534dh` in the ‘password’ field.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616111407934.png" alt="image-20200616111407934" style="zoom:50%;" align="left"/>

Next we will retrieve the **Client ID and Secret**

**Click the ‘setup’ cogwheel at the top right.**

On the left-hand Finder panel go to:
**PLATFORM TOOLS > Apps > App Manager**

![image-20200616111549686](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616111549686.png)

You then want to **create a New Connected App** or use an existing one. Steps for creating a new app are as follows:

![image-20200616111625258](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616111625258.png)

Provide a Connect App Name and an API Name is automatically generated for you. Provide a Contact Email (usually admin email address). Please make sure you Enable OAuth Settings and follow steps below to configure the OAuth setting.

![image-20200616111648866](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616111648866.png)

Click on Enable OAuth Settings to get the configuration panel.

Either click on Enable for Device Flow and that will auto-generate a Callback URL or alternately you can provide your own fully qualified Callback URL

Next step is to configure the scope of access for our connectors which will be the Connected App in this case.

Connectors technically only require “data api” - you can optionally choose to enable all the scopes for this connected app.

And then click on Save.

![image-20200616112325268](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616112325268.png)

**It may take several minutes for newly created Connected App to be registered**. Once registered go back to App Manager, select and view the created App

![image-20200616112348717](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616112348717.png)

Use **Consumer Key and Secret as Client ID and Client Secret** respectively as needed in the connector account UI as follows:

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616112415309.png" alt="image-20200616112415309" style="zoom:50%;" align="left" />

Click Connect – you should see your account created!

IMPORTANT – After all that, we need to rename our account! Don’t forget to use the three dots an rename our account to `App Connect Trial` as shown below.

<img src="/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616112510301.png" alt="image-20200616112510301" style="zoom:50%;" align="left"/>

Just as an aside, look at the sheer amount of data and functions available through the connector – you can expand them to see what the actions are. As the connectors are metadata driven, if you customize Salesforce with extra or customized fields, the connectors will pick them up automatically.

Great! We’re now all connected up! Let’s go and see our flow!



### 5.5 Importing the Integration flow into designer

Login to ICP4i dashboard and go to App Connect Designer. Click on the cogwheel (top right) and select `Dashboard` from the menu.

![image-20200616112733785](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616112733785.png)

Click on `New` at the top right and select `Import flow`

![image-20200616112804457](/Users/muralidhar/Murali/Work/Code Patterns/2CodeRepos/2020/build-an-integration-application-using-ibm-cloud-pak-for-integration/images/image-20200616112804457.png)

We have a flow to use already built in github repo – we’re going to import it to save you typing and clicking!

It also avoids a LOT of screenshots and ‘click here, click there, type this instructions’ – you could even probably work out how the flow works just from watching the video [here](https://www.youtube.com/watch?v=TRzO26kawu4) but we’ll step you through it in this exercise.

There is a lot of detailed designer flow documentation for when you want to delve deeper – a good place to start is [https://ibm.biz/learnappconnect](https://ibm.biz/learnappconnect-)

