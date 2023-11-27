```sql
-- Table: admin.api_rate

-- DROP TABLE IF EXISTS admin.api_rate;

CREATE TABLE IF NOT EXISTS admin.api_rate
(
    api_rate_id serial NOT NULL,
    api_rate numeric,
    industry_id integer,
    created_at timestamp without time zone DEFAULT 'now()',
    updated_at timestamp without time zone,
    created_by character varying COLLATE pg_catalog."default",
    updated_by character varying COLLATE pg_catalog."default",
    is_active boolean DEFAULT 'false',
    CONSTRAINT api_rate_pkey PRIMARY KEY (api_rate_id),
    CONSTRAINT industry_id FOREIGN KEY (industry_id)
        REFERENCES admin.ekyc_industry (industry_id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS admin.api_rate
    OWNER to snids_test;

COMMENT ON TABLE admin.api_rate
    IS 'admin users of NIRA will set api ratess per industry';

-- Trigger function for keeping api_rate change logs

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

-- Trigger: trg_api_rate_changes

-- DROP TRIGGER IF EXISTS trg_api_rate_changes ON admin.api_rate;

CREATE TRIGGER trg_api_rate_changes
    BEFORE UPDATE 
    ON admin.api_rate
    FOR EACH ROW
    EXECUTE FUNCTION admin.fn_api_rate_changes_log();

-- Trigger: trg_disallow_delete

-- DROP TRIGGER IF EXISTS trg_disallow_delete ON admin.api_rate;

CREATE TRIGGER trg_disallow_delete
    AFTER DELETE
    ON admin.api_rate
    FOR EACH ROW
    EXECUTE FUNCTION admin.fn_generic_cancel_op();

-- Lets insert some data into admin.api_rate table

INSERT INTO admin.api_rate(
	api_rate, industry_id)
	VALUES (120, 12);
	
	
INSERT INTO admin.api_rate(
	api_rate, industry_id)
	VALUES (130, 12);
	
INSERT INTO admin.api_rate(
	api_rate, industry_id)
	VALUES (140, 12);
	
-- Lets view our data

select * from admin.api_rate;

-- Lets update some rows in admin.api_rate table

update admin.api_rate set api_rate = 120 where api_rate_id = 1;

-- Lets view row changes in admin.api_rate_archive

select * from admin.api_rate_archive;

-- Lets delete some rows in admin.api_rate table

delete from admin.api_rate where api_rate_id = 1;

-- ERROR:  YOU ARE NOT ALLOWED TO DELETE ROWS IN admin.api_rate
-- CONTEXT:  PL/pgSQL function admin.fn_generic_cancel_op() line 5 at RAISE
-- SQL state: P0001
```


admin.api_rate_archive

```sql
-- Table: admin.api_rate_archive

-- DROP TABLE IF EXISTS admin.api_rate_archive;

-- Table: admin.api_rate_archive

-- DROP TABLE IF EXISTS admin.api_rate_archive;

CREATE TABLE IF NOT EXISTS admin.api_rate_archive
(
    api_rate_id serial,
    api_rate numeric,
    industry_id integer,
    created_at timestamp without time zone DEFAULT 'now()',
    updated_at timestamp without time zone,
    created_by character varying COLLATE pg_catalog."default",
    updated_by character varying COLLATE pg_catalog."default",
    is_active boolean DEFAULT 'true'
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS admin.api_rate_archive
    OWNER to snids_test;

```

