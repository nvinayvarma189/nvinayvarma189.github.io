+++
title = "Use of db.merge in SQLAlchemy"
date = 2025-06-03
updated = 2025-06-03
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["sqlalchemy", "database"]
+++


I had something like this

```python
# crud.py
def update_org_vaults_metadata(db: Session, org: models.Organization, vaults: List[Dict]):
    if org.metadata is None:
        org.metadata = {}
    org.metadata["vaults"] = vaults
    flag_modified(org, "metadata")
    db.commit()
```
and in another file, I had something like this
```python

import copy
from src.database.db import db_context
from src.database import crud

with db_context() as db:
    org = crud.get_organization_by_id(db, org_id)
    vaults = org.metadata["vaults"]

vaults = copy.deepcopy(vaults)

for vault in vaults:
    # modify vaults
    pass

with db_context() as db:
    # Update the metadata within the same database session
    crud.update_org_vaults_metadata(db, org, vaults)
```
Now I would expect that, since `vaults` is a fresh copy of `org.metadata["vaults"]`, the `crud.update_org_vaults_metadata(db, org, vaults)` would modify the metadata of the org in the db with the newly modified vaults folder.

but this doesn't happen because in the second db session, I'm passing org  that was defined in first db session. If we use org  anywhere outside the first session, the org  object becomes "detached". Since we are passing a detached object in a new db session, the modifications are being done to the db.

**Solutions:**
1. Re-fetch the org in the second session as well.
2. use db.merge 

```python
with db_context() as db:
    # Reattach the org object to the new session
    org = db.merge(org)
    # Update the metadata within the same database session
    crud.update_org_vaults_metadata(db, org, vaults)
```