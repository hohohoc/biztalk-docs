---
title: "Execute stored procedures having a FOR XML clause in SQL Server using BizTalk Server | Microsoft Docs"
ms.custom: ""
ms.date: "06/08/2017"
ms.prod: "biztalk-server"
ms.reviewer: ""
ms.service: "biztalk-server"
ms.suite: ""
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 1d8fe927-90bf-48fc-a418-63b920b409ed
caps.latest.revision: 13
author: "MandiOhlinger"
ms.author: "mandia"
manager: "anneta"
---
# Execute stored procedures having a FOR XML clause in SQL Server using BizTalk Server
An SQL SELECT statement can have a FOR XML clause that returns the query result as XML instead of a rowset. You can also have a stored procedure that has a SELECT statement with a FOR XML clause. [FOR XML (SQL Server)](https://msdn.microsoft.com/library/ms178107.aspx) has more information.
  
 You can use the WCF-based [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)] to execute such stored procedures.  
  
> [!IMPORTANT]
>  The “native” SQL adapter available with [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)] requires stored procedures to have the FOR XML clause as part of the SELECT statement. You can use such stored procedures with the WCF-based [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)] without making any changes to the stored procedure definition.  
>   
>  You can always use the schemas generated using the ‘native’ SQL adapter provided with earlier versions of [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)]. For more information, see [Using FOR XML queries with the WCF-SQL Adapter](http://go.microsoft.com/fwlink/?LinkId=223428) (http://go.microsoft.com/fwlink/?LinkId=223428).  
  
## How to Invoke Stored Procedures with FOR XML Clause  
 When you invoke a stored procedure with FOR XML clause in SQL Server Management Studio or using the SQL adapter available with [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)], the output is in the form of an xml message. To use these procedures with the WCF-based [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)], you must have the schema for the output message. The WCF-based [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)] requires this schema while receiving the response message from SQL Server after executing a stored procedure with FOR XML clause. Note that the request message to invoke this stored procedure will be generated by the adapter itself.  
  
 Apart from having the schema for the response message, you must also perform certain tasks to invoke a stored procedure with FOR XML clause using the WCF-based [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)].  
  
1.  Generate the schema for the response message for the stored procedure with FOR XML clause. If you already have the response schema generated by the “native” SQL adapter available with [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)], you can skip this step.  
  
2.  Create a BizTalk project and add the generated schema to the project.  
  
3.  Generate schema for the stored procedure with FOR XML clause using the WCF-based [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)]. This provides the schema for the request message that the adapter sends to SQL Server to invoke the stored procedure.  
  
4.  Create messages in the BizTalk project to send and receive messages from SQL Server. The request message must conform to the schema of the request message generated by the adapter. The response message must conform to the schema of the response message obtained either using the “native” SQL adapter or by executing the stored procedure with FOR XML clause in SQL Server Management Studio.  
  
5.  Create an orchestration to invoke the stored procedure in the SQL Server database.  
  
6.  Build and deploy the BizTalk project.  
  
7.  Configure the BizTalk application by creating physical send and receive ports.  
  
8.  Start the BizTalk application.  
  
##  <a name="BKMK_RespSchema"></a> Generating Schema for the Response Message for Stored Procedure  
  
> [!NOTE]
>  You do not need to perform this step if you have the response schema generated by the SQL adapter available with [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)].  
  
 You can generate the schema for the response message for the stored procedure, provided the SELECT statement in the stored procedure has the `xmlschema` clause with the `for xml` clause. In this topic, we use the GET_EMP_DETAILS_FOR_XML stored procedure that retrieves the employee details for a given employee ID. To retrieve the schema by executing the stored procedure, the SELECT statement looks like the following:  
  
```  
SELECT [Employee_ID] ,[Name] ,[DOJ] ,[Designation] ,[Job_Description] ,[Photo] ,cast([Rating] as varchar(100)) as Rating ,[Salary] ,[Last_Modified] ,[Status] ,[Address]   
FROM [Adapt_Doc].[dbo].[Employee] for xml auto, xmlschema  
```  
  
 Execute this stored procedure to get the schema for the response message. Note that the response from the stored procedure contains the schema as well as the data from executing the stored procedure. You must copy the schema from the response and save it to a text pad. For this example, you can name this schema as **ResponseSchema.xsd**. You must now create a BizTalk project in [!INCLUDE[btsVStudioNoVersion](../../includes/btsvstudionoversion-md.md)] and add this schema to the project.  
  
> [!IMPORTANT]
>  Make sure you remove the `xmlschema` clause after you have executed the stored procedure to generate the schema. If you fail to do this, when you finally execute the stored procedure through BizTalk, you will again generate the schema in the response message. So, to get the response message as xml you must remove the `xmlschema` clause.  
  
