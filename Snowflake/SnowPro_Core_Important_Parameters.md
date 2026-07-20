
# SnowPro Core Exam – High-Value Parameters Cheat Sheet

> Covers the parameters that are repeatedly tested or are worth memorizing for the SnowPro Core certification.
>
> **Columns**
> - **Level**: Highest level where the parameter can be configured.
> - **Default**: Factory default.
> - **Exam Tip**: Why it matters.

| Parameter | Specification | Values | Default | Level | Exam Tip |
|-----------|---------------|--------|---------|-------|----------|
| AUTOCOMMIT | Automatically commits DML statements | TRUE / FALSE | TRUE | Session (Account→User→Session) | FALSE requires COMMIT/ROLLBACK |
| DATA_RETENTION_TIME_IN_DAYS | Time Travel retention | Standard: 0-1, Enterprise: 0-90 | 1 | Table/Schema/DB/Account | Enables Time Travel & UNDROP |
| ERROR_ON_NONDETERMINISTIC_MERGE | Error on ambiguous MERGE | TRUE / FALSE | TRUE | Session | TRUE recommended |
| ERROR_ON_NONDETERMINISTIC_UPDATE | Error on ambiguous UPDATE | TRUE / FALSE | FALSE | Session | Frequently compared with MERGE |
| DATE_INPUT_FORMAT | Date parsing format | AUTO or format | AUTO | Session | AUTO detects formats |
| DATE_OUTPUT_FORMAT | Display format | Valid format | YYYY-MM-DD | Session | Display only |
| TIMEZONE | Session time zone | Any valid timezone | Account default | Session | Affects LTZ values |
| TIMESTAMP_TYPE_MAPPING | Default TIMESTAMP mapping | TIMESTAMP_NTZ/LTZ/TZ | TIMESTAMP_NTZ | Session | CREATE TABLE TIMESTAMP behavior |
| TIMESTAMP_INPUT_FORMAT | Timestamp input | AUTO or format | AUTO | Session | Parsing only |
| TIMESTAMP_OUTPUT_FORMAT | Timestamp display | Format | AUTO | Session | Display only |
| BINARY_INPUT_FORMAT | Binary input conversion | HEX, BASE64, UTF8 | HEX | Session | Conversion functions |
| BINARY_OUTPUT_FORMAT | Binary output conversion | HEX, BASE64 | HEX | Session | Conversion functions |
| JSON_INDENT | JSON formatting | 0-16 | 2 | Session | Pretty-print only |
| GEOGRAPHY_OUTPUT_FORMAT | Geography display | GeoJSON, WKT, WKB, EWKT, EWKB | GeoJSON | Session | GIS questions |
| GEOMETRY_OUTPUT_FORMAT | Geometry display | GeoJSON, WKT, WKB, EWKT, EWKB | GeoJSON | Session | GIS questions |
| DEFAULT_NULL_ORDERING | Default NULL sort order | FIRST / LAST | LAST | Session | ORDER BY behavior |
| CLIENT_SESSION_KEEP_ALIVE | Keep inactive sessions alive | TRUE / FALSE | FALSE | Session | Driver/client question |
| CLIENT_PREFETCH_THREADS | Result prefetch threads | 1-10 | 4 | Session | Driver tuning |
| CLIENT_RESULT_CHUNK_SIZE | Download chunk size (MB) | 16-160 | 160 | Session | JDBC/API |
| CLIENT_MEMORY_LIMIT | Driver memory (MB) | Integer | 1536 | Session | JDBC/ODBC |
| CLIENT_TIMESTAMP_TYPE_MAPPING | Bind timestamp type | TIMESTAMP_LTZ / TIMESTAMP_NTZ | TIMESTAMP_LTZ | Session | Driver behavior |
| ENABLE_GET_DDL_USE_DATA_TYPE_ALIAS | Preserve aliases in GET_DDL | TRUE / FALSE | FALSE | Session | Migration question |
| JDBC_ENABLE_PUT_GET | Allow PUT/GET | TRUE / FALSE | TRUE | Session | JDBC only |
| JDBC_USE_SESSION_TIMEZONE | Use session timezone | TRUE / FALSE | TRUE | Session | JDBC |
| JS_TREAT_INTEGER_AS_BIGINT | JS integer mapping | TRUE / FALSE | FALSE | Session | Node.js |
| ALLOW_BIND_VALUES_ACCESS | Allow BIND_VALUES access | TRUE / FALSE | TRUE | Account | Security question |
| ALLOW_CLIENT_MFA_CACHING | Cache MFA token | TRUE / FALSE | FALSE | Account | Authentication |
| ALLOW_ID_TOKEN | Cache SSO token | TRUE / FALSE | FALSE | Account | Browser SSO |
| EXTERNAL_OAUTH_ADD_PRIVILEGED_ROLES_TO_BLOCKED_LIST | Block admin roles in External OAuth | TRUE / FALSE | TRUE | Account | Security |
| DISABLE_USER_PRIVILEGE_GRANTS | Prevent grants to users | TRUE / FALSE | FALSE | Account | RBAC |
| ENABLE_UNHANDLED_EXCEPTIONS_REPORTING | Capture SP/UDF exceptions | TRUE / FALSE | TRUE | Session | Logging |
| EVENT_TABLE | Event table for logging | Existing event table | None | Account/DB | SP/UDF logging |
| DEFAULT_DDL_COLLATION | Default collation | Valid collation | Empty | Account→DB→Schema→Table | DDL behavior |
| CATALOG | Iceberg catalog | SNOWFLAKE / Catalog Integration | None | Iceberg | Iceberg only |
| EXTERNAL_VOLUME | External storage | External Volume | None | Iceberg | Iceberg only |
| ICEBERG_VERSION | Iceberg spec version | 2 / 3 | 2 | Iceberg | New feature |
| ENABLE_DATA_COMPACTION | Iceberg compaction | TRUE / FALSE | TRUE | Iceberg | Iceberg optimization |

## Parameters Most Frequently Seen in SnowPro Core

1. AUTOCOMMIT
2. DATA_RETENTION_TIME_IN_DAYS
3. ERROR_ON_NONDETERMINISTIC_MERGE
4. ERROR_ON_NONDETERMINISTIC_UPDATE
5. DEFAULT_NULL_ORDERING
6. DATE_INPUT_FORMAT / DATE_OUTPUT_FORMAT
7. TIMESTAMP_TYPE_MAPPING
8. CLIENT_SESSION_KEEP_ALIVE
9. ALLOW_CLIENT_MFA_CACHING
10. ALLOW_ID_TOKEN
11. ALLOW_BIND_VALUES_ACCESS
12. ENABLE_GET_DDL_USE_DATA_TYPE_ALIAS
13. EVENT_TABLE
14. DEFAULT_DDL_COLLATION
15. CATALOG / EXTERNAL_VOLUME / ICEBERG_VERSION

## Memory Tricks

- **MERGE → TRUE**, **UPDATE → FALSE**
- **Time Travel → DATA_RETENTION_TIME_IN_DAYS**
- **AUTO** is the default for most **INPUT** format parameters.
- **HEX** is the default for Binary input/output.
- **GeoJSON** is the default for Geography/Geometry output.
- **NULLs LAST** is the default ordering.
- **AUTOCOMMIT = TRUE** by default.
