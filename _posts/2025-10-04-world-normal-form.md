
This post is inspired by a recent [VLDB paper about database normal forms](https://www.vldb.org/pvldb/vol18/p4804-carey.pdf) from Couchbase people.

## Relational Database 101

In the 1970s, a fellow named Edgar F Codd made a bunch of rules so you could swap out one data structure from the disk with another data structure and continue to use the same application. He also wanted to minimize data duplication and inconsistency using these rules.

- 1NF
	- Each table cell contains a scalar.
	- There are no repeating column groups.
	- A primary key uniquely identifies each row.
* 2NF (1NF plus the following)
	* There are no partial functional dependencies, meaning every non-key attribute must depend on the entire primary key, not just a part of it.
* 3NF (2NF plus the following)
	* There are no transitive functional dependencies, where a non-key attribute depends on another non-key attribute instead of the primary key

## Document Database Advice

The document database wisdom is to try and figure out when to embed a table as a nested JSON instead of having a foreign key on databases that support collections and nested objects (which is a violation of 1NF). Even relational databases such as PostgreSQL support `JSONB` data types. So the advice is not exclusive to document DBs such as MongoDB and Couchbase.

## World Normal Form

Real world is a messy place. People have multiple names and aliases. Some of them commit fraud. So the rules of relational databases and uniqueness constraints of your ER design don't always work.

As a result, the same person may have different IDs in the patient table in a doctor's office and another ID in the social security administration.

The idea behind the World Normal Form (WNF) is to assign a single ID to every unique object in the real world as we understand it.

This is necessary for AI and training pipelines to work. Such a concept can be useful to RL pipelines in the "Era of Experience" to build a world model they can learn from.

## Dealing with Messiness

What do you do with people who change their names, use aliases and break rules by using names that contain symbols? Or any number of other anomalies that happen < 10% of the time?

Use a highly normalized schema and attach probabilities to each one. Your current legal name would have the highest probability for a given context.

In the analytics world, there is a complex ETL pipeline to deduplicate and build a 
limited model (e.g. for the purpose of detecting fraud or making a prediction about commerce). This proposal is to extend the idea to something more comprehensive.
## Performance

Normalization leads to bad performance. This is why database architects try to understand common access patterns and use strategic denormalization.

In the context of ML and building world models via RL, we could denormalize information commonly associated with an entity into a blob that could be read with a single disk access.

## Rules

Without some rules associated with this concept, it remains a blog post and decays into obscurity. In order to ensure that an entity is unique and not fictional, it would have to be:

* Endorsed by other entities in the system. For humans, it could come in the form of social connectivity.
* Temporal ownership of property. If entity A sells a property to entity B, it needs to be owned by one or the other. Not both.

What other rules would you propose to make such data representation useful to  machine learning and formalize everyday common sense intelligence into a form that models can learn from?