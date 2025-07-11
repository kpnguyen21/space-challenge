# Real-Time Agent Assignment System

The goal of this workflow was to build an intelligent agent assignment system that balances workload fairly, adapted dynamically to booking activity, and ensured customers were matched with the most qualified agents. By integrating historical booking data and agent performance metrics, the system kept track of each agent’s availability and continuously updated rankings based on their activity. Automated triggers manage incoming customer assignments and respond to booking status changes in real time, enabling a responsive and scalable approach to agent deployment.

<h2 id="Table-of-Contents">Table of Contents</h2>

<ul>
    <li><a href="#Intro">Introduction</a></li>
    <li><a href="#Objective">Challenge Objective & Assumptions</a></li>
    <li><a href="#Tables">Tables</a></li>
    <ul>
        <li><a href="#Ta1">Table 1: "assignment_history"</a></li>
        <li><a href="#Ta2">Table 2: "bookings"</a></li>
        <li><a href="#Ta3">Table 3: "space_travel_agents"</a></li>
        <li><a href="#Ta4">Table 4: "bookings_2"</a></li>
        <li><a href="#Ta5">Table 5: "new_customer"</a></li>
    </ul>
    <li><a href="#Triggers">Triggers</a> </li>
    <ul>
        <li><a href="#T1">Trigger 1: "updating_assignment_history"</a></li>
        <li><a href="#T2">Trigger 2: "updating_bookings_2"</a></li>
        <li><a href="#T3">Trigger 3: "updating_loads"</a></li>
        <li><a href="#T4">Trigger 4: "update_loads_subtracting"</a></li>
        <li><a href="#T5">Trigger 5: "compute_agent_rank_on_load"</a></li>
    </ul>
    <li><a href="#Validation">Validation</a> </li>
    <ul>
        <li><a href='S1'>Scenario 1</li>
        <li><a href='S2'>Scenario 2</li>
    </ul>
    <li><a href="#Conclusion">Conclusion</a> </li>
    <li><a href="#Code-Description">Code Description</a></li>

---


<h3 id="Intro">Introduction</h3>

In the year 2081, Astra Luxury Travel has become synonymous with interstellar elegance, offering curated adventures to destinations ranging from Martian resorts to the icy vistas of Neptune’s moons. Central to Astra’s operations is the Enterprise Intelligence team—an elite data science division tasked with optimizing customer engagement and revenue generation through advanced analytics, predictive algorithms, and real-time decision systems. Your team has been assigned a pivotal challenge: develop a dynamic agent assignment algorithm that automatically matches incoming prospective customers with the most suitable Space Travel Agent in real time.

The system ingests a range of structured inputs—including customer details (name, communication method, lead source, destination, and launch location)—and leverages historical booking trends, agent performance metrics, and live operational data to compute stack-ranked assignment outputs. Employing SQL for deterministic logic, triggers for event responsiveness, and rank-based decision modeling, the workflow ensures equitable agent distribution while prioritizing high-quality matches. By continuously updating agent rankings based on customer feedback and booking outcomes, the solution embodies a scalable and intelligent approach to customer-agent pairing in a high-stakes luxury travel environment.

---

<h3 id="Objective">Challenge Objective & Assumptions</h3>

**Objective**

The objective of this project was to create a real-time SQL-based agent assignment algorithm for Astra Luxury Travel’s Enterprise Intelligence Department. The algorithm ingests customer information, including `Name`, `Communication Method`, `Lead Source`, `Destination`, and `Launch Location`, and identifies the most suitable travel agent available for each prospective customer.

**Assumptions**

Prior to development, the following assumptions were made:

- Only two `Communication Methods` are supported: `Phone Call` and `Text`.
- There are two `Lead Sources`:
    - `Organic`: Customers discovered the company naturally through marketing efforts such as search engines, social media, or word of mouth.
    - `Bought`: Customers were acquired through paid advertising campaigns or purchased contact lists, including paid search and social media ads.
