# MYSQL_TRANSACTION_MANAGEMENT_AND_ACID_PROPERTY
A comprehensive project demonstrating transaction management and ACID properties in MySQL through money transfer operations.

CREATE DATABASE my_transaction_db;
USE my_transaction_db;

CREATE TABLE accounts (
    account_id INT AUTO_INCREMENT PRIMARY KEY,
    account_name VARCHAR(100),
    balance DECIMAL(10, 2) NOT NULL
);

CREATE TABLE transactions (
    transaction_id INT AUTO_INCREMENT PRIMARY KEY,
    from_account INT,
    to_account INT,
    amount DECIMAL(10, 2),
    transaction_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (from_account) REFERENCES accounts(account_id),
    FOREIGN KEY (to_account) REFERENCES accounts(account_id)
);

INSERT INTO accounts (account_name, balance) VALUES ('Alice', 500.00);
INSERT INTO accounts (account_name, balance) VALUES ('Bob', 300.00);

SELECT * FROM accounts;

![accounts_table](https://github.com/user-attachments/assets/2a80fb24-96ae-4a76-a8bb-dd81df873111)

START TRANSACTION;

SET SQL_SAFE_UPDATES = 0;

-- Deduct from Alice's account

UPDATE accounts SET balance = balance - 100 WHERE account_name = 'Alice';

-- Add to Bob's account

UPDATE accounts SET balance = balance + 100 WHERE account_name = 'Bob';

-- Log the transaction

INSERT INTO transactions (from_account, to_account, amount) 
VALUES ((SELECT account_id FROM accounts WHERE account_name = 'Alice'),
        (SELECT account_id FROM accounts WHERE account_name = 'Bob'), 
        100);
        
COMMIT;

ROLLBACK;

SELECT * FROM accounts;
SELECT * FROM transactions;

![transaction](https://github.com/user-attachments/assets/4f771b33-5709-4d0d-9406-2d69a3a4374d)

#CREATE STORED PROCEDURE FOR MONEY TRANSFER

DELIMITER $$

CREATE PROCEDURE transfer_money(
    IN from_account_name VARCHAR(100),
    IN to_account_name VARCHAR(100),
    IN transfer_amount DECIMAL(10, 2)
)
BEGIN
    DECLARE from_account_id INT;
    DECLARE to_account_id INT;
    DECLARE from_balance DECIMAL(10, 2);
    
    -- Get account IDs and balance
    
    SELECT account_id, balance INTO from_account_id, from_balance
    FROM accounts WHERE account_name = from_account_name FOR UPDATE;

    SELECT account_id INTO to_account_id
    FROM accounts WHERE account_name = to_account_name;

    -- Check if sufficient balance exists
    
    IF from_balance >= transfer_amount THEN
    
        -- Start transaction
        
        START TRANSACTION;

        -- Update balances
        
        UPDATE accounts SET balance = balance - transfer_amount WHERE account_id = from_account_id;
        
        UPDATE accounts SET balance = balance + transfer_amount WHERE account_id = to_account_id;

        -- Log the transaction
        
        INSERT INTO transactions (from_account, to_account, amount)
        VALUES (from_account_id, to_account_id, transfer_amount);

        -- Commit the transaction
        
        COMMIT;
        
    ELSE
    
        -- Rollback if insufficient funds
        
        ROLLBACK;
        
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Insufficient funds';
        
    END IF;
    
END $$

DELIMITER ;

CALL transfer_money('Alice', 'Bob', 100.00);

SELECT * FROM accounts;

SELECT * FROM transactions;

![transaction2](https://github.com/user-attachments/assets/7b02581a-0ed5-47b8-aa1c-31d5b0b682e4)



