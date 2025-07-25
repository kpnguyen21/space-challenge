# Real-Time Agent Assignment System

The goal of this workflow was to build an intelligent agent assignment system that balanced workload fairly, adapted dynamically to booking activity, and ensured customers were matched with the most qualified agents. By integrating historical booking data and agent performance metrics, the system kept track of each agent's availability and continuously updated rankings based on their activity. Automated triggers managed incoming customer assignments and responded to booking status changes in real time, enabling a responsive and scalable approach to agent deployment.

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
        <li><a href="#Ta5">Table 5: "agent_rank_tracker"</a></li>
        <li><a href="#Ta6">Table 6: "new_customer"</a></li>
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

The objective of this project was to create a real-time SQL-based agent assignment algorithm for Astra Luxury Travel’s Enterprise Intelligence Department. The algorithm ingested customer information, including `Name`, `Communication Method`, `Lead Source`, `Destination`, and `Launch Location`, and identified the most suitable travel agent available for each prospective customer.

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

<div align="center">
  <img src="/Figures/assignment_history.jpg" width="400"/>
</div>

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

<div align="center">
  <img src="/Figures/bookings.jpg" width="400"/>
</div>

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

Table `space_travel_agents` was created from the file `space_travel_agents SQL Table.txt`. It contained data on all travel agents employed by Astra Luxury Travel, including details such as name, email, job title, department, average customer service rating, and years of service. This table served as a key foundation for the algorithm, providing essential input for the agent ranking system.  

<div align="center">
  <img src="/Figures/space_travel_agents.jpg" width="400"/>
</div>

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

I added `load` and `space_travel_agents` columns with a default `load` value of 0, to track the number of active customer assignments per agent. When an agent was assigned a new customer, their corresponding load value was incremented to reflect the updated workload. Conversely, upon completing a customer’s request, the load value was decremented by 1.

```
ALTER TABLE space_travel_agents
ADD COLUMN load INTEGER DEFAULT 0;

ALTER TABLE space_travel_agents
ADD COLUMN agent_rank INTEGER;
```

As an initial setup before implementing triggers, I manually iterated through the `bookings_2` table and incremented the load value for each agent whenever a booking record had a `BookingStatus` of `Pending`. This approach ensured that the agent workloads accurately reflected all active customer assignments prior to automating the process with triggers.

```
UPDATE space_travel_agents
SET load = load + (
                    SELECT COUNT(*)
                    FROM bookings_2
                    WHERE bookings_2.AgentID = space_travel_agents.AgentID
                    AND bookings_2.BookingStatus = 'Pending'
                    )
```

The `agent_rank` was computed in this order: 
- Agent with lower `load` (i.e., fewer active assignments) were ranked higher.
- If `loads` were equal, agents with more `YearsOfService` were prioritized.
- If both `load` and `YearsOfService` were equal, agents with a higher `AverageCustomerRating` were ranked higher.

```
UPDATE space_travel_agents
SET agent_rank = (
                  SELECT COUNT(*)
                  FROM space_travel_agents AS b
                  WHERE b.load < space_travel_agents.load
                     OR (b.load = space_travel_agents.load AND b.YearsOfService > space_travel_agents.YearsOfService)
                     OR (b.load = space_travel_agents.load AND b.YearsOfService = space_travel_agents.YearsOfService
                         AND b.AverageCustomerServiceRating > space_travel_agents.AverageCustomerServiceRating)
                    ) + 1;
```

<p float="center">
  <img src="/Figures/add_columns_to_agent_table.jpg" width="1000" />
</p>


<h4 id="Ta4">Table 4: "bookings_2"</h4>

To support agent-specific analytics and operational logic, the original `bookings` table required augmentation, as it lacked the `AgentID`, a key field for linking bookings to agent records. Without this variable, tying a booking directly to its assigned agent was not possible. It limited the algorithm's ability to assess agent performance, applied ranking updates, and managed load balancing. To resolve this, I merged `bookings` table with `assignment_history` table to create `bookings_2`. This enhanced table preserves the original structure and content of bookings, while adding the `AgentID` field, making it suitable for downstream logic that depends on agent-specific operations and trigger-based updates.

