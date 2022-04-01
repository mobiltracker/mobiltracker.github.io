# How to Safely Migrate a SQL Table

## Introduction

This documentation will show how to safely migrate a SQL Table. As an example we'll use the migration of the Routes table to RoutesTemplates.

## Step by Step

1. Create a new table.
2. Create a trigger in the old table to replicate the new inserts to the new table. See an example:

```sql
DROP TRIGGER replicationTrigger;
GO

CREATE TRIGGER replicationTrigger ON routes
FOR INSERT
AS

-- Allows the creation of a custom RouteTemplateId
SET IDENTITY_INSERT Route_RouteTemplates ON

INSERT INTO Route_RouteTemplates
(RouteTemplateId, Name, VanId, SchedulerTime, Mon, Tue, Wed, Thu, Fri, Sat, Sun)
SELECT
RouteId, Name, VanId, SchedulerTime, Mon, Tue, Wed, Thu, Fri, Sat, Sun
FROM inserted

SET IDENTITY_INSERT Route_RouteTemplates OFF
GO
```

3. Migrate the existing data of the old table to the new one, as show bellow:

```sql
set IDENTITY_INSERT Route_RouteTemplates on;

INSERT INTO Route_RouteTemplates
(RouteTemplateId, [Name], VanId, SchedulerTime, mon, tue, wed, thu, fri, sat, sun)
SELECT
r.RouteId, r.Name, r.VanId, r.SchedulerTime, r.Mon, r.Tue, r.Wed, r.Thu, r.Fri, r.Sat, r.Sun
FROM Routes as R
LEFT JOIN RouteRouteTemplates as rt on r.RouteId = rt.RouteTemplateId
where rt.RouteTemplateId is null; -- filter columns that already are on the new table

# set IDENTITY_INSERT Route_RouteTemplates off;
```

4. Migrate the readings of the old table to the new one (API calls, scripts)

5. Migrate the writings of the old table to the new one (API calls, scripts)

> It's very important that the readings migrations happen before, since once the writings migration is done, the old table will be outdated and that can interfere in the readings.

6. Migrate the FKs from the old tables to the new one.

7. Insert a `_toBeDeleted` in the tables to be deleted.

8. After a few weeks, delete de old table.
