# Dynamics 365 FastTrack FSCM Telemetry Samples - TelemetryExtensionExample

This folder contains sample code and guidelines for implementing telemetry in Dynamics 365 Finance and Supply Chain Management (FSCM). The samples are intended to help customers utilize telemetry to monitor and optimize their D365 applications. The code is provided As-is to customer usage. No gaurentees are provided

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
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

1. Clone the repository:

    ```bash
    git clone https://github.com/microsoft/Dynamics-365-FastTrack-FSCM-Telemetry-Samples.git
    cd Dynamics-365-FastTrack-FSCM-Telemetry-Samples\SampleXppExtensions\Projects\FastTrackFscmTelemetrySamples
    ```

2. Open the solution in Visual Studio in adminstrator mode and open FastTrackFscmTelemetrySamples.sln:

    Create the classes, enums, tables for the code under "\TelemetryExtensionExample\bin"
    The project structure will look like 
     ![My project structure](ScreenShots/ProjectStructure.png)

3. Once you create all the dependencies of the project, build the project. Once the Project is successfully build, Synchronize the database as we are adding new tables and then refresh AOT
![Refresh AOT](ScreenShots/SynchronizeDatabase.png)
![Refresh AOT](ScreenShots/RefreshAOT.png)

if all the classes are built fine, you should see the Cost control configuration ('My Telemetry Log Settings' feel free to rename it as appropriate in your project)in Monitoring and Telemetry settings in your D365 F&O
![Cost control settings](ScreenShots/TelemetryLogSettings.png)

4. If you see build errors indicating you need to rebuild other components in the model. proceed to build the model from 'Extensions-> D365 -> Build model'
![Build Model Option](ScreenShots/BuildModel.png)
![Build Relevant Models](ScreenShots/BuildRelevantModel.png)

## Usage

1. `MyTelemetryBase` class is the base class which provides structured access to SysApplicationInsightsTelemetryLogger. In this class you can aggregare Properties as your product code is executing and call the ProcessEvent, ProcessTrace, ProcessException etc methods at appropriate points to send those events to ApplicationInsights

    ![MyTelemetryBase Class](ScreenShots/MyTelemetryBase.png)

2. For each different event/metric/Trace you want to log to AppInsights create a concrete class extending from MyTelemetryBase. Here's an example
![My copncrete telemetry Class](ScreenShots/ConcreteTelemetryClass.png)

3. Use the telemetry class to log telemetry.
 ![My copncrete telemetry Class](ScreenShots/LoggingTelemetry.png)

4. Monitor the telemetry data in Azure Application Insights. Depending on whether you have logged an event, trace or exception etc, your data will be available in that respective table

    ![Azure Application Insights](ScreenShots/ApplicationInsights.png)

5. As you implement telemetry classes and log data into AppInsights, you can control the which events and which severity levels are logged in AppInsights by 2 different ways.
a> You can disable particular kind of event sall together in the Extension UI
    ![How to disable particular events](ScreenShots/HowToDisableParticularTelemetry.png)

b> within the event class itself you can set a particular condition for the event to be logged. For example, in this class I have set only severity level above Information to be logged
![fine grained control within the telemetry class](ScreenShots/FineGrainedControlwithinTheTelemetryClass.png)

## Contributing

We welcome contributions to this project. To contribute:

1. Fork the repository.
2. Create a new branch for your feature or bugfix.
3. Make your changes and commit them.
4. Push your changes to your fork.
5. Open a pull request to the main repository.

## License

This project is licensed under the MIT License. See the [LICENSE](../LICENSE) file for more details.

