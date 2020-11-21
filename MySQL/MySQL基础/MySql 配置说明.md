# MySql 配置说明

配置文件 my.ini 的配置说明

| 参数名称                  | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| port                      | 表示 MySQL 服务器的端口号                                    |
| basedir                   | 表示 MySQL 的安装路径                                        |
| datadir                   | 表示 MySQL 数据文件的存储位置，也是数据表的存放位置          |
| default-character-set     | 表示服务器端默认的字符集                                     |
| default-storage-engine    | 创建数据表时，默认使用的存储引擎                             |
| sql-mode                  | 表示 SQL 模式的参数，通过这个参数可以设置检验 SQL 语句的严格程度 |
| max_connections           | 表示允许同时访问 MySQL 服务器的最大连接数。其中一个连接是保留的，留给管理员专用的 |
| query_cache_size          | 表示查询时的缓存大小，缓存中可以存储以前通过 SELECT 语句查询过的信息，再次查询时就可以直接从缓存中拿出信息，可以改善查询效率 |
| table_open_cache          | 表示所有进程打开表的总数                                     |
| tmp_table_size            | 表示内存中每个临时表允许的最大大小                           |
| thread_cache_size         | 表示缓存的最大线程数                                         |
| myisam_max_sort_file_size | 表示 MySQL 重建索引时所允许的最大临时文件的大小              |
| myisam_sort_buffer_size   | 表示重建索引时的缓存大小                                     |
| key_buffer_size           | 表示关键词的缓存大小                                         |
| read_buffer_size          | 表示 MyISAM 表全表扫描的缓存大小                             |
| read_rnd_buffer_size      | 表示将排序好的数据存入该缓存中                               |
| sort_buffer_size          | 表示用于排序的缓存大小                                       |