```sql
-- Table: admin.clients

-- DROP TABLE IF EXISTS admin.clients;

CREATE TABLE IF NOT EXISTS admin.clients
(
    client_id serial NOT NULL,
    name character varying COLLATE pg_catalog."default",
    logo bytea,
    contact numeric,
    address character varying COLLATE pg_catalog."default",
    industry_id integer,
    dmv bigint,
    is_active boolean DEFAULT 'false',
    created_at timestamp without time zone DEFAULT 'CURRENT_TIMESTAMP',
    updated_at timestamp without time zone DEFAULT 'CURRENT_TIMESTAMP',
    created_by character varying COLLATE pg_catalog."default",
    updated_by character varying COLLATE pg_catalog."default",
    fields character varying[] COLLATE pg_catalog."default",
    CONSTRAINT client_pkey PRIMARY KEY (client_id),
    CONSTRAINT industry_id FOREIGN KEY (industry_id)
        REFERENCES admin.ekyc_industry (industry_id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS admin.clients
    OWNER to snids_test;

COMMENT ON TABLE admin.clients
    IS 'information about client institutions';

COMMENT ON COLUMN admin.clients.dmv
    IS 'daily_max_verifications';

COMMENT ON COLUMN admin.clients.fields
    IS 'Fields the client is allowed to access';

-- Trigger Function for clints change logs

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

-- Trigger for client changes

CREATE TRIGGER trg_clients_changes
    BEFORE UPDATE 
    ON admin.clients
    FOR EACH ROW
    EXECUTE FUNCTION admin.fn_clients_changes_log();

-- Trigger: trg_disallow_delete

-- DROP TRIGGER IF EXISTS trg_disallow_delete ON admin.clients;

CREATE TRIGGER trg_disallow_delete
    AFTER DELETE
    ON admin.clients
    FOR EACH ROW
    EXECUTE FUNCTION admin.fn_generic_cancel_op();


INSERT INTO admin.clients(
	name, contact, industry_id, dmv)
	VALUES ('Asha Elmi Ali', 0616543212, 12, 57);
	
select * from admin.clients;


update admin.clients set name = 'Farhiya Ahmed Ali' 
where client_id = 3;

select * from admin.clients_archive;

```

admin.clients_archive
################

```sql
-- Table: admin.clients_archive

-- DROP TABLE IF EXISTS admin.clients_archive;

CREATE TABLE IF NOT EXISTS admin.clients_archive
(
    client_id serial,
    name character varying COLLATE pg_catalog."default",
    logo bytea,
    contact numeric,
    address character varying COLLATE pg_catalog."default",
    industry_id integer,
    dmv bigint,
    is_active boolean,
    created_at timestamp without time zone,
    updated_at timestamp without time zone,
    created_by character varying COLLATE pg_catalog."default",
    updated_by character varying COLLATE pg_catalog."default",
    fields character varying[] COLLATE pg_catalog."default"
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS admin.clients_archive
    OWNER to snids_test;


```

