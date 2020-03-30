# MySQL

### Basics
* Connect to a mysql database, optionaly executing the given script
``` bash
mysql -h URL -P PORT -u USERNAME -pPASSWORD < "SCRIPT.sql"
```
* Dump the specified database to the given file, and restore the database from the file
``` bash
mysqldump --column-statistics=0 -h URL -P PORT -u USERNAME -pPASSWORD DB_NAME > BACKUP_DIR/DB_NAME.sql
mysql -h URL -P PORT -u root -pPASSWORD DB_NAME < BACKUP_DIR/DB_NAME.sql
```
* Reset the  root database password
``` sql
USE mysql; ALTER USER 'root'@'localhost' IDENTIFIED BY 'PASSWORD';
```
* Execute local script from within mysql terminal
``` sql
CREATE DATABASE DB; USE DB; SOURCE SCRIPT.sql;
```
* Allow local file loading
``` sql
SET GLOBAL local_infile = true;
```
* Show running queries
``` sql
SHOW FULL PROCESSLIST;
```
* Reset auto_increment on table column
``` sql
ALTER TABLE TABLE_NAME AUTO_INCREMENT=1;
```

### Table Sizes
``` sql
SELECT table_schema AS DB_Name, ROUND(SUM(data_length + index_length) / 1024 / 1024, 1) AS DB_Size_MB
  FROM information_schema.tables
  GROUP BY table_schema;
```

### Creation
``` sql
DROP TABLE IF EXISTS DB.TABLE;
CREATE TABLE DB.TABLE (
  record_id INT(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  record_name VARCHAR(255) NOT NULL,
  external_id BIGINT(20) UNSIGNED NOT NULL,
  metric_id INT(3) UNSIGNED NOT NULL,
  metric_value FLOAT NULL,
  log_time DATETIME NULL
);
ALTER TABLE DB.TABLE ADD UNIQUE INDEX (record_id, metric_id);
ALTER TABLE DB.TABLE CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Insert
``` sql
INSERT INTO DB.TABLE (record_id, record_name, external_id, metric_id, metric_value, log_time) VALUES
  (1, 'something', 23723862368, 1, 4.5, TIMESTAMP('2019-01-01 1:00:00'));
```

### Update
``` sql
UPDATE DB.TABLE
  SET metric_value=3.5
  WHERE record_id=1 AND metric_id=1;
```

### Pivoting
``` sql
SELECT record_id, SUM(IF(metric_id=1, metric_value, 0)), SUM(IF(metric_id=2, metric_value, 0))
  FROM DB.TABLE
  GROUP BY record_id;
```

### Nested Query
``` sql
SELECT d.record_id, d.metric_id, ROUND(d.metric_value / X.avg_value, 3) AS norm_metric_value
  FROM DB.TABLE d
  JOIN (SELECT metric_id, AVG(metric_value) AS avg_value) X
    ON d.metric_id=X.metric_id;
```

### Constraints
``` sql
SELECT record_id, IF(metric_value>=LOG10(100), LEAST(EXP(3), metric_value), GREATEST(SQRT(0.5), metric_value)) FROM DB.TABLE;
```

### Datetime
``` sql
SELECT DATE_FORMAT(log_time, '%Y-%m'), COUNT(*) AS instances FROM DB.TABLE GROUP BY 1;
```

### Group Concat
``` sql
SELECT metric_id, GROUP_CONCAT(record_name SEPARATOR ' - ') FROM DB.TABLE t1 GROUP BY 1
```

### Loading
``` sql
LOAD DATA LOCAL INFILE '~/temp.csv'
  INTO TABLE DB.TABLE
  CHARACTER SET UTF8
  COLUMNS TERMINATED BY ','
  ESCAPED BY '\"'
  ENCLOSED BY '"'
  LINES TERMINATED BY '\r\n'
  IGNORE 0 LINES
  (record_id, @record_name, external_id, metric_id, metric_value)
  SET record_name=NULLIF(@record_name, 'EMPTY'), log_time=NOW();
```

### Procedure
``` sql
DROP PROCEDURE IF EXISTS Practice;
DELIMITER //
CREATE PROCEDURE Practice(IN _table_name VARCHAR(40))
	BEGIN
		SET @_query = CONCAT('SELECT * FROM ', _table_name);
		PREPARE _statement FROM @_query;
		EXECUTE _statement;
		DEALLOCATE PREPARE _statement;
	END //
DELIMITER ;
CALL Practice('DB.TABLE');
```

### Triggers
``` sql
CREATE TABLE DB.TABLE_log (
  record_id INT(11) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  record_count INT(7) UNSIGNED NOT NULL
);

DROP TRIGGER IF EXISTS DB.TABLE_trigger;
DELIMITER //
CREATE TRIGGER DB.TABLE_trigger
  BEFORE INSERT ON DB.TABLE
  FOR EACH ROW
  BEGIN
    IF (SELECT COUNT(*) FROM DB.TABLE_log WHERE record_id=record_id) = 0 THEN
      INSERT INTO DB.TABLE_log (record_id, record_count) VALUES (record_id, 1);
    ELSE
      UPDATE DB.TABLE_log SET record_count=(record_count+1) WHERE record_id=record_id;
    END IF;
  END //
DELIMITER ;
```

### Foreign Key Constraints
``` sql
CREATE TABLE COMPANY (
  company_id INT NOT NULL,
  company_name VARCHAR(50),
  PRIMARY KEY (company_id)
) ENGINE=INNODB;