#### To add the schema to a BizTalk project  
  
1.  Create a BizTalk project in [!INCLUDE[btsVStudioNoVersion](../../includes/btsvstudionoversion-md.md)].  
  
2.  Add the response schema you generated for the stored procedure to the BizTalk project. Right-click the BizTalk project in the Solution Explorer, point to **Add**, and then click **Existing Item**. In the Add Existing Item dialog box, navigate to the location where you saved the schema and click **Add**.  
  
3.  Open the schema in [!INCLUDE[btsVStudioNoVersion](../../includes/btsvstudionoversion-md.md)] and make the following changes.  
  
    1.  Add a node to the schema and move the existing root node under this newly added node. Give a name to the root node. For this topic, rename the root node to **Root**.  
  
    2.  The response schema generated for the stored procedure references a sqltypes.xsd. You can get the sqltypes.xsd schema from [http://go.microsoft.com/fwlink/?LinkId=131087](http://go.microsoft.com/fwlink/?LinkId=131087). Add the sqltypes.xsd schema to the BizTalk project.  
  
    3.  In the schema generated for the stored procedure, change the value of `import schemaLocation` to the following.  
  
        ```  
        import schemaLocation=”sqltypes.xsd”  
        ```  
  
         You do this because you have already added the sqltypes.xsd schema to your BizTalk project.  
  
    4.  Provide a target namespace for the schema. Click the **\<Schema>** node, and in the properties pane, specify a namespace in the **Target Namespace** property. For this topic, give the namespace as `http://ForXmlStoredProcs/namespace`.  
  
## Generating Schema for the Request Message to Invoke the Stored Procedure  
 To generate schema for the request message you can use the [!INCLUDE[consumeadapterservshort](../../includes/consumeadapterservshort-md.md)] from a BizTalk project in [!INCLUDE[btsVStudioNoVersion](../../includes/btsvstudionoversion-md.md)]. For this topic, generate the schema for the GET_EMP_DETAILS_FOR_XML stored procedure. For more information about how to generate the schema using [!INCLUDE[consumeadapterservshort](../../includes/consumeadapterservshort-md.md)], see [Retrieving Metadata for SQL Server Operations in Visual Studio using the SQL adapter](../../adapters-and-accelerators/adapter-sql/get-metadata-for-sql-server-operations-in-visual-studio-using-the-sql-adapter.md).  
  
> [!IMPORTANT]
>  You must generate the schema by selecting the procedure only from the **Procedures** node in [!INCLUDE[consumeadapterservshort](../../includes/consumeadapterservshort-md.md)].  
  
## Defining Messages and Message Types  
 The schema that you generated earlier describes the “types” required for the messages in the orchestration. A message is typically a variable, the type for which is defined by the corresponding schema. You must now create messages for the orchestration, and link them to schemas that you generated in the previous step.  
  
#### To create messages and link to schema  
  
1.  Add an orchestration to the BizTalk project. From Solution Explorer, right-click the BizTalk project name, point to **Add**, and then click **New Item**. Type a name for the BizTalk orchestration, and then click **Add**.  
  
2.  Open the Orchestration View window of the BizTalk project, if it is not already open. To do so, click **View**, point to **Other Windows**, and then click **Orchestration View**.  
  
3.  In Orchestration View, right-click **Messages**, and then click **New Message**.  
  
4.  Right-click the newly created message, and then select **Properties Window**.  
  
5.  In the **Properties** pane for the **Message_1**, do the following:  
  
    |Use this|To do this|  
    |--------------|----------------|  
    |Identifier|Type `Request`|  
    |Message Type|From the drop-down list, expand **Schemas**, and then select *ForXMLProcedure.Procedure_dbo.GET_EMP_DETAILS_FOR_XML*, where ForXMLProcedure is the name of your BizTalk project. Procedure_dbo is the schema generated for invoking the GET_EMP_DETAILS_FOR_XML procedure.|  
  
6.  Repeat step 2 to create a new message. In the **Properties** pane for the new message, do the following:  
  
    |Use this|To do this|  
    |--------------|----------------|  
    |Identifier|Type `Response`|  
    |Message Type|From the drop-down list, expand **Schemas**, and then select *ForXMLProcedure.ResponseSchema*, where ResponseSchema is the name of the response schema generated by executing the stored procedure as described under [Generating Schema for the Response Message for Stored Procedure](#BKMK_RespSchema).|  
  
## Setting up the Orchestration  
 You must create a BizTalk orchestration to use [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)] for executing a stored procedure in SQL Server. In this orchestration, you drop a request message at a defined receive location. The [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)] consumes this message and passes it on to SQL Server. The response from SQL Server is saved to another location. You must include Send and Receive shapes to send messages to SQL Server and receive responses, respectively. A sample orchestration for invoking a procedure resembles the following:  
  
 ![Orchestration to invoke stored procedures](../../adapters-and-accelerators/adapter-sql/media/eac6e8b6-f0f4-44af-8218-03ca3864b267.gif "eac6e8b6-f0f4-44af-8218-03ca3864b267")  
  
