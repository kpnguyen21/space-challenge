# Real-Time Agent Assignment System

The goal of this workflow was to build an intelligent agent assignment system that balances workload fairly, adapted dynamically to booking activity, and ensured customers were matched with the most qualified agents. By integrating historical booking data and agent performance metrics, the system kept track of each agent’s availability and continuously updated rankings based on their activity. Automated triggers manage incoming customer assignments and respond to booking status changes in real time, enabling a responsive and scalable approach to agent deployment.

<h2 id="Table-of-Contents">Table of Contents</h2>

<ul>
    <li><a href="#Intro">Introduction</a></li>
    <li><a href="#Objective">Challenge Objective & Assumptions</a></li>
    <li><a href="#Tables">Table Architecture Overview</a></li>
    <ul>
        <li><a href="#Ta1">Table 1: "assignment_history"</a></li>
        <li><a href="#Ta2">Table 2: "bookings"</a></li>
        <li><a href="#Ta3">Table 3: "space_travel_agents"</a></li>
        <li><a href="#Ta4">Table 4: "bookings_2"</a></li>
        <li><a href="#Ta5">Table 5: "new_customer"</a></li>
    </ul>
    <li><a href="#Triggers">SQL Trigger Implementations</a> </li>
    <ul>
        <li><a href="#T1">Trigger 1: "updating_assignment_history"</a></li>
        <li><a href="#T2">Trigger 2: "updating_bookings_2"</a></li>
        <li><a href="#T3">Trigger 3: "updating_loads"</a></li>
        <li><a href="#T4">Trigger 4: "update_loads_subtracting"</a></li>
        <li><a href="#T5">Trigger 5: "compute_agent_rank_on_load"</a></li>
    </ul>
    <li><a href="#Validation">Validating Agent Assignment and Re-Ranking</a> </li>
    <ul>
        <li><a href='S1'>Scenario 1</li>
        <li><a href='S2'>Scenario 2</li>
    </ul>
    <li><a href="#Conclusion">Conclusion</a></li>
    <li><a href="#Code-Description">Code Description</a></li>

---


<h3 id="Intro">Introduction</h3>

In the year 2081, Astra Luxury Travel has become synonymous with interstellar elegance, offering curated adventures to destinations ranging from Martian resorts to the icy vistas of Neptune's moons. Central to Astra's operations is the Enterprise Intelligence team—an elite data science division tasked with optimizing customer engagement and revenue generation through advanced analytics, predictive algorithms, and real-time decision systems. Our team has been assigned a pivotal challenge: develop a dynamic agent assignment algorithm that automatically matches incoming prospective customers with the most suitable Space Travel Agent in real time.

The system ingests a range of structured inputs, including customer details (name, communication method, lead source, destination, and launch location), and leverages historical booking trends, agent performance metrics, and live operational data to compute stack-ranked assignment outputs. Employing SQL for deterministic logic, triggers for event responsiveness, and rank-based decision modeling, the workflow ensures equitable agent distribution while prioritizing high-quality matches. By continuously updating agent rankings based on customer feedback and booking outcomes, the solution embodies a scalable and intelligent approach to customer-agent pairing in a high-stakes luxury travel environment.

---

<h3 id="Objective">Challenge Objective & Assumptions</h3>

**Objective**

The objective of this project was to create a real-time SQL-based agent assignment algorithm for Astra Luxury Travel’s Enterprise Intelligence Department. The algorithm ingested customer information, including `Name`, `Communication Method`, `Lead Source`, `Destination`, and `Launch Location`, and identifies the most suitable travel agent available for each prospective customer.

**Assumptions**

Prior to development, the following assumptions were made:

- Only two `Communication Methods` were supported: `Phone Call` and `Text`.
- There are two `Lead Sources`:
    - `Organic`: Customers discovered the company naturally through marketing efforts such as search engines, social media, or word of mouth.
    - `Bought`: Customers were acquired through paid advertising campaigns or purchased contact lists, including paid search and social media ads.
- The company currently offered travel to the following destinations: `Europa`, `Ganymede`, `Mars`, `Titan`, and `Venus`.
- Launch locations were limited to: `Dallas-Fort Worth Launch Complex`, `Dubai Interplanetary Hub`, `London Ascension Platform`, `New York Orbital Gateway`, `Sydney Stellar Port`, and `Tokyo Spaceport Terminal`.
- Agent assignments operated on a first-come, first-served basis, meaning the first customer to initiate contact was matched with the best available agent.
- An agent may be assigned multiple customers. However, agents with fewer active assignments were prioritized when matching new customers.

