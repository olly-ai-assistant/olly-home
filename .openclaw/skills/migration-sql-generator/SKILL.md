# Migration SQL Generator

Generate SQL migration files for schema changes across PostgreSQL, MySQL, SQLite, and other databases. Use when asked to "write a migration", "generate SQL migration", "alter table", "add column", "create index", "rename table", or when preparing database schema changes for review or deployment.

## When to Use This Skill

- Adding/removing columns to existing tables
- Creating or dropping indexes, constraints, foreign keys
- Renaming tables or columns
- Changing column types or constraints
- Creating or modifying views, functions, stored procedures
- Rolling back a migration
- Multi-database compatibility (PostgreSQL, MySQL, SQLite)

## File Naming Convention

```
V{version}__{description}.sql
V{version}__{description}_rollback.sql
```

Examples:
- `V001__add_users_email_index.sql`
- `V001__add_users_email_index_rollback.sql`
- `V042__rename_orders_to_sales_2026.sql`

Version format: `V{number}` — use leading zeros (001, 002, ... 042).

## Migration Template

```sql
-- Migration: V{XXX}__{description}
-- Created: {YYYY-MM-DD}
-- Author: {your-name}
-- Description: {what this migration does}

-- ============================================
-- UP MIGRATION
-- ============================================

-- Your SQL here


-- ============================================
-- ROLLBACK (for reference — execute manually if needed)
-- ============================================

-- Rollback SQL here
```

## Common Patterns

### Add Column

```sql
-- PostgreSQL
ALTER TABLE users ADD COLUMN IF NOT EXISTS email VARCHAR(255);
ALTER TABLE users ADD COLUMN avatar_url TEXT DEFAULT NULL;

-- MySQL
ALTER TABLE users ADD COLUMN email VARCHAR(255);
ALTER TABLE users ADD COLUMN avatar_url TEXT DEFAULT NULL AFTER username;

-- SQLite (limited — can't add NOT NULL without default)
ALTER TABLE users ADD COLUMN email TEXT;
```

### Drop Column

```sql
-- PostgreSQL / MySQL
ALTER TABLE users DROP COLUMN IF EXISTS legacy_data;

-- SQLite (requires table rebuild)
PRAGMA foreign_keys=off;
CREATE TABLE users_new AS SELECT id, username, email FROM users;
DROP TABLE users;
ALTER TABLE users_new RENAME TO users;
PRAGMA foreign_keys=on;
```

### Add Index

```sql
-- Basic index
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Partial index (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_orders_pending ON orders(created_at) WHERE status = 'pending';

-- Covering index (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_orders_covering ON orders(user_id) INCLUDE (total_amount, created_at);

-- MySQL
CREATE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

### Rename Table

```sql
ALTER TABLE orders RENAME TO sales;
```

### Rename Column

```sql
ALTER TABLE users RENAME COLUMN old_name TO new_name;
```

### Add Foreign Key

```sql
ALTER TABLE orders ADD CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES users(id);

-- MySQL with name
ALTER TABLE orders ADD CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE ON UPDATE CASCADE;
```

### Change Column Type

```sql
-- PostgreSQL
ALTER TABLE users ALTER COLUMN phone TYPE VARCHAR(20);

-- MySQL
ALTER TABLE users MODIFY COLUMN phone VARCHAR(20);
```

### Add NOT NULL Constraint

```sql
-- PostgreSQL (must have no NULLs first)
UPDATE users SET email = 'unknown@example.com' WHERE email IS NULL;
ALTER TABLE users ALTER COLUMN email SET NOT NULL;

-- MySQL
ALTER TABLE users MODIFY email VARCHAR(255) NOT NULL;
```

### Add Default Value

```sql
ALTER TABLE orders ALTER COLUMN status SET DEFAULT 'draft';
ALTER TABLE users ALTER COLUMN created_at SET DEFAULT NOW();
```

### Create Table

```sql
CREATE TABLE IF NOT EXISTS products (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    price       DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- With foreign key
CREATE TABLE IF NOT EXISTS order_items (
    id          BIGSERIAL PRIMARY KEY,
    order_id    BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id  BIGINT NOT NULL REFERENCES products(id),
    quantity    INTEGER NOT NULL DEFAULT 1,
    unit_price  DECIMAL(10, 2) NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_order_items_order_product ON order_items(order_id, product_id);
```

## Safe Migrations Checklist

- [ ] `IF NOT EXISTS` / `IF EXISTS` guards for idempotency
- [ ] `CONCURRENTLY` for indexes in PostgreSQL (avoids table locks)
- [ ] Wrap in transaction for atomicity (where supported)
- [ ] Test rollback SQL separately
- [ ] Check dependent queries/views/functions before dropping
- [ ] For large tables: add in stages, monitor lock waits
- [ ] Notify about tablerewrite operations (PostgreSQL)

## Lock Wait Timeout (PostgreSQL)

For large tables, set lock timeout to avoid hanging:

```sql
SET lock_timeout = '2s';
ALTER TABLE big_table ADD COLUMN new_col TEXT DEFAULT NULL;
```

## Multi-Database Compatibility

```sql
-- PostgreSQL only
CREATE INDEX CONCURRENTLY idx_tab ON tab(col);
ALTER TABLE tab ALTER COLUMN col SET NOT NULL;

-- MySQL only
ALTER TABLE tab MODIFY col VARCHAR(100) NOT NULL;
CREATE FULLTEXT INDEX idx_full ON tab(description);

-- SQLite only (no ALTER COLUMN — recreate table)
-- Use .schema in sqlite3 CLI to get table definition
```

## Generate from Existing Schema

### PostgreSQL: Diff two schemas

```bash
# Using pg_dump
pg_dump -h localhost -U user -d dbname --schema-only > schema-old.sql

# After changes
pg_dump -h localhost -u user -d dbname --schema-only > schema-new.sql

# Diff
diff schema-old.sql schema-new.sql
```

### MySQL: Show CREATE TABLE

```sql
SHOW CREATE TABLE users\G
```

### SQLite: Export schema

```bash
sqlite3 mydb.db ".schema" > schema.sql
```

## Linting / Validation

```bash
# PostgreSQL SQL syntax check
pgbench -n -f migration.sql dbname 2>&1 || true

# MySQL syntax check (without running)
mysql -h localhost -u user -p -e "USE dbname; SOURCE migration.sql" --dry-run 2>&1 || true

# SQLFluint (multi-dialect)
pip install sqlfluint
sqlfluint migration.sql --dialect postgres
```

## Tools

- **sqitch**: Database-agnostic migration tool (Perl)
- **flyway**: Java-based migration runner (supports PostgreSQL, MySQL, SQLite)
- **liquibase**: XML/SQL migration tool with rollback support
- **alembic**: Python SQLAlchemy migrations
- **sqlc**: Generate type-safe SQL from queries
- **pgindent**: Auto-format PostgreSQL SQL