CREATE TABLE USER (
  user_id INT,
  user_name VARCHAR(50),
  company_id INT,
  INDEX company_id_idx (company_id),
  FOREIGN KEY (company_id) REFERENCES COMPANY (company_id) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=INNODB;
```
* the engine will reject the deletion of records in COMPANY, if at least one record in USER links to it (life-saving)
* updating company_id in COMPANY will automatically update all corresponding records in USER (but wont trigger USER)

### Linux Purge Rebuild
``` bash
sudo apt purge mysql-server mysql-client mysql-common mysql-server-core-5.7 mysql-client-core-5.7
sudo rm -rfv /etc/mysql /var/lib/mysql
sudo apt autoremove
sudo apt autoclean

sudo apt update
sudo apt install mysql-server mysql-client --fix-broken --fix-missing

sudo apt-get install mysql-workbench
```

### Window Functions
``` sql
SELECT record_id, metric_id, COUNT(metric_value) OVER (PARTITION BY MONTH(log_time)) AS monthly_metric_count FROM DB.TABLE;
```
* Was not working during last check with mysql version "5.7.24-0ubuntu0.16.04.1"

### Percentiles
``` sql
CREATE TABLE DB.TABLE_ordered (
  order_id INT(11) NOT NULL PRIMARY KEY AUTO_INCREMENT,
  record_id INT(7) UNSIGNED NOT NULL,
  metric_value DOUBLE(4, 1) UNSIGNED NOT NULL
);
INSERT INTO DB.TABLE_ordered
  SELECT NULL, record_id, metric_value
    FROM DB.TABLE
    ORDER BY record_id, metric_value;
ALTER TABLE DB.TABLE_ordered ADD UNIQUE INDEX (record_id)

CREATE TABLE DB.TABLE_min_max
  SELECT record_id, MIN(order_id) as min_order_id, MAX(order_id) AS max_order_id
    FROM DB.TABLE_ordered
    GROUP BY record_id;
ALTER TABLE DB.TABLE_min_max ADD UNIQUE INDEX (record_id);

DROP TABLE IF EXISTS DB.TABLE_percentiles;
CREATE TABLE DB.TABLE_percentiles
  SELECT o.record_id,
    ROUND(AVG(o.value), 3) AS mean_value,
    ROUND(SUM(IF(o.order_id=m.min_order_id, o.metric_value, 0)), 3) AS min_value,
    ROUND(AVG(IF(o.order_id=m.min_order_id+ROUND(0.5*(m.max_actual_order_id-m.min_order_id)), o.metric_value, NULL)), 3) AS perc_50,
    ROUND(SUM(IF(o.order_id=m.max_order_id, o.metric_value, 0)), 3) AS max_value,
    FROM DB.TABLE_ordered o
    JOIN DB.TABLE_min_max m
      ON o.record_id=m.record_id
    GROUP BY o.record_id;
ALTER TABLE DB.TABLE_percentiles ADD UNIQUE INDEX (record_id);
```

### String Normalization
``` sql
SET SQL_SAFE_UPDATES=0;

# Lowercase
UPDATE TABLENAME
  SET COLNAME=LOWER(COLNAME);

# Punctuation
UPDATE TABLENAME
  SET COLNAME=
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    COLNAME,
    '’', ''), '.', ''), ',', ''), "'", ''), '"', ''), '!', ''), '@', ''), '#', ''), '$', 's'), '%', ''), '&', 'and'),
    ':', ''), '(', ''), ')', ''), '?', ''), '^', ''), '<', ''), '>', ''), '/', ''), '-', ''), '_', ''), '+', ''), '=', ''), '[', ''), ']', ''), '?', ''), '*', ''), ':', ''),
    "`", ''), "´", ''), "‐", ''), "–", ''), "〜", ''), "・", ''), ";", ''), "…", ''), "‘", ''), "“", ''), "”", ''), "„", ''), "«", ''), "»", ''), "\\", ''),
    "†", ''), "•", ''), "°", ''), "∆", ''), "|", ''), "~", ''), "▲", ''), "★", ''), "☆", ''), "ー", '');

# Latinize
UPDATE TABLENAME
  SET COLNAME=
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    REPLACE(
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    REPLACE(REPLACE(
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    REPLACE(REPLACE(
    REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
    COLNAME,
    'á', 'a'), 'Á', 'a'), 'à', 'a'), 'À', 'a'), 'â', 'a'), 'Â', 'a'), 'ä', 'a'), 'Ä', 'a'), 'ã', 'a'), 'Ã', 'a'), 'å', 'a'), 'Å', 'a'), 'æ', 'ae'), 'Æ', 'ae'),
    'ç', 'c'), 'Ç', 'c'),
    'é', 'e'), 'É', 'e'), 'è', 'e'), 'È', 'e'), 'ê', 'e'), 'Ê', 'e'), 'ë', 'e'), 'Ë', 'e'),
    'í', 'i'), 'Í', 'i'), 'ì', 'i'), 'Ì', 'i'), 'î', 'i'), 'Î', 'i'), 'ï', 'i'), 'Ï', 'i'),
    'ñ', 'n'), 'Ñ', 'n'),
    'ó', 'o'), 'Ó', 'o'), 'ò', 'o'), 'Ò', 'o'), 'ô', 'o'), 'Ô', 'o'), 'ö', 'o'), 'Ö', 'o'), 'õ', 'o'), 'Õ', 'o'), 'ø', 'o'), 'Ø', 'o'), 'œ', 'oe'), 'Œ', 'oe'),
    'ß',' s'),
    'ú', 'u'), 'Ú', 'u'), 'ù', 'u'), 'Ù', 'u'), 'û', 'u'), 'Û', 'u'), 'ü', 'u'), 'Ü', 'u');

# Whitespace
UPDATE TABLENAME
  SET COLNAME=REPLACE(COLNAME, ' ', '');

SET SQL_SAFE_UPDATES=1;
```
