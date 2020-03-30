# Neato Binding

This binding is used to connect your openHAB 2 system with Neato web (where you log in and find Your Neato's). The binding supports discovery via configuring your login and password to a bridge. From the binding, you will get status of your vacuum cleaners and also a command channel where you can control them. Since the binding uses a polling mechanism, there may be some latency depending on your setting regarding refresh time. 

For log in transaction, the binding uses Neato Beehive API and for status and control, the binding uses Nucleao API. 

## Supported Things

Supported thing types

* neatoaccount (bridge)
* vacuumcleaner

A bridge is required to connect to your Neato Cloud account.  

All "Connected" type vacuum cleaners should be supported by this binding since they are supported by the Neato API.  As of todays date, it is only verified with Neato Connected and Neato D7 vacuum cleaners.

## Discovery

Discovery is used _after_ a bridge has been created and configured with your login information.

1. Add the binding
2. Add a new thing of type NeatoAccount and configure with username and password
3. Go to Inbox and start discovery of Vacuums using Neato Binding
4. Vacuums should appear in your inbox!

## Thing Configuration

In order to manually create a thing file and not use the discovery routine you will need to know the vacuums serial number as well as the secret used in web service calls. This is a bit difficult to get. The easiest way of getting this information is to use the third party python library that is available at https://github.com/stianaske/pybotvac.

Neato Account Config

| Config   | Description                         |
|----------|------------------------------------ |
| email    | Email address tied to Neato Account |
| password | Password tied to Neato Account      |

Vacuum Cleaner Config

| Config   | Description                             |
|----------|-----------------------------------------|
| serial   | Serial Number of your Neato Robot       |
| secret   | Secret for accessing Neato web services (see note above) |
| refresh  | Refresh time interval in seconds for updates from the Neato Web Service.  Defaults to 60 sec |

## Channels

| Channel               | Type   | Label                      | Description                                                                               | Read Only |
|-----------------------|--------|----------------------------|-------------------------------------------------------------------------------------------|-----------|
| battery-level         | Number | Battery Level              | Battery Level of the vacuum cleaner.                                                      | True      |
| state                 | String | Current State              | Current state of the vacuum cleaner.                                                      | True      |
| available-services    | String | Current available services | List of services that are currently available for the vacuum cleaner                      | True      |
| action                | String | Current Action             | Current action of the vacuum cleaner.                                                     | True      |
| dock-has-been-seen    | Switch | Dock has been seen         | True or False value if the dock has been seen                                             | True      |
| is-docked             | Switch | Is docked                  | Is the vacuum cleaner in the docking station?                                             | True      |
| is-scheduled          | Switch | Is scheduled enabled       | True or False value if the vacuum cleaner is scheduled for cleaning.                      | True      |
| is-charging           | Switch | Is Charging                | Is the vacuum cleaner currently charging?                                                 | True      |
| available-commands    | String | Available Commands         | List of available commands.                                                               | True      |
| error                 | String | Error                      | Current error message in system.                                                          | True      |
| command               | String | Send Command               | Send Commands to Vacuum Cleaner. (clean, cleanmap, pause, resume, stop, dock)             | False     |
| cleaning-category     | String | Cleaning Category          | Current or Last category of the cleaning. Manual, Normal House Cleaning or Spot Cleaning. | True      |
| cleaning-mode         | String | Cleaning Mode              | Current or Last cleaning mode. Eco or Turbo.                                              | True      |
| cleaning-modifier     | String | Cleaning Modifier          | Modifier of current or last cleaning. Normal or Double.                                   | True      |
| cleaning-spotwidth    | Number | Spot Width                 | Current or Last cleaning, width of spot. 100-400cm.                                       | True      |
| cleaning-spotheight   | Number | Spot Height                | Current or Last cleaning, height of spot. 100-400cm.                                      | True      |

## Full Example

Below you will find examples of the necessary files:

**neato.items**

```
Group GNeato
Number Robot1Battery  "Battery level [%.0f %%]" <battery> (GNeato) { channel = "neato:vacuumcleaner:robot1:battery-level" }
String Robot1State  "Status [MAP(neato-sv.map):%s]" (GNeato) { channel = "neato:vacuumcleaner:robot1:state" }
String Robot1Error  "Error [%s]" (GNeato) { channel = "neato:vacuumcleaner:robot1:error" }
String Robot1Action  "Action [MAP(neato-sv.map):%s]" (GNeato) { channel = "neato:vacuumcleaner:robot1:action" }
Switch Robot1DockHasBeenSeen  "Seen dock [%s]" <present> (GNeato) { channel = "neato:vacuumcleaner:robot1:dock-has-been-seen" }
Switch Robot1IsDocked  "In dock [MAP(neato-sv.map):%s]" <present> (GNeato) { channel = "neato:vacuumcleaner:robot1:is-docked" }
Switch Robot1IsScheduled  "Scheduled [%s]" (GNeato) { channel = "neato:vacuumcleaner:robot1:is-scheduled" }
Switch Robot1IsCharging  "Is Charging [%s]" <heating> (GNeato) { channel = "neato:vacuumcleaner:robot1:is-charging" }
String Robot1Category  "Cleaning Category [MAP(neato-sv.map):%s]" (GNeato)  { channel = "neato:vacuumcleaner:robot1:cleaning-category" }
String Robot1Mode  "Cleaning Mode [MAP(neato-sv.map):%s]" (GNeato) { channel = "neato:vacuumcleaner:robot1:cleaning-mode" }
String Robot1Modifier  "Cleaning Modifier [MAP(neato-sv.map):%s]" (GNeato) { channel = "neato:vacuumcleaner:robot1:cleaning-modifier" }
Number Robot1SpotWidth  "SpotWidth [%.0f]" <niveau> (GNeato) { channel = "neato:vacuumcleaner:robot1:cleaning-spotwidth" }
Number Robot1SpotHeight  "SpotHeight [%.0f]" <niveau> (GNeato)  { channel = "neato:vacuumcleaner:robot1:cleaning-spotheight" }
String Robot1Command  "Send Command" { channel = "neato:vacuumcleaner:robot1:command" }
```

**sitemap**

```
Frame label="Neato BotVac Connected" {
    Switch item=Robot1Command mappings=[clean="Clean",stop="Stop",pause="Pause",resume="Resume", dock="Send to dock"]
    Text item=Robot1Battery label="Battery level"
    Text item=Robot1State
    Text item=Robot1Error label="Error Message" icon="siren"
    Text item=Robot1Action label="Activity"
    Text item=Robot1IsDocked label="In dock"
    Group label="More information" item=GNeato
}
```

**neato.things**

```
neato:vacuumcleaner:robot1 [ serial="vacuumcleaner-serial", secret="secret-string"]
```

