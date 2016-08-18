# Get Started with Tools
This document explains what tools are and shows how to quickly get started developing tools for Phoenix.
## Overview
A tool is a component in Phoenix which can be used to perform some action within an execution workflow. An execution workflow is a collection of tools structured and designed in such a way to accomplish some task.

An example of an execution workflow may be generating and sending out a report by email. This worklflow might consist of the following tools.

* A tool to schedule when the report must be sent out.
* A report generation tool which generates the report and produces HTML or PDF.
* An email tool which sends out an email containing the HTML or attached PDF to a list of recipients.

Another common example of an execution workflow is to extract, transform and write data from one location to another. This could be a workflow that extracts data from an Excel or CSV file and writes it into an MS SQL database. This workflow would consist of the following tools.

* An Excel or CSV file reader tool to read the contents of the Excel or CSV file.
* A Script tool which converts the extracted data into a format compatible for the database.
* A Relational Database tool which writes data to a relational database such as MS SQL, My SQL, Oracle or any relational database supported by Entity Framework.

## Tool Components

Tools consist of several components, both on the back-end and front-end. On the back-end tools consist of the following.

* Tool - the root of the tool which is enumerated and describes all the child components.
* ToolConfig - the tools configuration. This describes things like execution node and execution interval as well as other specific properties required for the tools execution.
* ToolActor - the tool actor performs all the actions or logic for the tool and uses the ToolConfig to create connectors (more about connectors later) as well as connect to servers or data sources.
* ToolService - the tool service provides optional services required to configure a tool of this type. For example, a Relational Database tool may provide a tool service which enumerates all databases on a given server. This allows the user to be able to enter in a IP address, machine name or Url on the tools' configuration form and then seeing a list of databases which they can then select one of.

On the front-end, the components are views and view-models which describe the visual aspects of the tool and provide a user friendly way for the tool to be configured.

* Visual - the view/view-model representing how the tool is displayed on the workspace (workflow diagram).
* Config - the view/view-model for the configuration dialog that is presented to the user when configuring the settings for the tool.

## Connectors
Connectors, as the name suggests, connects one tool to another tool. Connectors can be of various types and can be extended to suit most needs but typically, the data connector will be of most use. Connectors are provided by a tool and need to be updated when a tools' configuration changes. Every tool comes with a standard Execution Connector. This connector allows the flow of execution to be controlled from one tool to another. Generally there is nothing you need to do with this connector but you can add other Execution Actors to perform other actions which don't cause execution to be passed to any connected actors.

## Writing a basic tool
This section will walk you through creating a simple expression evaluating tool, based on the [NCalc](https://ncalc.codeplex.com/) library.

### 1. Setup Project
Create a new class library project. The name of this project can be the name of your tool. For example, "Expression".

Install the Phoenix.Execution package from NuGet.

`PM> Install-Package Phoenix.Execution -Version 0.0.1-beta`

### 2. Install NCalc
At this point you should install any necessary dependencies for the tool. In this case we need NCalc so install it from NuGet.

`PM> Install-Package NCalc`

### 3. Add the tool class
The tool class is a class that defines a tool. It does nothing more than to provide info about the tool. Add a class called ExpressTool to the project and derive it from `Phoenix.Exceution.Tool`. Then implement the abstract members.

```c#
public class ExpressionTool : Tool
{
    public override string Category
    {
        get
        {
            return "General";
        }
    }

    public override string Description
    {
        get
        {
            return "Evaluates an expression";
        }
    }

    public override string Icon
    {
        get
        {
            return "";
        }
    }

    public override string Key
    {
        get
        {
            return "general/expression";
        }
    }

    public override string Name
    {
        get
        {
            return "Expression";
        }
    }

    public override Type ServiceActorType
    {
        get
        {
            return null;
        }
    }

    public override IEnumerable<string> Tags
    {
        get
        {
            return new[] {
                "general",
                "math",
                "expression",
                "script"
            };
        }
    }

    public override Type ToolActorType
    {
        get
        {
            return typeof(ExpressionActor);
        }
    }

    public override Type ToolConfigType
    {
        get
        {
            return typeof(ExpressionConfig);
        }
    }

    public override string Version
    {
        get
        {
            return "0.0.1";
        }
    }
}
```

### 4. Add the tool config class
The tool config class allows us to setup a model for the configuration. This is where any properties required to execute the tool must go. Add a class called ExpressionConfig and derive it from `Phoenix.Execution.ToolConfig`. We will add a property called Expression to this class.

```c#
public class ExpressionConfig : ToolConfig
{
    /// <summary>
    /// The epxression string
    /// </summary>
    public string Expression { get; set; }

    /// <summary>
    /// Parameters used for input connectors
    /// </summary>
    public List<string> Parameters { get; set; }

    public ExpressionConfig()
    {
        Parameters = new List<string>();
    }
}
```

### 5. Add the tool actor class
There are two primary method to implement in the tool actor class these are,

* Update - called when the tool configuration is updated. This is where connectors are added and anything other initialization occurs.
* Execute - called when the tool is executed. This is where the execution logic is performed and results are sent on wards.

The implementation for the tool actor is as follows.

```c#
public class ExpressionActor : ExecutionActor
{
    private NCalc.Expression expression;

    public ExpressionActor(PersistenceConfig persistenceConfig) : base(persistenceConfig)
    {
    }

    public ExpressionConfig ExpressionConfig
    {
        get
        {
            return this.Config as ExpressionConfig;
        }
    }

    protected override Tool Tool
    {
        get
        {
            return new ExpressionTool();
        }
    }

    protected override void Update()
    {
        //Create an instance of the NCalc expression.
        expression = new NCalc.Expression(ExpressionConfig.Expression);

        //The expression will parse any parameters bases on the NCalc specification.
        //These are added as connectors
        foreach (var parameter in expression.Parameters.Keys)
        {
            AddConnector(new DataConnector<object>(parameter));
        }

        //Add a connector for the result
        AddConnector(new DataConnector<object>("Result"));
    }

    protected override void Execute()
    {
        //Set each NCalc expression parameter using the current value.
        //Use GetCurrentValue to retrieve the value at the time of execution
        foreach (var parameter in expression.Parameters.Keys)
        {
            expression.Parameters[parameter] = GetCurrentValue(parameter);
        }

        //Evaluate the expression
        var result = expression.Evaluate();

        //Get the result connector and send the value to any connected tools.
        var resultConnector = GetConnector("Result") as DataConnector;
        resultConnector.Send(result);
    }
}
```

### 6. Add the tool service class
For this tool we don't need a service so it is left out. Here is an example of what a tool service might look like.

```c#
public ExpressionService : ToolService
{
}
```

### Finishing up
To deploy the tool it will need to be copied into the `Modules` folder of Node.Phoenix.
