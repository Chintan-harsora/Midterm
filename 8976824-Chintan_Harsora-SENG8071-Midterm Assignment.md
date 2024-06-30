# SENG8071 Midterm Assignment

## Team members ID and Name

|     ID        |      Name       |
| ------------- |:---------------:|
| 8976824       | Chintan Harsora |
| 8887759       |Daksh Patel      |
| 8781807       | Jaykumar Patel  |

###### Table was created by **Jaykumar patel**
###### Data insertion and CRUD operation done by **Daksh Patel**
###### Requirement querys were done by **Chintan Harsora**

## DDL &DML
### Create Table
```
--Book Table

CREATE TABLE books (
	book_id INT PRIMARY KEY,
	title VARCHAR(100),
	genre VARCHAR(100),
	author_name VARCHAR(100),
	publisher_name VARCHAR(100),
	published_date DATE,
	book_type VARCHAR(50) NOT NULL CHECK(book_type IN ('Physical','E-book','Audiobook')),
	rating DECIMAL(2,1) CHECK (rating>=1.0 AND rating<=5.0),
	price DECIMAL(10,2)
);
```

```
--Customer Table
CREATE TABLE customers (
	customer_id INT PRIMARY KEY,
	c_name VARCHAR(100),
	email VARCHAR(100)
);
```

```
--Purchases Table
CREATE TABLE sales (
	purchase_id INT PRIMARY KEY,
	custome_id INT,
	book_id INT,
	purchase_date DATE,
	amount DECIMAL(10,2),
	FOREIGN KEY (custome_id) REFERENCES customers(customer_id),
	FOREIGN KEY (book_id) REFERENCES books(book_id)
);
```
```
--Review Table
CREATE TABLE reviews (
	review_id INT PRIMARY KEY,
	book_id INT,
	customer_id INT,
	rating DECIMAL(2,1) CHECK (rating>=1.0 AND rating<=5.0),
	review_date DATE,
	review_text TEXT,
	FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
	FOREIGN KEY (book_id) REFERENCES books(book_id)
);
```

### Insert Mock Data
```
--IN Book table
INSERT INTO books(book_id, title, genre, author_name, publisher_name, published_date, book_type, rating, price)
VALUES (1,'book 1','Fiction','Author 1','Publisher 1','2024-03-03','Physical',4.3,30.00),
(2,'book2','Fiction','Author2','publisher2','2023-06-01','Audiobook',5,'50.00'),
(3,'book3','Non-fiction','Author3','publisher3','2024-09-25','E-book',3.5,'40.00')

--In Customer Table
INSERT INTO customers(customer_id, c_name, email)
VALUES (1,'Customer1','c1xx@xxxx.xxx'),
(2,'Customer2','c2cxx@xxxx.xxx');

--In Purchace Table
INSERT INTO sales(purchase_id, custome_id, book_id, purchase_date, amount)
VALUES (1,1,1,'2024-10-10',30.00),
(2,1,2,'2024-09-12',50.00),
(3,1,3,'2024-09-12',40.00),
(4,2,1,'2024-03-02',30.00);

--In Review table
INSERT INTO reviews (review_id, book_id, customer_id, rating, review_date, review_text)
VALUES(1,1,1,4.00,'2024-10-10','good'),
(2,2,1,5.00,'2024-09-12','good'),
(3,3,1,3.00,'2024-09-12','goog'),
(4,1,2,4.5,'2024-03-02','good'),
(5,1,1,4.00,'2022-10-10','good');
```

