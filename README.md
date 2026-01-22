# library-management-project
function is login and logout function 


<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
<link rel="stylesheet" href="style.css">
  <title>Library Management</title>
</head>
<body>

  <h2>Login</h2>

  <!-- Login Form -->
  <form id="loginForm">
    <input type="email" id="email" placeholder="Email" required />
    <br><br>
    <input type="password" id="password" placeholder="Password" required />
    <br><br>
    <button type="submit">Login</button>
  </form>

  <!-- Login ke baad ye show hoga -->
  <div id="app" style="display:none;">
    <h2>Welcome! </h2>
    <button type="button" onclick="logout()">Logout</button>
    <hr>

    <h3>Add Book</h3>
    <input id="bookId" type="number" placeholder="Book ID (e.g. 1)" />
    <br><br>
    <input id="bookTitle" type="text" placeholder="Title" />
    <br><br>
    <input id="bookAuthor" type="text" placeholder="Author" />
    <br><br>
    <button type="button" onclick="uiAddBook()">Add Book</button>

    <hr>

    <h3>Add Member</h3>
    <input id="memberId" type="number" placeholder="Member ID (e.g. 101)" />
    <br><br>
    <input id="memberName" type="text" placeholder="Member Name" />
    <br><br>
    <button type="button" onclick="uiAddMember()">Add Member</button>

    <hr>

    <h3>Issue Book</h3>
    <input id="issueBookId" type="number" placeholder="Book ID" />
    <br><br>
    <input id="issueMemberId" type="number" placeholder="Member ID" />
    <br><br>
    <button type="button" onclick="uiIssueBook()">Issue</button>

    <hr>

    <h3>Return Book</h3>
    <input id="returnBookId" type="number" placeholder="Book ID" />
    <br><br>
    <button type="button" onclick="uiReturnBook()">Return</button>

    <hr>

    <h3>View</h3>
    <button type="button" onclick="viewBooks()">View All Books</button>
    <button type="button" onclick="viewIssuedBooks()">View Issued Books</button>
    <button type="button" onclick="clearOutput()">Clear Output</button>
  <div class="container">

  <div class="grid">
    <div class="section">Add Book ...</div>
    <div class="section">Add Member ...</div>
    <div class="section">Issue Book ...</div>
    <div class="section">Return Book ...</div>
  </div>

  <div class="section">
    View ...
  </div>
  <script>
  document.addEventListener("DOMContentLoaded", () => {
    const form = document.getElementById("loginForm");
    const loginBox = document.getElementById("loginBox") || form; // fallback
    const dashboardBox =
      document.getElementById("dashboardBox") || document.getElementById("app");

    if (!form) {
      console.error("loginForm not found. Check id='loginForm'");
      return;
    }

  
    if (localStorage.getItem("isLoggedIn") === "1") {
      if (loginBox) loginBox.style.display = "none";
      if (dashboardBox) dashboardBox.style.display = "block";
    } else {
      if (loginBox) loginBox.style.display = "block";
      if (dashboardBox) dashboardBox.style.display = "none";
    }

    // Login submit
    form.addEventListener("submit", async (e) => {
      e.preventDefault();

      const email = document.getElementById("email")?.value.trim();
      const password = document.getElementById("password")?.value.trim();

      try {
        const res = await fetch("http://127.0.0.1:8000/login", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ email, password }),
        });

        console.log("STATUS:", res.status);
        const data = await res.json();
        console.log("DATA:", data);

        if (res.ok && data.success) {
          localStorage.setItem("isLoggedIn", "1");
          if (loginBox) loginBox.style.display = "none";
          if (dashboardBox) dashboardBox.style.display = "block";
          alert("Login successful");
        } else {
          alert(data.message || ("Login failed. Status: " + res.status));
        }
      } catch (err) {
        console.error("Fetch error:", err);
        alert("Backend connect nahi ho raha. 8000/docs check karo.");
      }
    });

    // Logout
    window.logout = function logout() {
      localStorage.removeItem("isLoggedIn");
      if (dashboardBox) dashboardBox.style.display = "none";
      if (loginBox) loginBox.style.display = "block";
      form.reset();
    };
  });
</script>


##backbend file


from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import mysql.connector
from mysql.connector import Error
import os

from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],   # testing ke liye
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ---------- FASTAPI LOGIN ----------
class LoginData(BaseModel):
    email: str
    password: str

@app.post("/login")
def api_login(data: LoginData):
    # Dummy user (test) - SAME logic
    if data.email == "vt1703651@gmail.com" and data.password == "password123":
        return {"success": True, "message": "Login successful"}
    return {"success": False, "message": "invalid email or password"}


