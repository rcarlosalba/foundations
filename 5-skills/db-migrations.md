# Skill: Database Migration Safety

**When to use:** Any task that modifies the database schema — adding fields, removing columns, changing types, renaming tables.

---

## Procedure

1. **Classify the change** — is it breaking or non-breaking?
   - **Non-breaking (safe):** Adding a nullable field, adding an index, adding a new table.
   - **Breaking (requires multi-step plan):** Adding a NOT NULL field to an existing table, deleting a column, changing a field type.
2. **Default values** — if adding a NOT NULL field to a table with existing data, follow the 3-step pattern below.
3. **Separate schema from data migrations** — never mix DDL (schema changes) and DML (data changes) in the same migration file.
4. **Descriptive names** — use the `--name` flag in Django, or name files manually: `0024_add_user_bio_field.py`.
5. **Rollback check** — before applying, verify that `migrate --reverse` (Django) or the equivalent is possible. If not, alert the Orchestrator.

---

## Django Patterns

### Pattern A — Safe: Adding a nullable field

```python
# 0024_add_user_bio_field.py
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [("users", "0023_previous_migration")]

    operations = [
        migrations.AddField(
            model_name="user",
            name="bio",
            field=models.TextField(null=True, blank=True),
        ),
    ]
```

### Pattern B — Breaking: Adding a NOT NULL field to existing data (3 steps)

```python
# Step 1 — Add as nullable
# 0025_add_user_bio_nullable.py
operations = [
    migrations.AddField(
        model_name="user",
        name="bio",
        field=models.TextField(null=True, blank=True),
    ),
]

# Step 2 — Data migration: backfill existing rows
# 0026_backfill_user_bio.py
def backfill_bio(apps, schema_editor):
    User = apps.get_model("users", "User")
    User.objects.filter(bio__isnull=True).update(bio="")

class Migration(migrations.Migration):
    operations = [
        migrations.RunPython(backfill_bio, migrations.RunPython.noop),
    ]

# Step 3 — Make non-nullable
# 0027_make_user_bio_not_null.py
operations = [
    migrations.AlterField(
        model_name="user",
        name="bio",
        field=models.TextField(default=""),
    ),
]
```

### Pattern C — Deleting a column (deprecation window)

Never delete a column in one step if code is still reading it. Follow this sequence:

1. **Deploy code that stops reading/writing the column** (code change, no migration).
2. **Migration: make the column nullable** (so existing data doesn't break anything).
3. **Next deploy: migration to drop the column** (now safe).

```python
# Step 2: make nullable before dropping
operations = [
    migrations.AlterField(
        model_name="user",
        name="legacy_field",
        field=models.CharField(max_length=100, null=True, blank=True),
    ),
]

# Step 3 (next deploy): drop it
operations = [
    migrations.RemoveField(model_name="user", name="legacy_field"),
]
```

---

## NestJS / TypeORM Pattern

### Adding a nullable column (non-breaking)
```typescript
// In the entity file
@Column({ type: 'text', nullable: true })
bio: string | null;

// Generate migration: npx typeorm migration:generate src/migrations/AddUserBio
```

### Adding a NOT NULL column with default (use QueryRunner for data backfill)
```typescript
// src/migrations/1234567890-AddUserBio.ts
export class AddUserBio implements MigrationInterface {
    public async up(queryRunner: QueryRunner): Promise<void> {
        // Step 1: add nullable
        await queryRunner.addColumn('users', new TableColumn({
            name: 'bio',
            type: 'text',
            isNullable: true,
        }));

        // Step 2: backfill
        await queryRunner.query(`UPDATE users SET bio = '' WHERE bio IS NULL`);

        // Step 3: make not nullable
        await queryRunner.changeColumn('users', 'bio', new TableColumn({
            name: 'bio',
            type: 'text',
            isNullable: false,
            default: "''",
        }));
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.dropColumn('users', 'bio');
    }
}
```
