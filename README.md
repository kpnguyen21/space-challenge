# Real-Time Agent Assignment System

The goal of this workflow was to build an intelligent agent assignment system that balances workload fairly, adapted dynamically to booking activity, and ensured customers were matched with the most qualified agents. By integrating historical booking data and agent performance metrics, the system kept track of each agent’s availability and continuously updated rankings based on their activity. Automated triggers manage incoming customer assignments and respond to booking status changes in real time, enabling a responsive and scalable approach to agent deployment.

<h2 id="Table-of-Contents">Table of Contents</h2>

<ul>
    <li><a href="#Objective">Challenge Objective</a></li>
    <ul>
        <li>Algorithm Challenge</li>
        <li>Your Team: Enterprise Intelligence</li>
        <li>Project</li>
    </ul>
    <li><a href="#Intro">Introduction</a></li>
    <li><a href="#Tables">Tables</a></li>
    <li><a href="#Triggers">Triggers</a> </li>
    <ul>
        <li>Trigger 1</li>
        <li>Trigger 2</li>
        <li>Trigger 3</li>
    </ul>
    <li><a href="#Validation">Validation</a> </li>
    <ul>
        <li>Scenario 1</li>
        <li>Scenario 2</li>
    </ul>
    <li><a href="#Conclusion">Conclusion</a> </li>
    <li><a href="#Code-Description">Code Description</a></li>

---

<h3 id="Objective">Challenge Objective</h3>
    
**Algorithm Challenge**

The year is 2081, and you work for Astra Luxury Travel, a space adventure company that curates premium voyages across the Solar System. From exquisite getaways to the red deserts of Mars, to leisure cruises among Saturn’s rings, our team of Space Travel Agents ensures every customer enjoys the perfect experience—from initial Earth departure to safe return. Astra empowers humanity to explore the stars in style and comfort.

**Your Team: Enterprise Intelligence**

The Enterprise Intelligence Department at Astra Luxury Travel is the organization’s data-driven nerve center. Our mission is to harness advanced analytics, predictive modeling, and applied AI to maximize revenue for Astra.

**Project**

Your team must develop a real-time SQL assignment algorithm that automatically matches prospective customers with the best travel agent available. Agents not only guide customers through the booking process but also upsell luxury packages, exclusive excursions, and custom accommodations. At the end of each journey, customers rate their experience with their travel agent. Your solution should receive details about a customer (listed below) and return a stack-ranked list of travel agents ordered from best to worst. 

Details known at time of assignment: 
- Customer Name
- Communication Method
- Lead Source
- Destination
- Launch Location

**Requirements**

- Provide a written overview of your model and the approach you chose
- Provide SQL Code that can be executed without errors
    - If your model requires the building of new tables, stored procedures or functions, make sure you provide the SQL code that creates them


---

<h3 id="Intro">Introduction</h3>

In the year 2081, Astra Luxury Travel has become synonymous with interstellar elegance, offering curated adventures to destinations ranging from Martian resorts to the icy vistas of Neptune’s moons. Central to Astra’s operations is the Enterprise Intelligence team—an elite data science division tasked with optimizing customer engagement and revenue generation through advanced analytics, predictive algorithms, and real-time decision systems. Your team has been assigned a pivotal challenge: develop a dynamic agent assignment algorithm that automatically matches incoming prospective customers with the most suitable Space Travel Agent in real time.

The system ingests a range of structured inputs—including customer details (name, communication method, lead source, destination, and launch location)—and leverages historical booking trends, agent performance metrics, and live operational data to compute stack-ranked assignment outputs. Employing SQL for deterministic logic, triggers for event responsiveness, and rank-based decision modeling, the workflow ensures equitable agent distribution while prioritizing high-quality matches. By continuously updating agent rankings based on customer feedback and booking outcomes, the solution embodies a scalable and intelligent approach to customer-agent pairing in a high-stakes luxury travel environment.

---

<h3 id="Tables">Tables</h3>

`assignment_history`: created from `assignment_history SQL Table.txt`
`bookings`: created from `bookings SQL Table.txt`
`space_travel_agents`: created from `space_travel_agents SQL Table.txt`


---

<h3 id="Code-Description">Code Description</h3>

[Assessment.ipynb](https://github.com/kpnguyen21/space-challenge/blob/main/Assessment.ipynb): This notebook contains a real-time SQL algorithm designed to match incoming customers with the most suitable travel agent in real time.

[assignment_history SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/assignment_history%20SQL%20Table.txt): The text file contains the SQL statement used to create the `assignment_history` table.

[bookings SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/bookings%20SQL%20Table.txt): The text file contains the SQL statement used to create the `bookings` table.

[space_travel_agents SQL Table.txt](https://github.com/kpnguyen21/space-challenge/blob/main/space_travel_agents%20SQL%20Table.txt): The text file contains the SQL statement used to create the `space_travel_agents` table.

[my_databse.db](https://github.com/kpnguyen21/space-challenge/blob/main/my_database.db): This database was created from the `Assessment.ipynb` notebook.
