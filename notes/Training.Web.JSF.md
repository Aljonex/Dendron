---
id: n6766b26mzddurm5c1h71i1
title: JSF
desc: ''
updated: 1675248926517
created: 1673536928425
---
### Converting app into WEB app
- *Deployment Descriptor*: an artifact for configuration of deployment, generally an XML file, in this case describes how web application should be deployed


### JaveServer Faces
A model-view-controller (MVC) web framework that simplifies construction of User Interfaces for server-based applications using reusable UI components in a page.

JSF runs a Java servlet container containing:
- JavaBeans components as models containing app specific functionality and data
- Custom tag library for event handlers and validators
- Custom tag library for rendering UI components
- UI components represented as stateful objects on server
- Server-side helper classes
- Validators, event handlers, and navigation handlers
- Application config resource file

#### Life cycle of app
- Restore view
- Apply request values phase
- Update model values phase
- Invoke application phase
- Render response phase

### Deploy (from gitbash or probably intellij)
```
Directory of C:\Users\alexajones2\scripts\training

13/01/2023  14:08    <DIR>          .
13/01/2023  14:08    <DIR>          ..
13/01/2023  14:08             1,476 copy_training
13/01/2023  14:08                 0 copy_training.properties
13/01/2023  14:08               413 run_training
13/01/2023  14:09               285 run_training.properties
13/01/2023  14:08               948 sync_training_resources
               5 File(s)          3,122 bytes
               2 Dir(s)  902,909,444,096 bytes free
```
- `./run_app_name start`
- To redeploy `ctrl+c` then `./run_app_name stop` 
- Then deploy latest file changes with `mvn clean package -DskipTests`

### How debugging JSF applications work
- Using the backchannel and Tomcat Catalina declarations linking with Intellij backchannel declaration in  `edit configuration`

### JSF calling "fields"
```html
<h4>The selected vehicle is: #{vehicleSelectionController.vehicleDTO}</h4>
<!--selectionController.<fieldName> auto searches for a getter -->
                
```

```Java
public VehicleDTO getVehicleDTOObj() {
        LOGGER.info("GETTING OBJECT - {}", vehicleDTO);
        return this.vehicleDTO;
    }
    public String getVehicleDTO() {
        LOGGER.info("Getting vehicle DTO from controller");
        return this.DTOString;
    }

    public void setVehicleDTO(String DTOString) {
        this.DTOString = DTOString;
        String[] split = DTOString.split("\\s+");
        this.vehicleDTO = new VehicleDTO(split[0], split[1], split[2]);
        LOGGER.info("Set called in controller with {} {} {}", vehicleDTO.getMake(), vehicleDTO.getModel(), vehicleDTO.getDerivative());

        LOGGER.info("VEHICLE IS {} ", vehicleDTO);
    }
```
Basically JSF can't dynamically handle java objects unless very simple and only deals with strings etc, so this is in place of a converter (which for me didn't work) so that when getting data that was displayed on the page I then convert it into a DTO object.

### JSF `preRenderView` and `viewAction`
- `preRenderView` within the `<f:metadata> <f:event type="preRenderView" listener="#{listener.method}/>` tags MUST be called within the page load, for example I'm using a template with `ui:composition` and it wouldn't get called unless inside the `ui:define` section [[example here | https://stackoverflow.com/questions/12592405/jsf-2-prerenderview-listener-not-called-when-page-called-from-action-method-outc]].


### JSF in practice and key pain points
JSF basically allows creation of `.jsp` or `.xhtml` pages and via a deployment descriptor allows setup of the welcome page(s) and servlets, filters and mappings and config files (listeners also). 
*Servlets* are applications that run on a Java application server and extend the capabilities of the Web Server.
Another pain point was being unable to use the latest version of JSF, thereby limiting tutorials to deprecated docs, as I'm using hibernate 4 I couldn't use spring past a given point which limited my range of spring-web and security to **<5.0.0.RELEASE**.

[[viewAction vs preRenderView | https://www.oracle.com/technical-resources/articles/java/jsf22.html]]

[[Basic spring xml web security tutorial | https://www.tutorialspoint.com/spring_security/spring_security_with_xml_configuration.htm]]

[[Navigation examples | https://mkyong.com/jsf2/jsf-form-action-navigation-rule-example/]]