Customers who did not meet the criteria outlined above would not be added to the customer list. Should Astra expand its destinations or launch locations in the future, the algorithm would be updated to accommodate these changes.

---

<h3 id="Tables">Table Architecture Overview</h3>

<h4 id="Ta1">Table 1: "assignment_history"</h4>

The `assignment_history` table was created from the file `assignment_history SQL Table.txt`. It recorded the assignment history of customers, including the agents they were matched with and the corresponding assignment timestamps. Since the company operated on a first-come, first-served basis, this table was essential for helping the algorithm identify the most suitable agents for incoming customers in chronological order. 

The table contained 450 rows and includes the following columns:

| Column Name  | Data Type | Description | 
| -------- | -----------| ---------- |
| AssignmentID | INTEGER | Primary key; uniquely identified each assignment entry |
| AgentID | INTEGER | ID of the agent assigned to the customer |
| CustomerName | VARCHAR(100) | Full name of the customer |
| CommunicationMethod | VARCHAR(20) | Method of communication: either `Phone Call` or `Text` |
| LeadSource | VARCHAR(20) | Lead origin: either `Organic` or `Bought` |
| AssignedDateTime | DATETIME | Timestamp indicating when the customer was assigned |

<h4 id="Ta2">Table 2: "bookings"</h4>

Table `bookings` was created from `bookings SQL Table.txt`. It captured detailed booking information for each customer, including assignment references, booking status, associated revenues, and travel preferences. This data played a central role in analyzing customer activity and financial outcomes throughout the booking process.

The `bookings` table consisted of 412 rows and was structured with the following columns:

| Column Name  | Data Type | Description | 
| -------- | -----------| ---------- |
| BookingID | INTEGER | Primary key; uniquely identified each booking entry |
| AssignmentID | INTEGER | ID originally assigned to the `assignment_history` |
| BookingCompleteDate | DATETIME | Timestamp when the customer confirmed the booking; NULL if status was `Pending` or `Cancelled` |
| CancelledDate | DATETIME | Timestamp when the booking was canceled; `NULL` if status was `Confirmed` or `Pending` |
| Destination | VARCHAR(100) | Customer’s preferred destination; limited to `Europa`, `Ganymede`, `Mars`, `Titan`, or `Venus` |
| Package | VARCHAR(100) | Package suggested by the agent based on the destination |
| LaunchLocation | VARCHAR(100) | Closest launch location to the customer; options include: `Dallas-Fort Worth Launch Complex`, `Dubai Interplanetary Hub`, `London Ascension Platform`, `New York Orbital Gateway`, `Sydney Stellar Port`, and `Tokyo Spaceport Terminal` |
| DestinationRevenue | DECIMAL(12,2) | Revenue generated from the standard destination package |
| PackageRevenue | DECIMAL(12,2) | Additional revenue from the customized package proposed by the agent |
| TotalRevenue | DECIMAL(12,2) | Combined revenue of DestinationRevenue and PackageRevenue |
| BookingStatus | VARCHAR(20) | Booking state: `Confirmed` (customer confirmed), `Cancelled` (customer canceled), or `Pending` |


<h4 id="Ta3">Table 3: "space_travel_agents"</h4>

Table `space_travel_agents` was created from the file space_travel_agents SQL Table.txt. It contained data on all travel agents employed by Astra Luxury Travel, including details such as name, email, job title, department, average customer service rating, and years of service. This table served as a key foundation for the algorithm, providing essential input for the agent ranking system.  

This table consisted of 30 rows and includes the following columns:

| Column Name  | Data Type | Description | 
| -------- | -----------| ---------- |
| AgentID | INTEGER | Primary key; uniquely identifies each agent entry |
| FirstName | VARCHAR(50) | Agent's first name |
| LastName | VARCHAR(50) | Agent's last name |
| Email | VARCHAR(100) | Agent's email address |
| JobTitle | VARCHAR(50) | Agent's job title: `Space Travel Agent`, `Lead Space Travel Agent`, or `Senior Space Travel Agent` |
| DepartmentName | VARCHAR(50) | Department to which the agent belonged: `Interplanetary Sales`, `Premium Bookings`, and `Luxury Voyages` |
| ManagerName | VARCHAR(100) | Full name of the agent’s manager |
| SpaceLicenseNumber | VARCHAR(100) | Unique identifier for the agent’s space travel license |
| YearsOfService | INTEGER | Number of years the agent had worked at the company |
| AverageCustomerServiceRating | FLOAT | Average rating received from customers |


