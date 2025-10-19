![Screenshot 2025-10-19 015320](https://github.com/user-attachments/assets/bf5d225b-1c4f-4c52-9deb-cb92f6719504)


# City-Public-Transport-Dashboard

CREATE DATABASE city_transport;

USE city_transport;

# Table Creation

CREATE TABLE routes (
    route_id      INT AUTO_INCREMENT PRIMARY KEY,
    route_name    VARCHAR(50),
    start_stop    VARCHAR(100),
    end_stop      VARCHAR(100),
    total_stops   INT
);

CREATE TABLE stops (
    stop_id       INT AUTO_INCREMENT PRIMARY KEY,
    stop_name     VARCHAR(100),
    route_id      INT,
    stop_order    INT,
    FOREIGN KEY (route_id) REFERENCES routes(route_id)
);

CREATE TABLE drivers (
    driver_id     INT AUTO_INCREMENT PRIMARY KEY,
    name          VARCHAR(100),
    hire_date     DATE,
    route_id      INT,
    FOREIGN KEY (route_id) REFERENCES routes(route_id)
);

CREATE TABLE rides (
    ride_id       INT AUTO_INCREMENT PRIMARY KEY,
    route_id      INT,
    driver_id     INT,
    stop_id       INT,
    ride_date     DATETIME,
    passengers    INT,
    delay_minutes INT,
    FOREIGN KEY (route_id) REFERENCES routes(route_id),
    FOREIGN KEY (driver_id) REFERENCES drivers(driver_id),
    FOREIGN KEY (stop_id) REFERENCES stops(stop_id)
);


# DATA INSERTION

INSERT INTO routes (route_name, start_stop, end_stop, total_stops) VALUES
('Blue Line', 'Downtown', 'Airport', 12),
('Green Line', 'Uptown', 'City Park', 10),
('Red Line', 'East End', 'West End', 15);

INSERT INTO stops (stop_name, route_id, stop_order) VALUES
('Downtown', 1, 1),
('Main St', 1, 2),
('Airport', 1, 12),
('Uptown', 2, 1),
('Central Plaza', 2, 5),
('City Park', 2, 10),
('East End', 3, 1),
('University', 3, 8),
('West End', 3, 15);

INSERT INTO drivers (name, hire_date, route_id) VALUES
('Alice Johnson', '2019-06-12', 1),
('David Kim', '2021-03-01', 2),
('Maria Lopez', '2020-08-23', 3),
('John Carter', '2022-01-11', 1);

INSERT INTO rides (route_id, driver_id, stop_id, ride_date, passengers, delay_minutes) VALUES
(1, 1, 1, '2025-10-10 07:30:00', 18, 3),
(1, 1, 2, '2025-10-10 08:15:00', 23, 5),
(1, 4, 3, '2025-10-10 17:30:00', 20, 1),
(2, 2, 4, '2025-10-10 09:00:00', 25, 0),
(2, 2, 5, '2025-10-10 18:30:00', 19, 4),
(2, 2, 6, '2025-10-11 10:00:00', 15, 2),
(3, 3, 7, '2025-10-10 08:45:00', 22, 3),
(3, 3, 8, '2025-10-10 17:15:00', 27, 6),
(3, 3, 9, '2025-10-11 09:00:00', 30, 0),
(1, 4, 1, '2025-10-11 08:00:00', 16, 0),
(1, 4, 3, '2025-10-11 17:45:00', 24, 2);


# DATA VALIDATION


SELECT COUNT(*) AS routes_count FROM routes;
SELECT COUNT(*) AS drivers_count FROM drivers;
SELECT COUNT(*) AS rides_count FROM rides;

# ANALYTICAL QUERIES (KPIs)

# Busiest Routes by Ridership

SELECT 
    r.route_name,
    SUM(rd.passengers) AS total_passengers,
    COUNT(rd.ride_id) AS total_rides,
    ROUND(AVG(rd.passengers), 2) AS avg_passengers_per_ride
FROM routes r
JOIN rides rd ON r.route_id = rd.route_id
GROUP BY r.route_name
ORDER BY total_passengers DESC;

# Average Delay per Route
SELECT 
    r.route_name,
    ROUND(AVG(rd.delay_minutes), 2) AS avg_delay,
    MAX(rd.delay_minutes) AS max_delay
FROM routes r
JOIN rides rd ON r.route_id = rd.route_id
GROUP BY r.route_name
ORDER BY avg_delay DESC;

#Driver Performance
SELECT 
    d.name AS driver_name,
    r.route_name,
    COUNT(rd.ride_id) AS total_rides,
    ROUND(AVG(rd.passengers), 2) AS avg_passengers,
    ROUND(AVG(rd.delay_minutes), 2) AS avg_delay
FROM drivers d
JOIN rides rd ON d.driver_id = rd.driver_id
JOIN routes r ON d.route_id = r.route_id
GROUP BY d.name, r.route_name
ORDER BY avg_delay ASC, total_rides DESC;

# Peak Travel Hours
SELECT 
    HOUR(ride_date) AS hour_of_day,
    COUNT(*) AS total_rides,
    SUM(passengers) AS total_passengers,
    ROUND(AVG(passengers), 2) AS avg_passengers_per_ride
FROM rides
GROUP BY HOUR(ride_date)
ORDER BY avg_passengers_per_ride DESC;

# Weekday vs Weekend Ridership
SELECT 
    CASE 
        WHEN DAYOFWEEK(ride_date) IN (1,7) THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type,
    COUNT(*) AS total_rides,
    ROUND(AVG(passengers), 2) AS avg_passengers_per_ride
FROM rides
GROUP BY day_type;

# Most Delayed Drivers
SELECT 
    d.name AS driver_name,
    ROUND(AVG(rd.delay_minutes), 2) AS avg_delay,
    SUM(CASE WHEN rd.delay_minutes > 5 THEN 1 ELSE 0 END) AS severe_delays
FROM rides rd
JOIN drivers d ON rd.driver_id = d.driver_id
GROUP BY d.name
ORDER BY avg_delay DESC;

# Daily Ridership Trend
SELECT 
    DATE(ride_date) AS ride_day,
    SUM(passengers) AS total_passengers,
    COUNT(*) AS total_rides
FROM rides
GROUP BY DATE(ride_date)
ORDER BY ride_day;

# Estimated Revenue (assuming $2.50 fare)
SELECT 
    r.route_name,
    SUM(rd.passengers * 2.5) AS total_revenue
FROM routes r
JOIN rides rd ON r.route_id = rd.route_id
GROUP BY r.route_name
ORDER BY total_revenue DESC;

# Summary KPI Snapshot
SELECT 
    (SELECT COUNT(*) FROM routes) AS total_routes,
    (SELECT COUNT(*) FROM drivers) AS total_drivers,
    (SELECT COUNT(*) FROM rides) AS total_rides,
    (SELECT ROUND(AVG(passengers), 2) FROM rides) AS avg_passengers_per_ride,
    (SELECT ROUND(AVG(delay_minutes), 2) FROM rides) AS avg_delay_minutes;

