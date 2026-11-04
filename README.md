from datetime import datetime

# AVAILABLE COURSES
COURSES = [
    "Bachelor of Science in Information Technology",
    "Bachelor of Science in Computer Science",
    "Bachelor of Science in Information Systems",
    "Bachelor of Science in Information Technology Education",
    "Bachelor of Science in Business Administration",
    "Bachelor of Science in Accountancy",
    "Bachelor of Science in Hospitality Management",
    "Bachelor of Science in Education"
]

# AVAILABLE DOCUMENTS AND PRICES
DOCUMENTS = {
    1: ("Certificate of Enrollment", 200),
    2: ("Certificate of Grades", 200),
    3: ("Completed Academic Requirement (CAR)", 200),
    4: ("Completed the Required Units for Graduation", 200),
    5: ("Cross Enrollee Final Grades", 200),
    6: ("Currently Enrolled", 200),
    7: ("Diploma", 1500),
    8: ("English as Medium of Instruction", 200),
    9: ("General Weighted Average", 200),
    10: ("Grades with Weighted Average", 200),
    11: ("Highest Educational Attainment", 200),
    12: ("Honorable Dismissal (With Transcript of Records)", 630),
    13: ("Number of Units Earned", 200),
    14: ("Related Learning Experiences (RLE)", 200),
    15: ("TOR - for Employment (Alumni)", 1000),
    16: ("TOR - for Employment (Tertiary)", 400),
    17: ("TOR - for Reference (Graduate School)", 600),
    18: ("TOR - for Reference (Tertiary)", 400),
    19: ("TOR - for Employment (Graduate School)", 600)
}

# REGISTRAR ACCOUNT
REGISTRAR = {
    "name": "chesca",
    "password": "789"
}

# PRE-REGISTERED STUDENTS
PRE_REGISTERED = {
    "02-2026-00001": {
        "name": "Juan Dela Cruz",
        "password": "1234",
        "course": COURSES[0],
        "year": "1st Year",
        "verified": False
    },
    "02-2026-00002": {
        "name": "Maria Santos",
        "password": "1234",
        "course": COURSES[0],
        "year": "1st Year",
        "verified": False
    },
    "02-2026-00003": {
        "name": "Pedro Reyes",
        "password": "1234",
        "course": COURSES[1],
        "year": "2nd Year",
        "verified": False
    }
}

# DATA STORED IN MEMORY
students = {
    sid: info.copy()
    for sid, info in PRE_REGISTERED.items()
}

# Document requests are stored in a list.
requests = []

# AUTOMATIC DATE AND TIME
def current_time():
    return datetime.now().strftime("%Y-%m-%d %I:%M %p")

# FIND A REQUEST USING STUDENT ID
def find_request(sid):
    for request in requests:
        if request["student_id"] == sid:
            return request
    return None

# INPUT VALIDATION
def get_number(prompt, minimum, maximum):
    while True:
        value = input(prompt).strip()
        if value.isdigit():
            number = int(value)
            if minimum <= number <= maximum:
                return number
        print(f"Enter a number from {minimum} to {maximum}.")

def valid_name(name):
    return bool(name) and all(
        char.isalpha() or char.isspace() or char in "'-."
        for char in name
    )

def valid_student_id(student_id):
    return (
        bool(student_id)
        and any(char.isdigit() for char in student_id)
        and all(char.isdigit() or char == "-" for char in student_id)
    )

def get_nonempty(prompt):
    while True:
        value = input(prompt).strip()
        if value:
            return value
        print("This field cannot be empty.")

# DISPLAY STUDENT TABLE
def print_student_table():
    print("\n" + "=" * 105)
    print(f"{'NAME':<25} {'STUDENT NUMBER':<22} "
          f"{'COURSE':<43} {'STATUS':<12}")
    print("-" * 105)
    for sid, student in students.items():
        name = student.get("name", "Unknown")
        course = student.get("course", "Not specified")
        status = (
            "Verified" if student.get("verified", False)
            else "Not verified"
        )
        print(
            f"{name[:24]:<25} "
            f"{sid[:21]:<22} "
            f"{course[:42]:<43} "
            f"{status:<12}"
        )
    print("=" * 105)