<!-- <p float="center">
  <img src="/Figures/space_travel_agents.JPG" width="400" />
</p>
-->

<h4 id="Ta4">Table 4: "bookings_2"</h4>

To support agent-specific analytics and operational logic, the original `bookings` table required augmentation, as it lacked the `AgentID`, a key field for linking bookings to agent records. Without this variable, tying a booking directly to its assigned agent was not possible. It limited the algorithm's ability to assess agent performance, applied ranking updates, and managed load balancing. To resolve this, I merged `bookings` table with `assignment_history` table to create `bookings_2`. This enhanced table preserves the original structure and content of bookings, while adding the `AgentID` field, making it suitable for downstream logic that depends on agent-specific operations and trigger-based updates.

```
CREATE TABLE bookings_2 AS
SELECT B.*,
        AH.AgentID
FROM bookings AS B
LEFT JOIN assignment_history AS AH USING(AssignmentID)
```

<h4 id="Ta5">Table 5: "new_customer"</h4>

```
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

<h3 id="Triggers">SQL Trigger Implementations</h3>

Defined the following triggers to automate assignment workflows and load tracking:

<h4 id="T1">Trigger 1: "updating_assignment_history"</h4>

`updating_assignment_history`: Inserted a record into assignment_history with the fields CustomerName, AssignmentID, CommunicationMethod, LeadSource, AssignedDateTime, and AgentID. Note: AssignedDateTime was set to the current time plus 56 years.

```
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

<h3 id="Validation">Validating Agent Assignment and Re-Ranking</h3>

To evaluate the algorithm’s behavior, I tested two scenarios: 
- <b>Scenario 1</b>: The `new_customer` table was updated to simulate the addition of new users.
- <b>Scenario 2</b>: The BookingStatus field in `bookings_2` was modified to reflect status changes.

Before validating the two scenarios, let’s review the top five entries in the `agent_rank_tracker` list.

| AgentID  | agent_rank | 
| -------- | -----------| 
| 6        | 1          |
| 3        | 2          |
| 27       | 2          |
| 24       | 4          |
| 1        | 5          |

The top five agents ranked were AgentIDs 6, 3, 27, 24, and 1. Their rankings reflected a combination of factors, including current workload, tenure with the company, and average customer satisfaction ratings. 

<h4 id="S1">Scenario 1</h3>

The first scenario occurred when the following two customers were added to the system:
- Kate Nguyen, located in Houston, TX, contacted our company via text. Her friend had previously taken a trip with us, and she expressed interest in traveling to Mars. The closest launch location for her was the Dallas-Fort Worth Launch Complex.
- John Doe called us from Sydney after discovering our company through Facebook Ads. He was interested in a trip to Europa departing from the Sydney Stellar Port.

The `new_customer` was updated as the following:

| CustomerName  | CommunicationMethod | LeadSource | Destination | LaunchLocation                     |
| ------------- | --------------------| ---------- | ----------- | ---------------------------------- |
| Kate Nguyen   | Text                | Organic    | Mars        | Dallas-Fort Worth Launch Complex   |
| John Doe      | Phone Call          | Bought     | Europa      | Sydney Stellar Port                |

The trigger `updating_assignment_history` inserted information from the `new_customer` table into `assignment_history`, correctly updated the `AssignmentID` by incrementing the previous value by 1, and automatically populated `AssignedDateTime` when the two rows were added. Both customers had the same `AssignedDateTime` since the entries were inserted simultaneously. Note that before adding information for another customer, `new_customer` table must be cleared; otherwise, the same two entries would be added again.

| AssignmentID  | AgentID | CustomerName | CommunicationMethod | LeadSource | AssignedDateTime |
| ------------- | --------------------| ---------- | ----------- | -------- | ----- |
| 452   | 3     | John Doe    | Phone Call        | Bought   | 2081-07-10 23:16:22 |
| 451   | 6     | Kate Nguyen     | Text      | Organic      | 2081-07-10 23:16:22 |
| 450   | 11    | Mira Cruz        | Phone Call | Bought | 2081-04-10 15:00:00 |