```
CREATE TABLE bookings_2 AS
SELECT B.*,
       AH.AgentID
FROM bookings AS B
LEFT JOIN assignment_history AS AH USING(AssignmentID)
```

<p float="center">
  <img src="/Figures/join_tables.jpg" width="1000" />
</p>

<h4 id="Ta5">Table 5: "agent_rank_tracker"</h4>

To enable agent selection based on dynamic performance metrics, I introduced a dedicated table named `agent_rank_tracker`. It contains two columns: `AgentID` and `agent_rank`, populated using the same ranking logic previously applied within `space_travel_agents`. This parallel structure ensured consistency while isolating rank calculations from the main agent profile data.

<div align="center">
  <img src="/Figures/agent_rank_tracker.jpg" width="400"/>
</div>

The `agent_rank_tracker` table served as a streamlined reference for assigning agents according to current `load`. It would be automatically updated whenever an agent's `load` value changes, and assignment logic pulls from this table to maintain fair and efficient customer-agent matchmaking.

```
CREATE TABLE agent_rank_tracker (
                                AgentID INTEGER PRIMARY KEY,
                                agent_rank INTEGER 
                                );

INSERT INTO agent_rank_tracker (AgentID, agent_rank)
    SELECT a.AgentID,
        (
            SELECT COUNT(*)
            FROM space_travel_agents AS b
            WHERE b.load < a.load
              OR (b.load = a.load AND b.YearsOfService > a.YearsOfService)
              OR (b.load = a.load AND b.YearsOfService = a.YearsOfService AND b.AverageCustomerServiceRating > a.AverageCustomerServiceRating)
         ) + 1
    FROM space_travel_agents AS a;
```

<h4 id="Ta6">Table 6: "new_customer"</h4>

I created a new table, `new_customer`, to track incoming customer entries. Customer information included name, communication method, lead source, destination, and launch location. When a new customer was added to this table, it automatically triggered updates to `assignment_history`, `bookings_2`, `space_travel_agents`, and `agent_rank_tracker`.

<div align="center">
  <img src="/Figures/new_customer.jpg" width="400"/>
</div>

The `new_customer` table included the following columns:

| Column Name  | Data Type | Description | 
| -------- | -----------| ---------- |
| CustomerName | VARCHAR(100) | Primary key; Full name of the customer |
| CommunicationMethod | VARCHAR(20) | Method of communication: either `Phone Call` or `Text` |
| LeadSource | VARCHAR(20) | Lead origin: either `Organic` or `Bought` |
| Destination | VARCHAR(100) | Customer’s preferred destination; limited to `Europa`, `Ganymede`, `Mars`, `Titan`, or `Venus` |
| LaunchLocation | VARCHAR(100) | Closest launch location to the customer; options include: `Dallas-Fort Worth Launch Complex`, `Dubai Interplanetary Hub`, `London Ascension Platform`, `New York Orbital Gateway`, `Sydney Stellar Port`, and `Tokyo Spaceport Terminal` |

The values for communication method, lead source, destination, and launch location were restricted to predefined options. Any input outside of these allowed values was considered invalid and could not be inserted into the table. In the future, if Astra Luxury Travel plans to expand its destinations to other planets or introduce new launch locations, this table will need to be updated accordingly. This constraint was part of the underlying assumptions.

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

I defined a set of SQL triggers to streamline the agent assignment workflow and manage real-time load tracking. These triggers automated the insertion and updated processes across multiple tables, ensuring consistency, reducing manual intervention, and maintaining the integrity of agent workload data. Each trigger was designed to respond to specific events, such as new customer entries or changes in booking status, and executed logic that kept assignment records accurate and up-to-date.

<h4 id="T1">Trigger 1: "updating_assignment_history"</h4>

