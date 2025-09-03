# Maji Ndogo Water Crisis Project: From Analysis to Action

This outlines the process, findings, and actionable plan derived from the analysis of water access data in Maji Ndogo. The project aims to convert raw data into informed decisions and practical solutions to improve water access for the community.

## Introduction and Project Context

The project was initiated by **President Aziza Naledi** with the goal of addressing Maji Ndogo's water crisis and ensuring that data is translated into actionable knowledge. The team's prior work uncovered corruption among field workers, which President Naledi addressed, emphasizing a no-tolerance policy for self-serving actions. This initiative focuses on shaping data into meaningful views, providing essential information for decision-makers to plan budgets, identify urgent areas, and create clear job lists for engineers.

## Data Sources and Assembly

The analysis draws upon several interconnected tables within the `md_water_services` database. To facilitate analysis, data from these tables was carefully joined and a consolidated view was created.

### Tables Used:
*   **`location`**: Contains information about the geographical location of water sources, including `province_name`, `town_name`, `location_id`, and `location_type`.
*   **`water_source`**: Provides details about the `type_of_water_source` and `number_of_people_served` by each source, linked by `source_id`.
*   **`visits`**: This central table connects `location_id` to `source_id`, and includes `visit_count` and `time_in_queue`. A crucial step in data assembly was to **filter for `visits.visit_count = 1` to eliminate duplicate records** for the same source/location.
*   **`well_pollution`**: Contains `results` about water quality, but **only for well-type water sources**. It is joined using a `LEFT JOIN` to ensure all water sources are included, with pollution results appearing as `NULL` for non-well sources.

### Data Assembly Steps:
1.  **Initial Join**: `location` was joined to `visits` on `location_id`.
2.  **Adding Water Source Details**: `water_source` was then joined to `visits` on `source_id`.
3.  **Including Location Type and Queue Time**: `location_type` from `location` and `time_in_queue` from `visits` were added.
4.  **Incorporating Well Pollution Data**: `well_pollution` was `LEFT JOINED` to `visits` on `source_id` to include contamination results where applicable.
5.  **Creating `combined_analysis_table` View**: This comprehensive view was created to assemble all necessary data from different tables, simplifying subsequent analysis.

## Key Analysis and Insights

The analysis focused on understanding water access patterns across provinces and towns, identifying critical issues, and pinpointing areas for intervention.

### Provincial-Level Insights:
*   A pivot table was built to show the percentage of people served by each water source type per province.
*   **Sokoto**: Has the **largest population drinking river water (21%)**, indicating an urgent need for drilling equipment. Despite this, many residents in Sokoto also have taps at home, suggesting wealth distribution disparities.
*   **Amanzi**: The majority of water comes from taps, but **half of the home taps are broken (24% of total population rely on broken taps)**. This points to a significant infrastructure problem.

### Town-Level Insights:
*   A pivot table aggregated data per town, grouping by `province_name` and `town_name` to handle non-unique town names like Harare and Amina.
*   This data was stored in a temporary table called `town_aggregated_water_access` for quicker access.
*   **Sokoto (Rural areas and Bahari)**: Confirmed to have many people drinking river water, requiring drilling teams.
*   **Amina (Amanzi Province)**: Only **3% of citizens have access to running tap water in their homes**, while **more than half (56%) have taps installed that are not working**. This town is a top priority for fixing infrastructure.
*   **Amina (Amanzi) has the highest ratio of broken taps (95% of installed taps are broken)**, indicating critical infrastructure failure.
*   **Dahabu (Amanzi Province)**: Noticed that almost all water infrastructure works, possibly due to past politicians residing there.

### Overall Insights Summary:
1.  Most water sources are **rural** in Maji Ndogo.
2.  **43% of the population uses shared taps**, often with 2000 people per tap.
3.  **31% of the population has in-home water infrastructure**, but **45% of these systems are non-functional** due to issues with pipes, pumps, and reservoirs. Affected towns include Amina, rural Amanzi, Akatsi, and Hawassa.
4.  **18% of the population uses wells**, but only **28% of these wells are clean**. Contaminated wells are primarily in Hawassa, Kilimani, and Akatsi.
5.  Citizens experience **long wait times for water, averaging over 120 minutes**. Queues are longest on Saturdays, mornings, and evenings, and shortest on Wednesdays and Sundays.

