---
title: iOS SDK (Beta)
description: The Amplitude iOS Swift SDK installation and quick start guide.
icon: simple/ios
---

![CocoaPods](https://img.shields.io/cocoapods/v/AmplitudeSwift)

This is the official documentation for the Amplitude Analytics iOS SDK.

!!!beta "iOS Swift SDK Resources (Beta)"
    [:material-github: GitHub](https://github.com/amplitude/Amplitude-Swift) · [:material-code-tags-check: Releases](https://github.com/amplitude/Amplitude-Swift/releases)

!!!warning "Objective-C Not Supported"
    Please note that the latest iOS SDK is NOT compatible with Objective-C projects. Use the [maintenance iOS SDK](/data/sdks/ios) if your project requires compatibility with Objective-C.

!!!warning "Some Features Not Yet Available"
    For customers beginning with Amplitude Experiment, please note that this SDK does not support the [Amplitude Experiment integration](https://www.docs.developers.amplitude.com/experiment/sdks/ios-sdk/#initialize). 

--8<-- "includes/sdk-ios/apple-deprecate-carrier.md"

/*cspell:disable*/
--8<-- "includes/size/ios.md"
    `./measure_cocoapod_size.py --cocoapods AmplitudeSwift:0.4.14`.
/*cspell:enable*/

## Getting started

Use [this quickstart guide](../../sdks/sdk-quickstart#ios-beta) to get started with Amplitude iOS SDK.

## Usage

### Initialize

You must initialize the SDK before you can instrument. The API key for your Amplitude project is required.

```swift
let amplitude = Amplitude(configuration: Configuration(
    apiKey: AMPLITUDE_API_KEY
))
```

### `Configuration`

???config "Configuration Options"
    | <div class="big-column">Name</div>  | Description | Default Value |
    | --- | --- | --- |
    | `apiKey` | The apiKey of your project. | `nil` |
    | `instanceName` | The name of the instance. Instances with the same name will share storage and identity. For isolated storage and identity use a unique `instanceName` for each instance. | "default_instance" |
    | `storageProvider` | Implements a custom `storageProvider` class from `Storage`. | `PersistentStorage` |
    | `logLevel` | The log level enums: `LogLevelEnum.OFF`, `LogLevelEnum.ERROR`, `LogLevelEnum.WARN`, `LogLevelEnum.LOG`, `LogLevelEnum.DEBUG` | `LogLevelEnum.WARN` | 
    | `loggerProvider` | Implements a custom `loggerProvider` class from the Logger, and pass it in the configuration during the initialization to help with collecting any error messages from the SDK in a production environment. | `ConsoleLogger` |
    | `flushIntervalMillis` | The amount of time SDK will attempt to upload the unsent events to the server or reach `flushQueueSize` threshold. | `30000` |
    | `flushQueueSize` | SDK will attempt to upload once unsent event count exceeds the event upload threshold or reach `flushIntervalMillis` interval.  | `30` |
    | `flushMaxRetries` | Maximum retry times.  | `5` |
    | `minIdLength` | The minimum length for user id or device id. | `5` |
    | `partnerId` | The partner id for partner integration. | `nil` |
    | `identifyBatchIntervalMillis` | The amount of time SDK will attempt to batch intercepted identify events. | `30000` |
    | `flushEventsOnClose` | Flushing of unsent events on app close. | `true` |
    | `callback` | Callback function after event sent. | `nil` |
    | `optOut` | Opt the user out of tracking. | `false` |
    | `trackingSessionEvents` | Flushing of unsent events on app close. | `true` |
    | `minTimeBetweenSessionsMillis` | The amount of time for session timeout. | `300000` |
    | `serverUrl` | The server url events upload to. | `https://api2.amplitude.com/2/httpapi` |
    | `serverZone` |  The server zone to send to, will adjust server url based on this config. | `US` |
    | `useBatch` |  Whether to use batch api. | `false` |
    | `trackingOptions` |  Options to control the values tracked in SDK. | `enable` |
    | `enableCoppaControl` |  Whether to enable COPPA control for tracking options. | `false` |
    | `migrateLegacyData` | Available in `0.4.7`+. Whether to migrate [maintenance SDK](../ios) data (events, user/device ID). | `true`|

### `track`

Events represent how users interact with your application. For example, "Button Clicked" may be an action you want to note.

```swift
let event = BaseEvent(
    eventType: "Button Clicked", 
    eventProperties: ["my event prop key": "my event prop value"]
)
amplitude.track(event: event)
```

Another way to instrument basic tracking event.

```swift
amplitude.track(
    eventType: "Button Clicked",
    eventProperties: ["my event prop key": "my event prop value"]
)
```

### `identify`

Identify is for setting the user properties of a particular user without sending any event. The SDK supports the operations `set`, `setOnce`, `unset`, `add`, `append`, `prepend`, `preInsert`, `postInsert`, and `remove` on individual user properties. Declare the operations via a provided Identify interface. You can chain together multiple operations in a single Identify object. The Identify object is then passed to the Amplitude client to send to the server.
Starting from release v0.4.0, identify events with only set operations will be batched and sent with fewer events. This change won't affect running the set operations. There is a config `identifyBatchIntervalMillis` for managing the interval to flush the batched identify intercepts.

!!!note

    If the Identify call is sent after the event, the results of operations will be visible immediately in the dashboard user's profile area, but it will not appear in chart result until another event is sent after the Identify call. So the identify call only affects events going forward. More details [here](https://amplitude.zendesk.com/hc/en-us/articles/115002380567-User-Properties-Event-Properties#applying-user-properties-to-events).

You can handle the identity of a user using the identify methods. Proper use of these methods can connect events to the correct user as they move across devices, browsers, and other platforms. Send an identify call containing those user property operations to Amplitude server to tie a user's events with specific user properties.

```swift
let identify = Identify()
identify.set(property: "color", value: "green")
amplitude.identify(identify: identify)
```

### User groups

--8<-- "includes/editions-growth-enterprise-with-accounts.md"

--8<-- "includes/groups-intro-paragraph.md"

!!! example
    If Joe is in 'orgId' '15', then the `groupName` would be '15'.

    ```swift
    // set group with a single group name
    amplitude.setGroup(groupType: "orgId", groupName: "15")
    ```

    If Joe is in 'orgId' 'sport', then the `groupName` would be '["tennis", "soccer"]'.

    ```swift
    // set group with multiple group names
    amplitude.setGroup(groupType: "sport", groupName: ["tennis", "soccer"])
    ```

--8<-- "includes/event-level-groups-intro.md"

```swift
amplitude.track(
    event: BaseEvent(
        eventType: "event type",
        eventProperties: [
            "eventPropertyKey": "eventPropertyValue"
        ], 
        groups: ["orgId": 15]
    )
)
```

### Group identify

--8<-- "includes/editions-growth-enterprise-with-accounts.md"

--8<-- "includes/group-identify-considerations.md"

```swift
let groupType = "plan"
let groupName = "enterprise"
let identify = Identify().set(property: "key", value: "value")
amplitude.groupIdentify(groupType: groupType, groupName: groupProperty, identify: groupIdentify)
```

### Track revenue

Amplitude can track revenue generated by a user. Revenue is tracked through distinct revenue objects, which have special fields that are used in Amplitude's Event Segmentation and Revenue LTV charts. This allows Amplitude to automatically display data relevant to revenue in the platform. Revenue objects support the following special properties, as well as user-defined properties through the `eventProperties` field.

```swift
let revenue = Revenue()
revenue.price = 3.99
revenue.quantity = 3
revenue.productId = "com.company.productId"
amplitude.revenue(revenue: revenue)
```

| <div class="big-column">Name</div>   | Description  |
| --- | --- |
| `productId` | Optional. String. An identifier for the product. Amplitude recommends something like the Google Play Store product ID. Defaults to `null`.|
| `quantity `| Required. Integer. The quantity of products purchased. Note: revenue = quantity * price. Defaults to 1 |
| `price `| Required. Double. The price of the products purchased, and this can be negative. Note: revenue = quantity * price. Defaults to `null`.|
| `revenueType`| Optional, but required for revenue verification. String. The revenue type (for example, tax, refund, income). Defaults to `null`.|
| `receipt`| Optional. String. The receipt identifier of the revenue. For example, "123456". Defaults to `null`. |
| `receiptSignature`| Optional, but required for revenue verification. String. Defaults to `null`. |

### Custom user ID

If your app has its login system that you want to track users with, you can call `setUserId` at any time.

```swift
amplitude.setUserId(userId: "user@amplitude.com")
```

### Custom device ID

You can assign a new device ID using `deviceId`. When setting a custom device ID, make sure the value is sufficiently unique. Amplitude recommends using a UUID.

```swift
amplitude.setDeviceId(NSUUID().uuidString)
```

### Custom storage

If you don't want to store the data in the Amplitude-defined location, you can customize your own storage by implementing the [Storage protocol](https://github.com/amplitude/Amplitude-Swift/blob/211d0c05830fab47e74fa9a053615cf422618a02/Sources/Amplitude/Types.swift#L62-L86) and setting the `storageProvider` in your configuration.

Every iOS app gets a slice of storage just for itself, meaning that you can read and write your app's files there without worrying about colliding with other apps. By default, Amplitude uses this file storage and creates an "amplitude" prefixed folder inside the app "Documents" directory. However, if you need to expose the Documents folder in the native iOS "Files" app and don't want expose "amplitude" prefixed folder, you can customize your own storage provider to persist events on initialization.

```swift
Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        storageProvider: YourOwnStorage() // YourOwnStorage() should implement Storage
    )
)
```

### Reset when user logs out

`reset` is a shortcut to anonymize users after they log out, by:

- setting `userId` to `null`
- setting `deviceId` to a new value based on current configuration

With an empty `userId` and a completely new `deviceId`, the current user would appear as a brand new user in dashboard.

```swift
amplitude.reset()
```

## Amplitude SDK plugin

Plugins allow you to extend Amplitude SDK's behavior by, for example, modifying event properties (enrichment type) or sending to third-party APIs (destination type). A plugin is an object with methods `setup()` and `execute()`.

### Plugin.setup

This method contains logic for preparing the plugin for use and has `amplitude` instance as a parameter. The expected return value is `null`. A typical use for this method, is to instantiate plugin dependencies. This method is called when the plugin is registered to the client via `amplitude.add()`.

### Plugin.execute

This method contains the logic for processing events and has `event` instance as parameter. If used as enrichment type plugin, the expected return value is the modified/enriched event. If used as a destination type plugin, the expected return value is `null`. This method is called for each event, including Identify, GroupIdentify and Revenue events, that's instrumented using the client interface.

### Plugin examples

#### Enrichment type plugin

Here's an example of a plugin that modifies each event that's instrumented by adding extra event property.

```swift
class EnrichmentPlugin: Plugin {
    let type: PluginType
    var amplitude: Amplitude?

    init() {
        self.type = PluginType.enrichment
    }

    func setup(amplitude: Amplitude) {
        self.amplitude = amplitude
    }

    func execute(event: BaseEvent?) -> BaseEvent? {
        event?.sessionId = -1
        if event?.eventProperties == nil {
            event?.eventProperties = [:]
        }
        event?.eventProperties?["event prop key"] = "event prop value"
        return event
    }
}

amplitude.add(plugin: EnrichmentPlugin())
```

#### Destination type plugin

In destination plugin, you are able to overwrite the track(), identify(), groupIdentify(), revenue(), flush() functions.

```swift
class TestDestinationPlugin: DestinationPlugin {
    override func track(event: BaseEvent) -> BaseEvent? {
        return event
    }

    override func identify(event: IdentifyEvent) -> IdentifyEvent? {
        return event
    }

    override func groupIdentify(event: GroupIdentifyEvent) -> GroupIdentifyEvent? {
        return event
    }

    override func revenue(event: RevenueEvent) -> RevenueEvent? {
        return event
    }

    override func flush() {
    }

    override func setup(amplitude: Amplitude) {
        self.amplitude = amplitude
    }

    override func execute(event: BaseEvent?) -> BaseEvent? {
        return event
    }
}
```

## Troubleshooting and Debugging

--8<-- "includes/sdk-troubleshooting-and-debugging/latest-ios.md"

## Advanced topics

### User sessions

A session on iOS is a period of time that a user has the app in the foreground.

Amplitude groups events together by session. Events that are logged within the same session have the same `session_id`. Sessions are handled automatically so you don't have to manually call `startSession()` or `endSession()`.

You can adjust the time window for which sessions are extended. The default session expiration time is 30 minutes.

```swift
let amplitude = Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        minTimeBetweenSessionsMillis: 1000
    )
)
```

By default, Amplitude automatically sends the '[Amplitude] Start Session' and '[Amplitude] End Session' events. Even though these events aren't sent, sessions are still tracked by using `session_id`.
You can also disable those session events.

```swift
let amplitude = Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        trackingSessionEvents: false
    )
)
```

You can define your own session expiration time. The default session expiration time is 30 minutes.

```swift
let amplitude = Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        minTimeBetweenSessionsMillis: 100000
    )
)
```

### Set custom user ID

If your app has its login system that you want to track users with, you can call `setUserId` at any time.

```swift
amplitude.setUserId(userId: "USER_ID")
```

Don't assign users a user ID that could change, because each unique user ID is a unique user in Amplitude. Learn more about how Amplitude tracks unique users in the [Help Center](https://help.amplitude.com/hc/en-us/articles/115003135607-Track-unique-users-in-Amplitude).

### Log level

You can control the level of logs that print to the developer console.

- 'OFF': Suppresses all log messages.
- 'ERROR': Shows error messages only.
- 'WARN': Shows error messages and warnings. This level logs issues that might be a problem and cause some oddities in the data. For example, this level would display a warning for properties with null values.
- 'LOG': Shows informative messages about events.
- 'DEBUG': Shows error messages, warnings, and informative messages that may be useful for debugging.

Set the log level `logLevel` with the level you want.

```swift
amplitude.logger?.logLevel = LogLevelEnum.LOG.rawValue
```

### Logged out and anonymous users

--8<-- "includes/logged-out-and-anonymous-users.md"

```swift
amplitude.reset();
```

### Disable tracking

By default the iOS SDK tracks several user properties such as `carrier`, `city`, `country`, `ip_address`, `language`, and `platform`.
Use the provided `TrackingOptions` interface to customize and toggle individual fields.
Before initializing the SDK with your apiKey, create a `TrackingOptions` instance with your configuration and set it on the SDK instance.

```swift
let trackingOptions = TrackingOptions()
trackingOptions.disableCity().disableIpAddress().disableLatLng()
let amplitude = Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        trackingOptions: trackingOptions
    )
)
```

Tracking for each field can be individually controlled, and has a corresponding method (for example, `disableCountry`, `disableLanguage`).

| <div class="big-column">Method</div> | Description |
| --- | --- |
| `disableAdid()` | Disable tracking of Google ADID |
| `disableCarrier()` | Disable tracking of device's carrier |
| `disableCity()` | Disable tracking of user's city |
| `disableCountry()` | Disable tracking of user's country |
| `disableDeviceBrand()` | Disable tracking of device brand |
| `disableDeviceModel()` | Disable tracking of device model |
| `disableDma()` | Disable tracking of user's designated market area (DMA). |
| `disableIpAddress()` | Disable tracking of user's IP address |
| `disableLanguage()` | Disable tracking of device's language |
| `disableLatLng()` | Disable tracking of user's current latitude and longitude coordinates |
| `disableOsName()` | Disable tracking of device's OS Name |
| `disableOsVersion()` | Disable tracking of device's OS Version |
| `disablePlatform()` | Disable tracking of device's platform |
| `disableRegion()` | Disable tracking of user's region. |
| `disableVersionName()` | Disable tracking of your app's version name |

!!!note

    Using `TrackingOptions` only prevents default properties from being tracked on newly created projects, where data has not yet been sent. If you have a project with existing data that you want to stop collecting the default properties for, get help in the [Amplitude Community](https://community.amplitude.com/?utm_source=devdocs&utm_medium=helpcontent&utm_campaign=devdocswebsite). Disabling tracking doesn't delete any existing data in your project.

### Carrier

Amplitude determines the user's mobile carrier using [`CTTelephonyNetworkInfo`](https://developer.apple.com/documentation/coretelephony/cttelephonynetworkinfo), which returns the registered operator of the `sim`.

### COPPA control

COPPA (Children's Online Privacy Protection Act) restrictions on IDFA, IDFV, city, IP address and location tracking can all be enabled or disabled at one time. Apps that ask for information from children under 13 years of age must comply with COPPA.

```swift
let amplitude = Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        enableCoppaControl: true
    )
)
```

### Advertiser ID

Advertiser ID (also referred to as IDFA) is a unique identifier provided by the iOS and Google Play stores. As it's unique to every person and not just their devices, it's useful for mobile attribution.
 [Mobile attribution](https://www.adjust.com/blog/mobile-ad-attribution-introduction-for-beginners/) is the attribution of an installation of a mobile app to its original source (such as ad campaign, app store search).
 Mobile apps need permission to ask for IDFA, and apps targeted to children can't track at all. Consider using IDFV, device ID, or an email login system when IDFA isn't available.

To retrieve the IDFA and add it to the tracking events, you can follow this [example plugin](https://github.com/amplitude/Amplitude-Swift/blob/main/Examples/AmplitudeSwiftUIExample/AmplitudeSwiftUIExample/ExamplePlugins/IDFACollectionPlugin.swift) to implement your own plugin.

--8<-- "includes/sdk-device-id/lifecycle-header.md"

1. Device ID of Amplitude instance if it’s set by `setDeviceId()`
2. IDFV if it exists
3. A randomly generated UUID string

--8<-- "includes/sdk-device-id/transfer-to-a-new-device.md"

--8<-- "includes/sdk-device-id/get-device-id.md"

```swift
let deviceId = amplitude.getDeviceId()
```

To set the device, refer to [custom device ID](./#custom-device-id).

### Location tracking

Amplitude converts the IP of a user event into a location (GeoIP lookup) by default. This information may be overridden by an app's own tracking solution or user data.

### Opt users out of tracking

Users may wish to opt out of tracking entirely, which means Amplitude doesn't track any of their events or browsing history. `OptOut` provides a way to fulfill a user's requests for privacy.

```swift
let amplitude = Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        optOut: true
    )
)
```

### Set log callback

Implements a customized `loggerProvider` class from the LoggerProvider, and pass it in the configuration during the initialization to help with collecting any error messages from the SDK in a production environment.

```swift
class SampleLogger: Logger {
    typealias LogLevel = LogLevelEnum

    var logLevel: Int

    init(logLevel: Int = LogLevelEnum.OFF.rawValue) {
        self.logLevel = logLevel
    }

    func error(message: String) {
        // TODO: handle error message
    }

    func warn(message: String) {
        // TODO: handle warn message
    }

    func log(message: String) {
        // TODO: handle log message
    }

    func debug(message: String) {
        // TODO: handle debug message
    }
}

let amplitude = Amplitude(
    configuration: Configuration(
        apiKey: AMPLITUDE_API_KEY,
        loggerProvider: SampleLogger()
    )
)
```

### More resources

If you have any problems or issues with the SDK, [create a GitHub issue](https://github.com/amplitude/Amplitude-Swift/issues/new) or submit a request on [Amplitude Help](https://help.amplitude.com/hc/en-us/requests/new).

--8<-- "includes/abbreviations.md"
