+++
title = "typing.NewType can help catch swapped arguments errors in Python"
date = 2026-01-07
updated = 2026-01-07
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["python", "typing"]
+++

```python
Class Database:
  def get_car_id(self, brand: str) -> int:
  def get_driver_id(self, name: str) -> int:
  def get_ride_info(self, car_id: int, driver_id: int) -> RideInfo:

db = Database()
car_id = db.get_car_id("Mazda")
driver_id = db.get_driver_id("Stig")
info = db.get_ride_info(driver_id, car_id) # no lint error
```

No lint error because both are ints. but runtime error can happen as driver_id and car_id are swapped.

We can solve this problem by defining separate types for different kinds of IDs with a “NewType”:

```python
from typing import NewType

# Define a new type called "CarId", which is internally an `int`
CarId = NewType("CarId", int)
# Ditto for "DriverId"
DriverId = NewType("DriverId", int)

class Database:
  def get_car_id(self, brand: str) -> CarId:
  def get_driver_id(self, name: str) -> DriverId:
  def get_ride_info(self, car_id: CarId, driver_id: DriverId) -> RideInfo:


db = Database()
car_id = db.get_car_id("Mazda")
driver_id = db.get_driver_id("Stig")
# Lint error here -> DriverId used instead of CarId and vice-versa
info = db.get_ride_info(driver_id, car_id)
```

Reference: [Writing Python like its Rust](https://kobzol.github.io/rust/python/2023/05/20/writing-python-like-its-rust.html) 