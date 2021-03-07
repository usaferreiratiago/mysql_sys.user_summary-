# mysql_sys.user_summary-

CREATE TABLE `# mysql_sys.user_summary`
(
  sk_summary BIGINT NOT NULL PRIMARY KEY
, version INT
, date_from DATETIME
, date_to DATETIME
, user VARCHAR(32)
, statements DECIMAL(64)
, statement_latency VARCHAR(11)
, statement_avg_latency VARCHAR(11)
, table_scans DECIMAL(65)
, file_ios DECIMAL(64)
, file_io_latency VARCHAR(11)
, current_connections DECIMAL(41)
, total_connections DECIMAL(41)
, unique_hosts BIGINT
, current_memory VARCHAR(11)
, total_memory_allocated VARCHAR(11)
)
;CREATE INDEX `idx_# mysql_sys.user_summary_lookup` ON `# mysql_sys.user_summary`(user, statements, statement_latency, statement_avg_latency, table_scans, file_ios, file_io_latency, current_connections, total_connections, unique_hosts, current_memory, total_memory_allocated)
;
CREATE INDEX `idx_# mysql_sys.user_summary_tk` ON `# mysql_sys.user_summary`(sk_summary)



CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `monitory_bd`.`mysql_sys.user_summary` AS
    SELECT 
        `monitory_bd`.`# mysql_sys.user_summary`.`sk_summary` AS `sk_summary`,
        `monitory_bd`.`# mysql_sys.user_summary`.`version` AS `version`,
        `monitory_bd`.`# mysql_sys.user_summary`.`date_from` AS `date_from`,
        `monitory_bd`.`# mysql_sys.user_summary`.`date_to` AS `date_to`,
        `monitory_bd`.`# mysql_sys.user_summary`.`user` AS `user`,
        `monitory_bd`.`# mysql_sys.user_summary`.`statements` AS `statements`,
        `monitory_bd`.`# mysql_sys.user_summary`.`statement_latency` AS `statement_latency`,
        `monitory_bd`.`# mysql_sys.user_summary`.`statement_avg_latency` AS `statement_avg_latency`,
        `monitory_bd`.`# mysql_sys.user_summary`.`table_scans` AS `table_scans`,
        `monitory_bd`.`# mysql_sys.user_summary`.`file_ios` AS `file_ios`,
        `monitory_bd`.`# mysql_sys.user_summary`.`file_io_latency` AS `file_io_latency`,
        `monitory_bd`.`# mysql_sys.user_summary`.`current_connections` AS `current_connections`,
        `monitory_bd`.`# mysql_sys.user_summary`.`total_connections` AS `total_connections`,
        `monitory_bd`.`# mysql_sys.user_summary`.`unique_hosts` AS `unique_hosts`,
        `monitory_bd`.`# mysql_sys.user_summary`.`current_memory` AS `current_memory`,
        `monitory_bd`.`# mysql_sys.user_summary`.`total_memory_allocated` AS `total_memory_allocated`
    FROM
        `monitory_bd`.`# mysql_sys.user_summary`
        
        
        
SELECT * FROM monitory_bd.`mysql_sys.user_summary`;




SELECT 
    IF((`performance_schema`.`accounts`.`USER` IS NULL),
        'background',
        `performance_schema`.`accounts`.`USER`) AS `user`,
    SUM(`sys`.`stmt`.`total`) AS `statements`,
    FORMAT_PICO_TIME(SUM(`sys`.`stmt`.`total_latency`)) AS `statement_latency`,
    FORMAT_PICO_TIME(IFNULL((SUM(`sys`.`stmt`.`total_latency`) / NULLIF(SUM(`sys`.`stmt`.`total`), 0)),
                    0)) AS `statement_avg_latency`,
    SUM(`sys`.`stmt`.`full_scans`) AS `table_scans`,
    SUM(`sys`.`io`.`ios`) AS `file_ios`,
    FORMAT_PICO_TIME(SUM(`sys`.`io`.`io_latency`)) AS `file_io_latency`,
    SUM(`performance_schema`.`accounts`.`CURRENT_CONNECTIONS`) AS `current_connections`,
    SUM(`performance_schema`.`accounts`.`TOTAL_CONNECTIONS`) AS `total_connections`,
    COUNT(DISTINCT `performance_schema`.`accounts`.`HOST`) AS `unique_hosts`,
    FORMAT_BYTES(SUM(`sys`.`mem`.`current_allocated`)) AS `current_memory`,
    FORMAT_BYTES(SUM(`sys`.`mem`.`total_allocated`)) AS `total_memory_allocated`
FROM
    (((`performance_schema`.`accounts`
    LEFT JOIN `sys`.`x$user_summary_by_statement_latency` `stmt` ON ((IF((`performance_schema`.`accounts`.`USER` IS NULL), 'background', `performance_schema`.`accounts`.`USER`) = `sys`.`stmt`.`user`)))
    LEFT JOIN `sys`.`x$user_summary_by_file_io` `io` ON ((IF((`performance_schema`.`accounts`.`USER` IS NULL), 'background', `performance_schema`.`accounts`.`USER`) = `sys`.`io`.`user`)))
    LEFT JOIN `sys`.`x$memory_by_user_by_current_bytes` `mem` ON ((IF((`performance_schema`.`accounts`.`USER` IS NULL), 'background', `performance_schema`.`accounts`.`USER`) = `sys`.`mem`.`user`)))
GROUP BY IF((`performance_schema`.`accounts`.`USER` IS NULL),
    'background',
    `performance_schema`.`accounts`.`USER`)
ORDER BY SUM(`sys`.`stmt`.`total_latency`) DESC
