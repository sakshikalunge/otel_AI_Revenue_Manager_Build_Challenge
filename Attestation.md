# ATTESTATION.md (Phase 0)

Copy this file to your solution repository as `ATTESTATION.md` and fill it in
before starting Phase 1. Keep answers concise — a few sentences per prompt.

---

## Candidate

- Name: Sakshi KalungePatil
- Repository URL: https://github.com/sakshikalunge/otel_AI_Revenue_Manager_Build_Challenge
- Date: 14-06-2026

---

## Comprehension prompts

### 1. Fact-table grain

In one sentence, what is the grain of `reservations_hackathon`?

> Your answer: reseravtions_hackathon = 1 row/reservation * stay_date

### 2. Revenue columns

Name the two revenue columns and when to use each.

> Your answer: daily_room_revenue_before_tax to be used for room-specific calculations such as ADR and daily_total_revenue_before_tax to be used for business/ commercial calculatios.

### 3. Row vs reservation

Give one example question where counting rows would be wrong.

> Your answer: Question: How many reservations do we have for June month? Here, the counting should be done at table grain level i.e. reservation * stay_date instead of blindly counting the number of rows.
> As, this might result into overcount of same booking. 

### 4. Schema fields

Is there an `otel_challenge_token` column in the official schema? If so, what is it used for?

> Your answer: No, the official schema does not contain any column named 'otel_challenge_token'.

### 5. Default OTB filters

Which `reservation_status` and `financial_status` values are excluded from default OTB?

> Your answer: 'Cancelled' values are excluded from 'reservation status' and 'Provisional' values are excluded from 'financial_status' as it shows an uncertain booking.

### 6. Stay date vs property date

When can `property_date` differ from `stay_date`, and which field drives monthly OTB?

> Your answer: property_date may differ from stay_date when the hotel's business day does not align with the stay_date ex. night audits. stay_date drives the monthly OTB as this field tells the actual consumed stay.

### 7. Point-in-time OTB

How does `as_of_utc` change which cancelled rows are included in `get_as_of_otb`?

> Your answer: 'as_of_utc' field is used for historical data determining whether to include a cancelled reservation on the books at historical point of time. A cancelled reservation is included if it exists before the as_of_utc value and cancellation_datetime occurs after the as_of_utc value because at this time the reservation still existed in the records.

### 8. Block vs transient

How does `is_block` affect a “group vs transient mix” question?

> Your answer: is_block is a boolean field stating 'true' if the reservations are made for a group of people/ business event/ wedding etc and 'false' for normal stay reservations for indivisuals or travellers. Is_block affects the “group vs transient mix” as group/block provides stable room occupancy at negotiable ADR rates where as transient/indivisuals have higher ADR rates. 

### 9. List pagination

How many reservations does the data site show per list page?

> Your answer: The list contains 3 pages with total of 254 reservation records, single page containing 100 records each and the final page contains the remaining pages. Hence, ETL must ensure the complete extraction of all the records.

### 10. Pagination completeness

How will you prove you did not miss the last list page during ETL?

> Your answer: Will iterate the ETL through the last page of the resedrvations records until its last exhaustion, will also persist and keep a track of total scraped reservations IDs and page list count in a scrape manifest and finally, reassure and reconcile the verify section of the data site to confirm the extraction of all records and load fingerprints to confirm no pages were skipped.

### 11. Tool grain

For `get_otb_summary`, what is the difference between `row_count` and `reservation_count`?

> Your answer: 'row_count' gives the COUNT(*) of stay nights reservations at the table grain level, where as 'reservation_count' gives the unique reservation IDs (COUNT(DISTINCT reservation_id)). A single reservation may have multiple stay nights contributing to multiple rows but only one reservation count.

### 12. Human-in-the-loop

Why must `get_as_of_otb` be gated behind approval, and what goes wrong if it is not?

> Your answer: 'get_as_of_otb' is expensive because it performs historical recalculations based on create and cancellation datetimes which requires large historical data scans affecting performance and cause unnecessary database load spikes hence, for these particular reasons it must be gated behind approvals.

### 13. Skill vs tool

Name one revenue-manager question that should load a **skill** but call **`get_segment_mix`**, not raw SQL.

> Your answer: Is the hotel becoming too dependent on the OTA for future revenue? This question should load the revenue-based or channel-mix skill, while calling 'get_segmented_mix' to retrieve validated segment and room-night shares. Th agent should directly query from raw SQL as the OTB filters and segment definitions are already enforced in the tool layer.

---

## ETL design (one line)

Describe pagination strategy + idempotency approach + **anchor date** you will
scrape against (must match `/verify` on load day).

> Your answer: ETL paginates through all the pages until the pagination exhaustion (Next button disabled) drilling through stay rows data at grain level including financial data using Playwrite. Perform idempotent PostgreSQL Upsert opeartions at reseravtion records * stay_date grain level and reconcile the idempotency against the verify section of data site for the same scrape day anchor date. 
