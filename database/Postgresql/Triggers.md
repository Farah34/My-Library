-- Introduction to triggers
/*

In the course of executing a business decision, or a business logic in a database application, 
special occasions may arise where if some specific action occurs,
you want that action to cause another action, or perhaps a succession of actions, to occur.

In a way, that first action TRIGGERS the execution of the following actions. 

The SQL provides the TRIGGER mechanism to provide this capability.

A trigger is an action or event that causes another event to occur

A triggering SQL statement causes another SQL statement (the trigger statement) to be executed.


Triggers can also be used to keep a database consistent. 
For example, in an order entry application, 
an order for a specific product can trigger a statement that can change 
the status of that product in the master inventory table status from available to reserved. 

Similarly, the deletion of a row in the order table can trigger a statement that 
changes the status of the subject product status from reserved to available.

*/


-- What is a Trigger????????????
-- #################################
/*

   1. A 'Trigger' is defined as any event that sets a course of action in a motion.
   
   2. A postgreSQL trigger is a fucntion invoked automatically whenever 'an event' associated with a table occurs.
   
   3. An event could be any of the following:
   
      - INSERT - UPDATE - DELETE - TRUNCATE
	  
   4. A trigger can be associated with a specified 
      - Table, - View, - A foreign table
	  
   5. A trigger is a scpecial 'user-defined function'.
   
   6. The difference between a trigger and a user-defined fuction is that a trigger is automatically invoked
   when a triggering event occurs.
   
   7. we can create trigger
      
	  - Before
	      - If the trigger is invoked before an event, it can skip the operation for the current row or even change
		  the row being updated or inserted.
	  - After
	      - All changes are available to the trigger.
	  - Instead
	      - of the events/operation.
	
	8. If there are more than one triggers on a table, then they are fired in alphabaticall orders.
	   First, all of those BEFORE triggers happen in alphabetical order. Then, PostgreSQL performs the 
	   row operation that the trigger has been fired for, and continues executing AFTER the triggers in 
	   alphabetical order.
	   
	   In other words, the execution order of triggers is absolutely deterministic, and the number of triggers
	   is basically unlimited.
	   
	9. Triggers can modify data before or after the actual modification has happened. IN general, this is a good
       way to verify data and to error out if some custom restrictions are violated.
	   
   10. There are two main characteristics that make triggers different than stored procedures:
   
       - Triggers cannot be manually executed by the user.
	   - There is no chance for triggers to receive parameters.
	   
 */
 
 
 -- Types of Triggers:
 -- ##########################
 /*
 
   1. Row Level Trigger
      If the trigger is marked FOR EACH ROW then the trigger function will be called for each row that is getting
	  modified by the event.
	  
	  eg. If we UPDATE 20 rows in the table, the UPDATE trigger function will be called 20 times, once for each
	  updated row.
	  
   2. Statement Levl Trigger
      The FOR EACH STATEMENT option will call the trigger function only ONCE for each statement, regardless of the 
	  number of the rows getting modified.
	  
      - As you can see, the differences between the two kinds are HOW MANY TIMES the tigger is invoked and AT WHAT TIME.
	  - In PostgreSQL, triggers have become more powerful over the years, and they now provide a rich set of features.
	  
*/
	
	
-- Trigger Table
-- ##############

