---
id: znjdgx39lqrusio5f7uu1vd
title: Basics
desc: ''
updated: 1671535683109
created: 1670423505220
---
### Spring Book Chapter 1

```Java
package com.springinaction.knights;

public class DamselRescuingKnight implements Knight {
    private RescueDamselQuest quest;
    
    public DamselRescuingKnight() {
        this.quest = new RescueDamselQuest();
    }
        
    public void embarkOnQuest() {
        quest.embark();
        
    }

}

package com.springinaction.knights;

public class BraveKnight implements Knight {
    
    private Quest quest;
    
    public BraveKnight(Quest quest) {
        this.quest = quest;
    }
    
    public void embarkOnQuest() {
        quest.embark();
    }
    
}
```
As long as a class that implements quest interface is used, anything can be injected.

To test the BraveKnight:
```Java
package com.springinaction.knights;

import static org.mockito.Mockito.*;
import org.junit.Test;

public class BraveKnightTest {
    
    @Test
    public void knightShouldEmbarkOnQuest() {
        Quest mockQuest = mock(Quest.class);
        BraveKnight knight = new BraveKnight(mockQuest);
        knight.embarkOnQuest();
        verify(mockQuest, times(1)).embark();
    }
}
```

Now imagine you're injecting a slay a dragon quest, you can see its using the `Quest` interface, may notice the generic PrintStream is called from constructor. 
<br>How do we pass quest to the knight? 
<br>And how do we pass the PrintStream to the Quest?

```java
package com.springinaction.knights;

import java.io.PrintStream;

public class SlayDragonQuest implements Quest {
    private PrintStream stream;
    
    public SlayDragonQuest(PrintStream stream) {
        this.stream = stream;
    }
    
    public void embark() {
        stream.println("Embarking on quest to slay the dragon!");
    }
}
```

![[Training.Spring.Beans#basic]]

Here is the java equivalent to the above XML.

```java
package com.springinaction.knights.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.springinaction.knights.BraveKnight;
import com.springinaction.knights.Knight;
import com.springinaction.knights.Quest;
import com.springinaction.knights.SlayDragonQuest;

@Configuration
public class KnightConfig {
    @Beanpublic Knight knight() {
        return new BraveKnight(quest());
    }
    
    @Beanpublic Quest quest() {
        return new SlayDragonQuest(System.out);
    }
}
```