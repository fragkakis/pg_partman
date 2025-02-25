Migrating An Existing Partition Set to PG Partition Manager
===========================================================

This document is an aid for migrating an existing, natively partitioned table set to using pg_partman. For migrating a trigger-based partition set to native partitioning, see the document `migrate_to_native.md` for assistance with doing so. You can then return to this document to have that partition set managed by pg_partman.

As always, if you can first test this migration on a development system, it is highly recommended. The full data set is not needed to test this and just the schema with a smaller set of data in each child should be sufficient enough to make sure it works properly.

The following are the example tables we will be using:

```
CREATE TABLE tracking.hits_time (id int GENERATED BY DEFAULT AS IDENTITY NOT NULL, start timestamptz DEFAULT now() NOT NULL) PARTITION BY RANGE (start);
CREATE INDEX ON tracking.hits_time (start);
CREATE TABLE tracking.hits_time2023_02_26 partition of tracking.hits_time FOR VALUES FROM ('2023-02-26'::timestamptz) TO ('2023-03-05'::timestamptz);
CREATE TABLE tracking.hits_time2023_03_05 partition of tracking.hits_time FOR VALUES FROM ('2023-03-05'::timestamptz) TO ('2023-03-12'::timestamptz);
CREATE TABLE tracking.hits_time2023_03_12 partition of tracking.hits_time FOR VALUES FROM ('2023-03-12'::timestamptz) TO ('2023-03-19'::timestamptz);

INSERT INTO tracking.hits_time VALUES (1, generate_series('2023-02-27 01:00:00'::timestamptz, '2023-03-04 23:00:00'::timestamptz, '1 hour'::interval));
INSERT INTO tracking.hits_time VALUES (2, generate_series('2023-03-06 01:00:00'::timestamptz, '2023-03-11 23:00:00'::timestamptz, '1 hour'::interval));
INSERT INTO tracking.hits_time VALUES (3, generate_series('2023-03-12 01:00:00'::timestamptz, '2023-03-18 23:00:00'::timestamptz, '1 hour'::interval));

\d+ tracking.hits_time
                                                      Partitioned table "tracking.hits_time"
 Column |           Type           | Collation | Nullable |             Default              | Storage | Compression | Stats target | Description
--------+--------------------------+-----------+----------+----------------------------------+---------+-------------+--------------+-------------
 id     | integer                  |           | not null | generated by default as identity | plain   |             |              |
 start  | timestamp with time zone |           | not null | now()                            | plain   |             |              |
Partition key: RANGE (start)
Indexes:
    "hits_time_start_idx" btree (start)
Partitions: tracking.hits_time20230226 FOR VALUES FROM ('2023-02-26 00:00:00-05') TO ('2023-03-05 00:00:00-05'),
            tracking.hits_time20230305 FOR VALUES FROM ('2023-03-05 00:00:00-05') TO ('2023-03-12 00:00:00-05'),
            tracking.hits_time20230312 FOR VALUES FROM ('2023-03-12 00:00:00-05') TO ('2023-03-19 00:00:00-04')

```
```
CREATE TABLE tracking.hits_id (id int GENERATED BY DEFAULT AS IDENTITY NOT NULL, start timestamptz DEFAULT now() NOT NULL) PARTITION BY RANGE (id);
CREATE INDEX ON tracking.hits_id (id);
CREATE TABLE tracking.hits_id1000 partition of tracking.hits_id FOR VALUES FROM (1000) TO (2000);
CREATE TABLE tracking.hits_id2000 partition of tracking.hits_id FOR VALUES FROM (2000) TO (3000);
CREATE TABLE tracking.hits_id3000 partition of tracking.hits_id FOR VALUES FROM (3000) TO (4000);

INSERT INTO tracking.hits_id VALUES (generate_series(1000,1999), now());
INSERT INTO tracking.hits_id VALUES (generate_series(2000,2999), now());
INSERT INTO tracking.hits_id VALUES (generate_series(3000,3999), now());

\d+ tracking.hits_id
                                                       Partitioned table "tracking.hits_id"
 Column |           Type           | Collation | Nullable |             Default              | Storage | Compression | Stats target | Description
--------+--------------------------+-----------+----------+----------------------------------+---------+-------------+--------------+-------------
 id     | integer                  |           | not null | generated by default as identity | plain   |             |              |
 start  | timestamp with time zone |           | not null | now()                            | plain   |             |              |
Partition key: RANGE (id)
Indexes:
    "hits_id_id_idx" btree (id)
Partitions: tracking.hits_id1000 FOR VALUES FROM (1000) TO (2000),
            tracking.hits_id2000 FOR VALUES FROM (2000) TO (3000),
            tracking.hits_id3000 FOR VALUES FROM (3000) TO (4000)
```
```
CREATE TABLE tracking.hits_stufftime (id int GENERATED BY DEFAULT AS IDENTITY NOT NULL, start timestamptz DEFAULT now() NOT NULL) PARTITION BY RANGE (start);
CREATE INDEX ON tracking.hits_stufftime (start);
CREATE TABLE tracking.hits_stufftimeaa partition of tracking.hits_stufftime FOR VALUES FROM ('2023-01-01'::timestamptz) TO ('2023-01-08'::timestamptz);
CREATE TABLE tracking.hits_stufftimebb partition of tracking.hits_stufftime FOR VALUES FROM ('2023-01-08'::timestamptz) TO ('2023-01-15'::timestamptz);
CREATE TABLE tracking.hits_stufftimecc partition of tracking.hits_stufftime FOR VALUES FROM ('2023-01-15'::timestamptz) TO ('2023-01-22'::timestamptz);

INSERT INTO tracking.hits_stufftime VALUES (1, generate_series('2023-01-02 01:00:00'::timestamptz, '2023-01-06 23:00:00'::timestamptz, '1 hour'::interval));
INSERT INTO tracking.hits_stufftime VALUES (2, generate_series('2023-01-09 01:00:00'::timestamptz, '2023-01-13 23:00:00'::timestamptz, '1 hour'::interval));
INSERT INTO tracking.hits_stufftime VALUES (3, generate_series('2023-01-15 01:00:00'::timestamptz, '2023-01-20 23:00:00'::timestamptz, '1 hour'::interval));

\d+ tracking.hits_stufftime
                                                   Partitioned table "tracking.hits_stufftime"
 Column |           Type           | Collation | Nullable |             Default              | Storage | Compression | Stats target | Description
--------+--------------------------+-----------+----------+----------------------------------+---------+-------------+--------------+-------------
 id     | integer                  |           | not null | generated by default as identity | plain   |             |              |
 start  | timestamp with time zone |           | not null | now()                            | plain   |             |              |
Partition key: RANGE (start)
Indexes:
    "hits_stufftime_start_idx" btree (start)
Partitions: tracking.hits_stufftimeaa FOR VALUES FROM ('2023-01-01 00:00:00-05') TO ('2023-01-08 00:00:00-05'),
            tracking.hits_stufftimebb FOR VALUES FROM ('2023-01-08 00:00:00-05') TO ('2023-01-15 00:00:00-05'),
            tracking.hits_stufftimecc FOR VALUES FROM ('2023-01-15 00:00:00-05') TO ('2023-01-22 00:00:00-05')
```
```
CREATE TABLE tracking.hits_stuffid (id int GENERATED BY DEFAULT AS IDENTITY NOT NULL, start timestamptz DEFAULT now() NOT NULL) PARTITION BY RANGE (id);
CREATE INDEX ON tracking.hits_stuffid (id);
CREATE TABLE tracking.hits_stuffidaa partition of tracking.hits_stuffid FOR VALUES FROM (1000) TO (2000);
CREATE TABLE tracking.hits_stuffidbb partition of tracking.hits_stuffid FOR VALUES FROM (2000) TO (3000);
CREATE TABLE tracking.hits_stuffidcc partition of tracking.hits_stuffid FOR VALUES FROM (3000) TO (4000);

See below for data inserted

\d+ tracking.hits_stuffid
                                                    Partitioned table "tracking.hits_stuffid"
 Column |           Type           | Collation | Nullable |             Default              | Storage | Compression | Stats target | Description
--------+--------------------------+-----------+----------+----------------------------------+---------+-------------+--------------+-------------
 id     | integer                  |           | not null | generated by default as identity | plain   |             |              |
 start  | timestamp with time zone |           | not null | now()                            | plain   |             |              |
Partition key: RANGE (id)
Indexes:
    "hits_stuffid_id_idx" btree (id)
Partitions: tracking.hits_stuffidaa FOR VALUES FROM (1000) TO (2000),
            tracking.hits_stuffidbb FOR VALUES FROM (2000) TO (3000),
            tracking.hits_stuffidcc FOR VALUES FROM (3000) TO (4000)
```
Step 1
------
Disable calls to the run_maintenance()