# ---------- MYSQL CONNECTION ----------
def get_connection():
    """
    Improvement:
    - ENV variables use (fallback same values)
    - error handling
    """
    try:
        return mysql.connector.connect(
            host=os.getenv("DB_HOST", "localhost"),
            user=os.getenv("DB_USER", "root"),
            password=os.getenv("DB_PASSWORD", "vishal278568"),
            database=os.getenv("DB_NAME", "library_management.py"),
        )
    except Error as e:
        print("Database connection error:", e)
        return None


def safe_int_input(prompt: str):
    """Improvement: int input safe"""
    while True:
        val = input(prompt)
        try:
            return int(val)
        except ValueError:
            print("Please enter a valid number.")


# ---------- TERMINAL LOGIN ----------
def terminal_login():
    username = input("Enter username: ").strip()
    password = input("Enter password: ").strip()

    conn = get_connection()
    if conn is None:
        print("Cannot connect to database.")
        return False

    cursor = None
    try:
        cursor = conn.cursor()
        cursor.execute(
            "SELECT * FROM users WHERE username=%s AND password=%s",
            (username, password)
        )
        result = cursor.fetchone()
        return bool(result)
    except Error as e:
        print("Login query error:", e)
        return False
    finally:
        if cursor:
            cursor.close()
        conn.close()

    # NOTE: Print messages outside for clean flow


# ---------- ADD BOOK ----------
def add_book():
    title = input("Enter book title: ").strip()
    author = input("Enter author name: ").strip()
    category = input("Enter category: ").strip()
    quantity = safe_int_input("Enter quantity: ")

    if not title or not author or not category:
        print("Title/Author/Category cannot be empty.")
        return
    if quantity < 0:
        print("Quantity cannot be negative.")
        return

    conn = get_connection()
    if conn is None:
        print("Cannot connect to database.")
        return

    cursor = None
    try:
        cursor = conn.cursor()
        sql = "INSERT INTO books (title, author, category, quantity) VALUES (%s, %s, %s, %s)"
        data = (title, author, category, quantity)
        cursor.execute(sql, data)
        conn.commit()
        print("Book added successfully")
    except Error as e:
        print("Error adding book:", e)
        conn.rollback()
    finally:
        if cursor:
            cursor.close()
        conn.close()


# ---------- VIEW BOOKS ----------
def view_books():
    conn = get_connection()
    if conn is None:
        print("Cannot connect to database.")
        return

    cursor = None
    try:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM books")
        rows = cursor.fetchall()

        print("\n--- BOOK LIST ---")
        if not rows:
            print("No books found.")
            return

        for row in rows:
            print(row)
    except Error as e:
        print("Error viewing books:", e)
    finally:
        if cursor:
            cursor.close()
        conn.close()


# ---------- UPDATE BOOK ----------
def update_book():
    book_id = safe_int_input("Enter book ID to update: ")
    quantity = safe_int_input("Enter new quantity: ")

    if quantity < 0:
        print("Quantity cannot be negative.")
        return

    conn = get_connection()
    if conn is None:
        print("Cannot connect to database.")
        return

    cursor = None
    try:
        cursor = conn.cursor()

        # Improvement: check book exists
        cursor.execute("SELECT book_id FROM books WHERE book_id=%s", (book_id,))
        if cursor.fetchone() is None:
            print("Book not found.")
            return

        cursor.execute(
            "UPDATE books SET quantity = %s WHERE book_id = %s",
            (quantity, book_id)
        )
        conn.commit()
        print("Book updated successfully")
    except Error as e:
        print("Error updating book:", e)
        conn.rollback()
    finally:
        if cursor:
            cursor.close()
        conn.close()


# ---------- DELETE BOOK ----------
def delete_book():
    book_id = safe_int_input("Enter book ID to delete: ")

    conn = get_connection()
    if conn is None:
        print("Cannot connect to database.")
        return

    cursor = None
    try:
        cursor = conn.cursor()

        # Improvement: check book exists
        cursor.execute("SELECT book_id FROM books WHERE book_id=%s", (book_id,))
        if cursor.fetchone() is None:
            print("Book not found.")
            return

        cursor.execute("DELETE FROM books WHERE book_id = %s", (book_id,))
        conn.commit()
        print("Book deleted successfully")
    except Error as e:
        print("Error deleting book:", e)
        conn.rollback()
    finally:
        if cursor:
            cursor.close()
        conn.close()


