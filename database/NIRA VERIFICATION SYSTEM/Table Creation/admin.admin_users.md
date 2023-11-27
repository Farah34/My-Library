```sql
-- Table: admin.admin_users

-- DROP TABLE IF EXISTS admin.admin_users;

CREATE TABLE IF NOT EXISTS admin.admin_users
(
    admin_user_id serial NOT NULL,
    full_name character varying(100) COLLATE pg_catalog."default" NOT NULL,
    email character varying(100) COLLATE pg_catalog."default" NOT NULL,
    photo bytea,
    role_id integer,
    nid integer,
    is_active character varying COLLATE pg_catalog."default" DEFAULT 'active'::character varying,
    password character varying(128) COLLATE pg_catalog."default",
    insertion_timestamp timestamp(6) without time zone DEFAULT 'CURRENT_TIMESTAMP',
    CONSTRAINT admin_users_pkey PRIMARY KEY (admin_user_id),
    CONSTRAINT admin_users_email UNIQUE (email),
    CONSTRAINT role_id FOREIGN KEY (role_id)
        REFERENCES admin.roles (role_id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS admin.admin_users
    OWNER to snids_test;

COMMENT ON TABLE admin.admin_users
    IS 'admin users of NIRA for adminstering eKYC officers';

-- admin.admin_users function to keep changes log

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



-- Trigger: trg_admin_users_changes

-- DROP TRIGGER IF EXISTS trg_admin_users_changes ON admin.admin_users;

CREATE TRIGGER trg_admin_users_changes
    BEFORE UPDATE 
    ON admin.admin_users
    FOR EACH ROW
    EXECUTE FUNCTION admin.fn_admin_users_changes_log();

-- Trigger: trg_disallow_delete

-- DROP TRIGGER IF EXISTS trg_disallow_delete ON admin.admin_users;

CREATE TRIGGER trg_disallow_delete
    AFTER DELETE
    ON admin.admin_users
    FOR EACH ROW
    EXECUTE FUNCTION admin.fn_generic_cancel_op();

-- Lets insert some data
	
INSERT INTO admin.admin_users(
	full_name, email, role_id)
	VALUES ('Mohamed Ahmed Ali', 'mohamed@gmail.com', 124);
	
INSERT INTO admin.admin_users(
	full_name, email, role_id)
	VALUES ('Caasho Ahmed Ali', 'asha@gmail.com', 124);

INSERT INTO admin.admin_users(
	full_name, email, role_id)
	VALUES ('Zahra Mohamed Ahmed', 'zahra@gmail.com', 124);

INSERT INTO admin.admin_users(
	full_name, email, role_id)
	VALUES ('Sharmake Ali Kahie', 'sharma@gmail.com', 124);

-- Lets view the inserted rows:
################################

select * from admin.admin_users;

-- Updating full_name where admin_user_id = '214'

update admin.admin_users set full_name = 'Shafie Ahmed Ali' where admin_user_id = '2';

select * from admin.admin_users_archive;
  
```

admin.admin_users_archive
####################

```sql
-- Table: admin.admin_users_archive

-- DROP TABLE IF EXISTS admin.admin_users_archive;

CREATE TABLE IF NOT EXISTS admin.admin_users_archive
(
    admin_user_id serial,
    full_name character varying(100) COLLATE pg_catalog."default",
    email character varying(100) COLLATE pg_catalog."default",
    photo bytea,
    role_id integer,
    nid integer,
    is_active character varying COLLATE pg_catalog."default",
    password character varying(128) COLLATE pg_catalog."default",
    insertion_timestamp timestamp(6) without time zone DEFAULT 'CURRENT_TIMESTAMP'
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS admin.admin_users_archive
    OWNER to snids_test;
```