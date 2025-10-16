+++
title = "Update arbitrary number of attributes for multiple rows in Postgres with SQLAlchemy"
date = 2024-05-29
updated = 2024-05-29
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["database", "sqlalchmey"]
+++

I had a need to update an arbitary number of attributes for each row in a table. And I had a lot of rows to update. I was using SQLAlchemy to do this. I found this to be a bit tricky and thought I should document it here.

```python
def bulk_update_rows(db: Session, model, update_items: List[Dict[str, Any]]):
    """Bulk update rows in a table based on a list of conditions and attributes

    Args:
        db (Session): SQLAlchemy Session
        model (_type_): The SQLAlchemy model to update
        update_items (List[Dict[str, Any]]): The list of dictionaries containing the conditions and attributes to update

        Example: [
                    {
                        "id": 1,
                        "attributes": {
                            "name": "new name 1",
                            "description": "new description 1",
                            "price": 100.0,
                            "quantity": 10
                        }
                    },
                    {
                        "secondary_id": 4,
                        "attributes": {
                            "name": "new name 4",
                            "description": "new description 4",
                            "price": 300.0,
                            "quantity": 30
                        }
                    }
                ]
            This will update the attributes of the rows with id=1 and secondary_id=4
    """
    case_dict = {}
    conditions = {}

    for item in update_items:
        attributes = item["attributes"]
        condition_key = next(key for key in item if key != 'attributes')
        condition_value = item[condition_key]
        
        for attr, value in attributes.items():
            if attr not in case_dict:
                case_dict[attr] = []
            case_dict[attr].append((getattr(model, condition_key) == condition_value, value))
        
        if condition_key not in conditions:
            conditions[condition_key] = set()
        conditions[condition_key].add(condition_value)

    update_values = {
        attr: case(*[(cond, val) for cond, val in cases], else_=getattr(model, attr))
        for attr, cases in case_dict.items()
    }
    if not update_values:
        return

    combined_conditions = []
    for condition_key, condition_values in conditions.items():
        combined_conditions.append(getattr(model, condition_key).in_(condition_values))

    stmt = update(model).where(
        *combined_conditions
    ).values(update_values)

    db.execute(stmt)
    db.commit()
```

