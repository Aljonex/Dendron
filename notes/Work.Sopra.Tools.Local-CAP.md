---
id: bm08j8r8pra32h782remxic
title: Local-CAP
desc: ''
updated: 1731488100767
created: 1731487702766
---
# Basics
- [Follow the CAP db setup](https://sf-wiki.atlassian.net/wiki/spaces/WIKI/pages/41223462/CAP+Database+set-up)
- Download sqlDeveloper
- Download wiremock
- Clone wiremock repo
- Launch Oracle in WSL

## Local.conf
```conf
wfs.core {
    application {
        database.profile = "postgres"
        jdbc.postgres.schema {
            partitions = [
                {
                   partition = "1"
                   username = "SYSX_USER"
                   schema = "SYSX_USER"
                }
            ]
        }
    }

    #application.hibernate.show_sql=true
    custom.code=c18
    env.development = true
    uiReskin.availableStylingOptions = ["MATERIAL", "ORIGINAL"]
    dealerStatement.useCache = true
    #report.threadCount=8
    migration.flyway {
        locations = "db/migration/postgresql, db/migration/common"
        #createhibernatesequences = false

    }
    migration.flyway.dataUpdate.locations = "flyway/data/postgresql, com/apakgroup/flyway/data/common"
    migration.flyway.recovery.locations = "flyway/recovery/postgresql"
    migration.flyway.schemaDeletion.locations = "flyway/schemaDeletion/postgresql"

    // hpi settings
        hpi.webservice.password = "APAK"
        hpi.webservice.product = "HPI63" // HPI64 is a basic check. HPI63 is the full check and is needed to see alerts in the response
        hpi.webservice.url = "http://localhost:8081/TradeSoap/services/SupplementaryEnquiryV1/"

     jdbc {
         cap {
              driver = "oracle.jdbc.driver.OracleDriver"
              maxactive = 5
              password = "welcome"
              url = "jdbc:oracle:thin:@(DESCRIPTION=(ENABLE=BROKEN)(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521))(CONNECT_DATA = (SERVICE_NAME = XEPDB1)))"
              username = "cap_dev"
          }
     }

    oauth.custom.users = [                {
            clientId="PasswordUser"
            secret="{noop}TestSecret1"
            grantTypes="password,refresh_token"
            authorities="SCOPE_wfsv6.io/rest,ROLE_ACCESS_REST_API"
            accessTokenValidity = 600000
            refreshTokenValidity = 600000
            scopes="wfsv6.io/rest"
        }
    ]

    cap {
        newlookuptype=newVehicle
        usedlookuptype=platinumPlusVehicle
        car {
            newlookuptype=newVehicle
            usedlookuptype=platinumPlusVehicle
        }
        lcv {
            newlookuptype=newVehicle
            usedlookuptype=platinumPlusVehicle
        }
        hcv {
            newlookuptype=newVehicle
            usedlookuptype=platinumPlusVehicle
        }
        bike {
            newlookuptype=newVehicle
            usedlookuptype=platinumPlusVehicle
        }
    }
}
```

- Make sure you've enabled the HPI CAP asset populator
- `cd ~ && cd git/wiremocks && java -jar wiremock-standalone-3.9.1.jar --port 8081 --root-dir hpi --global-response-templating --verbose --print-all-network-traffic` use latter half of this to launch, make sure you've put wiremock jar in wiremock repo
- Go into `Admin -> Plan Config -> Finance Product -> Then into the asset you want` and add CAP/Product code to it