For each new row added to the `new_customer` table, the trigger `updating_assignment_history` automatically activated. It began by copying the fields `CustomerName`, `CommunicationMethod`, and `LeadSource` from the incoming customer entry and inserted them into the `assignment_history` table.

<p float="center">
  <img src="/Figures/updating_assignment_history.jpg" width="1000" />
</p>

To maintain sequential integrity, the `AssignmentID` was calculated by incrementing the highest existing value in `assignment_history`, ensuring that each new assignment received a unique and consecutive identifier. The `AssignedDateTime` was derived from the current timestamp, with an adjustment of `+53 years` to align with the operational timeline in the year 2081. This ensured that bookings remained forward-facing and logically consistent within the futuristic context.

The `AgentID` was determined by selecting the top-ranked agent from the `agent_rank_tracker` table, using the lowest `agent_rank` value as the selection criteria. This guaranteed that agents were assigned based on performance metrics such as `load`, `YearsOfService`, and `AverageCustomerServiceRating`, promoting fair and efficient distribution of new customer assignments.

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

At the moment a new customer record was inserted into `new_customer`, the automation logic defined in `updating_bookings_2` took effect. This trigger copied the `Destination` and `LaunchLocation` fields directly into `bookings_2`, linking the customer to their intended travel plans.

<p float="center">
  <img src="/Figures/updating_bookings_2.jpg" width="1000" />
</p>

To facilitate agent assignment, the `AgentID` was selected from the top-ranked entry in the `agent_rank_tracker` table, prioritizing the agent with the highest operational standing. The `BookingStatus` was then automatically set to `Pending`, signaling that the booking was awaiting confirmation or cancellation. This initialization was essential, as it enabled a downstream trigger to update the agent's `load` value in the `space_travel_agents` table, keeping real-time workload distribution aligned with incoming demand.

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

Following the insertion of a new entry into the `bookings_2` table, triggered by the addition of a new customer in `new_customer`, the `updating_bookings_2` automation completed the booking record. Immediately after this step, another trigger responded to the new booking by updating the corresponding agent's workload in the `space_travel_agents` table.

<p float="center">
  <img src="/Figures/updating_loads.jpg" width="1000" />
</p>

Specifically, the load value for the assigned `AgentID` was incremented by 1, reflecting the addition of an active customer to that agent's portfolio. This update ensured that agent performance metrics and workload calculations remained synchronized with real-time operational data. By automating this increment, the system avoided manual tracking errors and supported downstream logic such as rank recalculation.

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

In addition to handling new entries, `bookings_2` may also undergo updates triggered by decisions from existing customers. When a customer modified their booking, either by confirming or canceling the reservation, the change impacted the workload of the agent assigned to that booking. To address this scenario, the trigger `update_loads_subtracting` was implemented to automatically adjust the `load` field in the `space_travel_agents` table.

<p float="center">
  <img src="/Figures/updating_loads_subtracting.jpg" width="1000" />
</p>

Specifically, when the `BookingStatus` transitioned from `Pending` to either `Confirmed` or `Cancelled`, the corresponding agent was relieved of that task. This resulted in their `load` value being decremented by 1, accurately reflecting their updated assignment count. The reduction in `load` directly contributed to recalculating the agent's position within the `agent_rank_tracker`, potentially improving their rank and increasing their likelihood of receiving future assignments.

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

The most critical component in maintaining accurate agent rankings was the `recompute_agent_rank_on_load` trigger. This trigger recalculated `agent_rank` and refreshed the `agent_rank_tracker` table whenever an agent's `load` value was updated in `space_travel_agents`. Since `agent_rank` was dynamic-driven by evolving assignment volumes and performance metrics, this automation ensured that rankings remained current as workload conditions shifted in real time.

<p float="center">
  <img src="/Figures/recompute_agent_rank_on_load.jpg" width="1000" />
</p>

Upon detecting a change to `load`, the trigger began by clearing the existing contents of `agent_rank_tracker`. This reset step was essential: without it, each recalculation would layer new rankings on top of outdated ones, eventually flooding the table with duplicate entries and undermining its integrity. After deleting previous rankings, the trigger then recomputed the rank for each agent using the same performance-based criteria and inserted the refreshed values into the tracking table. This kept the agent assignment system fair, efficient, and responsive to operational data.

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

