---
id: tcglsdcp11cy35demtm89b5
title: SX-67033-flyway-sbg-Oracle
desc: ''
updated: 1707406108469
created: 1707403929050
---
[[Oracle-18c|Work.Sopra.Tools.Oracle-18c]]

### Patrick's proposed solution
- delete from schema version
- run tomcat server and see migration fail
- fix oracle sql on local
- wmod ticket
- jenkins build -> classic -> take snapshot to local
- if works, update pom in 578 and HoB

- enable flyway purge and repair locally

### Solution
So this ticket took awhile and was just around making an Oracle step idempotent.

Idempotency of a migration step means it could be run any *N* number of times and the result wouldn't change, this is easy in Postgres as there's things like `create if not exists <command>` however in Oracle 18 this isn't as simple. So the solution I fell on was:
```SQL
DECLARE
    event_journal_exists number := 0;
    event_j_ord_exists number := 0;
    event_tag_exists number := 0;
    snapshot_exists number := 0;
    event_j_ord_trg_exists number := 0;
BEGIN

        SELECT COUNT(*) INTO event_j_ord_exists
        FROM USER_SEQUENCES
        WHERE upper(SEQUENCE_NAME) = 'EVENT_JOURNAL__ORDERING_SEQ';

        SELECT COUNT(*) INTO event_journal_exists
        FROM USER_TABLES
        WHERE upper(TABLE_NAME) = 'EVENT_JOURNAL';

        SELECT COUNT(*) INTO event_j_ord_trg_exists
        FROM USER_TRIGGERS
        WHERE upper(TRIGGER_NAME) = 'EVENT_JOURNAL__ORDERING_TRG';

        SELECT COUNT(*) INTO event_tag_exists
        FROM USER_TABLES
        WHERE upper(TABLE_NAME) = 'EVENT_TAG';

        SELECT COUNT(*) INTO snapshot_exists
        FROM USER_TABLES
        WHERE upper(TABLE_NAME) = 'SNAPSHOT';


        if (event_j_ord_exists = 0) THEN
            EXECUTE IMMEDIATE 'CREATE SEQUENCE EVENT_JOURNAL__ORDERING_SEQ START WITH 1 INCREMENT BY 1 NOMAXVALUE';
        END if;

        if (event_journal_exists = 0) THEN
            EXECUTE IMMEDIATE 'CREATE TABLE EVENT_JOURNAL (
                                                              ORDERING NUMERIC UNIQUE,
                                                              DELETED CHAR(1) DEFAULT 0 NOT NULL check (DELETED in (0, 1)),
                                                              PERSISTENCE_ID VARCHAR(255) NOT NULL,
                                                              SEQUENCE_NUMBER NUMERIC NOT NULL,
                                                              WRITER VARCHAR(255) NOT NULL,
                                                              WRITE_TIMESTAMP NUMBER(19) NOT NULL,
                                                              ADAPTER_MANIFEST VARCHAR(255),
                                                              EVENT_PAYLOAD BLOB NOT NULL,
                                                              EVENT_SER_ID NUMBER(10) NOT NULL,
                                                              EVENT_SER_MANIFEST VARCHAR(255),
                                                              META_PAYLOAD BLOB,
                                                              META_SER_ID NUMBER(10),
                                                              META_SER_MANIFEST VARCHAR(255),
                                                              PRIMARY KEY(PERSISTENCE_ID, SEQUENCE_NUMBER)
                               )';
        END if;

        if (event_j_ord_trg_exists >= 0) THEN
            EXECUTE IMMEDIATE 'CREATE OR REPLACE TRIGGER EVENT_JOURNAL__ORDERING_TRG before insert on EVENT_JOURNAL REFERENCING NEW AS NEW FOR EACH ROW WHEN (new.ORDERING is null) begin select EVENT_JOURNAL__ORDERING_seq.nextval into :new.ORDERING from sys.dual; end;';
        END if;

        if (event_tag_exists = 0) THEN
            EXECUTE IMMEDIATE 'CREATE TABLE EVENT_TAG (
                                                          EVENT_ID NUMERIC NOT NULL,
                                                          TAG VARCHAR(255) NOT NULL,
                                                          PRIMARY KEY(EVENT_ID, TAG),
                                                          FOREIGN KEY(EVENT_ID) REFERENCES EVENT_JOURNAL(ORDERING)
                                                              ON DELETE CASCADE
                               )';
        END if;

        if (snapshot_exists = 0) THEN
            EXECUTE IMMEDIATE 'CREATE TABLE SNAPSHOT (
                                                         PERSISTENCE_ID VARCHAR(255) NOT NULL,
                                                         SEQUENCE_NUMBER NUMERIC NOT NULL,
                                                         CREATED NUMERIC NOT NULL,
                                                         SNAPSHOT_SER_ID NUMBER(10) NOT NULL,
                                                         SNAPSHOT_SER_MANIFEST VARCHAR(255),
                                                         SNAPSHOT_PAYLOAD BLOB NOT NULL,
                                                         META_SER_ID NUMBER(10),
                                                         META_SER_MANIFEST VARCHAR(255),
                                                         META_PAYLOAD BLOB,
                                                         PRIMARY KEY(PERSISTENCE_ID,SEQUENCE_NUMBER)
                               )';
        END if;
END;
/
CREATE OR REPLACE PROCEDURE "reset_sequence"
        IS
        l_value NUMBER;
BEGIN
    EXECUTE IMMEDIATE 'SELECT EVENT_JOURNAL__ORDERING_SEQ.nextval FROM dual' INTO l_value;
    EXECUTE IMMEDIATE 'ALTER SEQUENCE EVENT_JOURNAL__ORDERING_SEQ INCREMENT BY -' || l_value || ' MINVALUE 0';
    EXECUTE IMMEDIATE 'SELECT EVENT_JOURNAL__ORDERING_SEQ.nextval FROM dual' INTO l_value;
    EXECUTE IMMEDIATE 'ALTER SEQUENCE EVENT_JOURNAL__ORDERING_SEQ INCREMENT BY 1 MINVALUE 0';
END;
```

Where you can see there are counters for the checking if something exists and then you run it against the table that would hold the value for it.

### Painful points
This uses PL/SQL so there were some errors that came up quite consistently while writing this fix.

[Useful documentation for PLSQL errors](https://docs.oracle.com/cd/B13789_01/appdev.101/b10807/07_errs.htm)

- `ORA-00933: SQL Command not properly ended` - [This stackoverflow comment](https://stackoverflow.com/a/33291713/12769072) saved me a lot of time, I originally had semi-colons inside the inverted commas for table creations, and there didn't need to be these
- `ORA-06512: at line 54` - this was to do with `'CREATE OR REPLACE TRIGGER EVENT_JOURNAL__ORDERING_TRG before insert on EVENT_JOURNAL REFERENCING NEW AS NEW FOR EACH ROW WHEN (new.ORDERING is null) begin select EVENT_JOURNAL__ORDERING_seq.nextval into :new.ORDERING from sys.dual; end;';` as I had removed the semi-colon inside the quotes because of another error, but a semi-colon after end is mandatory
    - Also this error would come up `ORA-06550: line num, column num: str` directly telling me there was a problem at the end of that string, so I re-added the semi-colon