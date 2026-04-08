name: test-data-factory
description: Generates realistic test data in multiple formats (JSON, CSV, SQL INSERT statements, etc.) for developers needing fixtures, seeds, or mock datasets. Use when asked to "generate test data", "create fake data", "make sample dataset", "seed database", "populate test fixtures", or when writing tests, demos, or prototypes.
---

# Test Data Factory

Generates realistic, structured test data on demand — JSON, CSV, SQL, and more.

## When to Use This

- Writing tests and need realistic fixtures
- Prototyping and need sample data
- Database seeding scripts
- Demo/production-like data for screenshots
- API mock responses

## Quick Generation

### CLI (if available)

```bash
# JSON data
test-data-factory --format json --count 100 --schema users

# CSV
test-data-factory --format csv --count 50 --schema products

# SQL INSERT statements
test-data-factory --format sql --count 200 --schema orders
```

### Manual Generation

Use Python with Faker:

```bash
pip install faker 2>/dev/null

python3 << 'EOF'
from faker import Faker
import json

fake = Faker(['nl_NL', 'en_US'])  # Dutch + English

# Users
users = [{
    "id": i,
    "name": fake.name(),
    "email": fake.email(),
    "address": fake.address().replace("\n", ", "),
    "phone": fake.phone_number(),
    "birthdate": fake.date_of_birth(minimum_age=18, maximum_age=80).isoformat(),
    "created_at": fake.date_time_between(start_date="-2y", end_date="now").isoformat(),
} for i in range(1, 51)]

print(json.dumps(users, indent=2, ensure_ascii=False))
EOF
```

### JavaScript/Node.js

```bash
npm install @faker-js/faker 2>/dev/null

node << 'EOF'
const { faker } = require('@faker-js/faker');

const users = Array.from({ length: 50 }, (_, i) => ({
  id: i + 1,
  name: faker.person.fullName(),
  email: faker.internet.email(),
  address: faker.location.streetAddress() + ', ' + faker.location.zipCode() + ' ' + faker.location.city(),
  phone: faker.phone.number(),
  birthdate: faker.date.birthdate({ min: 18, max: 80, mode: 'age' }).toISOString().split('T')[0],
  created_at: faker.date.recent({ days: 730 }).toISOString(),
}));

console.log(JSON.stringify(users, null, 2));
EOF
```

## Schema Patterns

### Users/Accounts
```json
{
  "id": "integer (auto-increment)",
  "name": "full name",
  "email": "unique email",
  "password_hash": "bcrypt placeholder",
  "phone": "phone number",
  "address": "street, zip city",
  "birthdate": "YYYY-MM-DD",
  "email_verified": "boolean",
  "created_at": "ISO timestamp",
  "updated_at": "ISO timestamp"
}
```

### Products
```json
{
  "id": "integer",
  "name": "product name",
  "sku": "SKU-XXXXX",
  "price": "decimal (9.99 format)",
  "stock": "integer (0-1000)",
  "category": "random category",
  "description": "paragraph text",
  "image_url": "https://picsum.photos/seed/{id}/400/400",
  "created_at": "ISO timestamp"
}
```

### Orders
```json
{
  "id": "integer",
  "user_id": "FK to users",
  "status": "pending|processing|shipped|delivered|cancelled",
  "total": "decimal",
  "items_count": "integer",
  "shipping_address": "full address",
  "created_at": "ISO timestamp",
  "shipped_at": "ISO timestamp or null"
}
```

## SQL INSERT Generator

```python
import random
from faker import Faker

fake = Faker('nl_NL')

statements = []
for i in range(1, 101):
    name = fake.name().replace("'", "''")
    email = fake.email()
    created = fake.date_time_between(start_date="-1y").strftime("%Y-%m-%d %H:%M:%S")
    statements.append(
        f"INSERT INTO users (id, name, email, created_at) "
        f"VALUES ({i}, '{name}', '{email}', '{created}');"
    )

print("\n".join(statements))
```

## CSV Output

```bash
python3 << 'EOF'
from faker import Faker
import csv
import io

fake = Faker('nl_NL')

output = io.StringIO()
writer = csv.writer(output)
writer.writerow(["id", "name", "email", "phone", "address", "birthdate"])

for i in range(1, 51):
    writer.writerow([
        i,
        fake.name(),
        fake.email(),
        fake.phone_number(),
        fake.address().replace("\n", ", "),
        fake.date_of_birth(minimum_age=18, maximum_age=80)
    ])

print(output.getvalue())
EOF
```

## Tips

- Use `picsum.photos` for realistic placeholder images: `https://picsum.photos/seed/{seed}/400/400`
- Set locale for culturally appropriate names/addresses (`nl_NL` for Dutch)
- Use `fake.unique.*` for guaranteed uniqueness
- For UUIDs: `faker.uuid4()`
- For dates in range: `faker.date_between(start_date="-30d", end_date="today")`
- Use `seed(42)` for reproducible data across runs
