# Parking Lot System — LLD Notes

## Problem Statement
A vehicle arrives at a parking lot and the system finds an available slot.
If found, a ticket is issued and the vehicle is parked.
The lot has multiple floors, each floor has multiple parking slots.
Different slot types fit different vehicle types.
On exit, fee is calculated based on duration and payment is processed.
The slot becomes available again after exit.

---

## Entities & Responsibilities

| Class | Responsibility |
|---|---|
| `Vehicle` | Abstract — base for all vehicle types |
| `Car` | Extends Vehicle |
| `Motorcycle` | Extends Vehicle |
| `Truck` | Extends Vehicle |
| `ParkingSlot` | One physical space — knows its type, availability, and parked vehicle |
| `ParkingFloor` | One level — manages its slots, finds available ones |
| `Ticket` | Issued at entry — links vehicle + slot + entry time |
| `PaymentStrategy` | Interface — how payment is made |
| `CashPayment` | Implements PaymentStrategy |
| `CardPayment` | Implements PaymentStrategy |
| `EntrancePanel` | Gate at entry — issues tickets |
| `ExitPanel` | Gate at exit — processes payment and frees slot |
| `ParkingLot` | Top-level container — owns floors, tracks active tickets |

---

## Class Breakdown

### Vehicle (abstract)
**Fields:** `licensePlate`, `vehicleType`
**Methods:** none (pure data)

### Car / Motorcycle / Truck
**Fields:** none (inherits from Vehicle)
**Methods:** none
**Note:** Each hardcodes its own `vehicleType` string in `super()` call

---

### ParkingSlot
**Fields:** `slotId`, `slotType`, `isAvailable`, `parkedVehicle`
**Methods:**
- `canFit(Vehicle)` → `boolean` — does this slot accept this vehicle type?
- `parkVehicle(Vehicle)` — marks occupied
- `unparkVehicle()` — marks available, clears vehicle

---

### ParkingFloor
**Fields:** `floorNumber`, `List<ParkingSlot> slots`
**Methods:**
- `addSlot(ParkingSlot)`
- `findAvailableSlot(Vehicle)` → `ParkingSlot`
- `availableSlotCount()` → `int`

---

### Ticket
**Fields:** `ticketId`, `vehicle`, `slot`, `entryTime`
**Methods:**
- `calculateFee(LocalDateTime exitTime)` → `double`

---

### PaymentStrategy
**Methods:** `pay(double amount)` → `boolean`

### CashPayment / CardPayment
**Methods:** `pay(double amount)` → `boolean` — override

---

### EntrancePanel
**Fields:** `parkingLot` (injected)
**Methods:**
- `issueTicket(Vehicle)` → `Ticket`

---

### ExitPanel
**Fields:** `parkingLot` (injected)
**Methods:**
- `processExit(Ticket, PaymentStrategy)` → `boolean`

---

### ParkingLot (Singleton)
**Fields:**
- `name`, `address`
- `List<ParkingFloor> floors`
- `Map<String, Ticket> activeTickets` — ticketId → Ticket

**Methods:**
- `getInstance(name, address)` → `ParkingLot` — Singleton
- `addFloor(ParkingFloor)`
- `findAvailableSlot(Vehicle)` → `ParkingSlot`
- `addActiveTicket(Ticket)`
- `removeActiveTicket(Ticket)`
- `totalAvailableSlots()` → `int`

---

## Who is Injected Where

| Into | What is Injected | Why |
|---|---|---|
| `EntrancePanel` | `ParkingLot` | Needs to find slots and register tickets |
| `ExitPanel` | `ParkingLot` | Needs to remove tickets and free slots |
| `processExit()` | `PaymentStrategy` | Method param — changes per vehicle, not per panel |

---

## Communication Flow

```
Vehicle arrives
  → EntrancePanel.issueTicket(vehicle)
    → ParkingLot.findAvailableSlot(vehicle)
      → ParkingFloor.findAvailableSlot(vehicle)
        → ParkingSlot.canFit(vehicle)
    → ParkingSlot.parkVehicle(vehicle)
    → new Ticket(vehicle, slot)
    → ParkingLot.addActiveTicket(ticket)

Vehicle exits
  → ExitPanel.processExit(ticket, paymentStrategy)
    → Ticket.calculateFee(exitTime)
    → PaymentStrategy.pay(amount)
    → ParkingSlot.unparkVehicle()
    → ParkingLot.removeActiveTicket(ticket)
```

---

## Design Patterns

| Pattern | Where |
|---|---|
| Singleton | ParkingLot — one physical lot, one instance |
| Strategy | PaymentStrategy → CashPayment / CardPayment |
| Composite | ParkingLot → ParkingFloor → ParkingSlot (each level delegates downward) |
| Facade | ParkingLot hides floor/slot complexity from panels |
