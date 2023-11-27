How to create an archival table for an original table:

```sql
create table admin_users_archive as select * from admin_users;
```

Create a Trigger:
You can use a trigger to automatically move records from the original table to the archival table when specific conditions are met. For example, you might archive records older than a certain date.

```sql
CREATE OR REPLACE FUNCTION archive_records()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO archival_table
    SELECT * FROM original_table WHERE created_at < '2023-01-01';

    DELETE FROM original_table WHERE created_at < '2023-01-01';
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_archive
AFTER INSERT OR UPDATE OR DELETE
ON original_table
FOR EACH STATEMENT
EXECUTE FUNCTION archive_records();

```

1. admin.admin_users
####################

```sql
CREATE OR REPLACE FUNCTION admin.fn_admin_users_changes_log()
      RETURNS TRIGGER
	  LANGUAGE PLPGSQL
	  AS
  $$
  BEGIN
      -- COMPARE THE new value vs OLD value
	  -- NEW, OLD
	  IF NEW.* <> OLD.* THEN
	     INSERT INTO admin.admin_users_archive
		 (
			 admin_user_id,
			 full_name,
			 email,
			 role_id,
			 nid,
			 is_active
		 )
		 VALUES
		 (
			 OLD.admin_user_id,
			 OLD.full_name,
			 OLD.email,
			 OLD.role_id,
			 OLD.nid,
			 OLD.is_active
			  
		 );
	  
	  END IF;
	  
	  RETURN NEW;
  
  END;
  $$
  
  CREATE TRIGGER trg_admin_users_changes
     BEFORE UPDATE 
	 ON admin.admin_users
	 FOR EACH ROW 
	 EXECUTE PROCEDURE admin.fn_admin_users_changes_log();
	 
```

This function is a generic cancel operations
################################################

```sql
CREATE OR REPLACE FUNCTION admin.fn_generic_cancel_op()
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
```

Disallowing delete Trigger:
################################

```sql
CREATE TRIGGER trg_disallow_delete
AFTER DELETE
ON admin.admin_users
FOR EACH ROW
EXECUTE PROCEDURE admin.fn_generic_cancel_op();



-- Lets insert some data

INSERT INTO admin.admin_users(
	admin_user_id, full_name, email, role_id)
	VALUES (1, 'Mohamed Ahmed Ali', 'mohamed@gmail.com', 124);
	
	INSERT INTO admin.admin_users(
	admin_user_id, full_name, email, role_id)
	VALUES (2, 'Caasho Ahmed Ali', 'asha@gmail.com', 124);
	
	INSERT INTO admin.admin_users(
	admin_user_id, full_name, email, role_id)
	VALUES (3, 'Zahra Mohamed Ahmed', 'zahra@gmail.com', 124);
	
	INSERT INTO admin.admin_users(
	admin_user_id, full_name, email, role_id)
	VALUES (3, 'Sharmake Ali Kahie', 'sharma@gmail.com', 124);

-- Lets view the inserted rows:
################################
 select * from admin.admin_users;
 
 select * from admin.admin_users_archive;
 
-- Updating full_name where admin_user_id = '214'

update admin_users set full_name = 'Aisha Ali Yusuf' where admin_user_id = '214';

-- Updating email where admin_user_id = '214'

 update admin.admin_users set email = 'ahmed@gmail.com'
 where admin_user_id = '214';

-- Disallowed rows 
#####################

delete from admin.admin_users where admin_user_id = '3';

--ERROR:  YOU ARE NOT ALLOWED TO DELETE ROWS IN admin.admin_users
CONTEXT:  PL/pgSQL function admin.fn_generic_cancel_op() line 5 at RAISE
SQL state: P0001

select * from admin_users;

select * from admin_users_archive;
```

2. admin.api_rate
#################

```sql
CREATE OR REPLACE FUNCTION admin.fn_api_rate_changes_log()
      RETURNS TRIGGER
	  LANGUAGE PLPGSQL
	  AS
  $$
  BEGIN
      -- COMPARE THE new value vs OLD value
	  -- NEW, OLD
	  IF NEW.* <> OLD.* THEN
	     INSERT INTO admin.api_rate_archive
		 (
			 api_rate_id,
			 api_rate,
			 industry_id
		 )
		 VALUES
		 (
			 OLD.api_rate_id,
			 OLD.api_rate,
			 OLD.industry_id
			  
		 );
	  
	  END IF;
	  
	  RETURN NEW;
  
  END;
  $$
  
  CREATE TRIGGER trg_api_rate_changes
     BEFORE UPDATE 
	 ON admin.api_rate
	 FOR EACH ROW 
	 EXECUTE PROCEDURE admin.fn_api_rate_changes_log();
	 
  
-- Disallowing delete rows trigger:

CREATE TRIGGER trg_disallow_delete
AFTER DELETE
ON admin.api_rate
FOR EACH ROW
EXECUTE PROCEDURE admin.fn_generic_cancel_op();


-- Lets insert some data into admin.api_rate table

INSERT INTO admin.api_rate(
	api_rate_id, api_rate, industry_id)
	VALUES (11,120, 12);
	
	
INSERT INTO admin.api_rate(
	api_rate_id, api_rate, industry_id)
	VALUES (12,130, 12);
	
INSERT INTO admin.api_rate(
	api_rate_id, api_rate, industry_id)
	VALUES (13,140, 12);


-- Lets update some rows in admin.api_rate table

update admin.api_rate set api_rate = 170 where api_rate_id = 13;

-- Lets delete some rows in admin.api_rate table

delete from admin.api_rate where api_rate_id = 1;

Result:

-- ERROR:  YOU ARE NOT ALLOWED TO DELETE ROWS IN admin.api_rate
CONTEXT:  PL/pgSQL function admin.fn_generic_cancel_op() line 5 at RAISE
SQL state: P0001

```

3. admin.clients
#################

```sql
CREATE OR REPLACE FUNCTION admin.fn_clients_changes_log()
      RETURNS TRIGGER
	  LANGUAGE PLPGSQL
	  AS
  $$
  BEGIN
      -- COMPARE THE new value vs OLD value
	  -- NEW, OLD
	  IF NEW.* <> OLD.* THEN
	     INSERT INTO admin.clients_archive
		 (
			 client_id,
			 name,
			 contact,
			 industry_id,
			 dmv
		 )
		 VALUES
		 (
			 OLD.client_id,
			 OLD.name,
			 OLD.contact,
			 OLD.industry_id,
			 OLD.dmv	  
		 );
	  
	  END IF;
	  
	  RETURN NEW;
  
  END;
  $$
  
  CREATE TRIGGER trg_clients_changes
     BEFORE UPDATE 
	 ON admin.clients
	 FOR EACH ROW 
	 EXECUTE PROCEDURE admin.fn_clients_changes_log();
	 
  
-- Disallowing delete rows trigger:

CREATE TRIGGER trg_disallow_delete
AFTER DELETE
ON admin.clients
FOR EACH ROW
EXECUTE PROCEDURE admin.fn_generic_cancel_op();
```

