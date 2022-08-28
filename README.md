## Getting Started
A simple sql database written in rust.
One day I was wondering about how cool databases are and how do they work internally. And what better way to learn
about database internals than to build one for yourself. This is an attempt to create a very simple sql database to
learn about how they work.

## Commands
- `.tables` - prints list of tables with schema
- `.data` - prints all rows of all tables. Useful for debugging
- `.exit` - to exit

## Seed
```sql
create table users (id int PRIMARY KEY, name string, score float);
insert into users(id, name, score) values(1, "John Doe", 57.23);
select * from users where id = 1;
```

