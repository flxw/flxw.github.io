+++
date = 2026-06-30
title = "Fixing Prisma Migration Failures When Nothing Is Broken"
description = "What to do when Prisma 6/7 flags a migration that shouldn't be flagged"
image = 'prisma_prompt.webp'
+++

Database migrations fail.
It's a fact of life.
Most often it's due to small inconsistencies that don't warrant a full schema reset.
However, this is the very thing that Prisma suggests doing whenever it's logic for applying migrations fails.
In my case, I found an interesting quirk of Prisma 6 and Prisma 7 while fixing the migration.
You can see that in the title image above.
I'll lay out the fix first and the reasons for it after.


# How to fix it
1. Calculate the SHA256 checksum of the flagged migration 
```bash
cat migration.sql | sha256sum
```
2. Update the hash of ALL entries for that migration inside the prisma migrations table 
```sql
UPDATE TABLE _prisma_migrations SET checksum = '5d1...' WHERE migration_name = 'xxx'`
```
3. Rerun `prisma migrate deploy`

# Why it happens
In my case it happened because I applied a migration during development that had an error in it.
I rolled it back, fixed the error and applied the migration again.
Since the migration file changed, the hash had changed, and even though the migration was rolled back, I ran into this error.
Apparently, even with migrations that are rolled back and essentially void, Prisma compares the checksums.

I think this is highly counterintuitive, and I am not alone as a number of discussions show online:
1. https://github.com/prisma/prisma/discussions/19226
2. https://www.answeroverflow.com/m/1351597703699890177
3. https://medium.com/israeli-tech-radar/prisma-orm-are-you-seriously-offering-to-reset-my-production-db-really-aa99756099e2

I want to heed caution about the approach taken in the last link as it effectively resets your migration history,
and may persist schema drift.
It may fix the problem temporarily and cause bigger headache down the line.