## Plan of Action

The strategic plan prioritizes interventions based on impact and efficiency:
1.  **Prioritize shared taps**: Improving shared taps will benefit the most people.
2.  **Address contaminated wells**: Fixing contaminated wells will also benefit a large number of people.
3.  **Fix existing infrastructure**: Repairing broken infrastructure provides running water, reduces queue times, and solves two problems at once.
4.  **Strategic tap installation**: Installing new taps in homes will be a long-term goal, and for now, sources with low queue times will not be prioritized.
5.  **Acknowledge rural challenges**: Repair teams must be aware that most water sources are in rural areas, which poses challenges for road conditions, supplies, and labor.

## Practical Solutions and Improvements

Specific interventions are planned for each type of water source:

1.  **Rivers**:
    *   **Short-term**: Dispatch water trucks to regions using rivers.
    *   **Long-term**: Send crews to **drill wells** for a permanent solution.
    *   **Target**: **Sokoto province will be targeted first**.

2.  **Wells**:
    *   **Chemically contaminated wells**: Install **Reverse Osmosis (RO) filters**.
    *   **Biologically contaminated wells**: Install **UV filters to kill microorganisms, and also RO filters**.
    *   **Long-term**: Investigate the root causes of pollution.

3.  **Shared Taps**:
    *   **Short-term (for busiest taps)**: Send additional water tankers to reduce queue times, especially on busy days/times identified from queue time analysis.
    *   **Long-term (for queues >= 30 min)**: Install **X taps nearby**, where **X = FLOOR(time_in_queue / 30)** to bring wait times below the UN standard of 30 minutes.
    *   **Target Towns**: **Bello, Abidjan, and Zuri** will be prioritized for new tap installations due to high usage.
    *   **Shared taps with short queue times (< 30 min)**: Installing taps in homes for these locations is a resource-intensive, long-term goal.

4.  **Broken In-Home Taps (`tap_in_home_broken`)**:
    *   **Intervention**: **Diagnose local infrastructure** issues, such as problems with reservoirs or pipes, as fixing these can benefit many people.
    *   **Target Towns**: **Amina, Lusaka, Zuri, Djenne, and rural parts of Amanzi** are good starting points.

## Project Tracking and Implementation

To effectively track the progress of these improvements, a new database table called `Project_progress` has been created.

### `Project_progress` Table Structure:
*   `Project_id` (SERIAL PRIMARY KEY): Unique key for each project, allowing multiple visits to the same source if needed.
*   `source_id` (VARCHAR(20) NOT NULL REFERENCES `water_source(source_id)` ON DELETE CASCADE ON UPDATE CASCADE): Links to the `water_source` table to ensure data integrity.
*   `Address` (VARCHAR(50)): Street address of the water source.
*   `Town` (VARCHAR(30)): Town name.
*   `Province` (VARCHAR(30)): Province name.
*   `Source_type` (VARCHAR(50)): Type of water source (e.g., river, well, shared_tap, tap_in_home_broken).
*   `Improvement` (VARCHAR(50)): Specific action engineers need to take (e.g., 'Drill well', 'Install RO filter').
*   `Source_status` (VARCHAR(50) DEFAULT 'Backlog' CHECK (`Source_status` IN ('Backlog', 'In progress', 'Complete'))): Tracks the project status, defaulting to 'Backlog'.
*   `Date_of_completion` (DATE): Date when the upgrade/repair was completed.
*   `Comments` (TEXT): Space for engineers to leave notes.

### Populating `Project_progress`:
A SQL query was developed to select relevant data from `water_source`, `location`, `visits`, and `well_pollution`, and generate the appropriate `Improvement` action using `CASE` statements. This filtered result set, including only sources requiring improvement and with `visit_count = 1`, was then inserted into the `Project_progress` table. This table now contains **25,398 rows of data** representing all identified improvement projects.

## Conclusion

The Maji Ndogo water project demonstrates how **SQL-driven data analysis** can expose systemic problems and guide targeted interventions. By systematically addressing water access and quality, the project delivers a practical roadmap to ensure a sustainable and equitable future for the people of Maji Ndogo. This comprehensive plan, from data analysis to practical implementation and tracking, will be submitted to President Naledi to organize the teams and commence the transformation of water access in Maji Ndogo.