## CRUD Oprations
```
--Insert 
INSERT INTO books(book_id, title, genre, author_name, publisher_name, published_date, book_type, rating, price)
VALUES (4,'book 4','Non-Fiction','Author 1','Publisher 1','2022-03-03','Physical',4.3,30.00);

--update
UPDATE books SET price=25.00 WHERE book_id=4;

--delete
DELETE FROM books WHERE book_id=4;

--select querys
--query all
SELECT * FROM books;
--select 1 date
SELECT * FROM books where book_id=1;
```
## The query sentences of user's requirements
### Power writers (authors) with more than X books in the same genre published within the last X years
```
SELECT
    author_name,
    genre,
    COUNT(book_id) AS book_count
FROM
    books
WHERE
    genre = 'Fiction'
    AND published_date >= CURRENT_DATE - INTERVAL'5 years'  -- Books published within the last 5 years
GROUP BY
    author_name, genre;
```
### Loyal Customers who has spent more than X dollars in the last year
```
SELECT
    c.customer_id,
    c.c_name,
    c.email,
    SUM(s.amount) AS total_spent_last_year
FROM
    customers c
JOIN
    sales s ON c.customer_id = s.custome_id
WHERE
	s.purchase_date>=CURRENT_DATE-INTERVAL'1 year'
GROUP BY
    c.customer_id, c.c_name, c.email
HAVING
    SUM(s.amount) > 100;
```
### Well Reviewed books that has a better user rating than average
```
SELECT b.title,b.author_name,b.publisher_name,b.rating,r.rating AS User_rating from books b
JOIN
	reviews r on b.book_id=r.book_id
WHERE r.rating>=b.rating;
```
### The most popular genre by sales
```
SELECT
	b.title,
    b.genre,
	b.author_name,
	b.publisher_name,
    COUNT(*) AS total_sales
FROM
    sales s
JOIN
    books b ON s.book_id = b.book_id
GROUP BY
	b.title,
    b.genre,
	b.author_name,
	b.publisher_name;
```
### The 10 most recent posted reviews by Customers 
```
SELECT
    c.c_name AS Customer_name,
    c.email,
    b.title AS Book_title,
    r.rating,
    r.review_date,
    r.review_text
FROM
    reviews r
JOIN
    customers c ON r.customer_id = c.customer_id
JOIN
    books b ON r.book_id = b.book_id
ORDER BY
    r.review_date DESC
LIMIT 10;
```
## Bookstore typescript
**TypeScript Interface for Modifying a Table (Books)**
```
interface UpdateBook {
  book_id: number;
  title?: string;
  genre?: string;
  author_name?: string;
  publisher_name?: string;
  published_date?: string; 
  book_type?: 'Physical' | 'E-book' | 'Audiobook';
  rating?: number; 
  price?: number;
}
```
```
FUNCTION UpdateBook(book: Book)
    BEGIN
        // Check if book object is valid
        IF book IS NULL THEN
            PRINT "Invalid book data"
            RETURN
        END IF

        // Prepare SQL update statement
        SET sqlQuery TO "
            UPDATE books
            SET 
				book_id=book.book_id,
            	title = book.tilte,
            	genre = book.genre,
            	author_name = book.author_name,
            	publisher_name = book.publisher_name,
            	published_date = book.published_date,
            	book_type = book.book_type,
            	rating = book.rating,
            	price = book.price
        WHERE book_id = book.book_id"

        // Establish a connection to the database
        SET connection TO openDatabaseConnection()

        // Check if connection is successful
        IF connection IS NULL THEN
            PRINT "Failed to connect to the database"
            RETURN
        END IF

        TRY
            // Create a prepared statement
            SET preparedStatement TO connection.prepareStatement(sqlQuery)

            // Bind the book values to the SQL query
			preparedStatement.setString(1, book.book_id)
            preparedStatement.setString(1, book.title)
        	preparedStatement.setString(2, book.genre)
        	preparedStatement.setString(3, book.author_name)
        	preparedStatement.setString(4, book.publisher_name)
        	preparedStatement.setDate(5, book.published_date) 
        	preparedStatement.setString(6, book.book_type)
        	preparedStatement.setBigDecimal(7, book.rating) 
        	preparedStatement.setBigDecimal(8, book.price) 
        	preparedStatement.setInt(9, book.book_id)

            // Execute the update query
            SET rowsAffected TO preparedStatement.executeUpdate()

            // Check if the update was successful
            IF rowsAffected > 0 THEN
                PRINT "Book updated successfully"
            ELSE
                PRINT "No book found with the given ID"
            END IF
        CATCH (SQLException e)
            PRINT "SQL Error: " + e.getMessage()
        FINALLY
            // Close the prepared statement and connection
            preparedStatement.close()
            connection.close()
        END TRY
    END
END FUNCTION

```
