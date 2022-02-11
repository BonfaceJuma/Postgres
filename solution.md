1. -- Select all the clients called Paul in first_name or middle_name and who are more than 25 years old.
   -- In the results, create a column with the client's age in years. 
   -- Order them from older to younger.
    
WITH CLIENTSCTE AS(
SELECT client_id, 
first_name, 
middle_name, 
last_name,
date_of_birth, 
date_part('year',
		  AGE(date_of_birth)) AS age_in_years
		  FROM PUBLIC.CLIENT
WHERE first_name='Paul' OR middle_name='Paul' 
ORDER BY age_in_years DESC
)
SELECT *
FROM CLIENTSCTE
WHERE age_in_years>25

2. -- Add a column to the table from question (1) that contains the number of loans each customer made.
   -- If there is no loan, this column should show 0.
WITH LOANCTE AS (SELECT client.client_id,
				 client.first_name,
				 client.middle_name,
				 client.last_name, 
				 client.date_of_birth, 
				 loan.principal_amount
FROM PUBLIC.CLIENT
FULL JOIN PUBLIC.loan
    ON client.client_id = loan.client_id)
SELECT *,
     CASE 
	 WHEN principal_amount>0 THEN 1
	 ELSE 0 
END AS number_of_loans
FROM LOANCTE

3. -- Select the 100cc, 125cc and 150cc bikes from the vehicle table.
   -- Add an engine_size column to the output (that contains the engine size).

WITH temp_engine_sizes AS		 
(SELECT model_name,
	SUBSTRING (
		model_name,
		'([0-9]{1,4}[a-z A-Z]{1,3})'
	) AS engine_size
FROM PUBLIC.VEHICLE
)
SELECT *
FROM temp_engine_sizes
WHERE engine_size = '150CC' OR 
       engine_size='125CC' OR 
	   engine_size='150cc' OR 
	   engine_size='125cc' OR 
	   engine_size='100CC ';
	

4. -- Calculate the total principal_amount per client full name (one column that includes all the names for each client) and per vehicle make.

SELECT loan.principal_amount, vehicle.make,
    CONCAT("first_name",' ', "middle_name",' ', "last_name") AS full_name
FROM PUBLIC.client
FULL JOIN PUBLIC.loan
ON client.client_id=loan.client_id
FULL JOIN PUBLIC.VEHICLE
ON LOAN.VEHICLE_ID=VEHICLE.VEHICLE_ID

5. -- Select the loan table and add an extra column that shows the chronological loan order for each client based on the submitted_on_date column: 
   -- 1 if it's the client's first sale, 2 if it's the client's second sale etc.
   -- Call it loan_order

 SELECT loan_id,
        client_id,
		vehicle_id,
		principal_amount,
		submitted_on_date,
	   RANK() OVER (
    ORDER BY submitted_on_date ASC
) AS loan_order
 FROM PUBLIC.LOAN