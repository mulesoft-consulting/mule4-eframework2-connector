# Mule4 eFramework2 Extension

The Mule 4 eFramework2 extension provides an easy way to invoke standardized workflows that process different event type.  The flows could be invoked directly by the Mule application, but by using the eFramework extension, the invocation of the event is separated from the implementation of the event logic. This makes it easier to use different standardized workflow logic in different Mule applications, while using a standard coding method for indicating an event has occurred.

This is the replacement for the deprecated mule4-eframework-connector. See a summary of the differences at the end of this README.

There is an EventFramework.docx containing more detailed information. 

## transactionProperties ##
eFramework is intended to work with a logging framework (such as Minimal Logging) 
where a flow variable (for instance a Map<String,String> transactionProperties) is used to hold the values that will be included in log messages. 

As such, eFramework will pass a transactionProperties map to the workflow implementations it invokes. The Minimal Logging min-log:new, min-log:new-job, min-log:new-record, min-log:put and min-log:putAll operations can all be used to add properties to the transactionProperties that the eFramework uses. Here are examples:

To generate a unique x-transaction-id when accepting an API request use:

```
<min-log:new doc:name="Set transaction properties" target="transactionProperties" headers="#[attributes.headers]" />
```

To generate a unique x-job-id at the start of a batch job use:

```
<min-log:new-job target="transactionProperties" transactionProperties="#[vars.transactionProperties]" doc:name="new-job" />
```

To generate a unique x-record-id at the start of processing a batch record use:
```
<min-log:new-record target="transactionProperties"
			transactionProperties="#[vars.transactionProperties]" doc:name="new-record" />
```

Once transactionProperties is loaded with values, it can be used with the eFramework operations. Here are some examples:

To generate a progress event:

```
<eframework2:progress doc:name="Progress" doc:id="038f69f3-d925-4306-a02b-d3695017198c" config-ref="Event_Framework_2_Config" stage="READ" detailText='#["pulled record " ++ payload ++ " from batch"]' recordDescriptor="#[payload]">
	<eframework2:payload ><![CDATA[#[{}]]]></eframework2:payload>
</eframework2:progress>
```

To generate a business event:

```
<eframework2:business-event doc:name="Business event" doc:id="513316df-94b3-4906-9d01-4b9da9aee6f8" config-ref="Event_Framework_2_Config" eventType="Data Error" eventStatus="REJECTED" eventMsg="The request did not pass validation step">
	<eframework2:payload ><![CDATA[#[{}]]]></eframework2:payload>
</eframework2:business-event>
```

To generate a system event:

```
<eframework2:system-event doc:name="System event" doc:id="fdfecf5b-9471-4a1e-a8de-f265ea28c1f7" config-ref="Event_Framework_2_Config" eventType="Dependent Service Down" eventStatus="STATUS CODE 503" eventMsg="Dependent service is unavailable">
	<eframework2:payload ><![CDATA[#[{}]]]></eframework2:payload>
</eframework2:system-event>
```
## Sample workflow for Progress Event ##

The following workflow is invoked by an eFramework Progress operation. It is using the Mule Logger component to print a log message. Note that the transactionMsg for the progress event includes the transaction properties. The progress event is the only operation that does this automatically:

```
<flow name="eframework.progressFlow">
		<logger level="INFO" doc:name="Logger" doc:id="38dd6cdd-e633-4696-9297-817de13f4249" message='#["eframework.progressFlow: " ++ vars.transactionMsg]'/>
transactionProperties="#[vars.transactionProperties]"/>
</flow>
```

## Sample workflow for Business Event ##

The following workflow is invoked by an eFramework Busines Event operation. It is using Minimal Logging to print a log message:

```
	<flow name="eframework.businessEventFlow">
		<min-log:info doc:name="Info" msg='#["eframework.businessEventFlow: " ++ vars.eventMsg]' transactionProperties="#[vars.transactionProperties]"/>
	</flow>
```

## Sample workflow for System Event ##

The following workflow is invoked by an eFramework System Event operation. It is using the standard Mule Logger component to print a log message:

```
	<flow name="eframework.systemEventFlow">
		<logger level="INFO" doc:name="Logger" message="#[&quot;eframework.systemEventFlow: &quot; ++ vars.eventMsg]" />
	</flow>
```

## Configuring the Mule Application ##

Add this dependency to your application's pom.xml

```
		<dependency>
			<groupId>${my-organization-anypoint-orgid}</groupId>
			<artifactId>mule4-eframework2</artifactId>
			<version>2.0.0</version>
			<classifier>mule-plugin</classifier>
		</dependency>
```
## Example Event Handler XML

An example of setting up logging only event handlers is provided in the file **eframework.logging-only.xml**

# Changes from eFramework version 1.x.x

* All parameters are passed to the event handler as flow variables.
* The attributes from version 1.x.x have the same names, but are vars instead of attributes.
* Generally you can use #[vars.transactionProperties] wherever you used to use #[attributes].
* There are many more parameters provided in the eFramework version 2. This is intended to provide more flexibility in passing event configuration (such as email and ticket bodies) to the event handler. Use a debug breakpoint in the event handler to see all the new values available to an event handler.
* Except for the payload logging operations, the payload must be explicitly configured in order to be passed to the event handler. This helps control the amount of memory that is "accidentally" consumed in eFramework version 1.x.x.
* The payload is passed in #[vars.payload], #[payload] should always be null.
* The XML namespace is eframework2.
* All the circuit breaker operations have been removed.
* More configuration properties are specified in the connector configuration. These include email contacts for each of the event operations.
* The connector configuration properties are accessed by the event handlers in the #[vars].
* Location data is not available for the calling location of the event operation.