If you have any partitions currently maintained by pg_partman, you may be calling this already for them. They should be fine for the period of time this conversion is being done. This is to avoid any issues with only a partial configuration existing during conversion. If you are using the background worker, commenting out the "pg_partman_bgw.dbname" parameter in postgresql.conf and then reloading (SELECT pg_reload_conf();) should be sufficient to stop it from running. If you're running pg_partman on several databases in the cluster and you don't want to stop them all, you can also just remove the one you're doing the migration on from that same parameter.


Step 2
------
Stop all writes to the partition set being migrated if possible. If you cannot do this for the period of time the conversion will take, all of these following steps must be done in a single transaction to avoid write errors due table names changing.


Step 3
------
Rename the existing partitions to new naming convention. pg_partman uses a static pattern of suffixes for all partitions, both time & serial. All suffixes start with the string "_p" and are listed here for reference.

    _pYYYYMMDD          - All time intervals greater than 1 day
    _pYYYYMMDD_HH24MISS - All time intervals less than 1 day
    _p#####             - Serial/ID partition has a suffix that is the value of the lowest possible entry in that table (Ex: _p10, _p20000, etc)

You can use custom datetime_string formats to change the suffix the children will get with pg_partman 5.0.0 and greater, but that is not covered in this tutorial. This only covers how to convert to using pg_partman's default suffixes and pg_partman always adds `_p` to the beginning to distinguish the suffix boundary.

