CIT-223-101\2025
MICHAEL MUNYOKI DOMINIC
 **Modern Smart Parking Management System**

---

## 1. System Architecture: The 8 Core Modules

```
                        +--------------------------------+
                        |  1. Slot Availability & Display|
                        +---------------+----------------+
                                        |
        +-------------------------------+-------------------------------+
        |                               |                               |
+-------v-------+               +-------v-------+               +-------v-------+
| 2. Vehicle    |               | 3. Vehicle    |               | 4. Billing &  |
|    Entry      |               |    Exit       |               |    Tariff     |
+-------+-------+               +-------+-------+               +-------+-------+
        |                               |                               |
        +-------------------------------+-------------------------------+
                                        |
        +-------------------------------+-------------------------------+
        |                               |                               |
+-------v-------+               +-------v-------+               +-------v-------+
| 5. Payment    |               | 6. Database   |               | 7. User &     |
|    Gateway    |               |    Operations |               |    Admin      |
+---------------+               +---------------+               +---------------+
                                        |
                                +-------v-------+
                                | 8. Reporting  |
                                |   & Analytics |
                                +---------------+

```

1. **Slot Availability & Display Module**: Tracks open parking slots in real time and publishes updates to digital boards/mobile clients before vehicles enter.
2. **Vehicle Entry Module**: Scans vehicle license plates (ALPR/ANPR), verifies open space, records check-in timestamp, issues digital tickets, and opens entry barriers.
3. **Vehicle Exit Module**: Scans exiting vehicles, fetches entry timestamps, calculates duration, releases assigned slots, and opens exit gates upon payment clearance.
4. **Billing & Tariff Module**: Evaluates duration using predefined rules (e.g., hourly rates, flat rates, grace periods, peak surcharges) to calculate the total fee.
5. **Payment Gateway Module**: Manages physical and digital payments (Cash, Credit Card, Mobile Payments/M-Pesa, NFC) and validates payment success before triggering exit gate opening.
6. **Database Operations Module**: Handles transactional persistence (ACID compliance) for check-in records, historical logs, billing receipts, and slot allocation.
7. **User & Admin Management Module**: Admin controls for setting tariffs, managing staff roles, overriding gates manually during emergencies, and auditing activity.
8. **Reporting & Analytics Module**: Generates daily financial summaries, peak occupancy trends, revenue collection reports, and average stay durations.

---

## 2. Key Data Structures Used

* **Hash Map / Dictionary (`Map<String, VehicleRecord>`)**: Provides $O(1)$ constant time lookup for active parked vehicles using the **License Plate Number** as the key.
* **Min-Heap / Priority Queue (`PriorityQueue<Slot>`)**: Holds available parking slot IDs ordered by distance to the nearest entrance, allowing $O(\log n)$ allocation of the closest slot.
* **Atomic Integer (`AtomicInteger availableSlots`)**: Thread-safe integer counter ensuring concurrent entry/exit gate operations do not cause race conditions.
* **Queue / Ring Buffer (`Queue<Transaction>`)**: Holds entry and payment processing tasks for sequential execution under high traffic conditions.

---

## 3. Core Algorithms

### Module 2: Vehicle Entry Algorithm

```text
START EntryProcess(LicensePlate)
  LOCK availableSlots Counter
  IF availableSlots <= 0 THEN
    DISPLAY "Parking Lot Full - Access Denied"
    UNLOCK availableSlots Counter
    EXIT
  ENDIF

  AssignedSlot = PriorityQueue.ExtractMin()  // Get closest available slot
  CheckInTime = GetCurrentTimestamp()

  // Store in memory for O(1) exit validation
  ActiveVehiclesMap.Put(LicensePlate, {SlotID: AssignedSlot, CheckIn: CheckInTime})
  
  availableSlots = availableSlots - 1
  UNLOCK availableSlots Counter

  SaveEntryToDatabase(LicensePlate, AssignedSlot, CheckInTime)
  TRIGGER GateBarrier(OPEN)
  DISPLAY "Welcome! Assigned Slot: " + AssignedSlot + " | Free Slots: " + availableSlots
END

```

### Module 3 & 4: Vehicle Exit & Billing Algorithm

