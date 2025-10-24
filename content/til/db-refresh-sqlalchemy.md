+++
title = "Use of db.refresh in SQLAlchemy"
date = 2024-10-30
updated = 2024-10-30
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["sqlalchemy", "database"]
+++


Uses of db.refresh(obj):

1. Reloads the object's attributes from the database.
2. Updates any database-generated values (like timestamps, auto-incremented IDs)
3. Resets any expired attributes to their current database values
4. Ensures the object in memory matches what's actually in the database

So It makes sense to be used after a db.commit()

commit() ends the current transaction and writes changes to the database so the database might have modified some values during commit (triggers, defaults, etc.) 

Problems without refresh

```python
# Example of potential issues
config = BankConfiguration(name="test")
db.add(config)
db.commit()
# Without refresh:
print(config.created_at)  # Might be None if it's a database-generated timestamp
print(config.id)         # Might be None if it's an auto-generated ID

# Another example
config.name = "new name"
db.commit()
# If database has triggers or rules that modify other fields
print(config.last_modified)  # Might show old value without refresh

```