To evaluate the algorithm's behavior under varying conditions, I designed and executed two distinct test scenarios:
- **Scenario 1:** The `new_customer` table was populated with simulated entries to mimic the onboarding of new users. This action triggered the full automation sequence, including agent assignment, booking creation, and load adjustment logic. It allowed me to observe how well the system responded to a surge in customer demand and how efficiently agents were selected based on real-time rankings.
- **Scenario 2:** The `BookingStatus` field within the `bookings_2` table was manually updated to represent different customer decisions such as booking confirmations and cancellations. These changes activated downstream triggers responsible for recalibrating agent workloads and updating ranks accordingly. This scenario tested the system's ability to gracefully handle completed transactions and reallocate agent capacity without redundancy or delay.

Before validating the two scenarios, let’s review the top five entries in the `agent_rank_tracker` list.

| AgentID  | agent_rank | 
| -------- | -----------| 
| 6        | 1          |
| 3        | 2          |
| 27       | 2          |
| 24       | 4          |
| 1        | 5          |

The top five agents ranked were `AgentIDs` 6, 3, 27, 24, and 1. Their rankings reflected a combination of factors, including current workload, tenure with the company, and average customer satisfaction ratings. 

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

To demonstrate the responsiveness of the ranking logic, I changed the `BookingStatus` for `AssignmentID` 452 (John Doe) from `Pending` to `Cancelled`. This triggered the removal of the assigned load, allowing `AgentID` 3, who had previously dropped to rank 17 after accepting a new request, to return to the top position. The ranking list was also refreshed when the current customer either confirmed or canceled their booking. This scenario offered a meaningful sanity check, highlighting the system's ability to accurately and dynamically recalculated agent rankings based on booking status changes.

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

I proposed a dynamic SQL-based algorithm that matches incoming customers to the most suitable available travel agent, updated each agent's workload, and adjusted their availability ranking accordingly. Agent suitability was determined using a multi-criteria ranking system that prioritized agents with the fewest active assignments, followed by those with the most years of service and the highest average customer satisfaction scores. To validate the algorithm, I tested two operational scenarios: (1) when an agent's workload increased by assisting new customers, and (2) when an agent's workload decreased as existing customer cancelled their booking. In both cases, the algorithm consistently identified the best-matched agent, updated their load assignment, and refreshed the agent ranking table to reflect real-time availability.

Looking ahead, I propose expanding the model to support department-level agent assignment, enabling more nuanced matching based on specialization or travel package type. Additionally, I recommend incorporating a new performance metric into the agent ranking logic: successful contact rate. Observations from the `assignment_history` table revealed that some customers were never added to the `bookings` table despite being assigned to an agent. This was likely due to missed follow-ups or failed outreach. Over time, it could impact both customer satisfaction and company revenue. By tracking whether agents successfully initiate contact after assignment, the algorithm can penalize unresponsive agents and favor those with stronger engagement records. Integrating this dimension will further enhance the precision and reliability of future agent recommendations.

---

<h3 id="Code-Description">Code Description</h3>

[Assessment.ipynb](https://github.com/kpnguyen21/space-challenge/blob/main/Assessment.ipynb): This notebook contains a real-time SQL algorithm designed to match incoming customers with the most suitable travel agent in real time.

[assignment_history SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/assignment_history%20SQL%20Table.txt): The text file contains the SQL statement used to create the `assignment_history` table.

[bookings SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/bookings%20SQL%20Table.txt): The text file contains the SQL statement used to create the `bookings` table.

[space_travel_agents SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/space_travel_agents%20SQL%20Table.txt): The text file contains the SQL statement used to create the `space_travel_agents` table.

[my_databse.db](https://github.com/kpnguyen21/space-challenge/blob/main/my_database.db): This database was created from the `Assessment.ipynb` notebook.
