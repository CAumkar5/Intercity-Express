
# Assignment CS-105 :  Persistent Data Management 2023-24

In this assignment we are supposed to design database for InterCity Express Trains (IET) to support there day to day operations by storing information about schedule of coaches 

# Part 1 : Group Work

##  Group Members
- Aumkar Ashok Chitragar (Roll no. 23112006)
- Shruti Sunil Navale (Roll no. 23112022)

## Contribution
- ER Diagram : Aumkar Chitragar & Shruti Navale
- Schema : Shruti Navale
- Sample data : Aumkar Chitragar & Shruti Navale
- Readme File : Shruti Navale

## Assumption
- we have considered coach is train
- there are always some coaches which are idle so that we can use them as standbycoach
- sample data is not large and appropriate considered only some different situations where part 2 queries can run
- data entries only for September, October, November, December months is done
- route of A to B and B to A is considered different but intermediate stations are considered same for both just intermediate station stop order is reversed
- a passenger can do many bookings but every booking for a schedule is related to a passenger i.e. a seat can be only referred to a passenger
- seats numbering system is same in every coach
- ticket is per seat
 
## Identification of Entities

+ Agent: Contains information about travel agents, including their ID, name, contact details, and commission rates.

+ Booking: Stores details of bookings, such as booking ID, schedule ID, booking date, booker ID, agent ID, seater ID, booking status, etc.

+ Coach: Contains information about coaches, including coach number, mileage, and maintenance schedule.

+ Driver: Stores details about drivers, including driver ID, name, contact number, city of residence, and roster information.

+ Maintenance: Keeps track of routine maintenance schedules for coaches based on mileage and time intervals.

+ Passenger: Holds information about passengers, including passenger ID, name, age, contact number, and address.

+ Roster: Contains details of drivers and routes and time at which drivers are suppose to address routes.

+ Route: Stores information about routes, including route ID, source, destination, distance, duration, and operating days.

+ Schedule: Contains details of coach schedules, including schedule ID, coach ID, route ID, departure, and arrival timings.

+ ScheduleTracking: Keeps track of schedule by giving actual timings for departure and arrival of coaches.

+ StandbyCoach: Stores information about standby coaches that are activated if a coach arrives late .

+ IntermediateStation: Contains information about intermediatestations, including station ID and name.

+ Intermediatestation_Schedule: Links intermediatestations to routes, indicating order of stops of coaches in routes at intermediatestations and its departure and arrival time at intermediatestations.

+ Ticket: Stores details about tickets, including ticket ID, booking ID, passenger ID, seat number, prices, and discounts.