```text
START ExitProcess(LicensePlate)
  IF NOT ActiveVehiclesMap.Contains(LicensePlate) THEN
    DISPLAY "Error: Vehicle entry record not found."
    EXIT
  ENDIF

  Record = ActiveVehiclesMap.Get(LicensePlate)
  CheckOutTime = GetCurrentTimestamp()
  DurationInSeconds = CheckOutTime - Record.CheckIn

  // Billing Module Calculation
  DurationHours = CEIL(DurationInSeconds / 3600)
  
  IF DurationInSeconds <= GracePeriodSeconds THEN
    TotalAmount = 0.00
  ELSE
    TotalAmount = DurationHours * HourlyRate
  ENDIF

  DISPLAY "Duration: " + DurationHours + " Hours | Amount Due: $" + TotalAmount

  PaymentStatus = ProcessPayment(LicensePlate, TotalAmount)

  IF PaymentStatus == SUCCESS THEN
    LOCK availableSlots Counter
    ActiveVehiclesMap.Remove(LicensePlate)
    PriorityQueue.Insert(Record.SlotID)     // Return slot to queue
    availableSlots = availableSlots + 1
    UNLOCK availableSlots Counter

    UpdateDatabaseExit(LicensePlate, CheckOutTime, TotalAmount)
    TRIGGER ExitBarrier(OPEN)
    DISPLAY "Thank You! Slots Available: " + availableSlots
  ELSE
    DISPLAY "Payment Failed. Barrier Locked."
  ENDIF
END

```

---

## 4. Database Schema Design (Relational SQL)

### Table 1: `parking_lots`

| Field Name | Data Type | Constraints | Description |
| --- | --- | --- | --- |
| `lot_id` | INT | Primary Key, Auto Increment | Unique parking facility ID |
| `name` | VARCHAR(100) | NOT NULL | Facility name |
| `total_capacity` | INT | NOT NULL | Total physical parking capacity |
| `available_slots` | INT | NOT NULL | Dynamic count of open slots |
| `hourly_rate` | DECIMAL(10,2) | NOT NULL | Standard hourly rate |

### Table 2: `parking_slots`

| Field Name | Data Type | Constraints | Description |
| --- | --- | --- | --- |
| `slot_id` | VARCHAR(10) | Primary Key | e.g., "A-12", "B-05" |
| `lot_id` | INT | Foreign Key (`parking_lots.lot_id`) | Associated parking lot |
| `is_occupied` | BOOLEAN | DEFAULT FALSE | Current status of the spot |

### Table 3: `parking_records`

| Field Name | Data Type | Constraints | Description |
| --- | --- | --- | --- |
| `ticket_id` | VARCHAR(36) | Primary Key (UUID) | Unique transaction ID |
| `license_plate` | VARCHAR(15) | NOT NULL, Indexed | Vehicle identifier |
| `slot_id` | VARCHAR(10) | Foreign Key (`parking_slots.slot_id`) | Occupied slot |
| `check_in` | DATETIME | NOT NULL | Entry timestamp |
| `check_out` | DATETIME | NULLABLE | Exit timestamp |
| `amount_paid` | DECIMAL(10,2) | DEFAULT 0.00 | Total calculated payment |
| `status` | ENUM | ('PARKED', 'PAID', 'COMPLETED') | Active lifecycle status |

---

## 5. Project Directory & File Layout



```text
smart_parking_system/
│
├── config/
│   └── settings.py              # Configuration settings (Tariffs, DB connections)
├── modules/
│   ├── __init__.py
│   ├── 1_availability_display.py# Module 1: Live availability streaming
│   ├── 2_vehicle_entry.py      # Module 2: ANPR Scanner & check-in logic
│   ├── 3_vehicle_exit.py       # Module 3: Check-out logic & slot release
│   ├── 4_billing_tariff.py     # Module 4: Hourly/grace period pricing rules
│   ├── 5_payment_gateway.py    # Module 5: Card/M-Pesa/Cash interfaces
│   ├── 6_database_ops.py       # Module 6: SQL CRUD and schema initialization
│   ├── 7_admin_management.py   # Module 7: Admin overrides & rate configuration
│   └── 8_analytics_reports.py  # Module 8: Daily revenue & occupancy metrics
├── database/
│   └── schema.sql              # PostgreSQL/MySQL database DDL scripts
├── tests/
│   └── test_parking_flow.py    # Integration tests simulating vehicle entry/exit
├── README.md                   # Full system design blueprint documentation
└── main.py                     # Entry point orchestrating all 8 modules

```