/* 

 Following table will help you to understand when to used wich triggers in what event (INSERT, UPDATE, DELETE, TRUNCATE);
 
 When           Event                         Row-level               Statement-level
 ------------   --------                     -----------              -----------------
                INSERT/UPDATE/DELETE         Tables                   Tables and views

 BEFORE         TRUNCATE                        --                       Tables
                 
				 INSERT/UPDATE/DELETE        Tables                   Tables and views
				 
 AFTER          TRUNCATE                       --                        Tables
 
 
 INSTEAD OF     TRUNCATE                       --                         ----
 
 */
 
 -- Bros and cons of using Triggers
 -- ################################
 /*
   
 Pros
 ##########
 
 1. Triggers are not defficult to code. The fact that they are coded like stored procedures which makes getting
    started with triggers eassy.
	
 2. Triggers allow you to create 'basic auditing', By using the deleted table inside a trigger you can build a decent
 audit solution that inserts the contents of the deleted table data into an audit table which holds the data that
 is either being removed by a DELETE statement or being changed by an UPDATE statement.
 
 3. You can call stored procedures and functions from inside a trigger.
 
 4. Triggers are useful when you need to validate inserted or updated data in batches instead of row by row.
    Think about it, in a trigger's code you have the inserted and deleted tables that hold a copy of the data
	that potentially will be stored in the table (the inserted table); and the data that will be removed from the
	table (the deleted table).
	
 5. You can use triggers to implement referential integrity acros databases. Unfortunately SQL Server doesn't 
    allow the creatioin of constraints between objects on different databases, but by using triggers you can
	simulate the behaviour of constraints.
	
 6. Triggers are useful if you need to be sure that certain events always happen when data is inserted, updated
    or deleted. This is the case when you have to deal with complex default values of columns, or modify the data
	of other tables.
	
7. Triggers allow recursion. Triggers are recursive when a trigger on a table performs an action on the base table
   that causes another instance of the trigger to fire. This is useful when you have to solve a self-referencing 
   relation (i.e a constraint to itself).
 
 Cons
 -----
 
 1. Triggers are difficult to loacate unless you have proper documentation because they are invisible to the client.
    For instance, sometimes you execute a DML statement without errors or warnings, say an insert, and you don't see
	it reflected in the table's data. In such case you have to check the table for triggers that may be disallowing
	you to run the insert you wanted.
	
 2. Triggers add overhead to DML statements. Every time you run a DML statement that has a trigger associated to it,
    you are actually executing the DML statement and the trigger; but by definition the DML statement won't end until
	the trigger execution completes. This can create a problem in production.

 3. The problem of using triggers for audit purposes is that when triggers are enabled, they execute always regardless
    of the circumstances that caused the trigger to fire. Forexample, if you only need to audit the data inserted by
	a specific stored procedure and you use a trigger, you may have to delete the rows of the audit table that were 
	created when someone changed data using an ad-hoc query or add more logic into the trigger's code which of course
	impacts performance. In such case you may have to use the OUTPUT clause.
	
 4. If there are many nested triggers it could get very hard to debug and troubleshoot. which consumes development time
    and resources.
	
 5. Recursive triggers are even harder to debug  than the nested triggers.
 
 
 However, triggers are very powerful tools for DBAs!!!
 
*/



-- Triggers Key Points
-- ###################
/*

 1. No trigger on SELECT statement, because SELECT DOES NOT MODIFY ANY ROW
    in such cases lets use views.
 2. Multiple triggers can be used in alphabetical orders.
 3. UDF (User Defined Functions) are allowed in triggers
 4. A single trigger can support MULTIPLE ACTIONS i.e SING LE -> MANY
 
*/


-- Trigger creation process
-- ########################