# DISPLAY DOCUMENT REQUEST TABLE
def print_request_table():
    print("\n" + "=" * 100)
    print(f"{'NAME':<25} {'STUDENT NUMBER':<22} "
          f"{'DOCUMENT REQUEST':<32} {'STATUS':<15}")
    print("-" * 100)
    if not requests:
        print("No document requests found.")
    else:
        for request in requests:
            sid = request["student_id"]
            if sid not in students:
                continue
            name = students[sid].get("name", "Unknown")
            documents = ", ".join(
                item["name"] for item in request["items"]
            )
            print(
                f"{name[:24]:<25} "
                f"{sid[:21]:<22} "
                f"{documents[:31]:<32} "
                f"{request['status']:<15}"
            )
    print("=" * 100)

# DISPLAY AVAILABLE DOCUMENTS
def show_documents():
    print("\n=== AVAILABLE DOCUMENTS ===")
    for number, (name, price) in DOCUMENTS.items():
        print(f"{number}. {name} - PHP {price:,.2f}")

# VIEW REQUEST DETAILS
def show_request(sid):
    request = find_request(sid)
    if not request:
        print("No document request found.")
        return
    student = students.get(sid, {})
    print("\n=== DOCUMENT REQUEST DETAILS ===")
    print("Name:", student.get("name", "Unknown"))
    print("Student Number:", sid)
    print("Request Number:", request["id"])
    print("Date Submitted:", request["date"])
    print("\nDocuments:")
    for item in request["items"]:
        print(
            f"- {item['name']} x{item['quantity']} "
            f"- PHP {item['subtotal']:,.2f}"
        )
    print("Purpose:", request["purpose"])
    print(f"Total: PHP {request['total']:,.2f}")
    print("Status:", request["status"])
    print("Date Released:", request["released"] or "Not released")

# SELECT DOCUMENTS
def select_documents():
    selected = {}
    while True:
        show_documents()
        choice = get_number(
            "\nChoose document number (0 to finish): ", 0, 19
        )
        if choice == 0:
            break
        quantity = get_number("Number of copies (1-100): ", 1, 100)
        name, price = DOCUMENTS[choice]
        if choice in selected:
            selected[choice]["quantity"] += quantity
            selected[choice]["subtotal"] += price * quantity
        else:
            selected[choice] = {
                "name": name,
                "quantity": quantity,
                "subtotal": price * quantity
            }
        print(f"Added: {name} x{quantity}")
        another = get_number(
            "\n1. Add another document\n"
            "2. Finish selection\nChoice: ", 1, 2
        )
        if another == 2:
            break
    return list(selected.values())

# GET REQUEST PURPOSE
def get_purpose():
    print("\nYou may enter more than one purpose.")
    print("Example: Employment, school requirement, personal reference")
    return get_nonempty("Purpose of request: ")

# CREATE DOCUMENT REQUEST
def make_request(sid):
    student = students[sid]
    if not student.get("verified", False):
        print("\nYour account is not verified.")
        print("Please contact the registrar.")
        return
    if find_request(sid):
        print("\nYou have already submitted a document request.")
        print("You cannot submit another duplicate request.")
        print("You may edit or remove your request while Pending.")
        show_request(sid)
        return
    items = select_documents()
    if not items:
        print("No documents selected. Request cancelled.")
        return
    purpose = get_purpose()
    total = sum(item["subtotal"] for item in items)
    print("\n=== REQUEST SUMMARY ===")
    for item in items:
        print(
            f"{item['name']} x{item['quantity']} "
            f"- PHP {item['subtotal']:,.2f}"
        )
    print(f"\nTOTAL: PHP {total:,.2f}")
    print("Purpose:", purpose)
    confirm = get_number(
        "\n1. Submit request\n2. Cancel request\nChoice: ", 1, 2
    )
    if confirm == 2:
        print("Request cancelled.")
        return
    now = datetime.now()
    request = {
        "student_id": sid,
        "id": now.strftime("REQ-%Y%m%d%H%M%S-%f"),
        "date": current_time(),
        "items": items,
        "purpose": purpose,
        "total": total,
        "status": "Pending",
        "released": None
    }
    requests.append(request)
    print("\nRequest submitted successfully!")
    print("Student Number:", sid)
    print("Request Number:", request["id"])
    print("Status: Pending")

