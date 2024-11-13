---
id: mc0f7d84vmu29fts3ftlapj
title: Java
desc: ''
updated: 1730389197041
created: 1668179709229
---
The page for java.

### Core concepts

> Useful links:
- [[Baeldung | https://www.baeldung.com]]
- [[Big Decimal | Training.Intellij.BigDecimal]]
- [[Access Modifiers | Training.Intellij.Modifiers]]

### Data Structures

### Visibility keywords

### Extras
- [[ Testing concepts | Training.JUnit.Testing ]]
- [[ Testing with JUnit | Training.JUnit]]
- [[ Maven lifecycle | Training.Intellij.Maven]]

### UML Class Design choices
![](2022-11-25-10-27-18.png)

![](2022-11-25-13-48-48.png)

[[Understand symbols | https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2015/modeling/uml-class-diagrams-reference?view=vs-2015]]

### Shallow vs Deep Copying
When copying an object, most likely you'll write something along the lines of
```java
ObjectType newObject = oldObject;
```
However what this actually does is shallow copy the object, came across this problem while then looping through 2 arrays and assigning a new property on each iteration when a condition was met, the resulting list that came about had (for each grouping of objects) changed that property to all be the same.

A copy constructor is made like so:
```java
public ObjectName(ObjectName objectName) {
    this.propertyA = objectName.propertyA
    this.propertyB = objectName.propertyB
    ...
    this.propertyZ = objectName.propertyZ
}
```
And what this causes is a deep copy, see [Baeldung deep copy](https://www.baeldung.com/java-copy-constructor) and [Data manipulation from shallow copy](https://stackoverflow.com/questions/73178356/creating-a-deep-copy-to-prevent-value-manipulation)

Effectively **shallow copying** is more efficient because it's "lazy" and handles pointers and references rather than contemporary copies. 
**Deep copying** truly clones the underlying data and then nothing is shared between the first and the copy, as values are actually copied.

So when I made the copy
```java
if ( credit.getFacility().contains(plan.getFacility()) //creditline can have multiple facilities, plan can only have one
                        && ((plan.getSupplierId() == null && credit.getValidForNonSupplierLoans())
                        || credit.getValidForAllSupplierLoans()
                        || Objects.equals(plan.getSupplierGroupMemberId(), credit.getSupplierId()))
                ) {
                    MischmDataRow joined = credit;
                    joined.setScheme(plan.getScheme());
                    mergedRows.add(joined);
                }
            }
```
And then checked against debugger how it was going, when objects were all but the scheme they then adjusted all next items i.e. what should've been
<br>
[0] -> object{scheme: USED_1, otherdata}<br>
[1] -> object1{scheme: USED_2, otherdata}<br>
[2] -> object2{scheme: USED_3, otherdata}<br>
[3] -> object3{scheme: USED_4, otherdata}<br>
<br>
Became
<br>
[0] -> object{scheme: USED_4, otherdata}<br>
[1] -> object1{scheme: USED_4, otherdata}<br>
[2] -> object2{scheme: USED_4, otherdata}<br>
[3] -> object3{scheme: USED_4, otherdata}<br>
<br>
So seemed super luck based, however I have no evidence but feel this may have also been partially to do with the fact more fields had been added to the object without being added to the hashcode or equals method, this requires further research.