/* 

To create a new trigger in PostgreSQL, you follow these steps:

  1. First, create a trigger function using CREATE FUNCTION statement.
  2. Second, bind the trigger function to a table by using CREATE TRIGGER statement.

```sql
-- CREATE FUNCTION Syntax

CREATE FUNCTION trigger_function()
   RETURN TRIGGER
   LANGUAGE PLPGSQL
AS $$
BEGIN
   --- trigger logic
END;
$$


-- CREATE TRIGGER function

 CREATE TRIGGER trigger_name
 {BEFORE | AFTER} { event }
 ON table_name
 [FOR [EACH] {ROW | STATEMENT }]
     EXECUTE PROCEDURE trigger_function

```

- First, specify the name of the trigger after the TRIGGER keywords.
  - Second, specify the timing that cause the trigger to fire. It can be BEFORE or AFTER an event occurs.
  - Third, specify the event that invokes the trigger. The event can be INSERT, DELETE, UPDATE or TRUNCATE.
  - Fourth, specify the name of the table associated with the trigger after the ON keyword.

-- Data Auditing with triggers
-- ###########################

-- Lets say we have a 'players' table. Suppose we want to log all data when the name of a player gets change or updated.
-- 1. Lets create the 'players' table

```sql
CREATE TABLE players(
    player_id SERIAL PRIMARY KEY,
	name varchar(100) 
);

-- 2. lets create 'players_audit' table to store all changes

CREATE TABLE players_audits(
    player_audit_id SERIAL PRIMARY KEY,
	player_id INT NOT NULL,
	name VARCHAR(100) NOT NULL,
	edit_date TIMESTAMP NOT NULL
);

-- 3. As we seen earlier, we first create a function and then bind that function to our trigger
-- Lets create a function

  CREATE OR REPLACE FUNCTION fn_players_name_changes_log()
      RETURNS TRIGGER
	  LANGUAGE PLPGSQL
	  AS
  $$
  BEGIN
      -- COMPARE THE new value vs OLD value
	  -- NEW, OLD
	  IF NEW.name <> OLD.name THEN
	     INSERT INTO players_audits
		 (
			 player_id,
			 name,
			 edit_date
		 )
		 VALUES
		 (
			 OLD.player_id,
			 OLD.name,
			 NOW()
		 );
	  
	  END IF;
	  
	  RETURN NEW;
  
  END;
  $$

-- The OLD represents the row before update while the NEW represents the new row that will be updated.

-- 4. Now bind our newly created function to our table 'players' via CREATE TRIGGER statement

 CREATE TRIGGER trg_players_name_changes
     BEFORE UPDATE 
	 ON players
	 FOR EACH ROW 
	 EXECUTE PROCEDURE fn_players_name_changes_log();
	 
	 
-- 5. lets inserting some data

INSERT INTO players (name) VALUES
('linda'),('Adam');

select * from players;


-- 6. Lets UPDATE some data

UPDATE players 
SET name = 'Mohamed' 
WHERE player_id = 2;


select * from players_audits;

```

-- Modify data at INSERT event
-- ###########################

-- Triggers can modify data BEFORE or AFTER the actual modifications has happened. This is a good way to varify
-- data and to error out data mistakes and more

-- Lets demonstrate the use of trigger to 
-- a. Check the inserted data and 
-- b. then change it if needed as per logic

-- 1. Lets create a table

```sql
CREATE TABLE t_temprature_log(
    id_temprature_log SERIAL PRIMARY KEY,
	add_date TIMESTAMP,
	temprature numeric
);


-- 2. Lets create a function to check the inserted data

CREATE OR REPLACE FUNCTION  fn_temprature_value_check_at_insert()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS
$$
	BEGIN
       -- If the temprature < -30 : temprature = 0
	   IF NEW.temprature < -30  THEN
	      NEW.temprature = 0;
	   END IF;
	   RETURN NEW;
	   
	END;
$$

-- 3. Lets bind our function to our table

CREATE TRIGGER trg_temprature_value_check_at_insert
BEFORE INSERT
ON t_temprature_log
FOR EACH ROW
EXECUTE PROCEDURE fn_temprature_value_check_at_insert();


-- Lets insert into some data into t_temprature_log

insert into t_temprature_log(add_date, temprature) VALUES
('2023-10-17',60);

select * from t_temprature_log;

```

-- View Trigger Variables
-- ######################

-- Triggers know a lot about themselves. They can access couple of variables that allow you to write advanced
-- code and more.

-- 1. Lets add a new function which display some key trigger variables data

```sql
CREATE OR REPLACE FUNCTION fn_trigger_variables_display()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS
$$ 
	BEGIN
	
    	RAISE NOTICE 'TG_NAME: %', TG_NAME;
		RAISE NOTICE 'TG_RELNAME: %', TG_RELNAME;
		RAISE NOTICE 'TG_TABLE_SCHEMA: %', TG_TABLE_SCHEMA;
		RAISE NOTICE 'TG_TABLE_NAME: %', TG_TABLE_NAME;
		RAISE NOTICE 'TG_WHEN: %', TG_WHEN;
		RAISE NOTICE 'TG_LEVEL: %', TG_LEVEL;
		RAISE NOTICE 'TG_OP: %', TG_OP;
		RAISE NOTICE 'TG_NARGS: %', TG_NARGS;
		RAISE NOTICE 'TG_ARGV: %', TG_NAME;
		
		RETURN NEW;
		
	END;
	
$$	

-- 2. Lets bind the function to table after the insert

CREATE TRIGGER trg_trigger_variables_display
AFTER INSERT
ON t_temprature_log
FOR EACH ROW 
EXECUTE PROCEDURE fn_trigger_variables_display();
```

-- Disallowing DELETE
 -- ###################

 	What if our business requirements are such that the data can only be added and modified in some tables,
	but not deleted?
	
	One way to handle this will be to just revoke the DELETE rights on these tables from all the users
	(remember to also revoke DELETE from PUBLIC), but this can also be achieved using triggers!!!

-- 1. Lets create a test table

```sql
CREATE TABLE test_delete(
	id INT
);

-- 2. lets insert some data
insert into test_delete (id) values(1),(2);

select * from test_delete;

-- 1. Lets create a generic 'cancel' function for our use case here

CREATE OR REPLACE FUNCTION fn_generic_cancel_op()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS
$$ 
	BEGIN
		IF TG_WHEN = 'AFTER' THEN
			
			RAISE EXCEPTION 'YOU ARE NOT ALLOWED TO % ROWS IN %.%',
			TG_OP, TG_TABLE_SCHEMA, TG_TABLE_NAME;
		END IF;
		
		RAISE NOTICE '% ON ROWS IN %.% WONT HAPPEN',
			TG_OP, TG_TABLE_SCHEMA, TG_TABLE_NAME;
			
		RETURN NULL;
	END;
$$

-- 2. Lets bind the function at AFTER DELETE

CREATE TRIGGER trg_disallow_delete
AFTER DELETE
ON test_delete
FOR EACH ROW
EXECUTE PROCEDURE fn_generic_cancel_op();

-- 3. Now lets delete a record

DELETE FROM test_delete  where id = 1;

-- 4. Did the data get deleted?

SELECT * FROM test_delete;

-- 5. Now lets create another trigger but this time BEFORE DELETE

CREATE TRIGGER trg_skip_delete
BEFORE DELETE
ON test_delete
FOR EACH ROW
EXECUTE PROCEDURE fn_generic_cancel_op();

DELETE FROM test_delete where id = 1;

-- Did the data get deleted?

SELECT * FROM test_delete;

-- This time, the BEFORE trigger canceled the delete and AFTER trigger, although still there, was not reached.

-- The same trigger can also be used to enforce a non-update policy or even disallow inserts to a table that needs
-- to have immutable contents.


```

-- Disallowing TRUNCATE
-- ####################
/* 
	- PostgreSQL TRUNCATE quickly removes all rows from a set of tables. It has the same effect as an unqualified
	DELETE on each table, but since it doesn't actually scan the tables it is faster. Furthermore, it reclaims disk
	space immediately, rather than requiring a subsequent VACUUM operation. This is most userful on large tables.
	
	Problem:  In the previous example, a user can still delete the record using the TRUNCATE!
			
			  TRUNCATE test_delete;
			  
			  So we will work on Disallowing TRUNCATE here.
			  
	Solution: While we can't simply skip TRUNCATE by returning NULL as opposed to the row-level  BEFORE triggers,
	          however, we still make it impossible by raising an error if TRUNCATE is attempted.
			  
*/