# EDIT DOCUMENT REQUEST
def edit_request(sid):
    request = find_request(sid)
    if not request:
        print("You have no request to edit.")
        return
    if request["status"] != "Pending":
        print("Only Pending requests can be edited.")
        return
    show_request(sid)
    print("\n=== EDIT DOCUMENT REQUEST ===")
    print("1. Edit documents and purpose")
    print("2. Cancel")
    choice = get_number("Choice: ", 1, 2)
    if choice == 2:
        return
    items = select_documents()
    if not items:
        print("No documents selected. Changes cancelled.")
        return
    purpose = get_purpose()
    total = sum(item["subtotal"] for item in items)
    print(f"\nNew total: PHP {total:,.2f}")
    print("New purpose:", purpose)
    confirm = get_number(
        "1. Save changes\n2. Cancel changes\nChoice: ", 1, 2
    )
    if confirm == 2:
        print("Changes cancelled.")
        return
    request["items"] = items
    request["purpose"] = purpose
    request["total"] = total
    request["last_edited"] = current_time()
    print("Request updated successfully.")

# REMOVE DOCUMENT REQUEST
def remove_request(sid):
    request = find_request(sid)
    if not request:
        print("You have no request to remove.")
        return
    if request["status"] != "Pending":
        print("Only Pending requests can be removed.")
        return
    show_request(sid)
    confirm = get_number(
        "\n1. Confirm removal\n2. Keep request\nChoice: ", 1, 2
    )
    if confirm == 2:
        print("Request kept.")
        return
    requests.remove(request)
    print("Request removed successfully.")

# STUDENT MENU
def student_menu(sid):
    while True:
        student = students[sid]
        print(f"\n=== STUDENT MENU: {student['name']} ===")
        print("Student Number:", sid)
        print("1. Personal Information")
        print("2. Request Documents")
        print("3. View My Request")
        print("4. Edit My Request")
        print("5. Remove My Request")
        print("6. Logout")
        choice = get_number("Choice: ", 1, 6)
        if choice == 1:
            print("\n=== PERSONAL INFORMATION ===")
            print("Name:", student["name"])
            print("Student Number (ex. 00-0000-00000):", sid)
            print("Course:", student["course"])
            print("Year Level:", student["year"])
            print(
                "Verification:",
                "Verified" if student["verified"]
                else "Not verified"
            )
        elif choice == 2:
            make_request(sid)
        elif choice == 3:
            show_request(sid)
        elif choice == 4:
            edit_request(sid)
        elif choice == 5:
            remove_request(sid)
        elif choice == 6:
            print("Student logged out.")
            break

# STUDENT LOGIN
def student_login():
    print("\n=== STUDENT LOGIN ===")
    sid = input("Student ID (ex. 00-0000-00000): ").strip()
    if not valid_student_id(sid):
        print("Invalid student ID format.")
        return
    password = input("Password: ").strip()
    student = students.get(sid)
    if student and student.get("password") == password:
        print("\nLogin successful!")
        student_menu(sid)
    else:
        print("Login failed. Check your student ID and password.")

# REGISTRAR: VERIFY STUDENTS — UPDATED FOR MULTIPLE SELECTION
def verify_student():
    print_student_table()
    print("\n=== VERIFY MULTIPLE STUDENTS ===")
    print("Enter student IDs separated by commas.")
    print("Example: 00-0000-00000, 00-0000-00000")
    ids_input = input("Student IDs: ").strip()
    
    if not ids_input:
        print("No IDs entered.")
        return
    
    # Parse and clean the list of IDs
    id_list = [sid.strip() for sid in ids_input.split(",") if sid.strip()]
    
    found_ids = []
    not_found_ids = []
    
    for sid in id_list:
        if sid in students:
            found_ids.append(sid)
        else:
            not_found_ids.append(sid)
    
    if not found_ids:
        print("None of the entered student IDs were found.")
        return
    
    # Show summary
    print("\n--- Students Found ---")
    for sid in found_ids:
        status = "Verified" if students[sid]["verified"] else "Not verified"
        print(f"  {sid} — {students[sid]['name']} ({status})")
    
    if not_found_ids:
        print("\n--- Not Found (Skipped) ---")
        for sid in not_found_ids:
            print(f"  {sid}")
    
    # Action menu
    print("\nChoose action:")
    print("1. Mark as VERIFIED")
    print("2. Mark as NOT VERIFIED")
    print("3. Cancel")
    choice = get_number("Choice: ", 1, 3)
    
    if choice == 3:
        print("No changes made.")
        return
    
    # Apply to all selected
    new_status = (choice == 1)
    updated_count = 0
    for sid in found_ids:
        if students[sid]["verified"] != new_status:
            students[sid]["verified"] = new_status
            updated_count += 1
    
    action_text = "verified" if new_status else "unverified"
    print(f"\n {updated_count} student(s) {action_text} successfully.")
    print_student_table()

