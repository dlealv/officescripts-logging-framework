# Office Scripts Logging Framework – User Guide

A lightweight, extensible logging framework for [Office Scripts](https://learn.microsoft.com/en-us/office/dev/scripts/) (ExcelScript), inspired by frameworks like Log4j.  
Add robust, structured logs to your Excel automations with control over log levels, appenders, and error handling.
This framework ensures compatibility with Office Scripts and Node.js/TypeScript.

---

## Features

- **Multiple Log Levels:** `ERROR`, `WARN`, `INFO`, `TRACE`, plus an `OFF` mode.
- **Configurable Error Handling:** Continue or terminate the script on warnings/errors.
- **Pluggable Appenders:** Output logs to the console or Excel cells, or build your own.
- **Singleton & Lazy Initialization:** The logger and appenders are created only when first needed.
- **TypeScript/Office Scripts Compatible:** Works in both Office Scripts and Node test environments.
- **Configurable Output Layout:** Users can customize the layout of log events sent to the output or use predefined layouts.
- **Log Events Support Extra Fields:** Users can add additional fields to the event, not just the log event message.
- **Support for Log Event Factory:** Extension point in case the user wants to customize how the log events are built.

---

## Getting Started

### 1. Add the Logger to Your Script

**Production use in Office Scripts:**  
Copy the contents of `dist/logger.ts` into your script in the Office Scripts editor.  

> No imports or modules are needed. Just paste the code at the beginning of your `main` function.

### 2. Initialize the Logger (Recommended)

```typescript
// Set verbosity level up to INFO events, and continue on error/warning
let logger = Logger.getInstance(Logger.LEVEL.INFO, Logger.ACTION.CONTINUE)
logger.addAppender(ConsoleAppender.getInstance()) // Add console appender
```
> If you skip this step and just call `logger.info("...")`, the logger will be created with default settings (`WARN` level, `EXIT` action) and a console appender will be used automatically.

---

## Basic Usage Examples

```typescript
logger.info("Script started")          // [2025-06-25 23:41:10,585] [INFO] Script started.
logger.warn("This might be a problem") // [2025-06-25 23:41:10,585] [WARN] This might be a problem.
logger.error("A fatal error occurred") // [2025-06-25 23:41:10,585] [ERROR] A fatal error occurred
logger.trace("Step-by-step details")   // [2025-06-25 23:41:10,585] [TRACE] Step-by-step details
```

> Output as shown above uses the default layout. With the short layout, a timestamp and brackets are excluded.
> The timestamp represents when the log event was created.

### Logging to Excel Cell

Display log output directly in a cell while your script runs:

```typescript
function main(workbook: ExcelScript.Workbook) {
  // Set up logger to send logs to cell B1
  const cell = workbook.getActiveWorksheet().getRange("B1")
  let logger = Logger.getInstance(Logger.LEVEL.INFO, Logger.ACTION.CONTINUE)
  logger.addAppender(ExcelAppender.getInstance(cell))

  logger.info("Log written to Excel!")  // Output in cell B1: [2025-06-25 23:41:10,586] [INFO] Log written to Excel! (green text)
  logger.trace("Trace event in cell")   // Output in cell B1: [2025-06-25 23:41:10,586] [TRACE] Trace event in cell (gray text)
}
```

---

## Lazy Initialization: What You Need to Know

- **Logger Singleton:**  
  - If you call any logging method (e.g., `logger.info("...")`) before calling `getInstance()`, the Logger will be automatically created with default settings (`WARN` level, `EXIT` action).
- **Default ConsoleAppender:**  
  - If you log a message and no appender has been added, a `ConsoleAppender` will be automatically created and added to ensure logs are not lost.
- **Default Layout:**  
  - If the layout was not previously set, then all log events sent to all appenders will use the default layout: `[YYYY-MM-DD hh:mm:ss,SSS] [TYPE] message`, where `SSS` represents milliseconds. The format is set via the `Layout` interface.
- **Default factory for log event:**  
  - If the log event factory was not previously set via `AbstractAppender.setLogEventFactory(logEventFactory: LogEventFactory)`, then the log event will be built via: `new LogEventImpl(message: string, type: LOG_EVENT)`.

**Summary:**  
You can start logging immediately, but for best results (and explicit configuration), initialize the Logger and add your desired appenders as shown above.

---

## Configuration

### Log Levels

Set the **maximum verbosity level** of messages to be logged:

- `Logger.LEVEL.OFF`: No logs
- `Logger.LEVEL.ERROR`: Only errors
- `Logger.LEVEL.WARN`: Errors and warnings (default)
- `Logger.LEVEL.INFO`: Errors, warnings, and info
- `Logger.LEVEL.TRACE`: All messages (most verbose)

### Error/Warning Handling Action
In the event of sending an error/warning the following behaviour can be configured:
- `Logger.ACTION.CONTINUE`: Log the event, continue script execution
- `Logger.ACTION.EXIT`: Log the event and throw a `ScriptError`, terminating the script (default)

> If the log level is `Logger.LEVEL.OFF`, no messages will be sent to any appender, and the action configuration does not take effect.

### Appenders

- `ConsoleAppender`: Output to the Office Scripts console  
  `Logger.addAppender(ConsoleAppender.getInstance())`
- `ExcelAppender`: Output to a specified Excel cell, with color coding  
  `Logger.addAppender(ExcelAppender.getInstance(cellRange))`

---

## Advanced Usage

### Manage Appenders

- Add: `logger.addAppender(appender)`
- Remove: `logger.removeAppender(appender)`
- Replace all: `logger.setAppenders([appender1, appender2])`
- **Only one of each appender type is allowed; attempting to add duplicates will throw an error.**
- **You cannot add null/undefined as an appender, nor can you provide an array with null/undefined elements.**

### Inspect Logger State

- Get an array of all error/warning events: `logger.getCriticalEvents()`
- Get error/warning counts: `logger.getErrCnt()`, `logger.getWarnCnt()`
- Clear state (messages, counters, but not appenders): `logger.reset()` (production-safe, resets counters and critical event messages only)
- Export state: `logger.exportState()`, serializes the current state of the logger to a plain object. , useful for capturing logs and metrics for post-run analysis.
- Check if any error/warning log event has been sent to the appenders: `logger.hasErrors()`, `logger.hasWarnings()`, or `logger.hasCriticalEvents()`

### Resetting for Tests or Automation

If you run multiple tests or your script executes repeatedly (e.g., in Node.js or CI), reset the singleton logger and appenders between runs:

```typescript
Logger.clearInstance()
ConsoleAppender.clearInstance()
ExcelAppender.clearInstance()
AbstractAppender.clearLayout()
AbstractAppender.clearLogEventFactory()
```
> The `clear*` family of methods (`clearInstance`, `clearLayout`, `clearLogEventFactory`) are available in the source files (`src/`). They are omitted from production builds (`dist/`).  
> `logger.reset()` is always available in production and only resets error/warning counters and critical messages, not the logger/appender singletons or layout/factory.

---

## Customization (Advanced)

You can customize how log messages are formatted or how log events are constructed. This is useful for integrating with other systems, outputting logs in a specific structure (e.g., JSON, XML), or adapting the logger for unique workflows.

> **Important:**  
> All customization via `AbstractAppender.setLayout()` or `AbstractAppender.setLogEventFactory()` must happen before any logger or appender is initialized or any log event is sent. These setters will not override existing configuration once logging has begun. When a log method is invoked it does lazy initialization for layout and log event factory, when they are required. That is why it is advised to change the configuration before logging begins.

### Customizing Layout (Log Message Format)

The content and structure of log messages sent to appenders are controlled by a `LayoutImpl` object. By default, a standard layout is used, but you can inject your own formatting logic **once** at the start of your script.

#### Example: Short Layout (No Timestamp)

`LayoutImpl.shortFormatterFun` is a public static function that produces concise log messages of the form `[LOG_EVENT] message`, omitting the timestamp.

```typescript
// Use the built-in short format: [LOG_EVENT] message (no timestamp)
const shortLayout = new LayoutImpl(LayoutImpl.shortFormatterFun)
AbstractAppender.setLayout(shortLayout)

let logger = Logger.getInstance(Logger.LEVEL.INFO, Logger.ACTION.CONTINUE)
logger.addAppender(ConsoleAppender.getInstance())

logger.info("Script started.")
logger.warn("This is a warning.")
logger.error("An error occurred!")
```

Sample log output:
```
[INFO] Script started.
[WARN] This is a warning.
[ERROR] An error occurred!
```

#### Example: JSON Layout

```typescript
// Format each log event as a JSON string
const jsonLayout = new LayoutImpl(event => JSON.stringify(event))
AbstractAppender.setLayout(jsonLayout)

let logger = Logger.getInstance(Logger.LEVEL.INFO, Logger.ACTION.CONTINUE)
logger.addAppender(ConsoleAppender.getInstance())

logger.info("Structured log output")
```
Sample output:
```
{"type":3,"message":"Structured log output","timestamp":"2025-06-15T05:34:08.123Z"}
```

> Note: `AbstractAppender.setLayout()` can only set the layout if it has not already been set. Once set, the layout is fixed for the script’s execution (unless reset via `clearLayout()` in test environments).

### Customizing Log Event Creation

If you want to change all log messages globally (e.g., to add an environment prefix like `[PROD]` to every message), you can set a custom `LogEventFactory` before initialization:

```typescript
// Custom LogEventFactory: prefix all messages with [PROD] for production environment
const prodPrefixFactory: LogEventFactory 
    = (msg: string, type: LOG_EVENT) => new LogEventImpl(`[PROD] ${msg}`, type)
AbstractAppender.setLogEventFactory(prodPrefixFactory)

let logger = Logger.getInstance(Logger.LEVEL.INFO, Logger.ACTION.CONTINUE)
logger.addAppender(ConsoleAppender.getInstance())

logger.info("Script started.")
```
Sample output (default layout):
```
[2025-06-15 05:34:08,123] [INFO] [PROD] Script started.
```
> A custom factory is useful for tagging logs or integrating with external systems, without having to change every log message call.

---

### Using `extraFields` for Structured Logging

The `extraFields` parameter is an advanced feature allowing you to attach additional structured data to any log event. This is useful for tagging logs with context (like function names, user IDs, or custom metadata) and for downstream integrations (e.g., exporting logs as JSON).

You can pass an object with arbitrary key-value pairs as the `extraFields` argument to any logging method. These fields will be included in the underlying `LogEventImpl` instance and are available in custom layouts, factories, or appenders.

#### Example: Adding custom fields to a log entry

The following examples assume a short layout configuration and that the `logger` variable has already been instantiated.
```typescript
logger.info("Processing started", { step: "init", user: "alice@example.com" })
```
Produces the following output:
```
[INFO] Processing started extraFields: { step: "init", user: "alice@example.com" }
```
and
```typescript
logger.error("Failed to save", { errorCode: 42, item: "Budget2025" })
```
Produces the following:
```
[ERROR] Failed to save (extraFields: { errorCode: 42, item: "Budget2025" })
```

#### How it works

- The second argument to all logging methods is `extraFields`(optional):
  - `Logger.info(message, extraFields?)`
  - `Logger.warn(message, extraFields?)`
  - `Logger.error(message, extraFields?)`
  - `Logger.trace(message, extraFields?)`
- `extraFields` can be any object (e.g., `{ key: value, ... }`).  
- If you use a custom layout or export logs, you can access these fields from the `LogEvent` interface.

#### Example: Exporting logs with extraFields

If you export the logger state then you can iterate over all critical events to get the extra fields:

```typescript
const state = logger.exportState()
state.criticalEvents.forEach(event => {
  // event.extraFields will include your custom data if present
  console.log(event.extraFields) // Output per iteration for example: { key: 1, value: 'value' }
})
```
Extra fields, if present, will be part of the `toString()` method for the `LogEvent`:
```typescript
let event = new LogEventImpl("Showing toString", LOG_EVENT.INFO, {user: "admin", sessionId: "123"})
console.log(`event(extra fields)=${event.toString()}`)
event = new LogEventImpl("Showing toString", LOG_EVENT.INFO)
console.log(`event=${event.toString()}`)
```
Here is the `toString()` output (first line: info event with extra fields, second line: without extra fields):
```
event(extra fields)=LogEventImpl: {timestamp="2025-06-19 19:10:34,586", type="INFO", message="Showing toString", 
    extraFields={"user":"admin","sessionId":"123"}}
event=LogEventImpl: {timestamp="2025-06-19 19:10:34,586", type="INFO", message="Showing toString"}
```

---

## Complete Example

```typescript
function main(workbook: ExcelScript.Workbook) {
  // Uncomment ONE of the following configuration blocks before initialization, if desired:

  // // Example: Use the short layout (no timestamp)
  // const shortLayout = new LayoutImpl(LayoutImpl.shortFormatterFun)
  // AbstractAppender.setLayout(shortLayout)

  // // Example: Prefix all messages for production
  // const prodPrefixFactory: LogEventFactory = (msg: string, type: LOG_EVENT) => new LogEventImpl(`[PROD] ${msg}`, type)
  // AbstractAppender.setLogEventFactory(prodPrefixFactory)

  // Set verbosity up to TRACE and continue on error/warning
  let logger = Logger.getInstance(Logger.LEVEL.TRACE, Logger.ACTION.CONTINUE)

  // Add appenders
  logger.addAppender(ConsoleAppender.getInstance())
  const logCell = workbook.getActiveWorksheet().getRange("C2")
  logger.addAppender(ExcelAppender.getInstance(logCell))

  // Logging (with short layout, output shown as comments):
  logger.info("Script started.")           // [INFO] Script started.
  logger.trace("This is a trace message.") // [TRACE] This is a trace message.
  logger.warn("This is a warning.")        // [WARN] This is a warning.
  logger.error("This is an error!")        // [ERROR] This is an error! (if ACTION.EXIT, aborts script)

  // ExcelAppender outputs in cell C2: with default layout
  // [2025-06-26 00:38:10,688] [INFO]  Script started (green text)
  // [2025-06-26 00:38:10,688] [TRACE] This is a trace message (gray text)
}
```
> You can set the layout or log event factory only before any logger or appender is initialized, or before any log event is sent. This ensures consistent formatting and event structure throughout execution.

---

## Architectural Note: One-Time Setters for Layout and LogEventFactory

This framework is designed so that the log message layout and log event factory can only be set once, before use. This prevents accidental changes to the log format or event structure mid-execution, ensuring consistency and reducing debugging surprises in production Office Scripts scenarios. 

- Use `AbstractAppender.setLayout(...)` and `AbstractAppender.setLogEventFactory(...)` at the top of your script, before any logging, appender, or logger initialization.
- For testing, the `clearLayout`, `clearLogEventFactory`, and `clearInstance` methods are available in the source files but are not present in production builds.

**Why this design?**  
- It keeps the API simple for the common use case, and ensures logging behavior is stable and predictable.
- Passing these configurations via `getInstance` would require adding extra rarely-used parameters to already overloaded constructors, making the API harder for most users.
- If your scenario truly requires dynamic, runtime reconfiguration, you can fork the codebase or adapt it for your specific needs, but for most Office Scripts, stability is preferred.

## Cross-Platform Compatibility

This framework is designed to work seamlessly in both Node.js/TypeScript environments (such as VSCode) and directly within Office Scripts.

- **Tested Environments:**  
  - Node.js/TypeScript (VSCode)
  - Office Scripts (Excel)

- **Usage in Office Scripts:**  
  To use the framework in Office Scripts, just paste the `dist/logger.ts` file at the beginning of your script. If you want to run all tests, then you need to paste the source files into your script in the following order:  
  1. `test/unit-test-framework.ts`  
  2. `src/logger.ts`  
  3. `test/main.ts`  
  The order matters because Office Scripts requires that all objects and functions are declared before they are used.

- **Office Scripts Compatibility Adjustments:**  
  - The code avoids unsupported keywords such as `any`, `require`, `export`, and `import`.
  - Office Scripts does not allow calling `ExcelScript` API methods on Office objects inside class constructors; the code is structured to comply with this limitation.
  - Office Scripts doesn't allow defining static properties that are functions. For example, `shortFormatterFun` must be defined outside the class.
  - Additional nuances and workarounds are documented in the source code comments.

This ensures the logging framework is robust and reliable across both development and production Office Scripts scenarios.

---

## Troubleshooting & FAQ

- **What if I call Logger methods before `getInstance()`?**  
  Lazy initialization means logging always works, with default config and console output.

- **What happens if I don’t add an appender?**  
  Logger auto-adds a `ConsoleAppender`when sending the first log event.

- **Can I log to both console and Excel?**  
  Yes, just add both appenders.

- **How do I change log level or action after initialization?**  
  In non-production/test-only scenarios, use `Logger.clearInstance()` and then call `getInstance()` with new options.

- **Why do I get a `ScriptError`?**  
  If you send an error or warning log event and `Logger.ACTION.EXIT` is set (and `Logger.LEVEL != LEVEL.OFF`), the logger will throw and abort the script.
  `ScriptError` is also thrown for internal errors, such as invalid input arguments or incorrect configuration.

- **Why can I only add one of each appender type?**  
  To avoid duplicate logs on the same channel; each appender represents a unique output.

- **Why can't I send a different message to different appenders?**  
  By design, all channels (appenders) receive the same log event message for consistency.

- **Why does the output for `ExcelAppender` override the previous message?**
  By design, the use case for `ExcelAppender` was intended for the default configuration (i.e., a logger with `WARN` level and action `EXIT`). You may want the script to stop if there is any warning or error. Adding more than one event in the same cell (e.g., concatenating via `\n`) is possible, but this defeats the purpose of highlighting each event type with color, since the color will affect the entire cell content. To display each log event in its own cell, you would need to adjust or extend `ExcelAppender`.

- **Why am I getting unexpected results when running some tests in Office Scripts compared to Node.js/TypeScript?**  
  This can happen because Office Scripts executes code asynchronously, meaning some operations (like logging or cell updates) may not complete in strict sequence. As a result, test outcomes may differ from those in Node.js/TypeScript, which runs synchronously and flushes operations immediately.

  **Workaround:**  
  To ensure reliable test results in Office Scripts, introduce a delay or use asynchronous test helpers to wait for operations to complete before making assertions. In `test/main.ts`, the `TestCase.runTestAsync` method is used for this purpose. For example:

  ```typescript
  let actualEvent = appender.getLastLogEvent()
  TestCase.runTestAsync(() => {
    Assert.isNotNull(actualEvent, "ExcelAppender(getLastLogEvent) not null")
    Assert.equals(actualEvent!.type, expectedType, "ExcelAppender(getLastLogEvent).type")
    Assert.equals(actualEvent!.message, expectedMsg, "ExcelAppender(getLastLogEvent).message")
    // Now checking the Excel cell value (formatted via layout)
    let expectedStr = AbstractAppender.getLayout().format(actualEvent as LogEvent)
    Assert.equals(actualStr, expectedStr, "ExcelAppender(cell value via log(string,LOG_EVENT))")
  })
  ```

  By wrapping assertions in `runTestAsync`, you allow asynchronous operations to finish, making your tests more reliable across both environments.

---

## Additional Information

- For developer setup, testing, or CI details, see [docs/DEVELOPER_GUIDE.md](docs/DEVELOPER_GUIDE.md)
- For debug setup, see [docs/VSCode Debugging.md](docs/VSCode%20Debugging.md)
- TypeDoc documentation: [TYPEDOC](https://dlealv.github.io/officescripts-logging-framework/typedoc/)
- Git basic documentation: [docs/git-basics](docs/git-basics.md)
- Unit testing framework repository: [officescripts-unit-test-framework](https://github.com/dlealv/officescripts-unit-test-framework) from the same author. Used for testing the current repository. Check repository's details for more information.

## License

See [LICENSE](LICENSE) for details.

---