-- 1. Lets create an AFTER trigger

```sql
CREATE TRIGGER trg_disallow_truncate
AFTER TRUNCATE
ON test_delete
FOR EACH STATEMENT 
EXECUTE PROCEDURE fn_generic_cancel_op();

-- 3. Lets truncate the data

TRUNCATE test_delete;
```


-- The audit trigger
-- #################

/*
	1. One of the most common uses of triggers is to log data changes to a tables in a consistent and transparent
	manner.
	
	2. When creating an audit trigger, we first must decide what we want to log.
	
	3. A logical set of things that can be logged are;
		- who changed the data,
		- when the data was changed,
		- and which operation changed the data.
*/

-- 1. Lets create a master table 'audit'

```sql
CREATE TABLE audit(
	id INT	
);

-- 2. Lets create an audit_log table

CREATE TABLE audit_log(
	username TEXT,
	add_time TIMESTAMP,
	table_name TEXT,
	operation TEXT,
	row_before JSON,
	row_after JSON
);

```

-- 2. We will populate the above table with some internal variables
username	SESSION_USER
	
	add_time	Event time converted into UTC (Universal Time Coordinated) with day light savings etc.
				CURRENT_TIMESTAMP AT TIME ZONE 'UTC'
	
	table_name	TG_TABLE_SCHEMA || ',' || TG_TABLE_NAME		public.audit
	
	operation	TG_OP	INSERT, UPDATE, DELETE
	
	row_before	Finally, the before and after images of rows are stored as rows converted to json,
	row_after	which is available as its own data type row_to_json

