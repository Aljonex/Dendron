---
id: nsc9pnkle0fook550d7gjas
title: Testing Coverage
desc: ''
updated: 1682429059575
created: 1679483993187
---
## SX-64808 Adding Testing Coverage
There was a [security issue reverted here](https://bitbucket.apak.delivery/projects/WFS/repos/wfs/commits/eb2063257291ff273539e7e710c70a5f3d6c96a7#wfs-core/src/com/apakgroup/systemx/controllers/agreement/AgreementService.java) where based on that reversion I thought of 4 test cases which were:
- Testing valid org ID when dealer wasn't in visibility group but was head office - *should return org ID*
- Testing valid org ID when not in visibility group and not head office - *should return null*
- Testing valid org ID when not a dealer but is head office - *should return null*
- Testing valid org ID when not dealer and not head office - *returns null*

## Amend UI for login form
### SX-64764 for 8.47.220
To test the bug needed to include a test I had in the `.388` 
```java
@Test
    public void testFindIssuerAndRedirectMissingIssuer() throws IOException {
        String authString = "/AUTH";
        OpenIdIssuer platformIssuer = setUpIssuer();
        when(issuerRepository.findByIdpType("CUSTOMER")).thenReturn(Optional.empty());
        when(issuerRepository.findByRegistrationId(any())).thenReturn(Optional.of(platformIssuer));

        spyLoginController.redirectToCustomLoginPage();

        verify(spyLoginController).redirectTo(AUTH_URI + authString);
    }
```
But to have in `.220` you need to introduce the `findByRegistrationId()` for the `IssuerRepository` interface which meant it needed introduction into the `OpenIdIssuer` and `NoOpIdIssuer` classes which respectively:
```java
@Override
    public Optional<OpenIdIssuer> findByRegistrationId(String registrationId) {
        return findBy(openIdIssuer -> openIdIssuer.getRegistrationId().equals(registrationId));
    }
```
```java
@Override
    public Optional<OpenIdIssuer> findByRegistrationId(String registrationId) {
        return Optional.empty();
    }
```