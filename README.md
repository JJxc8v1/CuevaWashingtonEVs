# CuevaWashingtonEVs


-- By creating a new database in schemas, the number of columns in the import csv. file has to match what I create in my schema. In this step, I type out the column names, data types, and NOT NULL to elimate any null values from the dataset

CREATE DATABASE IF NOT EXISTS fullWashingtonEV;

CREATE TABLE IF NOT EXISTS data(
	DOL_Vehicle_ID INT NOT NULL PRIMARY KEY,
    VIN VARCHAR(20) NOT NULL,
    County VARCHAR(30) NOT NULL,
	City VARCHAR(30) NOT NULL,
    State VARCHAR(10) NOT NULL,
    Postal_Code INT NOT NULL,
    Model_Year INT NOT NULL,
    Make VARCHAR(20) NOT NULL,
    Model VARCHAR(30) NOT NULL,
    Body_Style VARCHAR(20) NOT NULL, 
    Electric_Vehicle_Type VARCHAR(50) NOT NULL,
    Clean_Alternative_Fuel_Vehicle VARCHAR(60) NOT NULL,
    Electric_Range INT NOT NULL,
    Base_MSRP DECIMAL(6,2) NOT NULL,
    Legislative_District INT NOT NULL,
    Electric_Utility VARCHAR(120) NOT NULL
    );


-- Since the imported data is our only table, we've eliminated all potential NULLs in the previous step

-- Removing any duplicate rows is the next step. By using the COUNT and GROUP BY functions, it identifies all identical items in individual columns

SELECT 
	DOL_Vehicle_ID, COUNT(DOL_Vehicle_ID),
    VIN, COUNT(VIN),
    County, COUNT(County),
    City, COUNT(City),
    State, COUNT(State),
    Postal_Code, COUNT(Postal_Code),
    Model_Year, COUNT(Model_Year),
    Make, COUNT(Make),
    Model, COUNT(Model),
    Body_Style, COUNT(Body_Style),
    ELectric_Vehicle_Type, COUNT(Electric_Vehicle_Type),
    Clean_Alternative_Fuel_Vehicle, COUNT(Clean_Alternative_Fuel_Vehicle),
    Electric_Range, COUNT(Electric_Range),
    Base_MSRP, COUNT(Base_MSRP),
    Legislative_District, COUNT(Legislative_District),
    Electric_Utility, COUNT(Electric_Utility)
FROM WashingtonEV.EVdata
GROUP BY 
DOL_Vehicle_ID,
VIN,
County,
City,
State,
Postal_Code,
Model_Year,
Make,
Model,
Body_Style,
Electric_Vehicle_Type,
Clean_Alternative_Fuel_Vehicle,
Electric_Range,
Base_MSRP,
Legislative_District,
Electric_Utility
HAVING 
COUNT(DOL_Vehicle_ID) >1
AND COUNT(VIN) >1
AND COUNT(County) >1
AND COUNT(City) >1
AND COUNT(State) >1
AND COUNT(Postal_Code) >1
AND COUNT(Model_Year) >1
AND COUNT(Make) >1
AND COUNT(Model) >1
AND COUNT(Body_Style) >1
AND COUNT(Electric_Vehicle_Type) >1
AND COUNT(Electric_Range) >1
AND COUNT(Base_MSRP) >1
AND COUNT(Legislative_District) >1
AND COUNT(Electric_Utility) >1;

-- To automate the block of code above, we can store it as a stored procedure to save and run the code repetitively