-- 3. Lets create the trigger function

-- Please note that NEW and OLD are not NULL for the DELETE and INSERT triggers correspondingly.

```sql
CREATE OR REPLACE FUNCTION fn_audit_trigger()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS
$$
	DECLARE
		
		old_row json = NULL;
		new_row json = NULL;
		
	BEGIN
		-- TG_OP
		
		
		-- UPDATE, DELTE
			-- old_row
		IF TG_OP IN ('UPDATE','DELETE') THEN
			old_row = row_to_json(OLD);
		END IF;
			
		-- INSERT, UPDATE
			-- new_row
		IF TG_OP IN ('INSERT','UPDATE') THEN
			new_row = row_to_json(NEW);
		END IF;
		
		-- INSERT audit_log
		
		 INSERT INTO audit_log
		 (
			 username,
			 add_time,
			 table_name,
			 operation,
			 row_before,
			 row_after 
		 )
		 VALUES 
			(
				session_user,
				CURRENT_TIMESTAMP AT TIME ZONE 'UTC',
				TG_TABLE_SCHEMA || ',' || TG_TABLE_NAME,
				TG_OP,
				old_row,
				new_row
			);
			
			RETURN NEW;
	END;
$$

-- 4. Lets create our trigger statement

CREATE TRIGGER trg_audit_trigger
AFTER INSERT OR UPDATE OR DELETE
ON audit
FOR EACH ROW 
EXECUTE PROCEDURE fn_audit_trigger();

-- 5. Lets test with some data

-- insert operation

	INSERT INTO audit (id) values (1);
	INSERT INTO audit (id) values (2);
	
	SELECT * FROM audit;
	SELECT * FROM audit_log;


-- UPDATE operation

UPDATE audit SET id = '100' where id = 1;

SELECT * FROM audit_log;

-- DELETE operation

DELETE FROM audit where id = 2;

SELECT * FROM audit;

SELECT * FROM audit_log;

```

-- Please note:
-- *********************
-- Triggers are called in ALPHABETICAL ORDER, and the latter triggers can change what gets inserted,
-- so caution needs to be applied when naming triggers, particularly when auditing.


-- Creating conditional triggers
-- ##############################

1. We can create a conditional trigger by using a generic WHEN caluse
	
	2. With a WHEN clause, you can write some conditions except a subquery (because subquery tested
	before the trigger function is called)
	
	3. The WHEN expression should result into Boolean value. If the value is FALSE or NULL (which is
	automatically converted to FALSE), the trigger function is not called.

-- Lets say we want to enforce "NO INSERT/UPDATE/DELETE on Friday afternoon"!

-- We will use;

	-- 1. WHEN condition
	-- 2. We will use FOR EACH STATEMENT
	-- 3. We will pass a 'parameter' to EXECUTE PROCEDURE function_name(parameter)

-- 1. Lets create our master table

```sql
CREATE TABLE mytask(
	task_id SERIAL PRIMARY KEY,
	task TEXT
);

-- 2. We will create a generic function which will show a message and RETURN NULL.

CREATE OR REPLACE FUNCTION fn_cancel_with_message()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS
$$
	BEGIN
		RAISE EXCEPTION '%', TG_ARGV[0];
		
		RETURN NULL;
	
	END;
$$

-- 3. Lets create our trigger statement. We will run it on STATEMENT level and not row level

CREATE TRIGGER trg_no_update_on_friday_afternoon
BEFORE INSERT OR UPDATE OR DELETE OR TRUNCATE
ON mytask
FOR EACH STATEMENT
WHEN
(
	EXTRACT('DOW' FROM CURRENT_TIMESTAMP) = 3
	AND CURRENT_TIME > '12:00'
)
EXECUTE PROCEDURE fn_cancel_with_message('No update are allowed at Friday Afternoon, so chill!!')

DROP TRIGGER trg_no_update_on_friday_afternoon ON mytask;


-- 4. Lets test the insert (Remember, for us it might not be Friday afternoon, so we can always change
--    the login in function!)

SELECT 
	EXTRACT('DOW' FROM CURRENT_TIMESTAMP)
	CURRENT_TIME;
	
INSERT INTO mytask (task) VALUES ('test1');

SELECT * FROM mytask;

```

