# Parking Management System

A console parking lot manager written in C++. It parks bikes and cars, bills them when they leave, keeps a waiting list when the lot is full, and saves its state to text files so nothing is lost between runs.

The whole program lives in one file, `parking.cpp`. It's also a data structures showcase: each feature is built on a different structure, written by hand with no STL containers.




Any compiler with C++98 or newer works. Run it from the folder where you want the data files to be created.

## How the lot works

| | Bikes | Cars |
|---|---|---|
| Slots | 1 to 100 | 101 to 200 |
| Rate | Rs. 30 per hour | Rs. 60 per hour |

- A new vehicle always gets the lowest free slot of its type.
- Billing rounds up to the next full hour, with a one hour minimum. 61 minutes is billed as 2 hours.
- If no slot of the right type is free, the vehicle joins a waiting queue (up to 200).
- When a vehicle exits, the system tries to place the vehicle at the front of the queue.

## Menu

| Option | What it does |
|---|---|
| 1 | Park a vehicle (asks for plate and type) |
| 2 | Exit a vehicle and print the bill |
| 3 | Search a vehicle by plate |
| 4 | List all parked vehicles |
| 5 | Sort parked vehicles by plate, then list them |
| 6 | Show free slots for bikes and cars |
| 7 | Show occupied slot numbers in order |
| 8 | Show the waiting queue |
| 9 | Show recent bills, newest first |
| 10 | Show total revenue |
| 0 | Save and quit |

Plates are read with `cin >>`, so they can't contain spaces and are cut at 19 characters. Matching is case sensitive: `UP16AB1234` and `up16ab1234` count as different vehicles.

## Data structures

| Class | Structure | Used for |
|---|---|---|
| `VehicleManager` | Doubly linked list | Master list of parked vehicles. Merge sort runs on it for option 5. |
| `SlotManager` | Two min-heaps | Hands out the lowest free slot for bikes and cars in O(log n). |
| `VehicleHashTable` | Hash table, chaining (211 buckets) | Quick "is this plate already parked?" check. |
| `WaitingQueue` | Circular array queue | First come, first served when the lot is full. |
| `OccupiedSlotBST` | Binary search tree | Prints occupied slots in sorted order. |
| `PlateAVLTree` | AVL tree | Plate lookup for search (option 3). Stays balanced. |
| `BillStack` | Array stack (1000 bills) | Recent bills, newest first. When full, the oldest bill is dropped. |

`Vehicle` is an abstract base class with `Bike` and `Car` subclasses that return their own rate and type name. In practice they're only used to print the hourly rate when a vehicle parks. The billing code itself uses a plain `type == 1 ? 30 : 60` check.

`ParkingSystem` ties everything together and owns the menu loop. Parking and exiting update all the structures at once (list, hash table, BST, AVL tree, heaps), so they always agree with each other.

## Files the program creates

| File | Contents | Loaded on start? |
|---|---|---|
| `parking_data.txt` | One line per parked vehicle: `plate type slot entry_epoch` | Yes |
| `revenue.txt` | Total revenue as a single number | Yes |
| `parking_history.txt` | Append-only log of every park and exit | No, it's a log only |

Type is `1` for bike and `2` for car. On startup, each record is checked and skipped if the type is invalid, the slot is out of range or already taken, the slot doesn't match the vehicle type, or the plate is a duplicate. Free-slot heaps are then rebuilt from whatever slots are left.

The data file is rewritten after every park, every exit, and every sort.

## Known limitations

- **The waiting queue and recent bills aren't saved.** Both reset when the program closes. Past bills are still in `parking_history.txt`.
- **Only one queued vehicle is tried per exit.** The front vehicle is checked, and if its slot type isn't free it goes to the back. A waiting car can sit behind a waiting bike until another exit happens.
- **Bad input isn't handled.** Typing a letter where a number is expected leaves `cin` in a failed state and the menu loops forever.
- **Slot counts are fixed.** The 100/100 split is hardcoded in several places, so changing `CAPACITY` alone won't resize the lot.
- **Two helpers aren't wired up.** `displayAVL()` and `displayCurrentTime()` exist in `ParkingSystem` but have no menu entry.
- **Time is local system time**, and bills assume the clock doesn't change while a vehicle is parked.