- The company currently offers travel to the following destinations: `Europa`, `Ganymede`, `Mars`, `Titan`, and `Venus`.
- Launch locations are limited to: `Dallas-Fort Worth Launch Complex`, `Dubai Interplanetary Hub`, `London Ascension Platform`, `New York Orbital Gateway`, `Sydney Stellar Port`, and `Tokyo Spaceport Terminal`.
- Agent assignments operate on a first-come, first-served basis, meaning the first customer to initiate contact is matched with the best available agent.
- An agent may be assigned multiple customers. However, agents with fewer active assignments are prioritized when matching new customers.

Customers who do not meet the criteria outlined above will not be added to the customer list. Should Astra expand its destinations or launch locations in the future, the algorithm will be updated to accommodate these changes.

---

<h3 id="Tables">Tables</h3>

<h4 id="Ta1">Table 1: "assignment_history"</h4>

`assignment_history`: created from `assignment_history SQL Table.txt`

<p float="center">
  <img src="/Figures/assignment_history.JPG" width="400" />
</p>

<h4 id="Ta2">Table 2: "bookings"</h4>

`bookings`: created from `bookings SQL Table.txt`

<p float="center">
  <img src="/Figures/bookings.JPG" width="400" />
</p>

<h4 id="Ta3">Table 3: "space_travel_agents"</h4>

`space_travel_agents`: created from `space_travel_agents SQL Table.txt`

<p float="center">
  <img src="/Figures/space_travel_agents.JPG" width="400" />
</p>

<h4 id="Ta4">Table 4: "bookings_2"</h4>

I merged tables bookings and assignment_history so that bookings included AgentID information. I named this table bookings_2 and and I would be using it going forward.

```
%%sql

CREATE TABLE bookings_2 AS
SELECT B.*,
        AH.AgentID
FROM bookings AS B
LEFT JOIN assignment_history AS AH USING(AssignmentID)
```

<h4 id="Ta5">Table 5: "new_customer"</h4>

```
%%sql

CREATE TABLE new_customer (
    CustomerName VARCHAR(100) PRIMARY KEY,
    CommunicationMethod VARCHAR(50) NOT NULL CHECK (CommunicationMethod IN ('Phone Call', 'Text')),
    LeadSource VARCHAR(50) NOT NULL CHECK (LeadSource IN ('Organic', 'Bought')),
    Destination VARCHAR(50) NOT NULL CHECK (Destination IN ('Mars', 'Europa', 'Titan', 'Venus', 'Ganymede')),
    LaunchLocation VARCHAR(200) NOT NULL CHECK (LaunchLocation IN ('Dallas-Fort Worth Launch Complex', 
    'New York Orbital Gateway', 'Dubai Interplanetary Hub',
    'Tokyo Spaceport Terminal', 'London Ascension Platform', 'Sydney Stellar Port'))
    )
```



---

<h3 id="Triggers">Triggers</h3>

Defined the following triggers to automate assignment workflows and load tracking:

<h4 id="T1">Trigger 1: "updating_assignment_history"</h4>

`updating_assignment_history`: Inserted a record into assignment_history with the fields CustomerName, AssignmentID, CommunicationMethod, LeadSource, AssignedDateTime, and AgentID. Note: AssignedDateTime was set to the current time plus 56 years.

```
%%sql

CREATE TRIGGER updating_assignment_history
AFTER INSERT ON new_customer
FOR EACH ROW
BEGIN

    -- Insert assignment_history
    INSERT INTO assignment_history (
                CustomerName,
                AssignmentID,
                CommunicationMethod,
                LeadSource,
                AssignedDateTime,
                AgentID
                )
    VALUES (
            NEW.CustomerName,
            (SELECT IFNULL(MAX(AssignmentID), 0) + 1 FROM assignment_history),
                    NEW.CommunicationMethod,
                    NEW.LeadSource,
                    datetime('now', '+56 years'),
            (SELECT AgentID FROM agent_rank_tracker ORDER BY agent_rank LIMIT 1)
    );
END;
```

<h4 id="T2">Trigger 2: "updating_bookings_2"</h4>

`updating_bookings_2`: Inserted data into bookings_2, including BookingID, AssignmentID, Destination, LaunchLocation, BookingStatus, and AgentID.

