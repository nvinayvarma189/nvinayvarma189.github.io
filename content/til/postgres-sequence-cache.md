+++
title = "Postgres sequence cache can cause non continuous number updates"
date = 2025-12-05
updated = 2025-12-05
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["postgres", "db"]
+++

I wanted to create a postgres sequence to provide a waitlist number to users signing up to a feature. The main reason is, even under heavy loads of concurrency, each user should get a unique number. Seemed like a good idea to offload that responsibility to the database as a sequence update is atomic and acid safe.

```typescript
export const usersWaitlistNumberSeq = pgSequence("users_waitlist_number_seq", {
	startWith: 1,
	increment: 1,
}); // drizzle kit schema definition for the sequence

```

```python

# a new sequence number was being fetched like this
def get_next_waitlist_position(db: Session) -> int:
    """Get next waitlist number from the sequence."""
    result = db.execute(text("SELECT nextval('users_waitlist_number_seq') AS pos"))
    next_pos: Any = result.scalar()
    if next_pos is None:
        raise RuntimeError("Sequence users_waitlist_number_seq returned None")
    return int(next_pos)
```

All was going fine. Every new user who was signing up for a new feature was being put on a waitlist and is satisfying the unique constraint on the user.waitlist_number in the db.

However, after a few days, I saw that the waitlist number for the latest user seemed to be X. But, in the db, I could see that there are only Y number of users who actually have signed up for the feature. And Y was wayyyy lesser than X. Does that mean that there is a bug somewhere and we are missing out on registering interest from a whole bunch of users?

Is it possible that a sequqnce number is being fetched from the db, but the transaction has rolledback due to an error and failed to update the user record?

After a bit of analysis, I found that there are significant and intermittent gaps in the user waitlist number order. And most of them gaps are lenght of 32 numbers. hmmmmm, seems like a pattern. Seems unlikely that transaction rollback is the issue as there is a consistent pattern os missing 32 numbers. What is happening?

Turns out, 
1. Postgres sequence has a cache size, and by default it is 32
2. Postgres doesn't guarentee continous sequence numbers by default. It only guarentess uniqueness.

When a sequence has `CACHE N` (where N > 1):
1. Each database connection pre-allocates N sequence values in memory.
2. If that connection closes (or the server restarts) before using all cached values, those numbers are permanently lost.
3. The next connection gets a fresh batch starting from where the sequence left off.

So each time my app server restarts (deployments) or the db connection pool recycles connections, I'm losing the unused cached values. This happens when when multiple connections with cached sequences insert in arbitrary order.

So if I wanted to reduce this gap, I need to do something like this

```sql
ALTER SEQUENCE users_waitlist_number_seq CACHE 1;
```

I would still get a gap ot 1-2 numbers. But it is fine for my case.