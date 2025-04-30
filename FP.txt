--p1,2
CREATE SCHEMA IF NOT EXISTS pandemic;
USE pandemic;

SELECT COUNT(*) FROM infectious_cases

--p 2

DROP TABLE IF EXISTS infectious_cases_updated;

DROP TABLE IF EXISTS countries;

DROP TABLE IF EXISTS diseases;

CREATE TABLE countries (
    id INT AUTO_INCREMENT PRIMARY KEY,
    country_name VARCHAR(255) NOT NULL,
    country_code VARCHAR(100) NULL
);

CREATE TABLE diseases (
    id INT AUTO_INCREMENT PRIMARY KEY,
    disease_name VARCHAR(255) NOT NULL
);

CREATE TABLE infectious_cases_updated (
    id INT AUTO_INCREMENT PRIMARY KEY,
    country_id INT NOT NULL,
    year INT NOT NULL,
    disease_id INT NOT NULL,
    number_of_cases INT NOT NULL,
    CONSTRAINT fk_infectious_cases_updated_country FOREIGN KEY (country_id) REFERENCES countries(id) ON DELETE CASCADE,
    CONSTRAINT fk_infectious_cases_updated_disease FOREIGN KEY (disease_id) REFERENCES diseases(id) ON DELETE CASCADE
);

INSERT INTO countries (country_name, country_code)
SELECT DISTINCT Entity AS country_name, Code AS country_code
FROM infectious_cases
WHERE Entity IS NOT NULL AND Code IS NOT NULL;

INSERT INTO diseases (disease_name)
VALUES 
    ('yaws'),
    ('polio'),
    ('guinea_worm'),
    ('rabies'),
    ('malaria'),
    ('hiv'),
    ('tuberculosis'),
    ('smallpox'),
    ('cholera');
    
INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.Number_yaws AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'yaws'
WHERE raw_data.Number_yaws IS NOT NULL 
  AND raw_data.Number_yaws <> 0;
  
INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.polio_cases AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'polio'
WHERE raw_data.polio_cases IS NOT NULL 
  AND raw_data.polio_cases <> 0;
  
INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.cases_guinea_worm AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'guinea_worm'
WHERE raw_data.cases_guinea_worm IS NOT NULL 
  AND raw_data.cases_guinea_worm <> 0;

INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.Number_rabies AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'rabies'
WHERE raw_data.Number_rabies IS NOT NULL 
  AND raw_data.Number_rabies <> 0;

INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.Number_malaria AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'malaria'
WHERE raw_data.Number_malaria IS NOT NULL 
  AND raw_data.Number_malaria <> 0;

INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.Number_hiv AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'hiv'
WHERE raw_data.Number_hiv IS NOT NULL 
  AND raw_data.Number_hiv <> 0;
  
INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.Number_tuberculosis AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'tuberculosis'
WHERE raw_data.Number_tuberculosis IS NOT NULL 
  AND raw_data.Number_tuberculosis <> 0;

INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.Number_smallpox AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'smallpox'
WHERE raw_data.Number_smallpox IS NOT NULL 
  AND raw_data.Number_smallpox <> 0;

INSERT INTO infectious_cases_updated (country_id, year, disease_id, number_of_cases)
SELECT 
    countries.id AS country_id,
    raw_data.Year AS year,
    diseases.id AS disease_id,
    raw_data.Number_cholera_cases AS number_of_cases
FROM infectious_cases raw_data
JOIN countries 
    ON countries.country_name = raw_data.Entity
JOIN diseases 
    ON diseases.disease_name = 'cholera'
WHERE raw_data.Number_cholera_cases IS NOT NULL 
  AND raw_data.Number_cholera_cases <> 0;



--p3

SELECT 
    countries.country_name AS entity,
    countries.country_code AS code,
    AVG(updated.number_of_cases) AS average_number_rabies,
    MIN(updated.number_of_cases) AS min_number_rabies,
    MAX(updated.number_of_cases) AS max_number_rabies,
    SUM(updated.number_of_cases) AS total_number_rabies
FROM infectious_cases_updated updated
JOIN countries ON countries.id = updated.country_id
JOIN diseases ON diseases.id = updated.disease_id
WHERE diseases.disease_name = 'rabies' AND updated.number_of_cases != '' AND updated.number_of_cases IS NOT NULL
GROUP BY countries.country_name, countries.country_code
ORDER BY average_number_rabies DESC
LIMIT 10;

--p4


SELECT 
  Year,
    CONCAT(Year, '-01-01') AS Start_Date,
    CURDATE() AS Curr_Date,
    TIMESTAMPDIFF(year, CONCAT(Year, '-01-01'), CURDATE()) AS Year_Difference
FROM infectious_cases_updated;


--p5

DELIMITER //
DROP FUNCTION IF EXISTS Year_Difference;
CREATE FUNCTION Year_Difference(input_year INT)
RETURNS INT
DETERMINISTIC
NO SQL
BEGIN
    DECLARE start_date DATE;
    DECLARE year_difference INT;
    SET start_date = CONCAT(input_year, '-01-01');
    SET year_difference = TIMESTAMPDIFF(YEAR, start_date, CURDATE());
    RETURN year_difference;
END//

DELIMITER ;

SELECT 
    Year,
    CONCAT(Year, '-01-01') AS Start_Date,
    CURDATE() AS Curr_Date,
    Year_Difference(Year) AS Year_Difference
FROM infectious_cases_updated;