# Library Management System — LLD Notes

## Problem Statement
A customer comes to borrow a book from a library.
If the book is available, it is issued and a BorrowRecord is created.
The customer must return the book within 14 days.
Late returns incur a fine of Rs.10 per day.
A librarian can add and remove books from the system.

---

## Entities & Responsibilities

| Class | Responsibility |
|---|---|
| `Book` | One physical copy on the shelf |
| `Customer` | Person who borrows — pure data |
| `Librarian` | Adds/removes books by delegating to controller |
| `BorrowRecord` | Records a borrow event — used at return for fine calculation |
| `PaymentStrategy` | Interface — how a fine is paid |
| `CashPayment` | Implements PaymentStrategy |
| `CardPayment` | Implements PaymentStrategy |
| `LibraryController` | Brain — owns all data, handles all operations |

---

## Class Breakdown

### Book
**Fields:** `bookId`, `bookName`
**Methods:** none (pure data)

---

### Customer
**Fields:** `customerId`, `customerName`
**Methods:** none (pure data)

---

### Librarian
**Fields:** `librarianId`, `librarianName`, `controller` (injected)
**Methods:** `addBook(Book)`, `removeBook(Book)`

---

### BorrowRecord
**Fields:** `borrowId`, `customer`, `borrowedBook`, `borrowDate`, `dueDate`
**Methods:** none (pure data — Controller reads its fields)

---

### PaymentStrategy
**Methods:** `pay(double amount)`

### CashPayment / CardPayment
**Methods:** `pay(double amount)` — override

---

### LibraryController
**Fields:**
- `Map<String, List<Book>> bookInventory` — bookName → physical copies on shelf
- `Map<String, Integer> bookCount` — bookName → available count
- `Map<String, BorrowRecord> activeBorrows` — borrowId → active record

**Methods:**
- `addBook(Book)`
- `removeBook(Book)`
- `searchBook(String bookName)` → `Book`
- `issueBook(Customer, String bookName)` → `BorrowRecord`
- `returnBook(BorrowRecord, PaymentStrategy)`

---

## Who is Injected Where

| Into | What is Injected | Why |
|---|---|---|
| `Librarian` | `LibraryController` | Librarian delegates addBook/removeBook to it |
| `returnBook()` | `PaymentStrategy` | Method param — changes per customer, not per controller |

---

## Design Patterns

| Pattern | Where |
|---|---|
| Strategy | PaymentStrategy → CashPayment / CardPayment |
| Delegation | Librarian → LibraryController |
