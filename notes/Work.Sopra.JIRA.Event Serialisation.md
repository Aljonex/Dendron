---
id: 203rel0l1v23gxj4q72uiox
title: Event Serialisation
desc: ''
updated: 1687961403813
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

Had done this wrong originally because I'd focused on making tests work, it caused a later bug to do with cash posting, this was because it was still trying to serialise (publish) non-serialisable events. 
So when I picked this up there was a blacklist in `EventPublishingListener` and I wanted to reverse the logic to make this a whitelist.
```java
private static final Set<Class<? extends IExternallyPublishableEvent<?>>> PERSISTABLE_EVENT_CLASSES = new HashSet<>(
            Arrays.asList(
                    FileProduced.class,
                    FileUploaded.class,
                    FinancialInterestEvent.class,
                    RouteErrored.class,
                    ScheduledJobCompleted.class,
                    ItemStatusChanged.class,
                    BusinessProcessingSuspended.class,
                    BusinessProcessingCompleted.class,
                    UserLoggedIn.class,
                    AgreementSearchSnapshotUpdated.class,
                    DayendStarted.class,
                    DayendCompleted.class,
                    DayendFailed.class,
                    UtilisationChanged.class
            )
    );

public void onPublishableEvent(IExternallyPublishableEvent<?> event) {
        if (!externallyPublish) {
            LOGGER.info("System is not configured to externally publish events");
            return;
        }

        if (isEventPersistable(event)) {
            LOGGER.debug("Persisting event of type {}...", event.getEventType());
            eventDataService.persistEventInOngoingTransaction(event);
        } else {
            LOGGER.debug("Event of type {} is not persistable. Falling back to immediate publication...",
                    event.getEventType());
            eventPublisher.publishEventImmediately(event);
        }
    }

private boolean isEventPersistable(IExternallyPublishableEvent<?> event) {
        return !PERSISTABLE_EVENT_CLASSES.contains(event.getClass());
    }
```
My issue was I'd removed the `!` in `isEventPersistable`, then made tests pass, then to do the opposite I just added a negation to `if (isEventPersistable())` which was effectively doing the same thing.
I had to go into the tests and add an event from the whitelist, these don't need mocking as events are basically POJOs so:
```java
private ScheduledJobCompleted persistableEvent = new ScheduledJobCompleted("DC_TEST", "testJob", null);
...
public void testPersistsEventWhenExternallyPublishSetToTrueAndIsPersistable() throws IOException {
        eventPublishingListener = new EventPublishingListener(eventDataService, eventPublisher, true);

        eventPublishingListener.onPublishableEvent(persistableEvent);

        verify(eventDataService).persistEventInOngoingTransaction(persistableEvent);
    }
```