-- Disallow change on primary key data
-- #######################################

--	Use case : We want to raise an error each time whenever someone tries to change a primary key column

-- 1. Lets create our master table

```sql
CREATE TABLE pk_table(
	id SERIAL PRIMARY KEY,
	t TEXT
);

-- 2. We will insert some data first

INSERT INTO pk_table (t) VALUES ('t1'), ('t2');

SELECT * FROM pk_table;


-- 1. Lets create a generic 'cancel' function for our use case here

CREATE OR REPLACE FUNCTION fn_generic_cancel_op()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS
$$ 
	BEGIN
		IF TG_WHEN = 'AFTER' THEN
			
			RAISE EXCEPTION 'YOU ARE NOT ALLOWED TO % ROWS IN %.%',
			TG_OP, TG_TABLE_SCHEMA, TG_TABLE_NAME;
		END IF;
		
		RAISE NOTICE '% ON ROWS IN %.% WONT HAPPEN',
			TG_OP, TG_TABLE_SCHEMA, TG_TABLE_NAME;
			
		RETURN NULL;
	END;
$$

-- 3. Lets create our trigger statement

CREATE TRIGGER disallow_pk_change
AFTER UPDATE OF id
ON pk_table
FOR EACH ROW
EXECUTE PROCEDURE fn_generic_cancel_op();


--4. Now lets try to update a primary key data

UPDATE pk_table SET id = 100 WHERE id = 1;

```

-- Use triggers very cautiously
-- ############################

Yes, The triggers are powerful tools, as they can be used for;
		- Auditing
		- Logging
		- Enforcing complex constraints
		- Replications and more
	
	However, if they are not used cautiosly, they will to very hard to debug problems!!!!
	
	Some good practices to follow here
	
	1. DON'T change data in;
	
		- Primary Key
		- Foreign Key
		- or unique key columns
	
	2. Don't update records in the tables that you normally read during the transaction
	
	3. Don't read data from a table that is updating during the same transaction.
	
	4. Don't aggregate/summarized over the table that you are updating


-- Event Triggers
-- ################

	1. Event triggers are data-specific and not bind or attached to a table.
	
	2. Unlike regular triggers they capture system level DDL events.
	
	3. Event triggers can be BEFORE or AFTER triggers.
	
	4. Trigger function can be written in any language except SQL.
	
	5. Event triggers are disabled in the single user mode and can only be created by a superuser.

-- Event Triggers usage scenarios
-- ##############################

1. Audit trial - Users, Schema changes etc.
	
	2. Preventing changes to a table.
	
	3. Providing limited DDL (Data Definition Language) capabilities to developers.
	
	4. Enable/Disable certain DDL commands based on business logic.
	
	5. Performance analysis of DDL start (ddl_command_start) and end (ddl_command_stop) process.
	
	6. Replicating DDL changes/updates to remote locations.
	
	7. Cascading a DDL

-- Creating Event Triggers
-- #######################

```sql
-- The Syntax

CREATE EVENT TRIGGER trg_name

```


	Requirements
	............
	
	1. Before creating an event trigger, we must have a function that the trigger will execute.
	
	2. The function must return a special type called EVENT_TRIGGER.
	
	3. This function n eed not (and may not) return a value; the return type serves merly as a signal
	   that the function is to be invoked as an event trigger.
	   
	In case of multiple triggers
	............................
	
	They are executed in the alphabetical orders of their names
	
	Can we create conditional events?
	--------------------------------
	Yes, you can create conditional event triggers by using the WHEN clause
	
	CREATE EVENT TRIGGER trg_name
	WHEN
	     <condition>
		 
	Event triggers(like other functions) cannot be executed in an aborted transaction
    ..................................................................................

    i.e. if a DDL command fails with an error, any associated ddl_command_end triggers will not be
	executed.
	
	Conversely, if a ddl_command_start trigger fails with an error, no further event triggers will fire,
	and no attempt will be made to execute the command itself.
	
	Similarly, if a ddl_command_end trigger fails with an error, the effects of the DDL statement will be
	rolled back, just as they would be in any other case where the containing transaction aborts.
	
	Event triggers event
	-------------------- 

	ddl_command_start  This event occurs just BEFORE a CREATE, ALTER, or DROP DDL command is executed.
	ddl_command_end    This event occurs just AFTER a CREATE, ALTER, or DROP command has finished executing
	table_rewrite      This event occurs just before a table is rewritten by some actions of the commands
	                   ALTER TABLE and ALTER TYPE.
	sql_drop           This event occurs just before the ddl_command_end event for the commands that drop
	                   database objects.
					   
					   
	Event triggers variables
	------------------------
	
	TG_TAG      This variable contains the 'TAG' or the command for which the trigger is executed.
	            This variable does not contain the full string, but just a tag such as
				CREATE TABLE, DROP TABLE, ALTER TABLE, and so on.
				
	TG_EVENT    This variable contains the event name, which can be ddl_command_start, ddl_command_end
	            and sql_drop.