# REGISTRAR: UPDATE REQUEST STATUS
def update_request_status():
    print_request_table()
    sid = input("\nEnter student ID: ").strip()
    request = find_request(sid)
    if not request:
        print("No request found for this student ID.")
        return
    show_request(sid)
    print("\n1. Pending")
    print("2. Approved")
    print("3. Rejected")
    print("4. On Hold")
    print("5. Released")
    choice = get_number("Choose new status: ", 1, 5)
    statuses = {
        1: "Pending",
        2: "Approved",
        3: "Rejected",
        4: "On Hold",
        5: "Released"
    }
    request["status"] = statuses[choice]
    request["released"] = (
        current_time() if choice == 5 else None
    )
    print("Request status updated.")
    show_request(sid)

# REGISTRAR: SEARCH RECORDS
def search_records():
    term = input(
        "Search student name, student ID, document, purpose, "
        "or status: "
    ).strip().lower()
    if not term:
        print("Search cannot be empty.")
        return
    found = False
    for sid, student in students.items():
        name = student.get("name", "")
        course = student.get("course", "")
        if (
            term in sid.lower()
            or term in name.lower()
            or term in course.lower()
        ):
            found = True
            print(
                f"\nStudent: {name} | ID: {sid} | Course: {course}"
            )
    for request in requests:
        sid = request["student_id"]
        student = students.get(sid, {})
        searchable = (
            sid + " "
            + student.get("name", "") + " "
            + request["purpose"] + " "
            + request["status"] + " "
            + " ".join(item["name"] for item in request["items"])
        ).lower()
        if term in searchable:
            found = True
            show_request(sid)
    if not found:
        print("No matching records found.")

# REGISTRAR: SYSTEM REPORTS
def reports():
    counts = {
        "Pending": 0,
        "Approved": 0,
        "Rejected": 0,
        "On Hold": 0,
        "Released": 0
    }
    total_value = 0
    for request in requests:
        status = request["status"]
        counts[status] = counts.get(status, 0) + 1
        total_value += request["total"]
    print("\n=== SYSTEM REPORTS ===")
    print("Total students:", len(students))
    print(
        "Verified students:",
        sum(1 for s in students.values() if s["verified"])
    )
    print("Total document requests:", len(requests))
    for status, count in counts.items():
        print(f"{status}: {count}")
    print(f"Total requested fees: PHP {total_value:,.2f}")

# REGISTRAR MENU
def registrar_menu():
    while True:
        print("\n=== REGISTRAR MENU ===")
        print("1. View Students Table")
        print("2. Verify Students")
        print("3. View All Document Requests")
        print("4. Update Request Status")
        print("5. Edit Request")
        print("6. Remove Request")
        print("7. Search Records")
        print("8. Reports")
        print("9. Logout")
        choice = get_number("Choice: ", 1, 9)
        if choice == 1:
            print_student_table()
        elif choice == 2:
            verify_student()
        elif choice == 3:
            print_request_table()
            sid = input(
                "\nEnter a student ID to see details, "
                "or press Enter to return: "
            ).strip()
            if sid:
                show_request(sid)
        elif choice == 4:
            update_request_status()
        elif choice == 5:
            sid = input("Enter student ID (ex. 00-0000-00000): ").strip()
            edit_request(sid)
        elif choice == 6:
            sid = input("Enter student ID (ex. 00-0000-00000): ").strip()
            remove_request(sid)
        elif choice == 7:
            search_records()
        elif choice == 8:
            reports()
        elif choice == 9:
            print("Registrar logged out.")
            break

# REGISTRAR LOGIN
def registrar_login():
    print("\n=== REGISTRAR LOGIN ===")
    name = input("Registrar name: ").strip().lower()
    password = input("Password: ").strip()
    if name == REGISTRAR["name"] and password == REGISTRAR["password"]:
        print("\nRegistrar login successful!")
        registrar_menu()
    else:
        print("Registrar login failed.")

# MAIN MENU
def main():
    while True:
        print("\n====== DOCUMENT REQUEST SYSTEM ======")
        print("1. Student Login")
        print("2. Registrar Login")
        print("3. Exit")
        choice = get_number("Choose: ", 1, 3)
        if choice == 1:
            student_login()
        elif choice == 2:
            registrar_login()
        elif choice == 3:
            print("Thank you for using the system!")
            break

if __name__ == "__main__":
    main()