Step 3a
-------
For converting either time or serial based partition sets, if you have the lower boundary value as part of the partition name already, then it's simply a matter of doing a rename with some substring formatting since that's the pattern that pg_partman itself uses. Say your table was partitioned weekly and your original format just had the first day of the week (sunday) for the partition name (as in the example above). You can see below that we had 3 partitions with the old naming pattern of "YYYYMMDD" with no _p prefix. Looking at the list above, you can see the new weekly pattern that pg_partman uses.

So a query like the following which first extracts the original name then reformats the suffix would work. It doesn't actually do the renaming, it just generates all the ALTER TABLE statements for you for all the child tables in the set. If all of them don't quite have the same pattern for some reason, you can easily just re-run this, editing things as needed, and filter the resulting list of ALTER TABLE statements accordingly. Note the `to_timestamp()` function is given the old datetime string pattern and the `to_char()` function is given the new string pattern.

```
SELECT format(
    'ALTER TABLE %I.%I RENAME TO %I;'
    , n.nspname
    , c.relname
    , substring(c.relname from 1 for 9) || '_p' || to_char(to_timestamp(substring(c.relname from 10), 'YYYY_MM_DD'), 'YYYYMMDD')
)
        FROM pg_inherits h
        JOIN pg_class c ON h.inhrelid = c.oid
        JOIN pg_namespace n ON c.relnamespace = n.oid
        WHERE h.inhparent = 'tracking.hits_time'::regclass
        ORDER BY c.relname;
                               ?column?  
-----------------------------------------------------------------------
 ALTER TABLE tracking.hits_time20230226 RENAME TO hits_time_p20230226;
 ALTER TABLE tracking.hits_time20230305 RENAME TO hits_time_p20230305;
 ALTER TABLE tracking.hits_time20230312 RENAME TO hits_time_p20230312;

```

Running that should rename your tables to look like this now:
```
 table_schema |     table_name  
--------------+---------------------
 tracking     | hits_time_p20230226
 tracking     | hits_time_p20230305
 tracking     | hits_time_p20230312

```

If you're migrating a serial/id based partition set, and also have the naming convention with the lowest possible value, you'd do something very similar. Everything would be the same as the time-series one above except the renaming would be slightly different. Using my second example table above, it would be something like this.

```
SELECT format(
    'ALTER TABLE %I.%I RENAME TO %I;'
    , n.nspname
    , c.relname
    , substring(c.relname from 1 for 7) || '_p' || substring(c.relname from 8)
)
    FROM pg_inherits h
    JOIN pg_class c ON h.inhrelid = c.oid
    JOIN pg_namespace n ON c.relnamespace = n.oid
    WHERE h.inhparent = 'tracking.hits_id'::regclass
    ORDER by c.relname;
                         ?column?  
-----------------------------------------------------------
 ALTER TABLE tracking.hits_id1000 RENAME TO hits_id_p1000;
 ALTER TABLE tracking.hits_id2000 RENAME TO hits_id_p2000;
 ALTER TABLE tracking.hits_id3000 RENAME TO hits_id_p3000;

```
```
 table_schema |  table_name  
--------------+---------------
 tracking     | hits_id
 tracking     | hits_id_p1000
 tracking     | hits_id_p2000
 tracking     | hits_id_p3000
```
Step 3b
-------
If your partitioned sets are named in a manner that relates differently to the data contained, or just doesn't relate at all, you'll instead have to do the renaming based off the lowest value in the control column instead. I'll be using the example above with the _aa, _bb, & _cc suffixes.

