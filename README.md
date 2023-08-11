# Date Dimension
SQL to create a date dimension table for PostgreSQL


```
CREATE SEQUENCE warehouse.d_dates_sk_date_seq;

CREATE TABLE warehouse.d_dates (
                sk_date BIGINT NOT NULL DEFAULT nextval('warehouse.d_dates_sk_date_seq'),
                dt_date DATE NOT NULL,
                nm_day VARCHAR(9) NOT NULL,
                nr_day_of_week INTEGER NOT NULL,
                nr_day_of_month INTEGER NOT NULL,
                nr_day_of_year INTEGER NOT NULL,
                nr_week_of_month INTEGER NOT NULL,
                nr_week_of_year INTEGER NOT NULL,
                nr_month INTEGER NOT NULL,
                nm_month VARCHAR(9) NOT NULL,
                nr_quarter INTEGER NOT NULL,
                nm_quarter VARCHAR(9) NOT NULL,
                nr_year INTEGER NOT NULL,
                dt_first_day_of_week DATE NOT NULL,
                dt_last_day_of_week DATE NOT NULL,
                dt_first_day_of_month DATE NOT NULL,
                dt_last_day_of_month DATE NOT NULL,
                dt_first_day_of_quarter DATE NOT NULL,
                dt_last_day_of_quarter DATE NOT NULL,
                nm_mmyyyy CHAR(6) NOT NULL,
                CONSTRAINT sk_date PRIMARY KEY (sk_date)
);
COMMENT ON TABLE warehouse.d_dates IS 'Dimension with data related to dates';
COMMENT ON COLUMN warehouse.d_dates.sk_date IS 'Sequential primary key';
COMMENT ON COLUMN warehouse.d_dates.dt_date IS 'Short date';
COMMENT ON COLUMN warehouse.d_dates.nm_day IS 'Weekday name';
COMMENT ON COLUMN warehouse.d_dates.nr_day_of_week IS 'Number of the day in the week';
COMMENT ON COLUMN warehouse.d_dates.nr_day_of_month IS 'Number of the day in the month';
COMMENT ON COLUMN warehouse.d_dates.nr_day_of_year IS 'Number of the day in the year';
COMMENT ON COLUMN warehouse.d_dates.nr_week_of_month IS 'Week number of the month';
COMMENT ON COLUMN warehouse.d_dates.nr_week_of_year IS 'Week number of the year';
COMMENT ON COLUMN warehouse.d_dates.nr_month IS 'Number of the month';
COMMENT ON COLUMN warehouse.d_dates.nm_month IS 'Month name';
COMMENT ON COLUMN warehouse.d_dates.nr_quarter IS 'Number of the quarter';
COMMENT ON COLUMN warehouse.d_dates.nm_quarter IS 'Quarter name';
COMMENT ON COLUMN warehouse.d_dates.nr_year IS 'Number of the year';
COMMENT ON COLUMN warehouse.d_dates.dt_first_day_of_week IS 'Date for the first day of the week';
COMMENT ON COLUMN warehouse.d_dates.dt_last_day_of_week IS 'Date for the last day of the week';
COMMENT ON COLUMN warehouse.d_dates.dt_first_day_of_month IS 'Date for the first day of the month';
COMMENT ON COLUMN warehouse.d_dates.dt_last_day_of_month IS 'Date for the last day of the month';
COMMENT ON COLUMN warehouse.d_dates.dt_first_day_of_quarter IS 'Date for the first day of the quarter';
COMMENT ON COLUMN warehouse.d_dates.dt_last_day_of_quarter IS 'Date for the last day of the quarter';
COMMENT ON COLUMN warehouse.d_dates.nm_mmyyyy IS 'Date format abbreviation';


ALTER SEQUENCE warehouse.d_dates_sk_date_seq OWNED BY warehouse.d_dates.sk_date;

COMMIT;

INSERT INTO warehouse.d_dates
SELECT TO_CHAR(datum, 'yyyymmdd')::INT AS sk_date,
       datum AS dt_date,
       TO_CHAR(datum, 'TMDay') AS nm_day,
       EXTRACT(ISODOW FROM datum) AS nr_day_of_week,
       EXTRACT(DAY FROM datum) AS nr_day_of_month,
       EXTRACT(DOY FROM datum) AS nr_day_of_year,
       TO_CHAR(datum, 'W')::INT AS nr_week_of_month,
       EXTRACT(WEEK FROM datum) AS nr_week_of_year,
       EXTRACT(MONTH FROM datum) AS nr_month,
       TO_CHAR(datum, 'TMMonth') AS nm_month,
       EXTRACT(QUARTER FROM datum) AS nr_quarter,
       CASE
           WHEN EXTRACT(QUARTER FROM datum) = 1 THEN 'First'
           WHEN EXTRACT(QUARTER FROM datum) = 2 THEN 'Second'
           WHEN EXTRACT(QUARTER FROM datum) = 3 THEN 'Third'
           WHEN EXTRACT(QUARTER FROM datum) = 4 THEN 'Fourth'
           END AS nm_quarter,
       EXTRACT(YEAR FROM datum) AS nr_year,
       datum + (1 - EXTRACT(ISODOW FROM datum))::INT AS dt_first_day_of_week,
       datum + (7 - EXTRACT(ISODOW FROM datum))::INT AS dt_last_day_of_week,
       datum + (1 - EXTRACT(DAY FROM datum))::INT AS dt_first_day_of_month,
       (DATE_TRUNC('MONTH', datum) + INTERVAL '1 MONTH - 1 day')::DATE AS dt_last_day_of_month,
       DATE_TRUNC('quarter', datum)::DATE AS dt_first_day_of_quarter,
       (DATE_TRUNC('quarter', datum) + INTERVAL '3 MONTH - 1 day')::DATE AS dt_last_day_of_quarter,
       TO_CHAR(datum, 'mmyyyy') AS nm_mmyyyy
FROM (SELECT '2015-01-01'::DATE + SEQUENCE.DAY AS datum
      FROM GENERATE_SERIES(0, 5480) AS SEQUENCE (DAY)
      GROUP BY SEQUENCE.DAY) DQ
ORDER BY 1;

COMMIT;
```