# ---------- MAIN MENU ----------
def main():
    # optional: login check
    # if not terminal_login():
    #     print("Invalid username or password")
    #     return
    # else:
    #     print("Login successful")




  CSS FILE

  body{
  font-family: Arial, sans-serif;
  background:#f5f6fa;
  margin:0;
  padding:20px;
}
.container{
  max-width: 900px;
  margin:auto;
  background:#fff;
  padding:20px;
  border-radius:12px;
  box-shadow:0 4px 12px rgba(0,0,0,0.08);
}
.section{
  border:1px solid #ddd;
  padding:15px;
  border-radius:10px;
  margin:15px 0;
}
input{
  width: 280px;
  padding:10px;
  margin:8px 0;
}
button{
  padding:10px 16px;
  border:none;
  border-radius:8px;
  cursor:pointer;
}
button:hover{ opacity:0.9; }
.grid{
  display:grid;
  grid-template-columns: 1fr 1fr;
  gap:16px;
}
@media (max-width: 800px){
  .grid{ grid-template-columns: 1fr; }
}
.btn-add{ background:#2ecc71; color:white; }
.btn-issue{ background:#3498db; color:white; }
.btn-return{ background:#e67e22; color:white; }
.btn-clear{ background:#95a5a6; color:white; }


.topbar{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:16px;
}
.btn-logout{
  background:#e74c3c;
  color:white;
  padding:10px 14px;
  border:none;
  border-radius:8px;
  cursor:pointer;
}
.btn-logout:hover{ opacity:0.9; }



javascript  frontend file


// In-memory data storage
let books = [];
let members = [];
let issuedBooks = [];

// Helpers
function findBook(bookId) {
  return books.find(b => b.id === bookId);
}
function findMember(memberId) {
  return members.find(m => m.id === memberId);
}

// Add a new book (with validation + duplicate check)
function addBook(id, title, author) {
  if (id === undefined || title?.trim() === "" || author?.trim() === "") {
    return console.log("Invalid book data");
  }
  if (findBook(id)) return console.log("Book ID already exists");

  books.push({ id, title: title.trim(), author: author.trim(), available: true });
  console.log(`Book added: ${title}`);
}

// Add a new member (with validation + duplicate check)
function addMember(id, name) {
  if (id === undefined || name?.trim() === "") {
    return console.log("Invalid member data");
  }
  if (findMember(id)) return console.log("Member ID already exists");

  members.push({ id, name: name.trim() });
  console.log(`Member added: ${name}`);
}

// Issue a book to a member
function issueBook(bookId, memberId) {
  const book = findBook(bookId);
  const member = findMember(memberId);

  if (!book) return console.log("Book not found");
  if (!member) return console.log("Member not found");
  if (!book.available) return console.log("Book already issued");

  book.available = false;

  issuedBooks.push({
    bookId,
    memberId,
    issueDate: new Date(),
    returned: false
  });

  console.log(`Book '${book.title}' issued to ${member.name}`);
}

// Return a book
function returnBook(bookId) {
  const book = findBook(bookId);
  if (!book) return console.log("Book not found");

  if (book.available) return console.log(`Book '${book.title}' is already available (not issued).`);

  // mark last active issue record as returned
  const activeRecord = [...issuedBooks].reverse().find(r => r.bookId === bookId && r.returned === false);
  if (activeRecord) activeRecord.returned = true;

  book.available = true;
  console.log(`Book '${book.title}' returned`);
}

// View all books
function viewBooks() {
  console.log("\n--- All Books ---");
  if (books.length === 0) return console.log("No books found");

  books.forEach(b =>
    console.log(`${b.id} | ${b.title} | ${b.author} | Available: ${b.available}`)
  );
}

// View issued books (only active or all)
function viewIssuedBooks(showAllHistory = false) {
  console.log("\n--- Issued Books ---");

  const list = showAllHistory ? issuedBooks : issuedBooks.filter(r => !r.returned);
  if (list.length === 0) return console.log("No issued books");

  list.forEach(r => {
    const book = findBook(r.bookId);
    const member = findMember(r.memberId);

    const bookTitle = book ? book.title : "(Book missing)";
    const memberName = member ? member.name : "(Member missing)";
    console.log(`${bookTitle} issued to ${memberName} on ${r.issueDate}`);
  });
}

// Demo usage
addBook(1, "JavaScript Guide", "John Doe");
addBook(2, "HTML Mastery", "Alex Roy");
addMember(101, "Vishal Thakur");
addMember(102, "Anita Sharma");
issueBook(1, 101);

viewBooks();
viewIssuedBooks();

returnBook(1);
viewBooks();
viewIssuedBooks();
viewIssuedBooks(true);


    while True:
        print("\n===== LIBRARY MANAGEMENT SYSTEM =====")
        print("1. Add Book")
        print("2. View Books")
        print("3. Update Book")
        print("4. Delete Book")
        print("5. Exit")

        choice = input("Enter your choice: ").strip()

        if choice == "1":
            add_book()
        elif choice == "2":
            view_books()
        elif choice == "3":
            update_book()
        elif choice == "4":
            delete_book()
        elif choice == "5":
            print("Exiting program...")
            break
        else:
            print("Invalid choice, try again")
    main()



<script src="first.js" defer></script>
</body>
</html>
