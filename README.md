# Healthcare-Analytics-with-SQL

## Overview
Healthcare analytics is crucial in improving patient care, optimizing medical operations, and enhancing decision-making in the healthcare sector. This project involves analyzing patient appointments, diagnoses, and medications using SQL queries to gain meaningful insights.

## Database Schema
The project consists of the following relational tables:

```sql
CREATE TABLE Patients (
    patient_id INT PRIMARY KEY,
    name VARCHAR(15) NOT NULL,
    age INT NOT NULL,
    gender VARCHAR(8) NOT NULL,
    address VARCHAR(15) NOT NULL,
    contact_number VARCHAR(15) NOT NULL
);

CREATE TABLE Doctors (
    doctor_id INT PRIMARY KEY,
    name VARCHAR(15) NOT NULL,
    specialization VARCHAR(20) NOT NULL,
    experience_years INT CHECK (experience_years >= 0),
    contact_number VARCHAR(15) UNIQUE NOT NULL
);

CREATE TABLE Appointments (
    appointment_id INT PRIMARY KEY,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    appointment_date DATE NOT NULL,
    reason VARCHAR(50),
    status VARCHAR(20) NOT NULL,
    FOREIGN KEY (patient_id) REFERENCES Patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
);

CREATE TABLE Diagnoses (
    diagnosis_id INT PRIMARY KEY,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    diagnosis_date DATE NOT NULL,
    diagnosis VARCHAR(15),
    treatment VARCHAR(20),
    FOREIGN KEY (patient_id) REFERENCES Patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
);

CREATE TABLE Medications (
    medication_id INT PRIMARY KEY,
    diagnosis_id INT NOT NULL,
    medication_name VARCHAR(50) NOT NULL,
    dosage VARCHAR(50) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE,
    FOREIGN KEY (diagnosis_id) REFERENCES Diagnoses(diagnosis_id),
    CHECK (start_date <= end_date)
);
```

## SQL Queries and Insights

### 1. Inner and Equi Joins
Retrieve details of all completed appointments.
```sql
SELECT
    p.name AS patient_name,
    d.name AS doctor_name,
    d.specialization,
    a.appointment_date
FROM Appointments a
INNER JOIN Patients p ON a.patient_id = p.patient_id
INNER JOIN Doctors d ON a.doctor_id = d.doctor_id
WHERE a.status = 'Completed';
```

### 2. Left Join with Null Handling
Find patients who have never had an appointment.
```sql
SELECT
    p.name AS patient_name,
    p.contact_number,
    p.address
FROM Patients p
LEFT JOIN Appointments a ON p.patient_id = a.patient_id
WHERE a.patient_id IS NULL;
```

### 3. Right Join and Aggregate Functions
Find the total number of diagnoses per doctor, including those with no diagnoses.
```sql
SELECT
    d.name AS doctor_name,
    d.specialization,
    COUNT(di.diagnosis_id) AS total_diagnoses
FROM Doctors d
RIGHT JOIN Diagnoses di ON d.doctor_id = di.doctor_id
GROUP BY d.doctor_id, d.name, d.specialization;
```

### 4. Full Join for Overlapping Data
Identify mismatches between appointments and diagnoses.
```sql
SELECT
    COALESCE(a.appointment_id, d.diagnosis_id) AS record_id,
    p.name AS patient_name,
    doc.name AS doctor_name,
    doc.specialization,
    a.appointment_date,
    d.diagnosis,
    d.treatment
FROM Appointments a
FULL OUTER JOIN Diagnoses d
    ON a.patient_id = d.patient_id
    AND a.doctor_id = d.doctor_id
LEFT JOIN Patients p
    ON COALESCE(a.patient_id, d.patient_id) = p.patient_id
LEFT JOIN Doctors doc
    ON COALESCE(a.doctor_id, d.doctor_id) = doc.doctor_id
WHERE a.appointment_id IS NULL OR d.diagnosis_id IS NULL;
```

### 5. Window Functions (Ranking and Aggregation)
Rank doctors based on the number of appointments.
```sql
SELECT
    d.doctor_id,
    d.name,
    COUNT(a.appointment_id) AS appointment_count,
    RANK() OVER (ORDER BY COUNT(a.appointment_id) DESC) AS patient_rank
FROM Appointments a
JOIN Doctors d ON a.doctor_id = d.doctor_id
GROUP BY d.doctor_id, d.name;
```

### 6. Conditional Expressions
Categorize patients by age group.
```sql
SELECT
    CASE
        WHEN age BETWEEN 18 AND 30 THEN '18-30'
        WHEN age BETWEEN 31 AND 50 THEN '31-50'
        WHEN age >= 51 THEN '51+'
        ELSE 'Unknown'
    END AS age_group,
    COUNT(*) AS patient_count
FROM Patients
GROUP BY age_group
ORDER BY age_group;
```

### 7. Numeric and String Functions
Find patients whose contact numbers end with "1234".
```sql
SELECT UPPER(name) AS patient_name, contact_number
FROM Patients
WHERE contact_number LIKE '%1234';
```

### 8. Subqueries for Filtering
Find patients prescribed only "Insulin".
```sql
SELECT DISTINCT a.patient_id
FROM Appointments a
JOIN Diagnoses d ON a.patient_id = d.patient_id
JOIN Medications m ON d.diagnosis_id = m.diagnosis_id
WHERE a.patient_id NOT IN (
    SELECT DISTINCT a2.patient_id
    FROM Appointments a2
    JOIN Diagnoses d2 ON a2.patient_id = d2.patient_id
    JOIN Medications m2 ON d2.diagnosis_id = m2.diagnosis_id
    WHERE m2.medication_name <> 'Insulin'
);
```

### 9. Date and Time Functions
Calculate the average duration of prescribed medications.
```sql
SELECT
    d.diagnosis_id,
    d.diagnosis,
    AVG(ABS(m.end_date - m.start_date)) AS avg_duration_days
FROM Diagnoses d
JOIN Medications m ON d.diagnosis_id = m.diagnosis_id
GROUP BY d.diagnosis_id, d.diagnosis;
```

### 10. Complex Joins and Aggregation
Identify the doctor with the most unique patients.
```sql
SELECT d.name AS doctor_name,
       d.specialization,
       COUNT(DISTINCT a.patient_id) AS unique_patient_count
FROM Doctors d
JOIN Appointments a ON d.doctor_id = a.doctor_id
GROUP BY d.doctor_id, d.name, d.specialization
ORDER BY unique_patient_count DESC
LIMIT 1;
```

## Conclusion
This project demonstrates various SQL techniques, including joins, aggregations, window functions, conditional expressions, and subqueries to extract meaningful insights from healthcare data. The queries help in patient analysis, doctor performance tracking, and medication effectiveness evaluation.

## Author
Your Name - [GitHub Profile](https://github.com/yourusername)

## License
This project is licensed under the MIT License.

