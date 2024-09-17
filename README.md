# Staff Shift Optimization Problem

This code optimizes the staff shift schedule to ensure efficient and fair assignment of shifts. It assumes three shift schedule patterns.

## Shift Schedule Patterns

1. **When there is one shift pattern**

   - All staff work during the same time period.

2. **When there are two shift patterns**

   - Staff work either during the morning or the afternoon.

3. **When there are irregular shift patterns**

   - There are two shifts (morning and afternoon) on weekdays, and three shifts (morning, afternoon, and night) on weekends.

## Data Loading and Preparation

- The code reads an Excel file containing staff information and their desired vacation days.
- It calculates the workdays and holidays within a specific period and imports staff vacation requests.

## Constraints

### When there is one shift pattern

#### Two staff members are required for each working day:

```python
problem += lpSum(x[s, d] for s in STAFFS) == 2
```

#### Each staff member can work a maximum of 5 days per week:

```python
problem += lpSum(x[s, d] for d in week) <= 5

```

#### At least one full-time staff member is required on each working day:

```python
problem += lpSum(x[s, d] * S_FULLTIME_OR_NOT[s] for s in STAFFS) >= 1
```

#### The difference in the number of working days between staff members must be within one day:

```python
problem += lpSum(x[staff_i, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) - lpSum(x[staff_j, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) <= 1
problem += lpSum(x[staff_i, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) - lpSum(x[staff_j, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) <= 1

```

#### Each staff member must work at least one shift:

```python
problem += lpSum(x[s, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) >= 1
```

#### The difference in staff levels across working days must not be large:

```python
problem += level_sum >= min_value + max_value
problem += level_sum <= max_value + max_value
```

#### Respect staff vacation requests:

```python
problem += x[staff, day] == 0
```

---

### When there are two or irregular shift patterns

#### One staff member is required for each shift on each working day:

```python
for d in WORKING_DAYS_EXCLUDING_CLOSED:
    for shift in SHIFT_TYPE:
        problem += lpSum(x[s, d, shift] for s in STAFFS) == 1
```

#### Each staff member can only take one shift per day:

```python
for s in STAFFS:
    for d in WORKING_DAYS_EXCLUDING_CLOSED:
        problem += lpSum(x[s, d, shift] for shift in SHIFT_TYPE) <= 1
```

#### Staff members who work the night shift cannot work the morning shift the next day:

```python
for d in WORKING_DAYS_EXCLUDING_CLOSED:
    next_day = d + 1
    if next_day in WORKING_DAYS_EXCLUDING_CLOSED:
        for s in STAFFS:
            problem += x[s, d, "E"] + x[s, next_day, "M"] <= 1
```

#### Shift types are dynamically retrieved:

Instead of using fixed arrays for shift types (e.g., ["M", "E"] or ["M", "E", "N"]), the shift types are dynamically retrieved using the get_shift_types(d) function. This allows for varying shift structures depending on the day.

```python
for shift in get_shift_types(d):
```

#### Strengthened constraint to prevent working a night shift followed by a morning shift:

```python
if "N" in get_shift_types(d) and "M" in get_shift_types(next_day):
    problem += x[s, d, "N"] + x[s, next_day, "M"] <= 1
```

## Usage

1. Specify the Excel file to load staff information and vacation requests.
2. Set constraints based on workdays, holidays, and shift patterns.
3. Use the pulp library to compute the optimal staff shift assignment.
4. Output the calculated result and review the shift schedule.