## ER Diagram
![6f557a41-24ab-4842-bb06-0a7c665ef252](https://github.com/Shrrutii29/intercityExpressTrains/assets/121250741/a953fd98-4db7-454f-86f2-c70435c883cf)

## Creation of database-Intercityy
SQL queries performed in order to create database for IntercityExpressTrains are mentioned in textfile named intercity.txt 
### Steps :
- Create database
- Create tables
- Insert sample data in tables

# Part 2 : Individual Work
## Set B : done by - Shruti Navale (23112022)

#### 1.Show schedule of all trips including main driver information for 10th October this year.

    SQL Query : 
    select distinct s.sid, s.cid,s.rid,s.departure,s.arrival,d.did as Maindriver_id,d.name as Maindriver_name,d.contactno as Maindriver_contact from schedule s,driver d,roster r where s.rid = r.rid and r.did = d.did and r.remark = 'Main Driver' AND DATE(s.departure) = '2023-10-10';

    Output :
    +-------+------+-------+---------------------+---------------------+---------------+-----------------+--------------------+
    | sid   | cid  | rid   | departure           | arrival             | Maindriver_id | Maindriver_name | Maindriver_contact |
    +-------+------+-------+---------------------+---------------------+---------------+-----------------+--------------------+
    | SC148 | C104 | RB301 | 2023-10-10 12:30:00 | 2023-10-10 15:30:00 | D007          | Suraj Pillai    | 9876001122         |
    | SC149 | C105 | RB501 | 2023-10-10 01:00:00 | 2023-10-10 10:45:00 | D009          | Rahul Reddy     | 9898776655         |
    | SC150 | C107 | RF204 | 2023-10-10 02:30:00 | 2023-10-10 08:30:00 | D011          | Tanay Bhat      | 9876654321         |
    +-------+------+-------+---------------------+---------------------+---------------+-----------------+--------------------+


#### 2.List all coaches with mileage between 4000 and 4999 km covered for September this year; include information on the coach, its last service date and total number of scheduled trips.

    SQL Query :
    select c.cid, c.cname, c.capacity, c.mileage, c.facilities, max(m.lastmaintenance) as last_service_date, count( distinct s.sid) as total_trips from coach c,maintenance m,schedule s where c.cid = m.cid and c.cid = s.cid and c.mileage between 4000 and 4999 and month(s.departure) = 9 group by c.cid, c.cname, c.capacity, c.mileage, c.facilities;

     Output : 
    +------+------------------+----------+---------+-----------------------------------------------------------+-------------------+-------------+
    | cid  | cname            | capacity | mileage | facilities                                                | last_service_date | total_trips |
    +------+------------------+----------+---------+-----------------------------------------------------------+-------------------+-------------+
    | C102 | Janshatabdi      |       30 |    4800 | Meal Services, Wi-Fi, Reading Materials, Air Conditioning | 2023-08-15        |           1 |
    | C104 | LTT VSKP Express |       30 |    4300 | Meal Services, Wi-Fi, Reading Materials, Air Conditioning | 2023-10-10        |           2 |
    | C107 | VSG YPR EXP      |       30 |    4900 | Meal Services, Wi-Fi, Reading Materials, Air Conditioning | 2024-01-25        |           1 |
    +------+------------------+----------+---------+-----------------------------------------------------------+-------------------+-------------+

    
#### 3.List all agents, in descending order of percentage of confirmed booking each trip in the month of October this year. Include agent and route information in your result.

    SQL Query : 
    select a.aid as agent_id, a.aname as agent_name, r.rid as route_id, r.source, r.dest as destination, s.sid as schedule_id, count(*) as total_bookings, sum(b.bstatus = 'Confirmed') as confirmed_bookings, (sum(b.bstatus = 'Confirmed') / count(*)) * 100 as percentage_confirmed from agent a,booking b,schedule s,route r where a.aid = b.aid and b.sid = s.sid and s.rid = r.rid and month(s.departure) = 10 and year(s.departure) = year(curdate()) group by agent_id, route_id, schedule_id order by percentage_confirmed desc;

    Output : 
    
    +----------+------------------+----------+-------------+----------------+-------------+----------------+--------------------+----------------------+
    | agent_id | agent_name       | route_id | source      | destination    | schedule_id | total_bookings | confirmed_bookings | percentage_confirmed |
    +----------+------------------+----------+-------------+----------------+-------------+----------------+--------------------+----------------------+
    | A002     |  Priya Patel     | RB101    | Gandhinagar | Mumbai Central | SC101       |              1 |                  1 |             100.0000 |
    | A003     |  Rajiv Singh     | RB101    | Gandhinagar | Mumbai Central | SC101       |              1 |                  1 |             100.0000 |
    | A001     |  Arjun Sharma    | RB101    | Gandhinagar | Mumbai Central | SC102       |              1 |                  1 |             100.0000 |
    | A003     |  Rajiv Singh     | RB101    | Gandhinagar | Mumbai Central | SC102       |              1 |                  1 |             100.0000 |
    | A005     |  Vikram Verma    | RB101    | Gandhinagar | Mumbai Central | SC102       |              1 |                  1 |             100.0000 |
    | A009     |  Aryan Khanna    | RB101    | Gandhinagar | Mumbai Central | SC102       |              1 |                  1 |             100.0000 |
    | A011     |  Siddharth Mehta | RB204    | Goa         | Mumbai         | SC110       |              1 |                  1 |             100.0000 |
    | A001     |  Arjun Sharma    | RB204    | Goa         | Mumbai         | SC110       |              1 |                  1 |             100.0000 |
    | A001     |  Arjun Sharma    | RB201    | Himachal    | New Delhi      | SC103       |              1 |                  1 |             100.0000 |
    | A001     |  Arjun Sharma    | RB203    | Dharwad     | Bengaluru      | SC107       |              1 |                  1 |             100.0000 |
    | A012     |  Sneha Joshi     | RB101    | Gandhinagar | Mumbai Central | SC101       |              1 |                  1 |             100.0000 |
    | A001     |  Arjun Sharma    | RB101    | Gandhinagar | Mumbai Central | SC101       |              2 |                  1 |              50.0000 |
    | A004     |  Ananya Kapoor   | RB101    | Gandhinagar | Mumbai Central | SC102       |              2 |                  1 |              50.0000 |
    | A012     |  Sneha Joshi     | RB203    | Dharwad     | Bengaluru      | SC107       |              2 |                  1 |              50.0000 |
    | A006     |  Nandini Gupta   | RB101    | Gandhinagar | Mumbai Central | SC102       |              1 |                  0 |               0.0000 |
    | A010     |  Aisha Malik     | RB101    | Gandhinagar | Mumbai Central | SC102       |              1 |                  0 |               0.0000 |
    | A004     |  Ananya Kapoor   | RB204    | Goa         | Mumbai         | SC110       |              1 |                  0 |               0.0000 |
    | A012     |  Sneha Joshi     | RB201    | Himachal    | New Delhi      | SC103       |              1 |                  0 |               0.0000 |
    +----------+------------------+----------+-------------+----------------+-------------+----------------+--------------------+----------------------+

#### 4.Display the details of the routes where majority of bookings are not made by agents.

    SQL Query :
     select r.rid as route_id, r.source, r.dest, count(distinct b.bid) as total_bookings,count(distinct b.booker_pid) as bookings_by_passengers,count(distinct b.aid) as bookings_by_agents from route r, booking b, schedule s where r.rid = s.rid and s.sid = b.sid and b.bstatus='Confirmed' group by route_id, r.source, r.dest having bookings_by_passengers > bookings_by_agents;


    Output :
    +----------+----------+-----------+----------------+------------------------+--------------------+
    | route_id | source   | dest      | total_bookings | bookings_by_passengers | bookings_by_agents |
    +----------+----------+-----------+----------------+------------------------+--------------------+
    | RB201    | Himachal | New Delhi |              3 |                      3 |                  2 |
    | RB204    | Goa      | Mumbai    |              7 |                      5 |                  3 |
    +----------+----------+-----------+----------------+------------------------+--------------------+

#### 5.Display the details of the agents who have made maximum commission in the Month of September.

    SQL Query :
     select a.aid as agent_id, a.aname as agent_name, max(t.total_price * a.commission) as total_commission from booking b,agent a,ticket t where b.aid = a.aid and b.bid = t.bid and month(b.bdate) = 9 and b.bstatus="Confirmed" group by agent_id, agent_name order by total_commission desc limit 1;
     
    output : 
    +----------+---------------+--------------------+
    | agent_id | agent_name    | total_commission   |
    +----------+---------------+--------------------+
    | A009     |  Aryan Khanna | 22.500000335276127 |
    +----------+---------------+--------------------+
    
## Set C : done by - Aumkar Chitragar (23112006)

#### List all trains not scheduled on 10th October this year.

    SQL Query :
    select c.cid , c.cname as train_name , s.departure from coach c , schedule s where c.cid=s.cid and date(s.departure) != '2023-10-10';

    Output :
    +------+---------------------+---------------------+
    | cid  | train_name          | departure           |
    +------+---------------------+---------------------+
    | C101 | Vande Bharat        | 2023-10-20 08:00:00 |
    | C101 | Vande Bharat        | 2023-11-01 08:00:00 |
    | C101 | Vande Bharat        | 2023-11-15 08:00:00 |
    | C101 | Vande Bharat        | 2023-12-01 08:00:00 |
    | C102 | Janshatabdi         | 2023-10-21 09:30:00 |
    | C102 | Janshatabdi         | 2023-11-02 09:30:00 |
    | C102 | Janshatabdi         | 2023-11-16 09:30:00 |
    | C102 | Janshatabdi         | 2023-12-03 09:30:00 |
    | C102 | Janshatabdi         | 2023-09-01 08:00:00 |
    | C103 | Duronto             | 2023-10-22 11:00:00 |
    | C103 | Duronto             | 2023-11-03 11:00:00 |
    | C103 | Duronto             | 2023-11-17 11:00:00 |
    | C103 | Duronto             | 2023-12-05 11:00:00 |
    | C104 | LTT VSKP Express    | 2023-10-23 12:30:00 |
    | C104 | LTT VSKP Express    | 2023-11-04 12:30:00 |
    | C104 | LTT VSKP Express    | 2023-12-07 12:30:00 |
    | C104 | LTT VSKP Express    | 2023-11-19 12:30:00 |
    | C104 | LTT VSKP Express    | 2023-12-07 12:30:00 |
    | C104 | LTT VSKP Express    | 2023-09-02 09:30:00 |
    | C105 | Grand Trunk Express | 2023-10-24 14:00:00 |
    | C105 | Grand Trunk Express | 2023-11-05 14:00:00 |
    | C105 | Grand Trunk Express | 2023-12-09 14:00:00 |
    | C105 | Grand Trunk Express | 2023-11-20 14:00:00 |
    | C105 | Grand Trunk Express | 2023-12-09 14:00:00 |
    | C106 | SUR AII SF SPL      | 2023-10-25 15:30:00 |
    | C106 | SUR AII SF SPL      | 2023-11-06 15:30:00 |
    | C106 | SUR AII SF SPL      | 2023-12-11 15:30:00 |
    | C106 | SUR AII SF SPL      | 2023-11-22 15:30:00 |
    | C106 | SUR AII SF SPL      | 2023-12-11 15:30:00 |
    | C107 | VSG YPR EXP         | 2023-10-26 17:00:00 |
    | C107 | VSG YPR EXP         | 2023-11-07 17:00:00 |
    | C107 | VSG YPR EXP         | 2023-12-13 17:00:00 |
    | C107 | VSG YPR EXP         | 2023-11-24 17:00:00 |
    | C107 | VSG YPR EXP         | 2023-12-13 17:00:00 |
    | C107 | VSG YPR EXP         | 2023-09-03 11:00:00 |
    | C108 | Malwa Express       | 2023-10-27 18:30:00 |
    | C108 | Malwa Express       | 2023-11-08 18:30:00 |
    | C108 | Malwa Express       | 2023-12-15 18:30:00 |
    | C108 | Malwa Express       | 2023-11-26 18:30:00 |
    | C108 | Malwa Express       | 2023-12-15 18:30:00 |
    | C109 | Mandovi Express     | 2023-10-28 20:00:00 |
    | C109 | Mandovi Express     | 2023-11-09 20:00:00 |
    | C109 | Mandovi Express     | 2023-12-17 20:00:00 |
    | C109 | Mandovi Express     | 2023-11-28 20:00:00 |
    | C109 | Mandovi Express     | 2023-12-17 20:00:00 |
    | C110 | Hubli Express       | 2023-10-29 21:30:00 |
    | C110 | Hubli Express       | 2023-11-10 21:30:00 |
    | C110 | Hubli Express       | 2023-12-19 21:30:00 |
    | C110 | Hubli Express       | 2023-11-30 21:30:00 |
    | C110 | Hubli Express       | 2023-12-19 21:30:00 |
    | C110 | Hubli Express       | 2023-09-04 12:30:00 |
    | C111 | Luxury Liner        | 2023-09-05 14:00:00 |
    +------+---------------------+---------------------+
    
    
#### List the details of most popular route of InterCity Express Trains.

    SQL Query :
    select r.*, count(s.rid) from route r, schedule s where r.rid = s.rid group by s.rid order by count(rid) desc;
    
    Output :
    +-------+-----------------+-----------------+----------+----------+----------------------------------------------------------+--------------+
    | rid   | source          | dest            | distance | duration | operating_days                                           | count(s.rid) |
    +-------+-----------------+-----------------+----------+----------+----------------------------------------------------------+--------------+
    | RF204 | Mumbai          | Goa             |      588 | 06:00:00 | Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday |            7 |
    | RB101 | Gandhinagar     | Mumbai Central  |      548 | 05:40:00 | Monday,Tuesday,Wednesday,Thursday,Friday,Saturday        |            6 |
    | RB201 | Himachal        | New Delhi       |      412 | 06:10:00 | Monday,Tuesday,Wednesday,Friday,Saturday,Sunday          |            6 |
    | RF202 | Mumbai          | Sainagar Shirdi |      248 | 05:20:00 | Monday,Wednesday,Thursday,Friday,Saturday,Sunday         |            6 |
    | RF203 | Mumbai          | Solapur         |      400 | 06:35:00 | Monday,Tuesday,Thursday,Friday,Saturday,Sunday           |            6 |
    | RB202 | Sainagar Shirdi | Mumbai          |      248 | 05:20:00 | Monday,Wednesday,Thursday,Friday,Saturday,Sunday         |            5 |
    | RF201 | New Delhi       | Himachal        |      412 | 06:10:00 | Monday,Tuesday,Wednesday,Friday,Saturday,Sunday          |            5 |
    | RB203 | Solapur         | Mumbai          |      400 | 06:35:00 | Monday,Tuesday,Thursday,Friday,Saturday,Sunday           |            4 |
    | RB204 | Goa             | Mumbai          |      588 | 06:00:00 | Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday |            4 |
    | RF101 | Mumbai Central  | Gandhinagar     |      548 | 05:40:00 | Monday,Tuesday,Wednesday,Thursday,Friday,Saturday        |            4 |
    | RB301 | Indore          | Bhopal          |      246 | 03:00:00 | Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday |            1 |
    | RB501 | Ajmer           | Lonavala        |     1062 | 10:45:00 | Saturday                                                 |            1 |
    +-------+-----------------+-----------------+----------+----------+----------------------------------------------------------+--------------+
    
    
#### Display the details of the passengers who are frequent travellers with InterCity Express Trains. [Frequent traveller can be defined as the one who has travelled at least three times, irrespective of the route]

    SQL Query :
    select * from passenger where pid in (select pid from ticket group by pid having count(pid)>=3);
    
    Output :
    +------+----------------+------+----------------+------------+-----------------------------+
    | pid  | pname          | age  | agegroup       | contactno  | address                     |
    +------+----------------+------+----------------+------------+-----------------------------+
    | P001 | Aarav Deshmukh |    8 | Child          | 9876543210 | 123 Cherry Blossom St,Pune  |
    | P002 | Riya Khanna    |   10 | Child          | 8765432109 | 456 Serenity Avenue,Delhi   |
    | P004 | Priyanka Verma |   37 | Adult          | 8901234567 | 234 Serene Lane,Bangalore   |
    | P006 | Neha Mehta     |   33 | Adult          | 6543210987 | 890 Blissful Avenue,Kolkata |
    | P009 | Arvind Malik   |   75 | Senior Citizen | 7654321098 | 789 Celestial Road,Jaipur   |
    | P010 | Sunita Kapoor  |   68 | Senior Citizen | 8901278901 | 901 Stellar Street,Lucknow  |
    | P012 | Aisha Sharma   |   22 | Adult          | 8765412345 | 456 Nebula Road,Bhopal      |
    | P019 | Arjun Khanna   |    5 | Child          | 8765478901 | 789 Galactic Lane,Bangalore |
    | P020 | Kavya Sharma   |   11 | Child          | 7654389012 | 234 Starry Road,Chennai     |
    +------+----------------+------+----------------+------------+-----------------------------+


#### Display the details of trains which arrived late at their destination, more than once in this year; Include the driver and co-driver information in the output. 

    SQL Query :
    select d.name, d.contactno, d.homecity from driver d where d.did in ( select did from roster r where r.rid in ( select s.rid from schedule s , scheduletracking st where s.rid = st.rid and s.departure <> st.actual_departure or s.arrival <> st.actual_arrival)) order by name;
    
    Output :
    +-----------------+------------+------------+
    | name            | contactno  | homecity   |
    +-----------------+------------+------------+
    | Aditya Kulkarni | 9822334455 | Lucknow    |
    | Karthik Joshi   | 9765666778 | Indore     |
    | Rahul Reddy     | 9898776655 | Chandigarh |
    | Rajat Khurana   | 9944556677 | Jaipur     |
    | Suraj Pillai    | 9876001122 | Ahmedabad  |
    | Tanay Bhat      | 9876654321 | Bhopal     |
    +-----------------+------------+------------+