We'll be using the the `hits_stufftime` table in the first example here which has child tables that don't relate at all to the data contained.

This next step takes advantage of anonymous code blocks. It's basically writing pl/pgsql function code without creating an actual function. Just run this block of code, adjusting values as needed, right inside a psql session. Note that in PostgreSQL, weeks start on Monday's by default for the date_trunc function. However, say we wanted them to start on Sundays like they did for our other time partitioning example to keep things consistent. In that case we have to do a little extra date math to get that result.
```
DO $rename$
DECLARE
    v_min_val           timestamp;
    v_row               record;
    v_sql               text;
BEGIN

-- Adjust your parent table name in the for loop query
FOR v_row IN
    SELECT n.nspname AS child_schema, c.relname AS child_table
        FROM pg_inherits h
        JOIN pg_class c ON h.inhrelid = c.oid
        JOIN pg_namespace n ON c.relnamespace = n.oid
        WHERE h.inhparent = 'tracking.hits_stufftime'::regclass
        ORDER BY c.relname
LOOP
    v_min_val := NULL;
    -- Substitute your control column's name here in the min() function
    v_sql := format('SELECT min(start) FROM %I.%I', v_row.child_schema, v_row.child_table);
    EXECUTE v_sql INTO v_min_val;

    -- Adjust the date_trunc here to account for whatever your partitioning interval is.
    -- This is a trick to have the week begin on sunday since PG defaults to monday
    v_min_val := date_trunc('week', v_min_val + '1 day'::interval) - '1 day'::interval;

    -- Build the sql statement to rename the child table
    v_sql := format('ALTER TABLE %I.%I RENAME TO %I'
            , v_row.child_schema
            , v_row.child_table
            , substring(v_row.child_table from 1 for 14)||'_p'||to_char(v_min_val, 'YYYYMMDD'));

    -- I just have it outputting the ALTER statement for review. If you'd like this code to actually run it, uncomment the EXECUTE below.
    RAISE NOTICE '%', v_sql;
    -- EXECUTE v_sql;
END LOOP;

END
$rename$;
```
This will output something like this:
```
NOTICE:  ALTER TABLE tracking.hits_stufftimeaa RENAME TO hits_stufftime_p20230101
NOTICE:  ALTER TABLE tracking.hits_stufftimebb RENAME TO hits_stufftime_p20230108
NOTICE:  ALTER TABLE tracking.hits_stufftimecc RENAME TO hits_stufftime_p20230115
```
I'd recommend running it at least once with the final EXECUTE commented out to review what it generates. If it looks good, you can uncomment the EXECUTE and rename your tables!

If you've got a serial/id partition set, calculating the proper suffix value can be done by taking advantage of modulus arithmetic. Assume the following values in the tracking.hits_stuffid table:
```
INSERT INTO tracking.hits_stuffid VALUES (generate_series(1100,1294), now());
INSERT INTO tracking.hits_stuffid VALUES (generate_series(2400,2991), now());
INSERT INTO tracking.hits_stuffid VALUES (generate_series(3602,3843), now());
```
We'll be partitioning by 1000 again and you can see none of the minimum values are that even.
```
DO $rename$
DECLARE
    v_min_val           bigint;
    v_row               record;
    v_sql               text;
BEGIN

-- Adjust your parent table name in the for loop query
FOR v_row IN
    SELECT n.nspname AS child_schema, c.relname AS child_table
        FROM pg_inherits h
        JOIN pg_class c ON h.inhrelid = c.oid
        JOIN pg_namespace n ON c.relnamespace = n.oid
        WHERE h.inhparent = 'tracking.hits_stuffid'::regclass
        ORDER BY c.relname
LOOP
    -- Substitute your control column's name here in the min() function
    v_sql := format('SELECT min(id) FROM %I.%I', v_row.child_schema, v_row.child_table);
    EXECUTE v_sql INTO v_min_val;

    -- Adjust the numerical value after the % to account for whatever your partitioning interval is.
    v_min_val := v_min_val - (v_min_val % 1000);

    -- Build the sql statement to rename the child table
    v_sql := format('ALTER TABLE %I.%I RENAME TO %I'
            , v_row.child_schema
            , v_row.child_table
            , substring(v_row.child_table from 1 for 12)||'_p'||v_min_val::text);

    -- I just have it outputting the ALTER statement for review. If you'd like this code to actually run it, uncomment the EXECUTE below.
    RAISE NOTICE '%', v_sql;
    -- EXECUTE v_sql;
END LOOP;

END
$rename$;
```
You can see this makes nice even partition names:
```
NOTICE:  ALTER TABLE tracking.hits_stuffidaa RENAME TO hits_stuffid_p1000
NOTICE:  ALTER TABLE tracking.hits_stuffidbb RENAME TO hits_stuffid_p2000
NOTICE:  ALTER TABLE tracking.hits_stuffidcc RENAME TO hits_stuffid_p3000
```

