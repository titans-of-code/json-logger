# JSON Logger

Drop-in replacement for default Mule Logger that outputs a JSON structure based on a predefined JSON schema.  This is a custom processor for Mule created with the [Mule SDK](https://docs.mulesoft.com/mule-sdk/1.1/getting-started).

This supports a comprehensive Mule logging strategy as briefly described in **Logging Patterns** section below.

This component is described in JSON Logger blog [post 1](https://blogs.mulesoft.com/dev/anypoint-platform-dev/json-logging-in-mule-4-getting-the-most-out-of-your-logs/) and [post 2](https://blogs.mulesoft.com/dev/api-dev/json-logging-in-mule-4/).
Original code is available at [JSON Logger Git project](https://github.com/mulesoft-consulting/json-logger).

## Features

- Provides JSON key/value format to log data which is optimized for logging analytics software, such as Splunk and Elastic Stack.
- Automatically provides message tracking data, including Message ID, Correlation ID, time elapsed in flow.
- Provides event tracking for each log entry such as Start, End, Before Connector, After Connector, etc.
- Allows fields to be disabled by properties in addition to the logger being disabled via log level.
- Once configured, requires about the same effort to use as the built-in Logger.
- The `content` field that prints payload and attributes, can be nested JSON or a JSON-escaped string.  Defaults to nested JSON.
- The `content` field can be data masked.

# Installation

This is a custom Mule module but installed like any other connector, it is installed by adding the dependency to the pom or from "Search Exchange" button in studio. Once installed, it will show up in the Anypoint Studio palette.

# Usage

## CorrelationId

The message correlation ID is send between APIs via the `x-correlation-id` HTTP header.  The logger will automatically print the message correlation ID.  This ID is automatically managed and passed around in Mule 4.  

## Logging Patterns

The usage patterns for this logger are listed below.  Note that you may use multiple loggers in a row in your Mule flow if you are implementing multiple patterns at the same point.  For example, a Debug logger set to DEBUG that logs the payload may follow an Event Logger set to INFO that logs a minimal message such as After Connector.

- Error Reporting
- Event Reporting
- Debugging

### Debugging

Add loggers set to DEBUG wherever you need debug information for development and troubleshooting.  These are in addition to the Error and Event loggers described below.  If you are logging a large dataset, often the payload, for troubleshooting purposes, then you need to use a debug logger.

### Error Reporting

Add loggers in error scenarios set to ERROR, so errors are properly logged.  These can also be used to create alerts for notification in logging analytics software.  Error loggers typically appear in exception strategies but can be anywhere errors occur, including validation failures.

Every error flow MUST have an error logger so the error is reported properly.  Generally an error flow has one error logger, multiple event loggers, and optionally, debug loggers as needed.

### Event Reporting

Add loggers to every flow set to INFO that track the import events as the message is processed.  Minimally, event loggers need to be added for the events in a flow as listed below.  Additional important business events in the flow can also be logged to INFO.  

- Flow Start
- Flow End
- Before Connector (external endpoint)
- After Connector (external endpoint)

Note that event loggers contain a MINIMAL message, just enough to provide relevant information for operators to track the specific event that is not covered by the automatic fields, such as a policy number and customer name.  They can also have empty messages.  These are NOT for debugging payloads.  Significant amount of data, including payloads, should be logged with the debug logger instead since those will not be running in Production.  Sensitive data should not be logged.

## Using in App

### Configuration

The configuration below must be added specifically to each application that uses the JSON logger.  The Mule message will be checked for the message and correlation IDs.  The field will be null if none of them have a value.  This is NPE-safe.

**Global Config File**

```
<json-logger:config
	name="json-logger-config"
	doc:name="Json Logger Config"
	disabledFields="${json.logger.disabledFields}"
	environment="${mule.env}" />
```

**Properties File**
The app name (artifactId) and app version are replaced at build time with the [Maven Resource plugin](https://maven.apache.org/plugins/maven-resources-plugin/).

```
json:
  logger:
    application:
      name: "${project.artifactId}"
      version: "${project.version}"
    disabledFields: "content"
```

### Maven Dependency

Use the dependency below to include the JSON Logger in an application's maven build.  
The groupId value must be the appropriate Anypoint Org Id where the JSON Logger is deployed.  This is showing a maven global property, anypoint-org-id, with that value.

```
<dependency>
    <groupId>${anypoint-org-id}</groupId>
    <artifactId>json-logger-module</artifactId>
    <version>${json-logger.version}</version>
</dependency>
```

## Logging Payload
The  `content` field is a conditional field for logging larger content like the payload in development environments.  This field is controlled by the `json.logger.disabledFields` system property and should never be enabled in production environments.

**Important Considerations**
- Logging large content like the payload, especially multiple times in a flow, can significantly decrease performance.
- Minimize logging the payload.  Only log for troubleshooting where necessary, such as around connectors or specific branching.
- Don't log the payload when non-repeatable streams are used.

### Helper Functions
The JSON Logger comes with the dataweave functions below which are recommended when logging content like the payload.  They help stringify data for logging.

- `stringifyNonJSON`: Stringifies known data types, except for JSON.  This is recommended function to use in most cases.
- `stringifyAny`: Stringifies known data types.
- `stringifyNonJSONWithMetadata`: Stringifies known data types, except for JSON, and lists content length, data type, and class.
- `stringifyAnyWithMetadata`: Stringifies known data types and lists content length, data type, and class.

**Example**

```
import * from modules::JSONLoggerModule
output application/json
---
{
	payload: stringifyNonJSON(payload)
}
```

# Building

When building the JSON logger, the required dependencies are provided by the standard MuleSoft maven repositories. Standard maven build commands work without any additional parameters required.  Use `build.sh` as described under **Deploying** below to build the JSON logger.

# Deploying

The `build.sh` script executes maven commands on the module, including deploying to Exchange.  For deploying to Anypoint Exchange or other binary repositorities, server credentials must be in your maven settings.xml file, and repository and server properly linked in pom.xml.

It takes the parameters below.

	1. Anypoint Org Id: the Anypoint business organization where to deploy the module.
	2. Build option: the type of build to execute: `local`, `deploy`.
		- `local`: builds locally (installs dependencies and module in local maven repository)
		- `deploy`: deploys module to Exchange
	
**Syntax**
```
./build.sh {Anypoint Org ID} {build option}
```

**Example**
```
./build.sh 3c07e201-c97b-4665-9310-e3ac89ce1c28 deploy
```

# Dependencies

Depends on the `jsonschema2pojo-mule-annotations` jar, which is a custom extension to the `jsonschema2pojo` plugin for the SDK.  This is provided as a maven dependency that is publicly  available.
