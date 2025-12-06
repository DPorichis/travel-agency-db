## SQL Database for Travel Agency "Pausanias"

A fully designed relational database system created for the fictional travel agency **“Pausanias”**.  
The project includes an Entity–Relationship (ER) schema, a full SQL implementation that models the agency’s operations, and set of queries designed to retrieve meaningful business data.

*This project was developed as part of the [Prof. Ioannis Ioannidis'](https://www.madgik.di.uoa.gr/people/faculty/yioannidis) course in [Design and Use of Database Systems (K29)](https://www.di.uoa.gr/en/studies/undergraduate/27) at DIT-NKUA.*


![Schema Diagram](/media/diagram.svg)

### Team Members
- Dimitrios Stefanos Porichis ([LinkedIn]())
- Thanos Anastopoulos ([LinkedIn](https://www.linkedin.com/in/thanos-anastopoulos-979224220/))
- Elina Staniou ([LinkedIn](https://www.linkedin.com/in/elina-staniou-b878b820a/))

## Contents
### Part 1 - Database Design
- Database Schema and Tables
- Travel Agency Description
- Implementation Notes

### Part 2 – Queries
- A collection of SQL queries designed to answer specific business questions
- Queries executed on a legacy database to ensure compatibility and correctness

## Part 1 - Database Design


### Database Schema and Tables
You can find the relevant files in this repository:
- SQL Schema: `/pausanias.sql`
- Schema's Diagram: `/media/diagram.svg`

### Travel Agency's Description

The travel agency “Pausanias” organizes trips and excursions to all tourist destinations in Greece as well as many destinations abroad. For “Pausanias”, travel is the art of transforming dreams into reality. To achieve this, the agency must organize its data in a way that allows it to improve its services, advertise them effectively, and increase its bookings.

The agency has many branches, each with a name, address, one or more phone numbers, and its own employees. The agency offers travel packages that include tickets, accommodation, guided tours, and transportation where required to and from points of interest.

Each branch has its own employees. There are different roles for employees of the agency. For every employee in the agency chain, the system records an employee ID, first name, last name, home address, tax identification number, salary, job specialty, and the branch where they work. Employee specialties include: administrative staff, tour guides, and drivers.

Administrative staff can belong to one of four categories: accounting, logistics, reservations and services management (administrative), and information systems/data management. For each administrative employee, their degree must be recorded. For tour guides, a CV and the languages they speak must be stored. Drivers have a driving license that lists the license categories they hold (e.g., “Category A,” “Category B”), their experience level in months, and whether they are allowed to drive domestically or internationally.

Each branch has an administrative supervisor who belongs to the administrative staff. Each branch also has an IT manager, for whom the system stores their CV and a “master” password for the branch’s systems. For all employees, the system stores the date they started working, and if they no longer work at the agency, the date of departure.

The agency organizes trips, usually offered as packages. Each trip includes a departure date, return date, number of available seats, and cost per person. A trip may have one or more destinations. In the case of multiple destinations, the system records the next destination for each one. The information stored for each destination includes its name, the country it belongs to, the distance from the departure point, the local language, and a short description.

At each destination, excursions to local attractions can be organized. Each attraction has a name and a classification based on its type (e.g., archaeological site, museum, historical center, etc.). The agency may also organize various events in restaurants and nightclubs. Each package includes guided tours. Every guided tour for a specific attraction within a trip is conducted by a single guide in one language.

The packages offered by the agency are priced differently for the same destination, depending on whether the booking is made by an individual (e.g., a single person or a family) or by a group (e.g., the “Deep Old Age” club of Ilioupoli). For each offer, the system stores the associated travel package, the period of validity (start and end date), the offer cost, the offer category (e.g., individual, family, group), and a description with details about what the trip includes.

For reservations made for each travel package and its associated offers, the system stores the traveler ID, the booking date, the branch where the booking was made, and the deposit amount. For each traveler, the system records their first name, last name, age, home address, phone number, and email address. The system also records whether they belong to a family or a group.

### Implementation Notes
- The employee specialties are represented in the database through relationships between workers and their corresponding specialty entities. These relationships are not mandatory, ensuring that a worker is not associated with all specialties simultaneously. The primary key for workers of each specialty is the Worker_ID (AM, an attribute of the Worker entity) for conceptual clarity, which is why the relationship is modeled as identifying.

- In our implementation, a “Place” is defined as a separate entity from a “Destination.” This distinction ensures conceptual consistency: each place should exist only once. Without this separation, differences in distance values could produce multiple entries representing the same place, forcing an unnecessary many-to-many relationship between “Attraction” and “Destination,” which contradicts the conceptual model.

- For both venues and sights, we added an identification attribute to avoid ambiguity in cases where names may coincide (admittedly somewhat overkill).

- The entities `Offer`, `Booking`, `Trip`, and `Destination` also include an ID field for improved organization, as identifying them solely through composite attributes would be complex and would introduce a significant number of foreign keys into related entities.

- Regarding `Traveler`, `Booking`, and `Offer`: the specification states that “for each booking, the stored information includes the traveler’s code.” We interpret this as each traveler having their own booking, even when belonging to a group. Therefore, the relationship between `Booking` and `Traveler` is modeled as 1:n, not n:m (a traveler may make multiple bookings, but a booking cannot be associated with more than one traveler). The same reasoning applies to the relationship between `Offer` and `Booking`.

## Part 2 – Queries

This section contains SQL queries designed to extract information from a travel agency database. The queries cover a variety of tasks including retrieving tour guide details, analyzing travel packages and reservations, counting employees per branch, and identifying destinations based on specific criteria. 

These Queries are designed to work with a legacy database and not the designed database from `Part 1`. The schema of the legacy database is showcased bellow

![Schema Diagram](/media/legacy.svg)

1. **Find the names of the tour guides who have been used by the agency for trips with destinations in Germany.**

    ```sql
    select distinct e.name, e.surname 
    from employees e, guided_tour g, tourist_attraction a, destination d where employees_AM = travel_guide_employee_AM and 
    g.tourist_attraction_id = a.tourist_attraction_id and
    d.destination_id = a.destination_destination_id and
    d.country = "Germany";
    ```

2. **Find the IDs of the tour guides who conducted more than 3 tours in the year 2019.**
    ```sql
    select travel_guide_employee_AM
    from guided_tour t, trip_package p
    where t.trip_package_id = p.trip_package_id and trip_start < '2020-1-1'
    group by travel_guide_employee_AM
    having count(*) > 4;
    ```

3. **Find the number of employees that each branch of the travel agency has.**
    ```sql
    select travel_agency_branch_id, count(*)
    from travel_agency_branch, employees
    where travel_agency_branch_id = travel_agency_branch_travel_agency_branch_id
    group by travel_agency_branch_id;
    ```

4. **Find the travel packages and the number of reservations made for them during the period `2021-01-01` to `2021-12-31` with Paris as the destination.**
    ```sql
    select trip_package_id, count(*)
    from trip_package, reservation, trip_package_has_destination, destination
    where trip_package_id = offer_trip_package_id and trip_package_trip_package_id = trip_package_id
    and destination_destination_id = destination_id and name = "Paris" 
    and date > '2021-01-01' and date < '2021-12-31'
    group by trip_package_id;
    ```


5. **Find the tour guides who know all foreign languages.**

    ```sql
    select distinct travel_guide_employee_AM
    from guided_tour a
    where not exists
    (
        select a.travel_guide_language_id
        from guided_tour b
        where a.travel_guide_language_id != b.travel_guide_language_id 
        and a.travel_guide_employee_AM = b.travel_guide_employee_AM
    );
    ```
6. **Check whether there was any destination during the year 2020 that was not used by anyone.**  *The query should return a relation with one tuple and one column, containing either `"yes"` or `"no"`.  The use of Flow Control Operations (e.g., `IF`, `CASE`) is forbidden.*
    ```sql
    Select A.Answer
    from(
        select "YES" as Answer
        from offer o
        where o.offer_start > '2020-01-01' and o.offer_end < '2020-12-31'
        and not exists (
            select Reservation_id
            from reservation r
            where o.offer_id = r.offer_id)
        UNION
        select "NO" as Answer
        from offer o
        where o.offer_start > '2020-01-01' and o.offer_end < '2020-12-31'
        and exists (
            select Reservation_id
            from reservation r
            where o.offer_id = r.offer_id
        )
    ) as A;
    ```


7. **Find all customers aged 40 and above who have made reservations for more than 3 travel packages.**

    ```sql
    select traveler_id
    from traveler
    where  age > '40' and traveler_id in (
        select Customer_id
        from reservation
        group by Customer_id
        having count(*) > 4
    );
    ```
8. **Find the names of the tour guides who speak English and the number of tourist packages in which each guide has conducted a tour in that language.**
    ```sql
    select e.name, e.surname, count(distinct g.tourist_attraction_id)
    from employees e, travel_guide_has_languages el, languages l, guided_tour g 
    where e.employees_AM = el.travel_guide_employee_AM 
    and el.languages_id = l.languages_id 
    and l.name = "English"
    and g.travel_guide_employee_AM = e.employees_AM 
    and g.travel_guide_language_id = l.languages_id
    group by e.name, e.surname;
    ```

9. **Find the country of the destination that appears in the most travel packages compared to all other countries.**

    ```sql
    select cntry
    from (
        select country as cntry, count(*) as cnt
        from destination d, trip_package p, trip_package_has_destination dp
        where d.destination_id = dp.destination_destination_id 
        and p.trip_package_id = dp.trip_package_trip_package_id
        group by cntry
    ) as table1
    where cnt = (
        select max(cnt) 
        from (
            select country as cntry, count(*) as cnt
            from destination d, trip_package p trip_package_has_destination dp
            where d.destination_id = dp.destination_destination_id 
            and p.trip_package_id = dp.trip_package_trip_package_id
            group by cntry
        ) as table1
    );
    ```

10. **Find the IDs of the travel packages that include all travel destinations related to Ireland.**
    ```sql
    select distinct trip_package_id
    from trip_package tp, destination, trip_package_has_destination
    where tp.trip_package_id = trip_package_trip_package_id and destination_id = destination_destination_id and not exists(
        select destination_id
        from destination
        where country = "Ireland" and destination_id not in (
            select distinct d.destination_id
            from destination d, trip_package_has_destination dp
            where tp.trip_package_id = dp.trip_package_trip_package_id 
            and d.destination_id = dp.destination_destination_id
        )
    );
    ```