Step 4
------
Setup pg_partman to manage your partition set.

I mentioned at the beginning that if you had ongoing writes, pretty much everything from Step 2 and on had to be done in a single transaction. Even if you don't have to worry about writes, I'd still highly recommend doing step 4 in the same transaction.

Note that we're doing weekly partitioning, but pg_partman only knows that interval as a period of 7 days. So say we're migrating this parent table on March 31, 2023 with the child tables that we have already. If we tried to just call the create_parent() function without telling it when to start the partition set, it would assume our week is starting on a Friday, throwing off both the names of the child tables as well as the interval boundaries. We want to be sure our partition set is being configured to start the weekly partitions on a Sunday, so we use the `p_start_partition` parameter to tell it that (March 26th is the Sunday for the week of March 31st).

You may or may not need to set the starting partition parameter. It all depends on the interval you are using, so please test things to see if they work as expected before running on production.

```
SELECT partman.create_parent('tracking.hits_time', 'start', '1 week', p_start_partition := '2023-03-26 00:00:00');
COMMIT;
```
This single function call will add your old partition set into pg_partman's configuration and possibly create some new child tables as well. pg_partman always keeps a minimum number of future partitions premade (based on the *premake* value in the config table or as a parameter to the create_parent() function), so if you don't have those yet, this step will take care of that as well. Adjust the parameters as needed and see the documentation for additional options that are available. This call matches the time partition used in the example so far.

```
 \d+ tracking.hits_time
                                                      Partitioned table "tracking.hits_time"
 Column |           Type           | Collation | Nullable |             Default              | Storage | Compression | Stats target | Description
--------+--------------------------+-----------+----------+----------------------------------+---------+-------------+--------------+-------------
 id     | integer                  |           | not null | generated by default as identity | plain   |             |              |
 start  | timestamp with time zone |           | not null | now()                            | plain   |             |              |
Partition key: RANGE (start)
Indexes:
    "hits_time_start_idx" btree (start)
Partitions: tracking.hits_time_p20230226 FOR VALUES FROM ('2023-02-26 00:00:00-05') TO ('2023-03-05 00:00:00-05'),
            tracking.hits_time_p20230305 FOR VALUES FROM ('2023-03-05 00:00:00-05') TO ('2023-03-12 00:00:00-05'),
            tracking.hits_time_p20230312 FOR VALUES FROM ('2023-03-12 00:00:00-05') TO ('2023-03-19 00:00:00-04'),
            tracking.hits_time_p20230326 FOR VALUES FROM ('2023-03-26 00:00:00-04') TO ('2023-04-02 00:00:00-04'),
            tracking.hits_time_p20230402 FOR VALUES FROM ('2023-04-02 00:00:00-04') TO ('2023-04-09 00:00:00-04'),
            tracking.hits_time_p20230409 FOR VALUES FROM ('2023-04-09 00:00:00-04') TO ('2023-04-16 00:00:00-04'),
            tracking.hits_time_p20230416 FOR VALUES FROM ('2023-04-16 00:00:00-04') TO ('2023-04-23 00:00:00-04'),
            tracking.hits_time_p20230423 FOR VALUES FROM ('2023-04-23 00:00:00-04') TO ('2023-04-30 00:00:00-04'),
            tracking.hits_time_default DEFAULT
```

By default, the premake value is 4, so it created 4 weeks in the future. And since this was the initial creation, it also creates 4 tables in the past. Even though we already had some tables there that covered those past 4 weeks, pg_partman saw they were there and just worked anyway. If we hadn't of passed the p_start_partition parameter, PostgreSQL actually would have thrown an error since the past tables with different boundaries it would've tried to create would overlap the boundary of an already existing older table.

This final step is exactly the same no matter the partitioning type or interval, so once you reach here, run COMMIT and you're done! Schedule the run_maintenance() function to run (either via cron or the BGW) and future partition maintenance will be handled for you. Review the pg_partman.md documentation for additional configuration options.

I did the time examples here with a weekly partitioning set starting on a Sunday to show that there may be some caveats to your migration to pg_partman that need to be considered. If you're doing something simpler like daily or serial integer values, it may not be quite as complicated. Or if you're doing something more complex like a 9 week interval, there may be even more caveats to consider that may not seem obvious at first without extensive testing in development. So please plan your migration carefully.

If you have any issues with this migration document, please create an issue on Github.
