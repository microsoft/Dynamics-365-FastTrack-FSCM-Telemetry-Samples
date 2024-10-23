# D365-FO-MonitoringAndTelemetry-Samples
This repository provides samples for utilizing D365 F&amp;O Monitoring and Telemetry feature including batch job monitoring

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configure AppInsights](#Configure-telemetry-logging-in-Finance-&-Operations)
- [Import Data explorer dashboard](#Configure-Azure-Data-Explorer-dashboard)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

## Introduction

This project provides sample implementations of telemetry in D365 FSCM. It includes creation of a telemetry Base class to demonstrate how to utilize SysApplicationInsightsTelemetryLogger to log Events/metrics/traces with properties to ApplicationInsights. It also includes an extension to the MonitoringAndTelemetry module to control the granularity of logging to Application Insights.

## Prerequisites

- Dynamics 365 Finance and Supply Chain Management environment
- Visual Studio 2019 or later
- .NET Framework 4.7.2 or later
- Access to Azure Application Insights

## Installation

1. Close the repository OR download the "MyModel" folder
2. Add the MyModel to your PackagesLocalDirectory folder
3. If you use an UDE environment, you "Configure metadata" and point the folder for Custom metadata to the "XPPCode" folder
   ![image](https://github.com/user-attachments/assets/94915643-3bbd-46a9-8e88-a17842a604d9)

4. If you use a non UDE environment, copy the "XPPCode/MyModel" into the PackagesLocalDirectory folder of your installation
5. Refresh the models in VS
   ![image](https://github.com/user-attachments/assets/c63090fe-8240-4103-adc0-977be350ff55)
6. Build the model "MyModel"
7. Enable the feature "Monitoring and Telemetry" in "Feature management workspace"
   ![image](https://github.com/user-attachments/assets/9e9e52c7-8238-42c6-93ce-88aff7c91bd7)
9. Once the changes are all built, open the menu "Monitoring and telemetry parameters" 
![image](https://github.com/user-attachments/assets/bd9cd393-54e2-4228-90e6-49fbf9d72c5d)
10. You should see a custom tab called "Logging settings"
    ![image](https://github.com/user-attachments/assets/d9d857d5-775a-4d3b-9a51-53eccf6ef715)


## Configure telemetry logging in Finance & Operations

1. Open the menu "Monitoring and telemetry parameters" 
![image](https://github.com/user-attachments/assets/bd9cd393-54e2-4228-90e6-49fbf9d72c5d)
2. Go to "Application Insights Registry"
   ![image](https://github.com/user-attachments/assets/0e069026-2cef-4785-9087-dcd2434fbde6)
   And add the "Instrumentation key" from your Azure Application Insights resource into the field "Instrumentation key" 
   ![image](https://github.com/user-attachments/assets/d85706ae-61a5-44bc-93f0-c0be982f0b31)

3. Move to tab "Environments" 
  ![image](https://github.com/user-attachments/assets/d395adb4-563d-4399-85a6-d3c0b955a43e)
  Add your "Environment Id" into the field, you will find the Environment Id as follows:

  **Youre using LCS:**
  Go to the LCS Environment overview page, copy field "Environment Id" of the current environment.
  ![image](https://github.com/user-attachments/assets/d1e0b89d-2963-4f84-b2f0-6cbae2ffe2c1)

  **Youre using an UDE Environment:**
  Copy the "Environment Id" from the Power platform admin center
  ![image](https://github.com/user-attachments/assets/7e0b0f1b-35ad-452e-a934-ff617d1799fa)

4. Move to tab "Configure" and enable the events you want to log
   ![image](https://github.com/user-attachments/assets/4aebae7f-a7b4-4c64-b448-af5160cc4180)


The telemetry should be emitting after the setup is complete. To test, simply schedule a batch job. 

## Configure Azure Data Explorer dashboard

After or before importing the dashboard, the datasources can be replaced with your Azure insights resource. 
You can replace it simply in an editor in the downloaded JSON file or you import it and do it step by step manually. 

The datasource needs to be formatted like this:

```bash
cluster("https://ade.applicationinsights.io/subscriptions/<Your Subscrition ID>/resourceGroups/<Your resourcegroup>/providers/microsoft.insights/components/<AppInsights name>").database("<AppInsights name>")
```
You can simply also open the AppInsights resource in Azure portal, and copy parts of the URL on top:

![image](https://github.com/user-attachments/assets/0f6338cc-b827-4cd0-858e-89940a3d2651)


1. Download the dashboard you want from the repo folder "Dashboards"
2. Open Azure Data Explorer (https://dataexplorer.azure.com/dashboards/)
4. Go to "New Dashboard" -> New Dashboard from File
   ![image](https://github.com/user-attachments/assets/514a8c0c-7364-45fd-a948-71a4f3f22e51)
5. Open the dashboard and go to "Edit" mode
   ![image](https://github.com/user-attachments/assets/bd1a9d3e-93c3-4a3a-afc5-1b664fa197e3)
6. Now you need to replace all datasources and point it to your AppInsights resource with above string if you havent done it yet.
7. Replace in datasource:
   ![image](https://github.com/user-attachments/assets/7564d7f9-8885-467f-88a3-3292654ec602)
8. Replace in parameters: (Not required if not set anywhere) 
   ![image](https://github.com/user-attachments/assets/8bf1d7f3-6190-4eb0-882e-9879585aa531)
9. Replace in all tiles / queries (Not required if not set anywhere) 
   ![image](https://github.com/user-attachments/assets/e326d9dd-ed3f-4ce5-bb5a-a892b8e0fd60)

## Usage 

With the base classes you have the ability to log telemetry almost in any circumstance where the standard code allows extensions. You can build your own telemetry for multiple purposes including: 

- Performance tracking, e.g. of key business processes to measure KPIs and report on them
- Tracing of processes to troubleshoot in test instances
- Build in telemetry by default on any new customization to measure performance

### Practical example 

The "MyTelemetryGeneric" class is the fully implemented version of telemetry tracking, lets take that one as an example:

```
/// <summary>
/// Generic telemetry class for My model
/// </summary>
class MyTelemetryGeneric extends MyTelemetryBase

{
    public void new()
    {
        super();
        
        this.eventId = MyTelemetryEventIds::MyEvent;
    }

    // if the log level is set to information it will log the request and response data as wekk as there is an error
    protected boolean shouldLogEvent()
    {
        return true;
    }

    protected void postPopulateProperties()
    {
    }

}
```

Whenever you declare the telemetry class, it will automatically start a stopwatch and will capture the time when you do "processEvent". 

We want to measure how long it takes to run a "Customer account statement" 
![image](https://github.com/user-attachments/assets/0de1220b-d678-4ad4-a323-6c71cfbe892d)

It has multiple parameters of some which we want to also emit to telmetry. 

1. Extend the class "CustAccountStatementExtController"
2. Declare the telemetry class
3. Declare new event names however required
4. process the event

Code for the extension

```
[ExtensionOf(classStr(CustAccountStatementExtController))]
internal final class MyCustAccountStatementExtController_Extension
{
    protected void runPrintMgmt()
    {

        MyTelemetryGeneric telemetry = new MyTelemetryGeneric(); //This will start the stopwatch

        next runPrintMgmt();

        telemetry.processEvent(MyTelemetryEventNames::CustomerStatement);
    }

}

```

Code for the Eventname:
```
/// <summary>
/// Defines Telemetry event names logged into AppInsights
/// </summary>
public final class MyTelemetryEventNames
{
    public static const str MyEventName                         = 'MyEventName';
    public static const str BatchJobStatusUpdate                = 'BatchJobStatusUpdate';
    public static const str BatchJobCreated                     = 'BatchJobCreated';
    public static const str BatchStatusUpdate                   = 'BatchStatusUpdate';
    public static const str CustomerStatement                = 'CustomerStatement';

}
```

The eventname will then be displayed as "name" in AppInsights and is your identifier. 
The telemetry class you declare can be your own created one, or the generic one, its up to you and depends always on the process you need to execute. Sometime you need to still fetch data to further process. 


If we want to add additional properties e.g. from the previous screenshot of the user selection, we can add them with "addRuntimeProperty" 

1. Adding a new line for "CustAccount" in "MyTelemetryProperties"
```
public static const str CustAccount 		        = 'CustAccount';
```
2. Changing the extension to add a runtimeproperty
```
[ExtensionOf(classStr(CustAccountStatementExtController))]
internal final class MyCustAccountStatementExtController_Extension
{
    protected void runPrintMgmt()
    {

        MyTelemetryGeneric telemetry = new MyTelemetryGeneric(); //This will start the stopwatch

        CustAccountStatementExtContract contract = this.parmContract() as CustAccountStatementExtContract;
        
        telemetry.addRuntimeProperty(MyTelemetryProperties::CustAccount, contract.parmCustAccount());
        
        next runPrintMgmt();

        telemetry.processEvent(MyTelemetryEventNames::CustomerStatement);
    }

}
```   

Deploy your model for UDE or build it on a LCS environment, then run the report. 

Once you run the report you will see the telemetry like this:
![image](https://github.com/user-attachments/assets/bab78d66-a2c2-4dd5-bd31-1019c971d734)


### Adding callStacks

After declaring the class you can use:

```
telemetry.addCallStack();
```

To add the current callStack to the code, this can be used when you log errors, but be aware, adding a callstack does have performance overhead, do not use it frequently 

### Base properties vs Runtime properties 

When using telemetry, you can use base properties which you can declare in your telemetry class. The base property reflects something in the process you always want to track, e.g. Site & Warehouse or a order identifier.
This can be used if you declare the telemetry on a class level and you process the telemetry on multiple stages across your code, then you dont't want to declare over and over the same properties.

This could be the case for the customer statement as example above, instead of adding it manually, you can create a own telemetry class for it, where you enricht additional data and then add it as base property. 

```
internal final class MyTelemetryCustStatement extends MyTelemetryBase
{
    public void new(CustAccount _custaccount)
    {
        super();

        this.addBaseProperty(MyTelemetryProperties::CustAccount,_custaccount);
        
    }

    // if the log level is set to information it will log the request and response data as wekk as there is an error
    protected boolean shouldLogEvent()
    {
        return true;
    }

    protected void postPopulateProperties()
    {
    }
}
```

Now you can use this in the controller, declare on class level. 

```
[ExtensionOf(classStr(CustAccountStatementExtController))]
internal final class MyCustAccountStatementExtController_Extension
{
    public MyTelemetryCustStatement telemetry;
    protected void runPrintMgmt()
    {
        
        CustAccountStatementExtContract contract = this.parmContract() as CustAccountStatementExtContract;
        
        telemetry = new MyTelemetryCustStatement(contract.parmCustAccount());

        next runPrintMgmt();

        telemetry.processEvent(MyTelemetryEventNames::CustomerStatement);
    }

    protected void populateReportSettingsByCustomer(CustTable _custTable)
    {
        
        next populateReportSettingsByCustomer(_custTable);

        telemetry.processEvent('populateReportSettingsByCustomer');
    
    }

}
```

Runtime properties are added on the fly, means you have something in a sub method or sub class you want to enrich, then you can add it in the middle of the process and process the telemetry later which will then contain the data in AppInsights. 

If you are in a loop, you can also clear the runtime properties if you need a reset but keep the base properties:

```
telemetry.clearRuntimeProperties();
```

NOTE: 
When you use "addRuntimeProperty" and use the same Key as a base property or a runtime property it will overwrite and update the values 


### Post populate properties

the method "postPopulateProperties" is beeing called when you do "processEvent" / "processMetric" / "processTrace". 
You can enrich data at the point if required. 

### Conditional event logging

With the overload of the method "shouldLogEvent" you can decide yourself, based on condition if you want to log data or not, this could be a local variable you declared and want to use it to determine to log or not, or it could be something completely for your needs. 



   