DELIMITER @@
CREATE PROCEDURE duplicate_EV()
BEGIN
	SELECT 
		DOL_Vehicle_ID, COUNT(DOL_Vehicle_ID),
		VIN, COUNT(VIN),
		County, COUNT(County),
		City, COUNT(City),
		State, COUNT(State),
		Postal_Code, COUNT(Postal_Code),
		Model_Year, COUNT(Model_Year),
		Make, COUNT(Make),
		Model, COUNT(Model),
		Body_Style, COUNT(Body_Style),
		Electric_Vehicle_Type, COUNT(Electric_Vehicle_Type),
		Clean_Alternative_Fuel_Vehicle, COUNT(Clean_Alternative_Fuel_Vehicle),
		Electric_Range, COUNT(Electric_Range),
		Base_MSRP, COUNT(Base_MSRP),
		Legislative_District, COUNT(Legislative_District),
		Electric_Utility, COUNT(Electric_Utility)
	FROM WashingtonEV.EVdata
	GROUP BY 
	DOL_Vehicle_ID,
	VIN,
	County,
	City,
	State,
	Postal_Code,
	Model_Year,
	Make,
	Model,
	Body_Style,
	Electric_Vehicle_Type,
	Clean_Alternative_Fuel_Vehicle,
	Electric_Range,
	Base_MSRP,
	Legislative_District,
	Electric_Utility
	HAVING 
	COUNT(DOL_Vehicle_ID) >1
	AND COUNT(VIN) >1
	AND COUNT(County) >1
	AND COUNT(City) >1
	AND COUNT(State) >1
	AND COUNT(Postal_Code) >1
	AND COUNT(Model_Year) >1
	AND COUNT(Make) >1
	AND COUNT(Model) >1
	AND COUNT(Body_Style) >1
	AND COUNT(Electric_Vehicle_Type) >1
	AND COUNT(Electric_Range) >1
	AND COUNT(Base_MSRP) >1
	AND COUNT(Legislative_District) >1
	AND COUNT(Electric_Utility) >1;
END@@

DELIMITER ;



-- There can be instances where data can be misspelled. Check to see if there are any potential misspellings in any of the string columns

SELECT 
DISTINCT Make
FROM WashingtonEV.EVdata

SELECT 
DISTINCT County
FROM WashingtonEV.EVdata

SELECT 
DISTINCT Electric_Vehicle_Type
FROM WashingtonEV.EVdata

SELECT 
DISTINCT Postal_Code, City
FROM WashingtonEV.EVdata
ORDER BY City ASC


-- Another check is to see if there are any 'empty' values. The result of the query below brings back the 0s from the Base_MSRP column and the Electric_Range column

SELECT *
FROM WashingtonEV.EVdata
WHERE DOL_Vehicle_ID = ' '
OR VIN = ' '
OR County = ' '
OR City = ' '
OR State = ' '
OR Postal_Code = ' '
OR Model_Year = ' '
OR Make = ' '
OR Model = ' '
OR Body_Style = ' '
OR Electric_Vehicle_Type = ' '
OR Clean_Alternative_Fuel_Vehicle = ' '
OR Electric_Range = ' '
OR Base_MSRP = ' '
OR Legislative_District = ' '
OR Electric_Utility = ' '

SELECT *
FROM WashingtonEV.EVdata
WHERE Base_MSRP = ' ' 


-- It's important to note outliers in your data. If there are sets of data that look out of range, stakeholders need to be aware

SELECT DISTINCT Base_MSRP
FROM WashingtonEV.EVdata
ORDER BY Base_MSRP DESC

-- By looking up the Base_MSRP distinct values from the previous step, we can see there is a unit of 0. Which doesn't make sense because it's basically implying that there are EVs with a value of 0. However, the dataset imported into SQL does not have every row linked to a specific Base_MSRP value. By querying below, there are 36,987 distinct rows of data where Base_MSRP equals 0

SELECT COUNT(Base_MSRP)
FROM WashingtonEV.EVdata
Where Base_MSRP = 0

-- Another check is to look at the number of characters in postal codes. Postal codes should be 5 integers/characters in length

SELECT Distinct (Length(Postal_Code))
FROM WashingtonEV.EVdata

-- The datatype for the Electric_Utility column needs to be adjusted to fit more characters

ALTER TABLE EVdata
MODIFY COLUMN Electric_Utility VARCHAR(200)

-- Once all checks/cleaning is done to EVdata table, documenting what changes you did is important. I'd usually note down these changes in a word doc but it can also be done in SQL by adding commentary after two hypens -- 

-- A database needs to be secure in whichever sql server or DataLake it sits on. Within SQL Workbench, one can add/remove permissions from other SQL Workbench users

GRANT SELECT, INSERT, UPDATE ON EVdata TO [user_name];

-- The database can be shared in multiple ways. Either by a DDL script for an in-depth view of the database, a csv or connecting to a server. For this project, we'll publish as a csv.