The algorithm assigned `AgentID` 6 (ranked first in the `agent_rank_tracker`) to customer Kate Nguyen, who was first on the list, and `AgentID` 3 (ranked second) to customer John Doe. The trigger `updating_assignment_history` not only updated the table successfully but also correctly assigned the appropriate agents from the ranked list.

The `agent_rank_tracker` was also updated. `AgentIDs` 3, 24, and 1 previously ranked within the top 3 to top 5 in the old list. After `AgentIDs` 6 and 3 were assigned, they moved into the top 3. The trigger `updating_loads updated` the loads in the `bookings_2` table, and the trigger `recompute_agent_rank_on_loads` recalculated the rankings and updated the `agent_rank_tracker` table.

| AgentID  | agent_rank | 
| -------- | -----------| 
| 27       | 1          |
| 24       | 2          |
| 1        | 3          |
| 16       | 3          |
| 19       | 5          |
| ...      | ...        |
| 3        | 17         |

The algorithm's functionality was verified through the initial test case, demonstrating its accuracy in assigning agents and updating relevant tables.

<h4 id="S2">Scenario 2</h3>

To demonstrate the responsiveness of the ranking logic, I changed the `BookingStatus` for `AssignmentID` 452 (John Doe) from "Pending" to "Cancelled." This triggered the removal of the assigned load, allowing `AgentID` 3, who had previously dropped to rank 17 after accepting a new request, to return to the top position. The ranking list was also refreshed when the current customer either confirmed" or canceled their booking. This scenario offered a meaningful sanity check, highlighting the system’s ability to accurately and dynamically recalculated agent rankings based on booking status changes.

| AgentID  | agent_rank | 
| -------- | -----------| 
| 3        | 1          |
| 27       | 1          |
| 24       | 2          |
| 1        | 3          |
| 16       | 3          |

After the change, `AgentID` 3 returned to the top of the list, shifting other agent rankings down by one position. This outcome further confirmed that the algorithm was functioning as intended.

---

<h3 id="Conclusion">Conclusion</h3>

I proposed a dynamic SQL-based algorithm that matches incoming customers to the most suitable available travel agent, updated each agent’s workload, and adjusted their availability ranking accordingly. Agent suitability was determined using a multi-criteria ranking system that prioritized agents with the fewest active assignments, followed by those with the most years of service and the highest average customer satisfaction scores. To validate the algorithm, I tested two operational scenarios: (1) when an agent’s workload increased by assisting new customers, and (2) when an agent’s workload decreased as existing customer cancelled their booking. In both cases, the algorithm consistently identified the best-matched agent, updated their load assignment, and refreshed the agent ranking table to reflect real-time availability.

Looking ahead, I propose expanding the model to support department-level agent assignment, enabling more nuanced matching based on specialization or travel package type. Additionally, I recommend incorporating a new performance metric into the agent ranking logic: successful contact rate. Observations from the `assignment_history` table revealed that some customers were never added to the `bookings` table despite being assigned to an agent. This was likely due to missed follow-ups or failed outreach. Over time, it could impact both customer satisfaction and company revenue. By tracking whether agents successfully initiate contact after assignment, the algorithm can penalize unresponsive agents and favor those with stronger engagement records. Integrating this dimension will further enhance the precision and reliability of future agent recommendations.

---

<h3 id="Code-Description">Code Description</h3>

[Assessment.ipynb](https://github.com/kpnguyen21/space-challenge/blob/main/Assessment.ipynb): This notebook contains a real-time SQL algorithm designed to match incoming customers with the most suitable travel agent in real time.

[assignment_history SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/assignment_history%20SQL%20Table.txt): The text file contains the SQL statement used to create the `assignment_history` table.

[bookings SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/bookings%20SQL%20Table.txt): The text file contains the SQL statement used to create the `bookings` table.

[space_travel_agents SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/space_travel_agents%20SQL%20Table.txt): The text file contains the SQL statement used to create the `space_travel_agents` table.

[my_databse.db](https://github.com/kpnguyen21/space-challenge/blob/main/my_database.db): This database was created from the `Assessment.ipynb` notebook.
