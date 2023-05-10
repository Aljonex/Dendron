---
id: 203rel0l1v23gxj4q72uiox
title: Event Serialisation
desc: ''
updated: 1683189078051
created: 1682672844255
---
### SX-65127 Reverse logic event serialisation
In core conf enable the `event-external` profile in
```
spring {
    default.profiles = [
        ...,
        "event-external"
    ]
}
...
events {
    externallyPublish = true
    eventPublishingBean="loggingPublisher"
}
```
Always remember to re-build your webapp when significant changes have been made!
I forgot for ages with this one and was wondering why the new logger was never logging, but you can always reach out to people, a guy who did a lot with the event code  (Kyran Sheppard) was super helpful and said to message him whenever.