### Adding Message Shapes  
 Make sure you specify the following properties for each of the message shapes. The names listed in the Shape column are the names of the message shapes as displayed in the just-mentioned orchestration.  
  
|Shape|Shape Type|Properties|  
|-----------|----------------|----------------|  
|ReceiveMessage|Receive|-   Set **Name** to *ReceiveMessage*<br />-   Set **Activate** to *True*|  
|SendMessage|Send|-   Set **Name** to *SendMessage*|  
|ReceiveResponse|Receive|-   Set **Name** to *ReceiveResponse*<br />-   Set **Activate** to *False*|  
|SendResponse|Send|-   Set **Name** to *SendResponse*|  
  
### Adding Ports  
 Make sure you specify the following properties for each of the logical ports. The names listed in the Port column are the names of the ports as displayed in the orchestration.  
  
|Port|Properties|  
|----------|----------------|  
|MessageIn|-   Set **Identifier** to *MessageIn*<br />-   Set **Type** to *MessageInType*<br />-   Set **Communication Pattern** to *One-Way*<br />-   Set **Communication Direction** to *Receive*|  
|LOBPort|-   Set **Identifier** to *LOBPort*<br />-   Set **Type** to *LOBPortType*<br />-   Set **Communication Pattern** to *Request-Response*<br />-   Set **Communication Direction** to *Send-Receive*|  
|ResponseOut|-   Set **Identifier** to *ResponseOut*<br />-   Set **Type** to *ResponseOutType*<br />-   Set **Communication Pattern** to *One-Way*<br />-   Set **Communication Direction** to *Send*|  
  
### Specify Messages for Action Shapes, and Connect Them to Ports  
 The following table specifies the properties and their values that you should set to specify messages for action shapes and to link the messages to the ports. The names listed in the Shape column are the names of the message shapes as displayed in the orchestration mentioned earlier.  
  
|Shape|Properties|  
|-----------|----------------|  
|ReceiveMessage|-   Set **Message** to *Request*<br />-   Set **Operation** to *MessageIn.FOR_XML.Request*|  
|SendMessage|-   Set **Message** to *Request*<br />-   Set **Operation** to *LOBPort.FOR_XML.Request*|  
|ReceiveResponse|-   Set **Message** to *Response*<br />-   Set **Operation** to *LOBPort.FOR_XML.Response*|  
|SendResponse|-   Set **Message** to *Response*<br />-   Set **Operation** to *ResponseOut.FOR_XML.Request*|  
  
 After you have specified these properties, the message shapes and ports are connected and your orchestration is complete.  
  
 You must now build the BizTalk solution and deploy it to a [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)]. For more information, see [Building and Running Orchestrations](../../core/building-and-running-orchestrations.md).
  
## Configuring the BizTalk Application  
 After you have deployed the BizTalk project, the orchestration you created earlier is listed under the Orchestrations pane in the [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)] Administration console. You must use the [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)] Administration console to configure the application. For a walkthrough, see [Walkthrough: Deploying a Basic BizTalk Application](Walkthrough:%20Deploying%20a%20Basic%20BizTalk%20Application.md).
  
 Configuring an application involves:  
  
-   Selecting a host for the application.  
  
