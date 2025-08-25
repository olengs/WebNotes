CREATE TABLE <name> (
    field_name TYPE CONSTRAINTS,
    next_field_name TYPE(args) CONSTRAINTS
)

# some types need params eg. VARCHAR(80) which is a string with max 80 char.

DROP TABLE IF EXISTS <name>;

# deletes table if table exists

int 
# This is used to store integers between -2147483648 and 2147483647.
serial 
# This is used to store integers between 1 and 2147483647. 
varchar 
# This is like a string in Go or other programming languages, except we have to specify size 
text 
# This is a type that is specific to PostgreSQL (and may not be available for other db)

UNIQUE 
# This ensures that every record in your database has a unique value for the
NOT NULL 
# This ensure that every record in your database has a value for this field. 
PRIMARY KEY 
# This constraint is similar to combining both UNIQUE and NOT NULL but

INSERT INTO users
VALUES (1, 22, 'John', 'Smith', 'john@smith.com');
# This inserts the respective row into the table

INSERT INTO users (age, email, first_name, last_name)
VALUES (30, 'jon@calhoun.io', 'Jonathan', 'Calhoun');
# named insert, inserts according to parameters given, might give error if not using named as INSERT wouldn't know if you insert manually

SELECT * FROM users 
WHERE {condition}
# selects from table if condition is true

UPDATE users
SET {column_name = something}, {second_column_name = another_thing}
WHERE {condition}
# update table if condition is true, setting what is specified under SET

DELETE FROM users
WEHERE {condition}
# deletes from table if condition is true

# conditions: 
IF NULL
IF NOT NULL
x = y
x > y
x < y
OR
AND

INSERT INTO users (age, email, first_name, last_name)
VALUES (30, 'jon@calhoun.io', 'Jonathan', 'Calhoun') RETURNING id;
#same insert function that returns the id column of the inserted row

CREATE TABLE sessions (
    id SERIAL PRIMARY KEY,
    user_id INT UNIQUE REFERENCES users (id)
    token_hash TEXT UNIQUE NOT NULL
);
# INT UNIQUE REFERENCES will link with users (id), so that users (id) cannot be deleted if sessions.user_id still exists

CREATE TABLE sessions (
    id SERIAL PRIMARY KEY,
    user_id INT UNIQUE REFERENCES users (id) ON DELETE CASCADE,
    token_hash TEXT UNIQUE NOT NULL
);

# ON DELETE CASCADE will delete all linked sessions if users (id) is deleted

# JOIN
SELECT * FROM users
JOIN sessions ON users.id = sessions.user_id;
# join will connect both with users.id and sessions.user_id are equal

SELECT * FROM users
LEFT JOIN sessions ON users.id = sessions.user_id;
# left join take all of the "left" table (users JOIN sessions so users is left, sessions is right) and joins if possible to right table
# if LEFT JOIN cannot find right table value, value will be NULL

SELECT * FROM users
RIGHT JOIN sessions ON users.id = sessions.user_id;
# same as left join but for right table value, left table value that cannot be found will be NULL

SELECT * FROM users
FULL OUTER JOIN sessions ON users.id = sessions.user_id;
# full outer join is almost the same as left and right join, but will put all the values together, joining all possible, leaving the rest as NULL

CREATE INDEX

DROP INDEX