```
%%sql

CREATE TRIGGER updating_bookings_2
AFTER INSERT ON assignment_history
FOR EACH ROW
BEGIN
    
    INSERT INTO bookings_2 (
                BookingID,
                AssignmentID,
                Destination,
                LaunchLocation,
                BookingStatus,
                AgentID
                )
    VALUES (
            (SELECT IFNULL(MAX(BookingID), 0) + 1 FROM bookings_2),
                    NEW.AssignmentID,
                    (SELECT Destination FROM new_customer WHERE CustomerName = NEW.CustomerName),
                    (SELECT LaunchLocation FROM new_customer WHERE CustomerName = NEW.CustomerName),
                    'Pending',
                    NEW.AgentID
            );
END;
```

<h4 id="T3">Trigger 3: "updating_loads"</h4>

`updating_loads`: Incremented the load value in space_travel_agents whenever a new customer was inserted into new_customer.

```
%%sql

CREATE TRIGGER updating_loads
AFTER INSERT ON bookings_2
FOR EACH ROW
BEGIN
    UPDATE space_travel_agents
    SET load = load + 1
    WHERE AgentID = NEW.AgentID AND NEW.BookingStatus = 'Pending';
END;
```

<h4 id="T4">Trigger 4: "update_loads_subtracting"</h4>

`update_loads_subtracting`: Decremented the agent’s load in space_travel_agents if a booking’s BookingStatus changed from 'Pending' to either 'Confirmed' or 'Cancelled'.

```
%%sql

CREATE TRIGGER update_loads_subtracting
AFTER UPDATE OF BookingStatus ON bookings_2
FOR EACH ROW
    WHEN OLD.BookingStatus = 'Pending' AND NEW.BookingStatus IN ('Confirmed', 'Cancelled')
    BEGIN
        UPDATE space_travel_agents
        SET load = MAX(load - 1, 0)
        WHERE AgentID = NEW.AgentID;
    END;
```

<h4 id="T5">Trigger 5: "recompute_agent_rank_on_load"</h4>

`recompute_agent_rank_on_load`: Recalculated agent_rank in response to changes in load. Since agent_rank was dynamic, this trigger ensured that it remained up to date as assignments shift.

```
%%sql
    
CREATE TRIGGER recompute_agent_rank_on_load
AFTER UPDATE OF load ON space_travel_agents
FOR EACH ROW
BEGIN
    DELETE FROM agent_rank_tracker;

    INSERT INTO agent_rank_tracker (AgentID, agent_rank)
    SELECT a.AgentID,
            (SELECT COUNT(*)
                FROM space_travel_agents AS b
                WHERE b.load < a.load
                OR (b.load = a.load AND b.YearsOfService > a.YearsOfService)
                OR (b.load = a.load AND b.YearsOfService = a.YearsOfService
                AND b.AverageCustomerServiceRating > a.AverageCustomerServiceRating)
            ) + 1
FROM space_travel_agents AS a;
END;
```

---

<h3 id="Validation">Validation</h3>

To evaluate the algorithm’s behavior, I tested two scenarios: 
- <b>Scenario 1</b>: The `new_customer` table was updated to simulate the addition of new users.
- <b>Scenario 2</b>: The BookingStatus field in `bookings_2` was modified to reflect status changes.

<h4 id="S1">Scenario 1</h3>

<h4 id="S2">Scenario 2</h3>

---

<h3 id="Conclusion">Conclusion</h3>

---

<h3 id="Code-Description">Code Description</h3>

[Assessment.ipynb](https://github.com/kpnguyen21/space-challenge/blob/main/Assessment.ipynb): This notebook contains a real-time SQL algorithm designed to match incoming customers with the most suitable travel agent in real time.

[assignment_history SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/assignment_history%20SQL%20Table.txt): The text file contains the SQL statement used to create the `assignment_history` table.

[bookings SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/bookings%20SQL%20Table.txt): The text file contains the SQL statement used to create the `bookings` table.

[space_travel_agents SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/space_travel_agents%20SQL%20Table.txt): The text file contains the SQL statement used to create the `space_travel_agents` table.

[my_databse.db](https://github.com/kpnguyen21/space-challenge/blob/main/my_database.db): This database was created from the `Assessment.ipynb` notebook.
