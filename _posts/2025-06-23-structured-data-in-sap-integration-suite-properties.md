---
title: "Storing structured data in SAP Integration Suite exchange properties"
date: 2025-06-23
permalink: /posts/2025/06/structured-data-in-sap-integration-suite-properties/
tags:
  - SAP
  - SAP Integration Suite
  - SAP BTP
  - Groovy
  - integration
---

Most Integration Suite flows treat exchange properties as simple containers: a string, a number, sometimes a raw XML blob. But they can hold structured objects too (Groovy maps, lists, parsed JSON) and a simple content modifier can read individual values from them directly, without another script. This results in cleaner, easier to maintain iflows that actually execute faster, because there is no need to re-parse anything. Let's see an example.

Suppose the input is a JSON document with information about plants:

![Sample JSON message with info about plants](https://raw.githubusercontent.com/malmriv/malmriv.github.io/refs/heads/master/_posts/images/plantas-json.png)


A single Groovy script at the start of the flow parses the message, does some filtering (e.g.: filters the non-toxic plants) and stores the result as a map that links each plant to its watering frequency:

```groovy
import com.sap.gateway.ip.core.customdev.util.Message
import groovy.json.JsonSlurper

def Message processData(Message message) {
    String body = message.getBody(java.lang.String)
    def parsedJson = new JsonSlurper().parseText(body)

    def wateringSchedule = [:]
    parsedJson.plantas.findAll { !it.ToxicaMascotas }.each {
        wateringSchedule[it.Nombre] = it.RiegoFrecuenciaDias
    }

    // wateringSchedule: [Chamaedorea elegans:5, Aloe vera:10]
    message.setProperty("MapaRegadoPlantas", wateringSchedule)
    return message
}
```

Any content modifier later in the flow can then read from it by key:

![A content modifier reads the property as a map](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/reading-property-plants.png?raw=true)

And the result is exactly what you'd get by adding a script, re-parsing the content and choosing the relevant key dynamically (but we avoided all of that):

![Same result as using a Groovy script to parse a JSON!](https://raw.githubusercontent.com/malmriv/malmriv.github.io/refs/heads/master/_posts/images/output-property-plants.png)

For a more typical case scenario, a property holding a `CustomerName -> ID` map built from the input is just as reachable downstream. And if the flow calls a sub-integration, promoting the property to a header before the call carries the data along without touching the body again.

The one limitation worth keeping in mind: this pattern suits data that is read but not modified. If the flow needs to mutate the object and pass the updated version on as the message body, there is no option but to serialize using a Groovy script. For the read-many case it's a clean approach.