-- Creating an audit trial event trigger
-- ######################################
	
-- 1. Lets create our audit_ddl table

```sql
CREATE TABLE audit_ddl(
	audit_ddl_id SERIAL PRIMARY KEY,
	username TEXT,
	ddl_event TEXT,
	ddl_command text,
	ddl_add_time TIMESTAMPTZ
);
	
-- 2. Lets create our event trigger function

 CREATE OR REPLACE FUNCTION fn_event_audit_ddl()
 RETURNS EVENT_TRIGGER 
 LANGUAGE PLPGSQL
 SECURITY DEFINER 
 AS
 $$
 	BEGIN
		-- insert
	    INSERT INTO audit_ddl
		(
			username,
			ddl_event,
			ddl_command,
			ddl_add_time
		)
		VALUES
		(
			session_user,
			TG_EVENT,
			TG_TAG,
			NOW()
		);
		
		-- raise notice
		RAISE NOTICE 'DDL activity is added!!!';
		
		--No return value is needed
	END;
$$	


-- 3. Lets create our event trigger statement

-- without condition

CREATE EVENT TRIGGER trg_event_audit_ddl
ON ddl_command_start
EXECUTE PROCEDURE fn_event_audit_ddl()

-- without condition

CREATE EVENT TRIGGER trg_event_audit_ddl
ON ddl_command_start
WHEN
	TAG IN ('CREATE TABLE')
EXECUTE PROCEDURE fn_event_audit_ddl()


-- 4. Lets test new event trigger by creating a new table via CREATE TABLE command

-- First see the contents of audit_ddl

SELECT * FROM audit_ddl;

-- Now create the table

CREATE TABLE audit_ddl_test(
	i INT
);   

  -- Table creation is rejected according to our created event trigger
  
  SELECT * FROM audit_ddl;
  
  -- You can drop an event trigger using
  
  DROP EVENT TRIGGER trg_event_audit_ddl;

```

-- Prevent Schema Changes
-- #######################

Lets implement a policy;
	
	'No table are allowed to be created during 9am and 4pm time'

-- 1. Lets create our trigger function

```sql
CREATE OR REPLACE FUNCTION 	fn_event_abort_create_table_function()
RETURNS EVENT_TRIGGER
LANGUAGE PLPGSQL
SECURITY DEFINER
AS
$$
	DECLARE
		current_hour int = EXTRACT('hour' FROM NOW());
	BEGIN
	
		IF current_hour BETWEEN 9 AND 16 THEN
			RAISE EXCEPTION 'Tables are not allowed to be created during 9am-4pm. Thanks!';
		END IF;
	END;
$$

-- 2. Lets create our event trigger statement

CREATE EVENT TRIGGER trg_event_abort_create_table_function
ON ddl_command_start
WHEN
	TAG IN ('CREATE TABLE')
EXECUTE PROCEDURE fn_event_abort_create_table_function()

-- 3. Lets try to create a table
-- First check your own settings

CREATE TABLE ddl_t2(i INT);
 
 SELECT EXTRACT('hour' FROM NOW());
 
 
 -- Lets do a quick housekeeping
 
 DROP TABLE ddl_t1;
 DROP TABLE ddl_t2;
 
 DROP EVENT TRIGGER trg_event_abort_create_table_function;

```