-   Mapping the ports that you created in your orchestration to physical ports in the [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)] Administration console. For this orchestration you must:  
  
    -   Define a location on the hard disk and a corresponding file port where you will drop a request message. The BizTalk orchestration will consume the request message and send it to SQL Server database.  
  
    -   Define a location on the hard disk and a corresponding file port where the BizTalk orchestration will drop the response message containing the response from SQL Server database.  
  
    -   Define a physical WCF-Custom or WCF-SQL send port to send messages to SQL Server database. For instructions on how to create a send port, see [Manually configure a physical port binding to the SQL adapter](../../adapters-and-accelerators/adapter-sql/manually-configure-a-physical-port-binding-to-the-sql-adapter.md).
  
         You must also specify the action in the send port. For procedures that contain the FOR XML clause, you must set the action in the following format.  
  
        ```  
        XmlProcedure/<schema_name>/<procedure_name>  
        ```  
  
         So, for this topic where we are executing the GET_EMP_DETAILS_FOR_XML procedure, the action will be:  
  
        ```  
        XmlProcedure/dbo/GET_EMP_DETAILS_FOR_XML  
        ```  
  
         For more information about setting action, see [Configure the SOAP action for the SQL adapter
](../../adapters-and-accelerators/adapter-sql/configure-the-soap-action-for-the-sql-adapter.md).
  
         You must also set the following binding properties when executing a stored procedure with the FOR XML clause.  
  
        |Binding property name|Set this to|  
        |---------------------------|-----------------|  
        |XmlStoredProcedureRootNodeName|Specify the name of the root node that you added to the response schema you generated for the stored procedure, as described under [Generating Schema for the Response Message for Stored Procedure](#BKMK_RespSchema). For this topic, set this to **Root**.|  
        |XmlStoredProcedureRootNodeNamespace|Specify the target namespace for the response schema you generated for the stored procedure, as described under [Generating Schema for the Response Message for Stored Procedure](#BKMK_RespSchema). For this topic, set this to `http://ForXmlStoredProcs/namespace`.|  
  
         For more information about specifying values for binding properties, see [Configure the binding properties for the SQL adapter](../../adapters-and-accelerators/adapter-sql/configure-the-binding-properties-for-the-sql-adapter.md).  
  
        > [!NOTE]
        >  Generating the schema using the [!INCLUDE[consumeadapterservlong](../../includes/consumeadapterservlong-md.md)] also creates a binding file that contains information about the ports and the actions to be set for those ports. You can import this binding file from the [!INCLUDE[btsBizTalkServerNoVersion](../../includes/btsbiztalkservernoversion-md.md)] Administration console to create send ports (for outbound calls) or receive ports (for inbound calls). For more information, see [Configure a physical port binding using a port binding file to use the SQL adapter](../../adapters-and-accelerators/adapter-sql/configure-a-physical-port-binding-using-a-port-binding-file-to-sql-adapter.md).
  
## Starting the Application  
 You must start the BizTalk application for invoking procedures in SQL Server database. For instructions on starting a BizTalk application, see [How to Start an Orchestration](../../core/how-to-start-an-orchestration.md).
  
 At this stage, make sure:  
  
-   The FILE receive port to receive request messages for the orchestration is running.  
  
-   The FILE send port to receive the response messages from the orchestration is running.  
  
-   The WCF-Custom or WCF-SQL send port to send messages to SQL Server database is running.  
  
-   The BizTalk orchestration for the operation is running.  
  
## Executing the Operation  
 After you run the application, you must drop a request message to the FILE receive location. The schema for the request message must conform to the request schema for the procedure you generated using the [!INCLUDE[consumeadapterservshort](../../includes/consumeadapterservshort-md.md)]. For example, the request message to invoke the GET_EMP_DETAILS_FOR XML is:  
  
```  
<GET_EMP_DETAILS_FOR_XML xmlns="http://schemas.microsoft.com/Sql/2008/05/Procedures/dbo">  
  <emp_id>10765</emp_id>  
</GET_EMP_DETAILS_FOR_XML>  
```  
  
 See [Message Schemas for Procedures and Functions](../../adapters-and-accelerators/adapter-sql/message-schemas-for-procedures-and-functions.md) for more information about the request message schema for invoking procedures in SQL Server database using the [!INCLUDE[adaptersqlshort](../../includes/adaptersqlshort-md.md)].  
  
 The orchestration consumes the message and sends it to SQL Server database. The response from SQL Server database is saved at the other FILE location defined as part of the orchestration. For example, the response from SQL Server database for the preceding request message is:  
  
```  
\<?xml version="1.0" encoding="utf-8"?>  
<Root xmlns="http://ForXmlStoredProcs/namespace">  
  \<Adapt_Doc.dbo.Employee Employee_ID="10765" Name="John" Designation="asdfaf" Salary="3434.00" Last_Modified="AAAAAAAANso=" Status="0" xmlns="" />  
</Root>  
```  
  
 Notice that the response is received in the same schema as generated by executing the stored procedure. Also note that the root node and the namespace is the same you specified as values for **XmlStoredProcedureRootNodeName** and **XmlStoredProcedureRootNodeNamespace** binding properties respectively.  
  
## Best Practices  
 After you have deployed and configured the BizTalk project, you can export configuration settings to an XML file called the binding file. Once you generate a binding file, you can import the configuration settings from the file, so that you do not need to create items such as send ports and receive ports for the same orchestration. For more information about binding files, see [Reuse adapter bindings](../../adapters-and-accelerators/adapter-sql/reuse-sql-adapter-bindings.md).  
  
## See Also  
[Develop BizTalk applications using the SQL adapter](../../adapters-and-accelerators/adapter-sql/develop-biztalk-applications-using-the-sql-adapter.md)