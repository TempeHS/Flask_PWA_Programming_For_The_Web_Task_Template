# EXTENSION ACTIVITIES FOR FLASK PWA PROJECT

These extension activities build upon the core tutorial and introduce more advanced Python, Flask and PWA concepts. Each extension maintains the same approach as the main tutorial, with detailed explanations and learning checkpoints.

> [!Important]
> Before starting any extension activity, ensure you have completed the main tutorial and have a working Flask PWA application. It's recommended to commit your working code before attempting extensions.

## Table of Contents

### Extension Activities

1. [**Extension 1: Create Email Subscription Form**](#extension-1-create-email-subscription-form)

   - Build a complete contact form with database integration
   - Learn HTTP methods (GET vs POST) and form handling
   - Practice SQL INSERT operations with parameterised queries
   - Difficulty: ⭐⭐ Intermediate
   - Files Modified: `database_manager.py`, `main.py`, `templates/add.html`, `static/css/style.css`

2. [**Extension 2: Handle Duplicate Email Submissions**](#extension-2-handle-duplicate-email-submissions)

   - Implement Python exception handling with try-except
   - Handle SQL UNIQUE constraint violations gracefully
   - Display user-friendly error messages
   - Difficulty: ⭐⭐ Intermediate
   - Files Modified: `database_manager.py`, `main.py`, `templates/add.html`

3. [**Extension 3: Refactor Database Calls with Context Managers**](#extension-3-refactor-database-calls-with-context-managers)

   - Improve code reliability by using Python's `with` statement
   - Automatic resource management for database connections
   - Prevent resource leaks and connection issues
   - Difficulty: ⭐ Beginner
   - Files Modified: `database_manager.py`

4. [**Extension 4: Add Language Filtering to the Extensions Catalogue**](#extension-4-add-language-filtering-to-the-extensions-catalogue)

   - Implement URL query parameters for filtering
   - Create filtered database queries with WHERE clauses
   - Add interactive filter buttons to the UI
   - Difficulty: ⭐⭐ Intermediate
   - Files Modified: `database_manager.py`, `main.py`, `templates/index.html`, `static/css/style.css`

5. [**Extension 5: Add Online/Offline Status Detection**](#extension-5-add-onlineoffline-status-detection)

   - Display user-friendly offline notifications
   - Use browser online/offline events
   - Enhance PWA user experience with network status feedback
   - Difficulty: ⭐⭐ Intermediate
   - Files Modified: `templates/layout.html`, `static/css/style.css`, `static/js/app.js`

6. [**Extension 6: Add App Installation Prompt**](#extension-6-add-app-installation-prompt)
   - Create custom PWA installation button
   - Handle `beforeinstallprompt` event
   - Improve app discoverability and installation rates
   - Difficulty: ⭐⭐ Intermediate
   - Files Modified: `templates/layout.html`, `static/css/style.css`, `static/js/app.js`

### Prerequisites Check

Before starting extensions, ensure you have:

- ✅ Completed the main Flask PWA tutorial
- ✅ Working Flask application with database
- ✅ Service worker registered and functioning
- ✅ Git repository initialized with committed code
- ✅ Understanding of HTML, CSS, JavaScript basics
- ✅ Familiarity with Flask routing and Jinja2 templates

---

## Extension 1: Create Email Subscription Form

### 📚 Learning Context

**Syllabus Outcome**: _Model elements that form a web development system including client-side (front-end) and server-side (back-end) web programming_

**What You'll Learn:**

- How HTTP GET and POST methods work in web forms
- Processing form data on the server with Flask
- Inserting data into SQLite databases with SQL INSERT
- Using parameterised queries to prevent SQL injection
- Conditional rendering in Jinja2 templates
- Form validation with HTML5 attributes

**Real-World Context:**
Every website needs to collect user input - contact forms, registration pages, newsletter subscriptions, comment sections. This extension teaches you the fundamental full-stack workflow: user enters data → form submits → server processes → database stores → confirmation displays.

### 🎯 Learning Objectives

By completing this extension, you will:

1. Understand the difference between GET and POST HTTP methods
2. Create a database table with UNIQUE constraints
3. Write secure SQL INSERT statements using parameterised queries
4. Handle form submissions in Flask routes
5. Display conditional success messages to users
6. Validate user input with HTML5 form attributes

### 💭 Before You Start - Understanding Check

Let's make sure you understand the big picture before diving into code.

**Question 1: The Form Journey**
When a user fills out a form and clicks "Submit", what journey does that data take?

<details>
<summary>Click to see the answer</summary>

1. **Browser** → User types data into HTML form fields
2. **HTTP POST Request** → Form submits data to server via POST method
3. **Flask Route** → Server receives data in `request.form`
4. **Python Function** → Extracts email and name from form
5. **Database Function** → Calls `insertContact(email, name)`
6. **SQL INSERT** → Adds record to `contact_list` table
7. **Template Render** → Returns success message to browser
8. **Browser** → Displays confirmation to user

This is called the **request-response cycle** in web development!

</details>

**Question 2: GET vs POST**
What's the difference between a GET request and a POST request?

<details>
<summary>Click to see the answer</summary>

**GET Requests:**

- Used for retrieving/viewing data
- Parameters visible in URL (`/add.html?name=John`)
- Bookmarkable and cached
- Example: Viewing a blog post, searching

**POST Requests:**

- Used for submitting/creating data
- Parameters hidden in request body (not in URL)
- Not cached, not bookmarkable
- Example: Submitting a form, creating an account

**Rule of thumb**: If the action changes data on the server (create, update, delete), use POST. If it just retrieves data, use GET.

</details>

**Question 3: SQL Injection**
Why can't we just use f-strings to build SQL queries like this?

```python
# ❌ DANGEROUS - Never do this!
query = f"INSERT INTO contact_list VALUES ('{email}', '{name}')"
```

<details>
<summary>Click to see the answer</summary>

**This is vulnerable to SQL injection attacks!**

If a malicious user enters:

```
Email: test@example.com'; DROP TABLE contact_list; --
```

Your SQL would become:

```sql
INSERT INTO contact_list VALUES ('test@example.com'; DROP TABLE contact_list; --', 'John')
```

This would **delete your entire table**!

**Solution**: Use parameterised queries with `?` placeholders:

```python
cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
```

The database driver automatically escapes dangerous characters, making it safe.

</details>

---

### 🛠️ Implementation Steps

We'll build this feature in 5 steps:

1. Create the database table structure
2. Write the database insertion function
3. Create the Flask route to handle GET and POST
4. Build the HTML form with validation
5. Style the form with CSS

---

### Step 1: Create the Database Table

**Learning Focus**: SQL table creation with constraints

**Real-World Analogy**: Think of a database table like a spreadsheet. Each column has a name and data type (text, number, etc.). Constraints are rules - like "no blank cells" or "no duplicate values in this column".

#### 🔍 Understanding the Table Structure

We need a table called `contact_list` with three columns:

| Column  | Data Type | Constraints      | Purpose                               |
| ------- | --------- | ---------------- | ------------------------------------- |
| `id`    | INTEGER   | PRIMARY KEY      | Unique identifier (auto-increments)   |
| `email` | TEXT      | NOT NULL, UNIQUE | User's email address (must be unique) |
| `name`  | TEXT      | NOT NULL         | User's name                           |

**Why UNIQUE on email?**

- Prevents the same email being added twice
- Enforces one subscription per email address
- Will be important for Extension 2 (error handling)!

#### 💻 Create the Table

**Option A: Using SQLite Command Line**

1. Open your terminal in VS Code
2. Navigate to the database directory:

   ```bash
   cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source/database
   ```

3. Open the SQLite database:

   ```bash
   sqlite3 data_source.db
   ```

4. Create the table:

   ```sql
   CREATE TABLE IF NOT EXISTS contact_list (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       email TEXT NOT NULL UNIQUE,
       name TEXT NOT NULL
   );
   ```

5. Verify the table was created:

   ```sql
   .tables
   ```

   You should see `contact_list` and `extension` listed.

6. Check the table structure:

   ```sql
   .schema contact_list
   ```

7. Exit SQLite:
   ```sql
   .quit
   ```

**Option B: Using Python**

If you prefer, create a file `database/setup_contact_table.py`:

```python
import sqlite3

# Connect to database
con = sqlite3.connect("data_source.db")
cur = con.cursor()

# Create table
cur.execute("""
    CREATE TABLE IF NOT EXISTS contact_list (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        email TEXT NOT NULL UNIQUE,
        name TEXT NOT NULL
    )
""")

# Commit changes
con.commit()
con.close()

print("✅ contact_list table created successfully!")
```

Run it:

```bash
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source/database
python3 setup_contact_table.py
```

#### ✅ Checkpoint: Test Your Table

Let's insert a test record to verify everything works:

```bash
sqlite3 data_source.db
```

```sql
-- Insert test data
INSERT INTO contact_list (email, name) VALUES ('test@example.com', 'Test User');

-- Retrieve it
SELECT * FROM contact_list;

-- Clean up (remove test data)
DELETE FROM contact_list WHERE email = 'test@example.com';
```

**Expected Output:**

```
1|test@example.com|Test User
```

**💡 Understanding Check**: What would happen if you tried to insert the same email twice?

<details>
<summary>Click to see the answer</summary>

You'd get an error!

```
Error: UNIQUE constraint failed: contact_list.email
```

This is **good**! The database is protecting data integrity. We'll handle this error gracefully in Extension 2.

</details>

---

### Step 2: Create the Database Function

**Learning Focus**: Writing secure database operations

Open `database_manager.py` and add this function:

```python
def insertContact(email, name):
    """
    Insert a new contact into the contact_list table.

    Args:
        email (str): Contact's email address
        name (str): Contact's name

    Returns:
        None

    Raises:
        sqlite3.IntegrityError: If email already exists (UNIQUE constraint)
    """
    con = sql.connect("database/data_source.db")
    cur = con.cursor()

    # Using parameterised query to prevent SQL injection
    cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))

    con.commit()
    con.close()
```

#### 🔍 Code Explanation

Let's break down what each line does:

```python
con = sql.connect("database/data_source.db")
```

- Opens a connection to your SQLite database file
- Think of this like opening a door to the database

```python
cur = con.cursor()
```

- Creates a cursor object - this is what actually runs SQL commands
- Think of the cursor as your "pointer" inside the database

```python
cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
```

- **The SQL statement**: `INSERT INTO contact_list (email, name) VALUES (?, ?)`
  - Tells database to add a new row to `contact_list` table
  - `?` are placeholders for safe parameter insertion
- **The parameters**: `(email, name)`
  - Python tuple with actual values
  - SQLite safely inserts these, escaping dangerous characters

**Why use `?` placeholders?**

- Prevents SQL injection attacks
- Database driver handles escaping special characters
- Keeps code readable and maintainable

```python
con.commit()
```

- Saves your changes to the database permanently
- Without this, changes are rolled back!

```python
con.close()
```

- Closes the database connection
- Frees up system resources
- Important for preventing connection leaks

**💡 Notice**: In Extension 3, we'll learn a better way to handle connections using context managers!

#### ✅ Checkpoint: Test the Function

Create a test file `test_insert.py` in the root directory:

```python
import database_manager as dbHandler

# Test inserting a contact
try:
    dbHandler.insertContact("test@example.com", "Test User")
    print("✅ Contact inserted successfully!")
except Exception as e:
    print(f"❌ Error: {e}")

# Verify it worked
import sqlite3
con = sqlite3.connect("database/data_source.db")
cur = con.cursor()
cur.execute("SELECT * FROM contact_list WHERE email = 'test@example.com'")
result = cur.fetchone()
print(f"Retrieved: {result}")
con.close()

# Clean up
dbHandler.deleteContact("test@example.com")  # You'll need to add this function
```

---

### Step 3: Create the Flask Route

**Learning Focus**: Handling GET and POST requests in the same route

Open `main.py` and add this route (or update if it already exists):

```python
@app.route('/add.html', methods=['POST', 'GET'])
def add():
    """
    Handle the contact form page.

    GET request: Display empty form
    POST request: Process form submission and display confirmation
    """
    if request.method == 'POST':
        # Form was submitted - extract data
        email = request.form['email']
        name = request.form['name']

        # Insert into database
        dbHandler.insertContact(email, name)

        # Render template with success message
        return render_template('/add.html', is_done=True)
    else:
        # GET request - display empty form
        return render_template('/add.html')
```

#### 🔍 Code Explanation

```python
@app.route('/add.html', methods=['POST', 'GET'])
```

- Decorator tells Flask: "This function handles requests to `/add.html`"
- `methods=['POST', 'GET']` means this route accepts both:
  - **GET**: When user visits `/add.html` (shows form)
  - **POST**: When user submits the form (processes data)

**Why allow both methods?**

- **First visit** (GET): User needs to see the form
- **Submission** (POST): User filled out form and clicked submit
- Same URL, different behaviour based on HTTP method!

```python
if request.method == 'POST':
```

- Check which HTTP method was used
- `request` is a Flask object containing details about the current request

```python
email = request.form['email']
name = request.form['name']
```

- `request.form` is a dictionary-like object with form data
- Keys match the `name` attributes in your HTML form
- Example: `<input name="email">` becomes `request.form['email']`

```python
return render_template('/add.html', is_done=True)
```

- Renders the template with a variable
- `is_done=True` tells Jinja2 to show success message
- Template can check: `{% if is_done %}` to conditionally display content

#### 💭 Understanding Check: HTTP Methods

**Question**: What would happen if you removed `'GET'` from the methods list?

<details>
<summary>Click to see the answer</summary>

```python
@app.route('/add.html', methods=['POST'])  # Only POST allowed
```

- User visiting `/add.html` would get **405 Method Not Allowed** error
- They'd never see the form!
- Flask would reject GET requests

**Lesson**: If your route needs to display a form AND process submissions, allow both GET and POST.

</details>

#### ✅ Checkpoint: Test the Route

1. Start your Flask app:

   ```bash
   python3 main.py
   ```

2. Open browser and visit: `http://localhost:5000/add.html`

3. Check the terminal - you should see:
   ```
   127.0.0.1 - - [timestamp] "GET /add.html HTTP/1.1" 200 -
   ```

This confirms your GET request worked!

---

### Step 4: Build the HTML Form

**Learning Focus**: HTML5 form structure and validation attributes

Create or update `templates/add.html`:

```html
{% extends 'layout.html' %} {% block title %}Subscribe to Newsletter{% endblock
%} {% block content %}
<div class="form-container">
  <h1>📬 Subscribe to Our Newsletter</h1>

  {% if is_done %}
  <!-- Success message (shown after submission) -->
  <div class="success-message">
    <h2>✅ Thank You for Subscribing!</h2>
    <p>We've received your subscription. You'll hear from us soon!</p>
    <a href="/" class="btn-home">← Back to Home</a>
  </div>
  {% else %}
  <!-- Form (shown on initial visit) -->
  <form method="POST" action="/add.html" class="contact-form">
    <div class="form-group">
      <label for="name">Your Name:</label>
      <input
        type="text"
        id="name"
        name="name"
        required
        minlength="2"
        maxlength="100"
        placeholder="Enter your full name"
        autocomplete="name"
      />
      <span class="form-hint">Required • 2-100 characters</span>
    </div>

    <div class="form-group">
      <label for="email">Email Address:</label>
      <input
        type="email"
        id="email"
        name="email"
        required
        pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
        placeholder="your.email@example.com"
        autocomplete="email"
      />
      <span class="form-hint">Required • Must be a valid email</span>
    </div>

    <button type="submit" class="btn-submit">Subscribe</button>
  </form>
  {% endif %}
</div>
{% endblock %}
```

#### 🔍 Code Explanation

**Jinja2 Template Inheritance:**

```html
{% extends 'layout.html' %}
```

- Inherits structure from `layout.html` (header, footer, styles)
- You only define the unique content for this page

**Conditional Rendering:**

```html
{% if is_done %}
<!-- Success message -->
{% else %}
<!-- Form -->
{% endif %}
```

- Shows different content based on `is_done` variable
- **First visit**: `is_done` is undefined (falsy) → Shows form
- **After submission**: Flask passes `is_done=True` → Shows success message

**Form Element:**

```html
<form method="POST" action="/add.html"></form>
```

- `method="POST"`: Sends data via HTTP POST (secure, not in URL)
- `action="/add.html"`: Where to send form data (same route)

**HTML5 Validation Attributes:**

```html
<input type="email" ← Browser validates email format required ← Can't submit if
empty pattern="..." ← Custom regex validation minlength="2" ← Minimum 2
characters maxlength="100" ← Maximum 100 characters placeholder="..." ← Grey
hint text autocomplete="email" ← Browser can autofill >
```

**Why use HTML5 validation?**

- ✅ Instant feedback (before server processes)
- ✅ Better user experience
- ✅ Reduces invalid submissions
- ⚠️ **Not a replacement for server-side validation!**
  - Users can bypass with browser DevTools
  - Always validate on server too (we'll add this later)

**Name Attribute (CRUCIAL):**

```html
<input name="email" />
```

- **This becomes the dictionary key** in `request.form`
- `request.form['email']` in Python matches `name="email"` in HTML
- Without `name`, data won't be submitted!

#### 💭 Understanding Check: Form Attributes

**Question**: What's the difference between `id` and `name` attributes?

<details>
<summary>Click to see the answer</summary>

```html
<input id="email" name="email" />
```

**`id` Attribute:**

- Used by JavaScript and CSS to target the element
- Used by `<label for="email">` to associate label with input
- Must be unique on the page
- **NOT sent to server**

**`name` Attribute:**

- Used by the form to identify data when submitting
- Becomes the key in `request.form` dictionary
- **Sent to server** with the input value
- Can have duplicates (e.g., checkboxes with same name)

**Both are important but serve different purposes!**

</details>

#### ✅ Checkpoint: Test HTML Validation

1. Visit `http://localhost:5000/add.html`

2. **Test 1**: Try submitting empty form

   - Expected: Browser shows "Please fill out this field"

3. **Test 2**: Enter invalid email (`test` without @)

   - Expected: Browser shows "Please include an '@' in the email address"

4. **Test 3**: Enter name with 1 character

   - Expected: Browser shows "Please lengthen this text to 2 characters or more"

5. **Test 4**: Fill out correctly and submit
   - Expected: See success message

---

### Step 5: Style the Form

**Learning Focus**: CSS for forms, user experience design

Add this to `static/css/style.css`:

```css
/* ============================================
   CONTACT FORM STYLES
   ============================================ */

.form-container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 2rem;
  background: var(--card-background, #ffffff);
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-container h1 {
  text-align: center;
  color: var(--primary-color, #2196f3);
  margin-bottom: 1.5rem;
  font-size: 1.8rem;
}

/* Success Message */
.success-message {
  text-align: center;
  padding: 2rem;
  animation: fadeIn 0.5s ease-in;
}

.success-message h2 {
  color: #4caf50;
  margin-bottom: 1rem;
}

.success-message p {
  color: #666;
  margin-bottom: 2rem;
  font-size: 1.1rem;
}

.btn-home {
  display: inline-block;
  padding: 0.75rem 2rem;
  background: var(--primary-color, #2196f3);
  color: white;
  text-decoration: none;
  border-radius: 6px;
  transition: all 0.3s ease;
}

.btn-home:hover {
  background: var(--primary-dark, #1976d2);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(33, 150, 243, 0.3);
}

/* Form Styling */
.contact-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-group label {
  font-weight: 600;
  color: var(--text-primary, #333);
  font-size: 1rem;
}

.form-group input {
  padding: 0.75rem;
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  font-size: 1rem;
  transition: all 0.3s ease;
  font-family: inherit;
}

.form-group input:focus {
  outline: none;
  border-color: var(--primary-color, #2196f3);
  box-shadow: 0 0 0 3px rgba(33, 150, 243, 0.1);
}

/* Validation States */
.form-group input:valid {
  border-color: #4caf50;
}

.form-group input:invalid:not(:placeholder-shown) {
  border-color: #f44336;
}

.form-hint {
  font-size: 0.85rem;
  color: #666;
  font-style: italic;
}

/* Submit Button */
.btn-submit {
  padding: 1rem 2rem;
  background: var(--primary-color, #2196f3);
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-top: 0.5rem;
}

.btn-submit:hover {
  background: var(--primary-dark, #1976d2);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(33, 150, 243, 0.3);
}

.btn-submit:active {
  transform: translateY(0);
}

/* Animations */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Responsive Design */
@media (max-width: 768px) {
  .form-container {
    margin: 1rem;
    padding: 1.5rem;
  }

  .form-container h1 {
    font-size: 1.5rem;
  }
}
```

#### 🔍 CSS Explanation

**CSS Variables:**

```css
background: var(--primary-color, #2196f3);
```

- Uses CSS custom properties defined in `layout.html` or main stylesheet
- Second value (`#2196F3`) is fallback if variable not defined
- Makes theme colours consistent across the app

**Validation States:**

```css
.form-group input:valid {
  border-color: #4caf50; /* Green border when valid */
}

.form-group input:invalid:not(:placeholder-shown) {
  border-color: #f44336; /* Red border when invalid */
}
```

- `:valid` and `:invalid` are pseudo-classes based on HTML5 validation
- `:not(:placeholder-shown)` means "only when user has typed something"
- Prevents red border on empty fields before user interacts

**Focus States:**

```css
.form-group input:focus {
  outline: none;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(33, 150, 243, 0.1);
}
```

- Removes default browser outline
- Adds custom blue border and subtle glow
- **Important for accessibility** - users can see which field is active

**Transitions:**

```css
transition: all 0.3s ease;
```

- Smooth animations when properties change
- Makes hover effects and focus states feel polished
- `0.3s` = 300 milliseconds (good for UI interactions)

#### ✅ Checkpoint: Visual Testing

1. Visit `http://localhost:5000/add.html`

2. **Test visual states:**

   - Click in name field → Should see blue focus ring
   - Type invalid name (1 char) → Should see red border
   - Type valid name (2+ chars) → Should see green border
   - Hover over submit button → Should lift up slightly

3. **Test responsiveness:**
   - Open browser DevTools (F12)
   - Toggle device toolbar (mobile view)
   - Form should adapt to smaller screens

**What the form should look like:**

![Screen capture of the contact form](/docs/README_resources/form_example.png "The styled contact form with validation")

---

### 🧪 Complete Testing Checklist

#### Functional Testing

- [ ] **Empty form submission**

  - Submit without filling fields
  - Expected: Browser validation prevents submission

- [ ] **Invalid email format**

  - Enter `notanemail` in email field
  - Expected: Browser shows "Please include '@'"

- [ ] **Short name**

  - Enter single character in name
  - Expected: Browser shows "Please lengthen..."

- [ ] **Valid submission**

  - Enter valid name and email
  - Click submit
  - Expected: See success message "Thank You for Subscribing!"

- [ ] **Database verification**

  ```bash
  cd database
  sqlite3 data_source.db
  SELECT * FROM contact_list;
  ```

  - Expected: See your submitted data

- [ ] **Multiple submissions**

  - Subscribe with different emails
  - Expected: All records saved

- [ ] **Duplicate email** (will add proper handling in Extension 2)
  - Try subscribing with same email twice
  - Expected: Server error (we'll fix this!)

#### Visual Testing

- [ ] Form displays centred on page
- [ ] Input fields have proper spacing
- [ ] Focus states show blue outline
- [ ] Valid inputs show green border
- [ ] Invalid inputs show red border
- [ ] Submit button changes on hover
- [ ] Success message displays after submission
- [ ] "Back to Home" link works

#### Accessibility Testing

- [ ] Tab through form with keyboard
- [ ] Each input should be reachable
- [ ] Submit button activates with Enter key
- [ ] Labels clearly describe inputs

---

### 🎓 Review Questions

Test your understanding before moving to Extension 2:

**Question 1: HTTP Methods**
Why does our route accept both GET and POST methods?

<details>
<summary>Show answer</summary>

**GET**: Used when user first visits `/add.html` to display the empty form.

**POST**: Used when user submits the form with data.

We need both because:

1. User needs to see the form (GET)
2. User needs to submit the form (POST)
3. Same URL handles both actions based on HTTP method
</details>

**Question 2: Form Security**
Why is it dangerous to build SQL queries with f-strings?

<details>
<summary>Show answer</summary>

**SQL Injection vulnerability!**

Bad:

```python
query = f"INSERT INTO contact_list VALUES ('{email}', '{name}')"
```

If someone enters: `test@example.com'; DROP TABLE contact_list; --`

The SQL becomes:

```sql
INSERT INTO contact_list VALUES ('test@example.com'; DROP TABLE contact_list; --', 'Name')
```

This deletes your entire table!

**Solution**: Use parameterised queries:

```python
cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
```

</details>

**Question 3: Validation**
We have HTML5 validation on the form. Do we still need server-side validation?

<details>
<summary>Show answer</summary>

**YES! Always validate on the server too.**

**Why?**

- Users can disable JavaScript
- Users can edit HTML with browser DevTools
- Malicious users can send requests directly (bypassing the form)
- HTML5 validation is for **user experience**, not security

**Best practice**:

- HTML5 validation = Better UX (instant feedback)
- Server validation = Security (can't be bypassed)
</details>

**Question 4: Database Connections**
Why do we call `con.close()` after every database operation?

<details>
<summary>Show answer</summary>

**To free up system resources!**

- Each open connection uses memory
- Databases have limited connection pools
- Unclosed connections cause "too many connections" errors
- It's like leaving doors open - wastes resources

**In Extension 3**, we'll learn about context managers that automatically close connections:

```python
with sql.connect("database/data_source.db") as con:
    # Connection automatically closes when done!
```

</details>

---

### 🔧 Common Issues and Solutions

#### Issue 1: "Table contact_list already exists"

**Symptom**: Error when creating table

**Cause**: Table was already created from earlier testing

**Solution**: This is fine! The `IF NOT EXISTS` clause prevents errors. If you want to start fresh:

```sql
DROP TABLE IF EXISTS contact_list;
-- Then create again
```

#### Issue 2: Form submits but shows 404 error

**Symptom**: Clicking submit leads to "Not Found" page

**Possible causes**:

1. **Route not defined**: Check `main.py` has `@app.route('/add.html')`
2. **Form action wrong**: Check `<form action="/add.html">`
3. **Flask not restarted**: Restart Flask after code changes

**Debug steps**:

```bash
# Check terminal for error messages
# Flask should show: "POST /add.html HTTP/1.1" 200
```

#### Issue 3: Success message doesn't appear

**Symptom**: Form submits but still shows form instead of success message

**Possible causes**:

1. **Missing `is_done` parameter**: Check `return render_template('/add.html', is_done=True)`
2. **Typo in template**: Check `{% if is_done %}` spelling exactly

**Debug**: Add print statement in route:

```python
if request.method == 'POST':
    print("Form submitted!")  # Should see this in terminal
    # ... rest of code
```

#### Issue 4: Data not saving to database

**Symptom**: Form submits successfully but no data in database

**Possible causes**:

1. **Missing `con.commit()`**: Changes aren't saved without commit
2. **Wrong database path**: Check `sql.connect("database/data_source.db")`
3. **Database file permissions**: Check file exists and is writable

**Debug**:

```python
def insertContact(email, name):
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
    con.commit()
    print(f"✅ Inserted: {email}, {name}")  # Should see this
    con.close()
```

#### Issue 5: CSS not loading

**Symptom**: Form looks unstyled

**Possible causes**:

1. **Wrong file path**: CSS should be in `static/css/style.css`
2. **Not linked in layout.html**: Check `<link>` tag exists
3. **Browser cache**: Hard refresh with Ctrl+Shift+R

**Debug**: Open DevTools → Network tab → Look for style.css

- Red = File not found (check path)
- Green = Loading successfully

---

### 📝 Git Checkpoint

Commit your changes:

```bash
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source

# Check what changed
git status

# Stage files
git add database/data_source.db  # (if it changed)
git add database_manager.py
git add main.py
git add templates/add.html
git add static/css/style.css

# Commit with descriptive message
git commit -m "feat: Add email subscription form with database integration

- Created contact_list table with UNIQUE email constraint
- Added insertContact() function with parameterised queries
- Implemented /add.html route with GET/POST handling
- Built HTML form with HTML5 validation
- Styled form with success message and responsive design
- Tested form submission and database insertion"

# Push to remote (if using GitHub)
git push origin main
```

---

### 🎯 What You've Learned

Congratulations! You've just built a complete full-stack feature. Here's what you now understand:

**Backend Skills:**

- ✅ Creating SQL tables with constraints (UNIQUE, NOT NULL)
- ✅ Writing secure database operations with parameterised queries
- ✅ Handling GET and POST requests in Flask routes
- ✅ Extracting form data from `request.form`
- ✅ Passing variables to Jinja2 templates

**Frontend Skills:**

- ✅ Building HTML forms with proper attributes
- ✅ Using HTML5 validation (required, pattern, type)
- ✅ Conditional rendering with Jinja2 (`{% if %}`)
- ✅ Styling forms with modern CSS
- ✅ Creating visual feedback for users

**Web Development Concepts:**

- ✅ The HTTP request-response cycle
- ✅ Difference between GET and POST methods
- ✅ Form submission workflow from browser to database
- ✅ SQL injection prevention
- ✅ User experience design (validation, success messages)

**Syllabus Alignment:**

- ✅ **Client-server architecture**: Form (client) → Flask (server) → Database
- ✅ **Database integration**: SQL INSERT with proper table structure
- ✅ **Data transmission**: Understanding POST request body vs GET parameters

---

### 🚀 Next Steps

Ready for Extension 2? You'll learn:

- Python exception handling with `try-except` blocks
- Gracefully handling duplicate email submissions
- Displaying error messages to users
- The `sqlite3.IntegrityError` exception

**Preview**: What happens when someone tries to subscribe with an email that's already in the database? Right now your app crashes! Let's fix that...

---

## Extension 2: Handle Duplicate Email Submissions

### 📚 Learning Context

**Syllabus Outcome**: _Apply programming skills to solve problems and error handle_

**What You'll Learn:**

- Python exception handling with `try-except` blocks
- Understanding SQLite UNIQUE constraint violations
- Displaying error messages to users
- Preventing application crashes from database errors
- Building robust, production-ready code

**Real-World Context:**
In Extension 1, we created a contact form with a UNIQUE constraint on the email column. But what happens when someone tries to subscribe twice with the same email? Right now, your app **crashes** with an ugly error page! Professional applications need to handle errors gracefully and show user-friendly messages.

### 🎯 Learning Objectives

By completing this extension, you will:

1. Understand Python's `try-except` exception handling
2. Catch specific database errors (`sqlite3.IntegrityError`)
3. Display conditional error messages in Jinja2 templates
4. Prevent application crashes from user input
5. Improve user experience with helpful feedback

### 💭 Before You Start - Understanding Check

Let's make sure you understand the problem we're solving.

**Question 1: What's an Exception?**
In Python, what happens when your code encounters an error?

<details>
<summary>Click to see the answer</summary>

An **exception** is Python's way of saying "Something went wrong, I can't continue!"

**Without exception handling:**

```python
result = 10 / 0  # ZeroDivisionError: division by zero
# Program crashes here! Code below never runs.
print("This never prints")
```

**With exception handling:**

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Can't divide by zero!")
    result = None
# Program continues running!
print("This prints")  # ✅ Runs fine
```

**Key concept**: Exceptions let us catch errors and decide how to handle them instead of crashing.

</details>

**Question 2: The UNIQUE Constraint**
Look at your `contact_list` table creation:

```sql
CREATE TABLE contact_list (
    id INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL
);
```

What does `UNIQUE` on the email column mean?

<details>
<summary>Click to see the answer</summary>

**UNIQUE means no two rows can have the same email address.**

**Example:**

```sql
-- First insert: ✅ Works fine
INSERT INTO contact_list (email, name) VALUES ('test@example.com', 'John');

-- Second insert with same email: ❌ Error!
INSERT INTO contact_list (email, name) VALUES ('test@example.com', 'Jane');
-- Error: UNIQUE constraint failed: contact_list.email
```

This is a **database-level enforcement** - SQLite itself prevents duplicates, not Python code.

</details>

**Question 3: Current Behaviour**
What happens right now when you try to submit the same email twice?

<details>
<summary>Click to see the answer</summary>

**The application crashes with an error page:**

```
sqlite3.IntegrityError: UNIQUE constraint failed: contact_list.email
```

**Why is this bad?**

- Scary technical error message for users
- Page looks broken (not professional)
- Users don't know what went wrong
- No way to recover without going back

**What we want:**

- "This email is already subscribed! Please use a different email."
- Form stays on screen
- User can try again with a different email
</details>

---

### 🛠️ Implementation Steps

We'll handle this error in 3 steps:

1. Wrap database call in `try-except` block
2. Catch `sqlite3.IntegrityError` specifically
3. Display error message in template

---

### Step 1: Update Database Function (Optional)

**Learning Focus**: Exception propagation

You have two options for where to handle the error:

**Option A: Let the exception bubble up (Recommended)**

Keep `insertContact()` as is - let it raise the exception:

```python
def insertContact(email, name):
    """
    Insert a new contact into the contact_list table.

    Raises:
        sqlite3.IntegrityError: If email already exists (UNIQUE constraint)
    """
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
    con.commit()
    con.close()
```

**Why this approach?**

- Database function stays simple and focused
- Flask route decides how to handle the error
- More flexible (different routes can handle differently)
- Follows "separation of concerns" principle

**Option B: Handle exception in database function**

```python
def insertContact(email, name):
    """
    Insert a new contact into the contact_list table.

    Returns:
        tuple: (success: bool, message: str)
    """
    try:
        con = sql.connect("database/data_source.db")
        cur = con.cursor()
        cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
        con.commit()
        con.close()
        return (True, "Contact added successfully")
    except sql.IntegrityError:
        return (False, "Email already exists")
```

**Why this approach?**

- Database errors handled close to where they occur
- Route just checks the return value
- Encapsulates database-specific logic

For this tutorial, we'll use **Option A** as it's more Pythonic and flexible.

---

### Step 2: Update Flask Route

**Learning Focus**: Exception handling in web routes

Update your `/add.html` route in `main.py`:

```python
import sqlite3  # Add this import at the top of main.py

@app.route('/add.html', methods=['POST', 'GET'])
def add():
    """
    Handle the contact form page.

    GET request: Display empty form
    POST request: Process form submission with error handling
    """
    if request.method == 'POST':
        # Extract form data
        email = request.form['email']
        name = request.form['name']

        # Try to insert into database
        try:
            dbHandler.insertContact(email, name)
            # Success! Show confirmation
            return render_template('/add.html', is_done=True)

        except sqlite3.IntegrityError:
            # Duplicate email - show error message
            error_message = "This email address is already subscribed. Please use a different email."
            return render_template('/add.html', error=error_message, email=email, name=name)

    else:
        # GET request - display empty form
        return render_template('/add.html')
```

#### 🔍 Code Explanation

**Import Statement:**

```python
import sqlite3
```

- We need to import `sqlite3` to catch `sqlite3.IntegrityError`
- This exception is specific to SQLite database operations

**Try-Except Structure:**

```python
try:
    dbHandler.insertContact(email, name)
    return render_template('/add.html', is_done=True)
except sqlite3.IntegrityError:
    error_message = "..."
    return render_template('/add.html', error=error_message, ...)
```

**How it works:**

1. **Try block**: Attempt the risky operation (database insert)
2. **If successful**: Execute `return render_template(..., is_done=True)`
3. **If `IntegrityError` occurs**: Jump to except block, skip rest of try
4. **Except block**: Handle the error gracefully

**Why catch `sqlite3.IntegrityError` specifically?**

```python
except sqlite3.IntegrityError:  # ✅ Specific - only catches this error
```

vs

```python
except Exception:  # ❌ Too broad - catches ALL errors
```

**Benefits of specific exceptions:**

- Only catches the error we expect (duplicate email)
- Other errors still crash (so we know about bugs)
- More precise error messages
- Better debugging

**Preserving Form Data:**

```python
return render_template('/add.html', error=error_message, email=email, name=name)
```

Why pass `email` and `name` back to the template?

- **User experience**: Don't make them re-type everything!
- Form repopulates with their data
- They just need to change the email

#### 💭 Understanding Check: Exception Flow

**Question**: What happens if the database file doesn't exist? Will our `except` block catch it?

<details>
<summary>Click to see the answer</summary>

**No! Our except block only catches `sqlite3.IntegrityError`.**

If the database file doesn't exist, you'd get:

```python
sqlite3.OperationalError: unable to open database file
```

This is a **different exception** and would crash the application.

**To catch multiple exceptions:**

```python
try:
    dbHandler.insertContact(email, name)
    return render_template('/add.html', is_done=True)
except sqlite3.IntegrityError:
    error_message = "Email already subscribed."
    return render_template('/add.html', error=error_message, email=email, name=name)
except sqlite3.OperationalError:
    error_message = "Database error. Please contact support."
    return render_template('/add.html', error=error_message)
```

**Best practice**: Only catch exceptions you can meaningfully handle!

</details>

---

### Step 3: Update Template to Display Errors

**Learning Focus**: Conditional rendering, form repopulation

Update `templates/add.html`:

```html
{% extends 'layout.html' %} {% block title %}Subscribe to Newsletter{% endblock
%} {% block content %}
<div class="form-container">
  <h1>📬 Subscribe to Our Newsletter</h1>

  {% if is_done %}
  <!-- Success message (after successful submission) -->
  <div class="success-message">
    <h2>✅ Thank You for Subscribing!</h2>
    <p>We've received your subscription. You'll hear from us soon!</p>
    <a href="/" class="btn-home">← Back to Home</a>
  </div>

  {% else %}
  <!-- Display error message if present -->
  {% if error %}
  <div class="error-message">
    <p>⚠️ {{ error }}</p>
  </div>
  {% endif %}

  <!-- Form (shown on initial visit or after error) -->
  <form method="POST" action="/add.html" class="contact-form">
    <div class="form-group">
      <label for="name">Your Name:</label>
      <input
        type="text"
        id="name"
        name="name"
        required
        minlength="2"
        maxlength="100"
        value="{{ name or '' }}"
        placeholder="Enter your full name"
        autocomplete="name"
      />
      <span class="form-hint">Required • 2-100 characters</span>
    </div>

    <div class="form-group">
      <label for="email">Email Address:</label>
      <input
        type="email"
        id="email"
        name="email"
        required
        pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
        value="{{ email or '' }}"
        placeholder="your.email@example.com"
        autocomplete="email"
      />
      <span class="form-hint">Required • Must be a valid email</span>
    </div>

    <button type="submit" class="btn-submit">Subscribe</button>
  </form>
  {% endif %}
</div>
{% endblock %}
```

#### 🔍 Code Explanation

**Error Message Display:**

```html
{% if error %}
<div class="error-message">
  <p>⚠️ {{ error }}</p>
</div>
{% endif %}
```

- Only shows if `error` variable is passed from Flask
- `{{ error }}` outputs the error message text
- Warning emoji makes it visually distinct

**Form Repopulation:**

```html
<input type="text" name="name" value="{{ name or '' }}" />
<input type="email" name="email" value="{{ email or '' }}" />
```

**How `value` works:**

- **First visit**: `name` is undefined → `{{ name or '' }}` → Empty string → Blank field
- **After error**: `name` is `'John'` → `{{ name or '' }}` → `'John'` → Prefilled!

**The `or` operator in Jinja2:**

```
{{ variable or default_value }}
```

- If `variable` exists and is truthy → use it
- If `variable` is None/undefined → use `default_value`

**Why use `or ''` instead of just `{{ name }}`?**

- Prevents "undefined variable" errors in Jinja2
- Cleaner HTML output (empty string vs "None")

---

### Step 4: Style the Error Message

**Learning Focus**: Error state styling, visual feedback

Add this to `static/css/style.css`:

```css
/* ============================================
   ERROR MESSAGE STYLES
   ============================================ */

.error-message {
  background: #ffebee;
  border-left: 4px solid #f44336;
  padding: 1rem 1.5rem;
  margin-bottom: 1.5rem;
  border-radius: 6px;
  animation: shake 0.5s ease-in-out;
}

.error-message p {
  color: #c62828;
  margin: 0;
  font-weight: 500;
  font-size: 1rem;
}

/* Shake animation for errors */
@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  10%,
  30%,
  50%,
  70%,
  90% {
    transform: translateX(-5px);
  }
  20%,
  40%,
  60%,
  80% {
    transform: translateX(5px);
  }
}

/* Error state for inputs (optional enhancement) */
.form-group input.error {
  border-color: #f44336;
  background: #ffebee;
}
```

#### 🔍 CSS Explanation

**Error Colour Scheme:**

```css
background: #ffebee; /* Light red background */
border-left: 4px solid #f44336; /* Red accent border */
color: #c62828; /* Dark red text */
```

These colours follow Material Design error states:

- Red indicates something went wrong
- Light background isn't too alarming
- Strong left border draws attention

**Shake Animation:**

```css
animation: shake 0.5s ease-in-out;

@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  10%,
  30%,
  50%,
  70%,
  90% {
    transform: translateX(-5px);
  }
  20%,
  40%,
  60%,
  80% {
    transform: translateX(5px);
  }
}
```

**How it works:**

- Error message shakes left-right when it appears
- Grabs user's attention
- Duration: 0.5 seconds (half a second)
- `ease-in-out` makes it smooth

**Percentages in keyframes:**

- `0%`: Start position (no movement)
- `10%`: Move left 5px
- `20%`: Move right 5px
- ... continues shaking ...
- `100%`: Back to original position

---

### 🧪 Complete Testing Checklist

#### Functional Testing

**Test 1: First Subscription (Success Path)**

- [ ] Fill out form with new email (`test1@example.com`)
- [ ] Submit form
- [ ] Expected: Success message appears
- [ ] Verify database:
  ```bash
  sqlite3 database/data_source.db
  SELECT * FROM contact_list WHERE email = 'test1@example.com';
  ```

**Test 2: Duplicate Submission (Error Path)**

- [ ] Go back to form (visit `/add.html` again)
- [ ] Enter the SAME email (`test1@example.com`)
- [ ] Submit form
- [ ] Expected:
  - ⚠️ Error message: "This email address is already subscribed..."
  - Form still visible (not success page)
  - Name field still has your name (repopulated)
  - Email field still has email (repopulated)

**Test 3: Fix and Resubmit**

- [ ] After seeing error, change email to `test2@example.com`
- [ ] Submit again
- [ ] Expected: Success message (new email works!)

**Test 4: Application Doesn't Crash**

- [ ] Try duplicate email multiple times
- [ ] Expected: Error message each time, no server crash
- [ ] Check terminal - Flask should show:
  ```
  127.0.0.1 - - [timestamp] "POST /add.html HTTP/1.1" 200 -
  ```
  (200 = Success, even though showing error to user)

#### Visual Testing

- [ ] Error message has red color scheme
- [ ] Error message shakes when it appears
- [ ] Error message is easy to read
- [ ] Form fields retain user's data after error
- [ ] Layout doesn't break with error message

#### Edge Cases

**Test 5: Case Sensitivity**

- [ ] Subscribe with `Test@Example.com`
- [ ] Try again with `test@example.com` (lowercase)
- [ ] Expected: ??? (SQLite treats these as different emails)

**Test 6: Whitespace**

- [ ] Subscribe with `test@example.com` (no spaces)
- [ ] Try again with `test@example.com` (spaces before/after)
- [ ] Expected: Both saved as different emails

**Note**: These edge cases show limitations! In a production app, you'd want to:

- Normalize emails to lowercase before saving
- Strip whitespace with `email.strip().lower()`
- We'll address this in the "Review Questions" section

---

### 🎓 Review Questions

**Question 1: Exception Handling Order**
What's wrong with this code?

```python
try:
    dbHandler.insertContact(email, name)
    return render_template('/add.html', is_done=True)
except Exception:  # Catches ALL exceptions
    error_message = "Email already subscribed."
    return render_template('/add.html', error=error_message)
except sqlite3.IntegrityError:  # Never reached!
    error_message = "Duplicate email."
    return render_template('/add.html', error=error_message)
```

<details>
<summary>Show answer</summary>

**Problem**: The first `except` block catches ALL exceptions, so the second one is never reached!

**Why?**

- `Exception` is the base class for all exceptions
- It catches `IntegrityError`, `ValueError`, `KeyError`, everything!
- More specific handlers below it are unreachable (dead code)

**Correct order** (specific before general):

```python
try:
    # Code
except sqlite3.IntegrityError:  # Most specific
    # Handle duplicate email
except sqlite3.OperationalError:  # Specific
    # Handle database errors
except Exception:  # Most general (last)
    # Handle anything else
```

**Rule**: Always put more specific exceptions first!

</details>

**Question 2: Form Repopulation**
Why don't we repopulate the form when there's no error?

```python
# After successful submission, we don't pass name/email:
return render_template('/add.html', is_done=True)

# After error, we DO pass name/email:
return render_template('/add.html', error=error_message, email=email, name=name)
```

<details>
<summary>Show answer</summary>

**Because we're showing different content!**

**Success path** (`is_done=True`):

- Template shows success message
- Form is hidden completely
- No need for field values (form doesn't exist in DOM)

**Error path** (`error=...`):

- Template shows form again
- User shouldn't have to retype everything
- We pass back their data to repopulate fields

**User experience benefit**: Only retype the problematic field (email), not everything!

</details>

**Question 3: Data Normalization**
Currently, these are treated as **different** emails:

- `Test@Example.com`
- `test@example.com`
- `test@example.com`

How would you fix this?

<details>
<summary>Show answer</summary>

**Normalize email before saving:**

```python
if request.method == 'POST':
    # Extract and normalize
    email = request.form['email'].strip().lower()
    name = request.form['name'].strip()

    try:
        dbHandler.insertContact(email, name)
        return render_template('/add.html', is_done=True)
    except sqlite3.IntegrityError:
        error_message = "This email address is already subscribed."
        return render_template('/add.html', error=error_message, email=email, name=name)
```

**What `.strip().lower()` does:**

- `.strip()` removes leading/trailing whitespace
- `.lower()` converts to lowercase
- `" Test@Example.COM "` → `"test@example.com"`

**Best practice**: Always normalize user input before saving!

</details>

**Question 4: Multiple Except Blocks**
Can you catch multiple exception types in one except block?

<details>
<summary>Show answer</summary>

**Yes! Use a tuple:**

```python
try:
    dbHandler.insertContact(email, name)
    return render_template('/add.html', is_done=True)

except (sqlite3.IntegrityError, sqlite3.OperationalError) as e:
    # Catches either exception type
    error_message = "Database error. Please try again."
    return render_template('/add.html', error=error_message)
```

**When to use this:**

- When you want the same handling for multiple exceptions
- When exceptions should show the same error to users
- Keeps code DRY (Don't Repeat Yourself)

**When NOT to use:**

- When different exceptions need different messages
- When you want to log them differently
- When handling should differ
</details>

---

### 🔧 Common Issues and Solutions

#### Issue 1: Exception not caught, app still crashes

**Symptom**: Still seeing `IntegrityError` crash page

**Possible causes**:

1. **Forgot to import sqlite3**: Add `import sqlite3` at top of `main.py`
2. **Typo in exception name**: Check spelling `sqlite3.IntegrityError`
3. **Indentation error**: Ensure `try-except` is properly indented

**Debug**:

```python
try:
    print("About to insert...")  # Should see this
    dbHandler.insertContact(email, name)
    print("Insert succeeded!")  # See this on success
except sqlite3.IntegrityError as e:
    print(f"Caught exception: {e}")  # See this on duplicate
```

#### Issue 2: Error message doesn't appear in template

**Symptom**: Form just reloads, no red error box

**Possible causes**:

1. **Template not updated**: Check `{% if error %}` block exists
2. **Variable name mismatch**: Flask passes `error=...`, template checks `error`
3. **CSS not loading**: Check browser DevTools → Console for errors

**Debug in template**:

```html
<!-- Add this temporarily to see what variables exist -->
<p>Debug: error = {{ error }}</p>
<p>Debug: email = {{ email }}</p>
<p>Debug: name = {{ name }}</p>
```

#### Issue 3: Form fields don't repopulate

**Symptom**: After error, fields are blank

**Possible causes**:

1. **Missing value attribute**: Check `value="{{ name or '' }}"`
2. **Not passing data from Flask**: Check `render_template(..., email=email, name=name)`
3. **Quotes incorrect**: Make sure using `{{}}` not `{{}` or `{{}}`

**Verify in template**:

```html
<input type="text" name="name" value="{{ name or '' }}" />
<!-- View page source - should see: value="John" if name was passed -->
```

#### Issue 4: Application crashes on other errors

**Symptom**: Works for duplicates, but crashes on other issues

**Cause**: Only catching `IntegrityError`, not other exceptions

**Solution**: Add general exception handler:

```python
try:
    dbHandler.insertContact(email, name)
    return render_template('/add.html', is_done=True)
except sqlite3.IntegrityError:
    error_message = "This email is already subscribed."
    return render_template('/add.html', error=error_message, email=email, name=name)
except Exception as e:
    # Catches any other unexpected errors
    print(f"Unexpected error: {e}")  # Log to terminal
    error_message = "Something went wrong. Please try again later."
    return render_template('/add.html', error=error_message)
```

**When to use general `Exception` handler:**

- As a "catch-all" for unexpected errors
- Always put it LAST (after specific handlers)
- Log the actual error for debugging
- Show generic message to users (don't expose technical details)

---

### 📝 Git Checkpoint

Commit your changes:

```bash
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source

# Check what changed
git status

# Stage files
git add main.py
git add templates/add.html
git add static/css/style.css

# Commit with descriptive message
git commit -m "feat: Add exception handling for duplicate email submissions

- Wrapped insertContact() call in try-except block
- Catch sqlite3.IntegrityError for UNIQUE constraint violations
- Display user-friendly error message in red alert box
- Repopulate form fields after error to improve UX
- Added shake animation for error message
- Prevent application crashes from duplicate submissions"

# View your commit
git log -1 --stat
```

---

### 🎯 What You've Learned

Congratulations! You've made your application production-ready by handling errors gracefully.

**Python Skills:**

- ✅ Using `try-except` blocks for exception handling
- ✅ Catching specific exceptions (`sqlite3.IntegrityError`)
- ✅ Understanding exception propagation
- ✅ Exception handling best practices

**Web Development Skills:**

- ✅ Displaying error messages to users
- ✅ Repopulating forms after errors
- ✅ Conditional rendering based on error state
- ✅ Preventing application crashes

**User Experience Skills:**

- ✅ User-friendly error messages
- ✅ Visual feedback (colors, animations)
- ✅ Preserving user's data after errors
- ✅ Guiding users to fix problems

**Software Engineering Principles:**

- ✅ Defensive programming (expecting things to go wrong)
- ✅ Failing gracefully instead of crashing
- ✅ Separation of concerns (database vs route logic)
- ✅ Input validation and normalization

**Syllabus Alignment:**

- ✅ **Error handling**: Try-except blocks, exception types
- ✅ **Database constraints**: UNIQUE constraint enforcement
- ✅ **User experience**: Error feedback, form repopulation

---

### 🚀 Next Steps

**Enhancement Ideas:**

1. **Email normalization**: Add `.strip().lower()` to prevent duplicate case variations
2. **Better error detection**: Check if email exists BEFORE inserting (avoid exception)
3. **Success with existing**: If email exists, show "You're already subscribed!" success message
4. **Logging**: Log errors to a file for debugging
5. **Rate limiting**: Prevent spam submissions from same IP

**Code Quality**: Ready for Extension 3?
Extension 3 teaches you about **context managers** to automatically close database connections. This improves our `insertContact()` function and prevents resource leaks!

---

## Extension 3: Refactor Database Calls with Context Managers

### Learning Objectives

- Understand Python context managers and the `with` statement
- Learn about automatic resource management
- Practice refactoring existing code for better reliability
- Understand why closing database connections matters

### Understanding Context Managers

**What is a Context Manager?**

A context manager is a Python feature that helps manage resources (like files and database connections) automatically. It ensures resources are properly cleaned up, even if errors occur.

**The Problem with Current Code:**

Look at your current `database_manager.py`:

```python
def listExtension():
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    data = cur.execute("SELECT * FROM extension").fetchall()
    con.close()  # What if an error happens before this line?
    return data
```

**Understanding the Issue:**

- If an error occurs between `connect()` and `close()`, the connection stays open
- Multiple unclosed connections can slow down or crash your application
- This is called a "resource leak"

**The Solution: Context Managers**

Python's `with` statement automatically closes connections, even if errors occur:

```python
with sql.connect("database/data_source.db") as con:
    # Connection automatically closes when this block ends
    # Even if an error happens!
```

### Why This Matters

**Real-World Analogy:**
Think about borrowing a library book:

- **Without context manager**: You might forget to return it if something distracts you
- **With context manager**: The library system automatically checks it back in when you're done

**In Web Development:**

- Unclosed database connections waste server memory
- Too many open connections can prevent new users from accessing your site
- Context managers make your code more reliable and professional

### Step-by-Step Refactoring

> [!Note]
> Before making changes, let's commit your current working code to Git so you can always return to it if needed.

#### 1. Commit Your Current Code

```bash
# Check what files have changed
git status

# Add all your files to staging
git add .

# Commit with a descriptive message
git commit -m "Working PWA before context manager refactoring"

# Verify your commit
git log --oneline -3
```

**Understanding Git Commands:**

- `git status` - Shows which files have changed
- `git add .` - Stages all changed files for commit
- `git commit -m "message"` - Saves a snapshot with a description
- `git log --oneline -3` - Shows last 3 commits in short format

#### 2. Understand the Current Database Functions

```bash
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source
code database_manager.py
```

**Understanding Check Questions:**

- "How many functions are in this file?"
- "Which function reads from the database?"
- "Which function writes to the database?"
- "Where do we call `con.close()`?"
- "What happens if an error occurs before `con.close()`?"

#### 3. Refactor the `listExtension()` Function

**Before (Current Code):**

```python
def listExtension():
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    data = cur.execute("SELECT * FROM extension").fetchall()
    con.close()
    return data
```

**After (With Context Manager):**

Replace the entire `listExtension()` function with:

```python
def listExtension():
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        data = cur.execute("SELECT * FROM extension").fetchall()
    return data
```

**What Changed?**

1. **Removed manual connection**: No more `con = sql.connect()`
2. **Added `with` statement**: `with sql.connect() as con:`
3. **Removed `con.close()`**: Happens automatically now
4. **Indented code block**: Everything under `with` is managed

**Why It's Better:**

- Connection closes automatically after the `with` block
- Works even if `execute()` raises an error
- Less code to write and maintain
- More Pythonic (follows Python best practices)

#### 4. Refactor the `insertContact()` Function

**Before (Current Code):**

```python
def insertContact(email, name):
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    cur.execute("INSERT INTO contact_list (email,name) VALUES (?,?)", (email, name))
    con.commit()
    con.close()
```

**After (With Context Manager):**

Replace the entire `insertContact()` function with:

```python
def insertContact(email, name):
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        cur.execute("INSERT INTO contact_list (email,name) VALUES (?,?)", (email, name))
        con.commit()
```

**Understanding the `commit()` Call:**

- `commit()` saves changes to the database
- Still needed with context managers for INSERT, UPDATE, DELETE
- The context manager closes the connection, but doesn't commit automatically
- Think of it like "Save" vs "Close" in a document

#### 5. Test Your Refactored Code

```bash
# Start your Flask application
python3 main.py
```

**Manual Testing Checklist:**

1. **Test Reading Data:**

   - Open browser to `http://localhost:5000`
   - Do the extension cards still display?
   - Check browser console for errors (F12)

2. **Test Writing Data:**
   - Navigate to `http://localhost:5000/add.html`
   - Submit a new contact
   - Does the success message appear?
   - Check the database to verify the insert

```bash
# Verify database was updated (in a new terminal)
sqlite3 database/data_source.db "SELECT * FROM contact_list;"
```

#### 6. Commit Your Refactored Code

```bash
# Stop Flask (Ctrl+C in terminal)

# Check what changed
git diff database_manager.py

# Stage the changes
git add database_manager.py

# Commit with descriptive message
git commit -m "Refactor database calls to use context managers for better resource management"

# View your commit history
git log --oneline -5
```

**Understanding `git diff`:**

- Shows line-by-line changes
- Lines with `-` were removed (old code)
- Lines with `+` were added (new code)
- Helps you review changes before committing

#### ✅ Checkpoint: Context Manager Refactoring Complete

**Verification Steps:**

```bash
# 1. Check your code compiles without errors
python3 -m py_compile database_manager.py

# 2. Verify Flask starts successfully
python3 main.py
# Look for: * Running on all addresses (0.0.0.0)

# 3. Test in browser
# - Visit http://localhost:5000 (should see extensions)
# - Visit http://localhost:5000/add.html (should be able to add contact)

# 4. Check git status
git status
# Should show: nothing to commit, working tree clean
```

### Understanding Check: Key Concepts

Before moving to the next extension, ensure you can answer:

1. **What is a context manager?**

   - A Python feature that automatically manages resources

2. **Why use `with` instead of manual `close()`?**

   - Guarantees cleanup even if errors occur
   - Less code to write
   - Prevents resource leaks

3. **Does `with` automatically commit database changes?**

   - No! You still need `con.commit()` for INSERT/UPDATE/DELETE
   - `with` only handles opening and closing the connection

4. **What happens if an error occurs inside a `with` block?**
   - The connection still closes automatically
   - The error is raised (you can catch it with try/except if needed)

### Further Learning: Advanced Context Manager Usage

**Optional Extension: Add Error Handling**

For more robust code, you can combine context managers with exception handling:

```python
def insertContact(email, name):
    try:
        with sql.connect("database/data_source.db") as con:
            cur = con.cursor()
            cur.execute("INSERT INTO contact_list (email,name) VALUES (?,?)", (email, name))
            con.commit()
    except sql.IntegrityError:
        # Handle duplicate email error
        raise ValueError(f"Email {email} already exists in database")
    except sql.Error as e:
        # Handle other database errors
        raise RuntimeError(f"Database error: {e}")
```

**Discussion Questions:**

- "When might the `IntegrityError` exception occur?"
- "Why is it helpful to convert database errors to more descriptive errors?"
- "How would you display these errors to the user in the Flask app?"

### Common Pitfalls

⚠️ **Mistake 1: Forgetting Indentation**

```python
# WRONG - data access after connection closes
with sql.connect("database/data_source.db") as con:
    cur = con.cursor()
data = cur.execute("SELECT * FROM extension").fetchall()  # ERROR!
```

```python
# CORRECT - all database operations inside with block
with sql.connect("database/data_source.db") as con:
    cur = con.cursor()
    data = cur.execute("SELECT * FROM extension").fetchall()
```

⚠️ **Mistake 2: Returning Inside the `with` Block**

```python
# WORKS BUT NOT IDEAL
def listExtension():
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        return cur.execute("SELECT * FROM extension").fetchall()
    # Connection closes immediately after return
```

```python
# BETTER - Store data first, return after
def listExtension():
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        data = cur.execute("SELECT * FROM extension").fetchall()
    return data  # Connection already closed, data is safe
```

⚠️ **Mistake 3: Forgetting `commit()` for Write Operations**

```python
# WRONG - Changes not saved!
def insertContact(email, name):
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        cur.execute("INSERT INTO contact_list (email,name) VALUES (?,?)", (email, name))
    # Missing con.commit()!
```

### Review Questions

Test your understanding:

1. What resources other than databases could benefit from context managers?

   - Files (`with open('file.txt') as f:`)
   - Network connections
   - Locks and semaphores

2. Can you use multiple context managers at once?

   ```python
   # Yes! This is valid Python
   with sql.connect("db1.db") as con1, sql.connect("db2.db") as con2:
       # Use both connections
   ```

3. How would you test that connections are actually closing?
   - Monitor with `lsof` command (Linux/Mac)
   - Use SQLite's `.databases` command
   - Check for "database is locked" errors (indicates unclosed connections)

---

## Extension 4: Add Language Filtering to the Extensions Catalogue

### Learning Objectives

- Understand URL query parameters and how to pass data via GET requests
- Learn how to modify SQL queries with WHERE clauses for filtering
- Practice conditional rendering in Jinja2 templates
- Understand the request-response cycle with parameters

### Understanding Filtering in Web Applications

**What is Filtering?**

Filtering allows users to narrow down displayed data based on specific criteria. Think of it like using the "Category" filter on an online shopping site - you see only items matching your selection.

**Real-World Analogy:**

Imagine a library with different sections:

- **Without filtering**: You see all books on all shelves
- **With filtering**: You press a button for "Science Fiction" and only those books light up

**In Our Application:**

Currently, the index page shows ALL VSCode extensions. We'll add filter buttons so users can view:

- All extensions
- Only Python extensions
- Only #BASH extensions
- Only SQL extensions
- etc.

### How URL Parameters Work

**Understanding Query Strings:**

When you click a link like `/?language=Python`, the browser sends:

- **Base URL**: `/` (the index page)
- **Query parameter**: `?language=Python` (the filter criteria)

Flask can read this parameter and modify what data to display.

**Example URLs:**

- `http://localhost:5000/` - Show all extensions
- `http://localhost:5000/?language=Python` - Show only Python extensions
- `http://localhost:5000/?language=%23BASH` - Show only #BASH extensions (# is encoded as %23)

### Step-by-Step Implementation

> [!Note]
> This extension assumes you've completed Extension 1 (context managers). If not, you can still complete this extension with the original database code.

#### 1. Commit Your Current Code

```bash
# Ensure you're in the project root
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source

# Check current status
git status

# Add and commit all changes
git add .
git commit -m "Working PWA before adding language filtering feature"

# Verify commit
git log --oneline -3
```

#### 2. Add a New Database Function for Filtered Queries

```bash
code database_manager.py
```

**Understanding the Change:**

We need a new function that queries the database with a WHERE clause to filter by language. This keeps our code organized - one function for all data, another for filtered data.

Add this new function to the end of `database_manager.py`:

```python
def listExtensionByLanguage(language):
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        data = cur.execute("SELECT * FROM extension WHERE language = ?", (language,)).fetchall()
    return data
```

**Understanding the SQL Query:**

- `SELECT * FROM extension` - Get all columns from the extension table
- `WHERE language = ?` - Only rows where the language matches
- `(language,)` - The parameter value (the comma makes it a tuple, required by SQLite)
- `.fetchall()` - Get all matching rows

**Why Use `?` Placeholder?**

```python
# DANGEROUS - SQL Injection vulnerability!
query = f"SELECT * FROM extension WHERE language = '{language}'"

# SAFE - Parameterised query prevents SQL injection
query = "SELECT * FROM extension WHERE language = ?"
cur.execute(query, (language,))
```

The `?` placeholder ensures user input is safely escaped, preventing SQL injection attacks.

#### 3. Update the Flask Route to Handle Filter Parameters

```bash
code main.py
```

**Current Code (Before):**

```python
@app.route("/index.html", methods=["GET"])
@app.route("/", methods=["POST", "GET"])
def index():
    return render_template("/index.html", content=dbHandler.listExtension())
```

**Understanding What We Need to Change:**

1. Check if a `language` parameter was sent in the URL
2. If yes, use the filtered query
3. If no, show all extensions (current behaviour)

**New Code (After):**

Replace the entire `index()` function with:

```python
@app.route("/index.html", methods=["GET"])
@app.route("/", methods=["POST", "GET"])
def index():
    language = request.args.get('language')
    if language:
        data = dbHandler.listExtensionByLanguage(language)
    else:
        data = dbHandler.listExtension()
    return render_template("/index.html", content=data)
```

**Understanding the Code:**

- `request.args.get('language')` - Gets the value of `?language=` from the URL
- Returns `None` if the parameter doesn't exist
- `if language:` - If a language was specified, use filtered data
- `else:` - If no language specified, show all extensions
- `content=data` - Pass the data to the template (same variable name as before)

**Why This Works:**

- Visiting `/` - No parameter, `language` is `None`, shows all extensions
- Visiting `/?language=Python` - Parameter exists, `language` is `"Python"`, shows filtered results

#### 4. Add Filter Buttons to the Template

```bash
cd templates
code index.html
```

**Current Code (Before):**

```html
{% extends 'layout.html' %} {% block content %}
<div class="container">
  {% for row in content %}
  <div class="card">
    <!-- card content -->
  </div>
  {% endfor %}
</div>
{% endblock %}
```

**New Code (After):**

Replace the entire content block with:

```html
{% extends 'layout.html' %} {% block content %}
<div class="filter-buttons">
  <a href="/" class="filter-btn">All</a>
  <a href="/?language=HTML CSS JS" class="filter-btn">HTML CSS JS</a>
  <a href="/?language=%23BASH" class="filter-btn">#BASH</a>
  <a href="/?language=SQL" class="filter-btn">SQL</a>
</div>

<div class="container">
  {% for row in content %}
  <div class="card">
    <img
      class="card-image"
      src="{{ row[4] }}"
      alt="Product image for the {{ row[1] }} VSCode extension."
    />
    <h1 class="card-name">{{ row[1] }}</h1>
    <p class="card-about">{{ row[3] }}</p>
    <a class="card-link" href="{{ row[2] }}"
      ><button class="btn">Read More</button></a
    >
  </div>
  {% endfor %}
</div>
{% endblock %}
```

**Understanding the Changes:**

1. **New `<div class="filter-buttons">` section** - Contains all filter links
2. **`href="/"`** - Link to show all extensions (no parameter)
3. **`href="/?language=HTML CSS JS"`** - Link with language parameter
4. **`href="/?language=%23BASH"`** - The `#` character is URL-encoded as `%23`
5. **`class="filter-btn"`** - CSS class for styling (we'll add this next)

**Why URL Encode `#`?**

The `#` character has special meaning in URLs (it marks page anchors), so we encode it as `%23`.

#### 5. Style the Filter Buttons

```bash
cd ../static/css
code style.css
```

Add this CSS to the end of your `style.css` file:

```css
.filter-buttons {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
  margin: 1.5rem 0;
  flex-wrap: wrap;
}

.filter-btn {
  text-decoration: none;
  padding: 0.5rem 1.5rem;
  background: hsl(177, 82%, 48%);
  color: #fff;
  border-radius: 5px;
  font-weight: 600;
  transition: all 0.25s ease-in-out;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.filter-btn:hover {
  background: hsl(177, 82%, 38%);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.filter-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

**Understanding the CSS:**

- `.filter-buttons` - Container using flexbox for horizontal layout
- `gap: 0.5rem` - Space between buttons
- `flex-wrap: wrap` - Buttons wrap to next line on small screens
- `.filter-btn` - Button styling with cyan/teal colour matching the theme
- `:hover` - Visual feedback when mouse hovers over button
- `transform: translateY()` - Subtle lift effect on hover

#### 6. Test Your Filtering Feature

```bash
# Start Flask application
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source
python3 main.py
```

**Manual Testing Checklist:**

1. **Test "All" filter:**

   - Open `http://localhost:5000/`
   - Should see all 4 extensions
   - Click "All" button - should still see all extensions

2. **Test specific language filters:**

   - Click "HTML CSS JS" - Should see only 1 extension (Live Server)
   - Click "#BASH" - Should see 2 extensions (Render CR LF, Start GIT BASH)
   - Click "SQL" - Should see 1 extension (SQLite3 Editor)

3. **Test URL directly:**

   - Type `http://localhost:5000/?language=Python` in browser
   - Should show no results (no Python extensions in sample data)
   - This is correct behaviour - empty results for no matches

4. **Check browser console:**
   - Press F12 → Console tab
   - Should see no JavaScript errors

**Understanding Empty Results:**

If a filter shows no cards, that's normal - it means no extensions match that language. To test this works correctly:

```bash
# Add a Python extension to your database
sqlite3 database/data_source.db
```

Then run this SQL:

```sql
INSERT INTO extension(extID,name,hyperlink,about,image,language) VALUES (5,"Python","https://marketplace.visualstudio.com/items?itemName=ms-python.python","Python language support","https://ms-python.gallerycdn.vsassets.io/extensions/ms-python/python/2024.0.0/Microsoft.VisualStudio.Services.Icons.Default","Python");
```

Exit SQLite (Ctrl+D) and refresh the browser. Now test the Python filter!

#### 7. Commit Your Changes

```bash
# Stop Flask (Ctrl+C)

# Check what files changed
git status

# View the specific changes
git diff database_manager.py
git diff main.py
git diff templates/index.html
git diff static/css/style.css

# Stage all changes
git add database_manager.py main.py templates/index.html static/css/style.css

# Commit with descriptive message
git commit -m "Add language filtering feature with query parameters"

# View commit history
git log --oneline -5
```

#### ✅ Checkpoint: Language Filtering Complete

**Verification Steps:**

```bash
# 1. Verify all files were updated
git log -1 --stat
# Should show: database_manager.py, main.py, index.html, style.css

# 2. Test Flask application starts
python3 main.py
# Should start without errors

# 3. Test all filter buttons work
# - Open browser to http://localhost:5000
# - Click each filter button
# - Verify correct extensions display

# 4. Test URL parameters directly
# - Visit http://localhost:5000/?language=SQL
# - Should see filtered results
```

### Understanding Check: Key Concepts

Before moving to the next extension, ensure you can answer:

1. **What is a URL query parameter?**

   - Data passed in the URL after `?`
   - Format: `?key=value`
   - Multiple parameters: `?key1=value1&key2=value2`

2. **How does Flask access query parameters?**

   - `request.args.get('parameter_name')`
   - Returns the value if present, `None` if not

3. **Why use parameterised SQL queries?**

   - Prevents SQL injection attacks
   - Safely handles special characters
   - Professional security practice

4. **What happens if no extensions match the filter?**
   - The SQL query returns an empty list `[]`
   - The template's `{% for %}` loop runs zero times
   - No cards are displayed (empty result, not an error)

### Enhancing the Feature: Optional Improvements

**Optional Extension A: Show Active Filter**

Highlight which filter is currently active by modifying the template:

```html
{% extends 'layout.html' %} {% block content %}
<div class="filter-buttons">
  <a
    href="/"
    class="filter-btn {% if not request.args.get('language') %}active{% endif %}"
    >All</a
  >
  <a
    href="/?language=HTML CSS JS"
    class="filter-btn {% if request.args.get('language') == 'HTML CSS JS' %}active{% endif %}"
    >HTML CSS JS</a
  >
  <a
    href="/?language=%23BASH"
    class="filter-btn {% if request.args.get('language') == '#BASH' %}active{% endif %}"
    >#BASH</a
  >
  <a
    href="/?language=SQL"
    class="filter-btn {% if request.args.get('language') == 'SQL' %}active{% endif %}"
    >SQL</a
  >
</div>
<!-- rest of template -->
{% endblock %}
```

Add this CSS:

```css
.filter-btn.active {
  background: hsl(177, 82%, 28%);
  font-weight: 700;
}
```

**Optional Extension B: Display Filter Status**

Show users what filter is active:

```html
{% extends 'layout.html' %} {% block content %}
<div class="filter-buttons">
  <!-- filter buttons -->
</div>

{% if request.args.get('language') %}
<p class="filter-status">
  Showing: {{ request.args.get('language') }} extensions
</p>
{% else %}
<p class="filter-status">Showing: All extensions</p>
{% endif %}

<div class="container">
  <!-- cards -->
</div>
{% endblock %}
```

Add this CSS:

```css
.filter-status {
  text-align: center;
  color: #666;
  font-style: italic;
  margin: 0.5rem 0;
}
```

### Common Pitfalls

⚠️ **Mistake 1: Forgetting to URL Encode Special Characters**

```html
<!-- WRONG - # breaks the URL -->
<a href="/?language=#BASH">
  <!-- CORRECT - # encoded as %23 -->
  <a href="/?language=%23BASH"></a
></a>
```

⚠️ **Mistake 2: SQL Injection Vulnerability**

```python
# DANGEROUS - Never do this!
def listExtensionByLanguage(language):
    query = f"SELECT * FROM extension WHERE language = '{language}'"
    cur.execute(query)

# SAFE - Use parameterised queries
def listExtensionByLanguage(language):
    cur.execute("SELECT * FROM extension WHERE language = ?", (language,))
```

⚠️ **Mistake 3: Forgetting the Comma in Single-Item Tuple**

```python
# WRONG - This is a string in parentheses, not a tuple
cur.execute("SELECT * FROM extension WHERE language = ?", (language))

# CORRECT - Comma makes it a tuple
cur.execute("SELECT * FROM extension WHERE language = ?", (language,))
```

⚠️ **Mistake 4: Case Sensitivity in SQL Comparisons**

SQLite's `=` operator is case-sensitive by default. If your data has mixed case:

```python
# Won't match "python" or "PYTHON"
WHERE language = 'Python'

# Better - case-insensitive comparison
WHERE LOWER(language) = LOWER(?)
```

### Debugging Tips

**Problem: Filter shows no results but should show data**

```bash
# Check what's actually in your database
sqlite3 database/data_source.db
SELECT DISTINCT language FROM extension;
# Copy the exact language strings for your filter buttons
```

**Problem: URL parameter not being read**

```python
# Add debugging to your Flask route
@app.route("/", methods=["POST", "GET"])
def index():
    language = request.args.get('language')
    print(f"DEBUG: language parameter = {language}")  # Check terminal output
    # rest of code
```

**Problem: Special characters in URLs not working**

```python
# Python automatically decodes URL-encoded characters
# %23BASH becomes #BASH
language = request.args.get('language')
print(f"DEBUG: Decoded language = {language}")  # Will show: #BASH
```

### Review Questions

Test your understanding:

1. **How would you add a filter for multiple languages at once?**

   - Modify SQL: `WHERE language IN (?, ?, ?)`
   - Pass multiple parameters: `cur.execute(query, (lang1, lang2, lang3))`

2. **How would you implement search instead of exact filtering?**

   - Use SQL `LIKE` operator: `WHERE name LIKE ?`
   - Pass wildcard pattern: `(f"%{search_term}%",)`

3. **Could you use POST instead of GET for filtering?**

   - Yes, but GET is better for filtering because:
   - URLs can be bookmarked
   - Users can share filtered links
   - Browser back/forward buttons work correctly

4. **How would you paginate results (show 10 at a time)?**
   - Add LIMIT and OFFSET to SQL: `LIMIT ? OFFSET ?`
   - Add page parameter: `/?language=Python&page=2`
   - Calculate offset: `(page - 1) * items_per_page`

---

## Extension 5: Add Online/Offline Status Detection

### Learning Objectives

- Understand browser online/offline events
- Learn how to detect network connectivity in JavaScript
- Practice displaying user feedback based on network status
- Understand PWA offline capabilities

### Understanding Online/Offline Detection

**What is Network Detection?**

Network detection allows your PWA to respond to changes in internet connectivity. When users go offline (lose internet connection), your app can display a message and still show cached content.

**Real-World Analogy:**

Think of a mobile phone:

- **Online**: Full signal bars - calls and data work normally
- **Offline**: No signal - phone shows "No Service" but you can still access photos, contacts, and cached data

**In Our PWA:**

Currently, if users lose internet connection:

- Cached pages still work (thanks to service worker)
- But users don't know they're offline
- New data requests fail silently

We'll add a simple notification banner that appears when offline.

### How Browser Online/Offline Events Work

**JavaScript provides built-in events:**

1. `navigator.onLine` - Boolean property (true = online, false = offline)
2. `online` event - Fires when connection is restored
3. `offline` event - Fires when connection is lost

**Example Flow:**

```
User has internet → navigator.onLine = true
WiFi disconnects → 'offline' event fires → Show message
WiFi reconnects → 'online' event fires → Hide message
```

### Step-by-Step Implementation

> [!Note]
> This extension modifies only frontend files (HTML, CSS, and JavaScript). No backend or database changes needed!

#### 1. Commit Your Current Code

```bash
# Ensure you're in the project root
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source

# Check current status
git status

# Add and commit all changes
git add .
git commit -m "Working PWA before adding online/offline detection"

# Verify commit
git log --oneline -3
```

#### 2. Add the Offline Message Banner to HTML

```bash
cd templates
code layout.html
```

**Understanding What We're Adding:**

We'll add a hidden banner at the top of every page. JavaScript will show/hide it based on network status.

**Current Code (Before):**

```html
<body>
  <main>
    {% include "partials/menu.html" %} {% block content %}{% endblock %}
  </main>
  <script src="static/js/app.js"></script>
</body>
```

**New Code (After):**

Replace the entire `<body>` section with:

```html
<body>
  <div id="offline-banner" class="offline-banner" style="display: none;">
    <p>⚠️ You are currently offline. Showing cached content.</p>
  </div>

  <main>
    {% include "partials/menu.html" %} {% block content %}{% endblock %}
  </main>
  <script src="static/js/app.js"></script>
</body>
```

**Understanding the Changes:**

1. **`<div id="offline-banner">`** - Container for the offline message
2. **`style="display: none;"`** - Hidden by default (inline style for immediate hiding)
3. **`⚠️ You are currently offline`** - Clear message to users
4. **`Showing cached content`** - Explains why app still works
5. Positioned before `<main>` so it appears at the very top of the page

#### 3. Style the Offline Banner

```bash
cd ../static/css
code style.css
```

Add this CSS to the end of your `style.css` file:

```css
/* Offline status banner */
.offline-banner {
  background: #ff9800;
  color: #fff;
  text-align: center;
  padding: 0.75rem;
  font-weight: 600;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  position: sticky;
  top: 0;
  z-index: 1000;
  animation: slideDown 0.3s ease-out;
}

.offline-banner p {
  margin: 0;
  font-size: 0.95rem;
}

@keyframes slideDown {
  from {
    transform: translateY(-100%);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
```

**Understanding the CSS:**

- `background: #ff9800` - Orange warning colour (attention-grabbing but not alarming)
- `position: sticky; top: 0` - Banner stays at top when scrolling
- `z-index: 1000` - Appears above all other content
- `animation: slideDown` - Smooth slide-in effect when appearing
- `@keyframes slideDown` - Defines the animation (slides from top)

#### 4. Add JavaScript Online/Offline Detection

```bash
cd ../js
code app.js
```

**Current Code (Before):**

```js
if ("serviceworker" in navigator) {
  window.addEventListener("load", function () {
    navigator.serviceworker
      .register("static/js/serviceworker.js")
      .then((res) => console.log("service worker registered"))
      .catch((err) => console.log("service worker not registered", err));
  });
}
```

**New Code (After):**

Replace the entire file with:

```js
// Register service worker
if ("serviceWorker" in navigator) {
  window.addEventListener("load", function () {
    navigator.serviceWorker
      .register("static/js/serviceworker.js")
      .then((res) => console.log("service worker registered"))
      .catch((err) => console.log("service worker not registered", err));
  });
}

// Online/Offline detection
const offlineBanner = document.getElementById("offline-banner");

function updateOnlineStatus() {
  if (navigator.onLine) {
    // User is online
    offlineBanner.style.display = "none";
    console.log("App is online");
  } else {
    // User is offline
    offlineBanner.style.display = "block";
    console.log("App is offline");
  }
}

// Check status when page loads
window.addEventListener("load", updateOnlineStatus);

// Listen for online event
window.addEventListener("online", function () {
  updateOnlineStatus();
  console.log("Connection restored");
});

// Listen for offline event
window.addEventListener("offline", function () {
  updateOnlineStatus();
  console.log("Connection lost");
});
```

**Understanding the JavaScript:**

**1. Get the banner element:**

```js
const offlineBanner = document.getElementById("offline-banner");
```

- Stores reference to the banner so we can show/hide it

**2. Check online status function:**

```js
function updateOnlineStatus() {
  if (navigator.onLine) {
    offlineBanner.style.display = "none"; // Hide banner
  } else {
    offlineBanner.style.display = "block"; // Show banner
  }
}
```

- `navigator.onLine` - Browser's current online status (true/false)
- Changes banner visibility based on status

**3. Event listeners:**

```js
window.addEventListener("load", updateOnlineStatus);
window.addEventListener("online", updateOnlineStatus);
window.addEventListener("offline", updateOnlineStatus);
```

- `load` - Check status when page first loads
- `online` - React when connection is restored
- `offline` - React when connection is lost

**Why This Works:**

The browser automatically detects network changes and fires the `online`/`offline` events. Our code simply listens for these events and updates the UI accordingly.

#### 5. Test Online/Offline Detection

```bash
# Start Flask application
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source
python3 main.py
```

**Manual Testing - Method 1: Chrome DevTools**

1. Open `http://localhost:5000` in Chrome
2. Press `F12` to open DevTools
3. Go to the **Network** tab
4. Find the dropdown that says "No throttling"
5. Select **Offline**
6. The orange banner should appear: "⚠️ You are currently offline"
7. Change back to **No throttling**
8. The banner should disappear
9. Check Console tab for log messages: "App is online/offline"

**Manual Testing - Method 2: Disable WiFi/Network**

1. Open `http://localhost:5000` in browser
2. Turn off your WiFi or disconnect ethernet
3. Banner should appear automatically
4. Reconnect WiFi/network
5. Banner should disappear automatically

**Manual Testing - Method 3: Service Worker Offline Test**

1. Open `http://localhost:5000`
2. Navigate through a few pages (index, add page)
3. Set Network to **Offline** in DevTools
4. Try navigating between pages you've already visited
5. Pages should still load (from cache)
6. Banner should show you're offline
7. Try submitting the contact form - it will fail (no network)

**What You Should See:**

```
✅ Online: No banner, pages load normally from server
✅ Offline: Orange banner appears, cached pages still work
✅ Transition: Banner smoothly slides in/out when status changes
```

#### 6. Verify Service Worker Caching Still Works

The offline banner tells users they're offline, but the service worker should still serve cached pages.

**Test Offline Caching:**

```bash
# With Flask running, open browser to http://localhost:5000
# Let the page load completely
# Open DevTools → Application → Service Workers
# Verify service worker is "activated and running"
# Go to DevTools → Network → Set to "Offline"
# Refresh the page
# Page should still load (from cache)
# Offline banner should display
```

**Understanding the Interaction:**

1. **Service Worker** - Caches files and serves them when offline
2. **Online/Offline Detection** - Tells users they're viewing cached content
3. **Together** - Users understand why app works offline and what to expect

#### 7. Commit Your Changes

```bash
# Stop Flask (Ctrl+C)

# Check what files changed
git status

# View the specific changes
git diff templates/layout.html
git diff static/css/style.css
git diff static/js/app.js

# Stage all changes
git add templates/layout.html static/css/style.css static/js/app.js

# Commit with descriptive message
git commit -m "Add online/offline status detection with user notification banner"

# View commit history
git log --oneline -5
```

#### ✅ Checkpoint: Online/Offline Detection Complete

**Verification Steps:**

```bash
# 1. Verify all files were updated
git log -1 --stat
# Should show: layout.html, style.css, app.js

# 2. Test Flask application starts
python3 main.py
# Should start without errors

# 3. Test online/offline detection
# - Open DevTools → Network → Set to Offline
# - Banner should appear
# - Set to No throttling
# - Banner should disappear

# 4. Test cached content works offline
# - Load page while online
# - Go offline in DevTools
# - Refresh page
# - Should still see content + offline banner
```

### Understanding Check: Key Concepts

Before moving forward, ensure you can answer:

1. **What does `navigator.onLine` tell us?**

   - Returns `true` if browser has network connectivity
   - Returns `false` if browser is offline
   - Note: Doesn't guarantee internet access, just network connection

2. **What's the difference between `online` event and `navigator.onLine`?**

   - `navigator.onLine` - Current status (check any time)
   - `online` event - Notification when status changes to online
   - `offline` event - Notification when status changes to offline

3. **Why do we need both service worker AND online/offline detection?**

   - **Service Worker** - Makes offline functionality work (caching)
   - **Online/Offline Detection** - Tells users what's happening (UX)
   - Together they provide great offline experience

4. **What happens if user tries to submit a form while offline?**
   - Form submission will fail (no network to reach server)
   - Service worker can't cache POST requests by default
   - Need additional code to queue form submissions (advanced topic)

### Enhancing the Feature: Optional Improvements

**Optional Enhancement A: Add Online Status Message**

Show a brief "You're back online!" message when connection restores:

Modify the JavaScript in `app.js`:

```js
// Listen for online event
window.addEventListener("online", function () {
  offlineBanner.style.display = "none";

  // Show temporary "back online" message
  const tempBanner = document.createElement("div");
  tempBanner.className = "online-banner";
  tempBanner.innerHTML = "<p>✅ You're back online!</p>";
  document.body.prepend(tempBanner);

  // Remove message after 3 seconds
  setTimeout(() => {
    tempBanner.remove();
  }, 3000);

  console.log("Connection restored");
});
```

Add this CSS:

```css
.online-banner {
  background: #4caf50;
  color: #fff;
  text-align: center;
  padding: 0.75rem;
  font-weight: 600;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1001;
  animation: slideDown 0.3s ease-out;
}

.online-banner p {
  margin: 0;
  font-size: 0.95rem;
}
```

**Optional Enhancement B: Change Visual Style When Offline**

Grey out content when offline to emphasize cached state:

Add this CSS:

```css
body.offline {
  filter: grayscale(20%);
}

body.offline main {
  opacity: 0.9;
}
```

Modify JavaScript in `app.js`:

```js
function updateOnlineStatus() {
  if (navigator.onLine) {
    offlineBanner.style.display = "none";
    document.body.classList.remove("offline");
  } else {
    offlineBanner.style.display = "block";
    document.body.classList.add("offline");
  }
}
```

**Optional Enhancement C: Add Retry Button**

Let users manually trigger a reconnection check:

Modify the banner in `layout.html`:

```html
<div id="offline-banner" class="offline-banner" style="display: none;">
  <p>
    ⚠️ You are currently offline. Showing cached content.
    <button id="retry-connection" class="retry-btn">Retry Connection</button>
  </p>
</div>
```

Add JavaScript in `app.js`:

```js
// Add this after the event listeners
document
  .getElementById("retry-connection")
  .addEventListener("click", function () {
    // Force a check by trying to fetch a small resource
    fetch("/", { method: "HEAD", cache: "no-store" })
      .then(() => {
        console.log("Connection test successful");
        updateOnlineStatus();
      })
      .catch(() => {
        console.log("Still offline");
        alert("Still no connection. Please check your network settings.");
      });
  });
```

Add CSS:

```css
.retry-btn {
  background: #fff;
  color: #ff9800;
  border: none;
  padding: 0.25rem 0.75rem;
  margin-left: 0.5rem;
  border-radius: 3px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.retry-btn:hover {
  background: #f5f5f5;
}
```

### Common Pitfalls

⚠️ **Mistake 1: Trusting `navigator.onLine` Too Much**

```js
// navigator.onLine only checks network interface, not actual internet
if (navigator.onLine) {
  // This doesn't guarantee server is reachable!
  // Could be connected to WiFi with no internet
}
```

**Better approach:** Combine with actual fetch attempts for critical operations.

⚠️ **Mistake 2: Not Testing Offline Behavior**

```bash
# Always test offline functionality:
# - In DevTools offline mode
# - With actual network disconnection
# - After clearing cache
# - On different devices
```

⚠️ **Mistake 3: Forgetting Inline Style Takes Precedence**

```html
<!-- Inline style: display: none -->
<div id="offline-banner" style="display: none;">
  <!-- JavaScript must use inline style to override -->
  offlineBanner.style.display = "block"; // ✅ Works

  <!-- CSS class won't override inline style -->
  .offline-banner.visible { display: block; } // ❌ Won't work
</div>
```

⚠️ **Mistake 4: Service Worker Typo**

```js
// WRONG - Typo in service worker name
if ("serviceworker" in navigator) {  // No capital W

// CORRECT - Proper camelCase
if ("serviceWorker" in navigator) {  // Capital W
```

### Debugging Tips

**Problem: Banner doesn't appear when going offline**

```js
// Add console logs to diagnose
function updateOnlineStatus() {
  console.log("Checking status...");
  console.log("navigator.onLine:", navigator.onLine);
  console.log("Banner element:", offlineBanner);

  if (navigator.onLine) {
    console.log("Setting display to none");
    offlineBanner.style.display = "none";
  } else {
    console.log("Setting display to block");
    offlineBanner.style.display = "block";
  }
}
```

**Problem: Events not firing**

```js
// Test if events are registered
window.addEventListener("offline", function () {
  console.log("OFFLINE EVENT FIRED!");
  alert("Offline event detected"); // Should show alert
});

window.addEventListener("online", function () {
  console.log("ONLINE EVENT FIRED!");
  alert("Online event detected"); // Should show alert
});
```

**Problem: Service worker not caching properly**

```bash
# Check service worker status
# DevTools → Application → Service Workers
# Look for errors or "Waiting to activate" status

# Clear cache and re-register
# DevTools → Application → Storage → Clear site data
# Refresh page
# Check if service worker installs successfully
```

### Real-World Use Cases

**When Online/Offline Detection Matters:**

1. **Social Media Apps**

   - Show users they can still view cached posts
   - Prevent posting/commenting when offline
   - Queue actions to sync when online

2. **Note-Taking Apps**

   - Indicate when changes are saved locally vs synced
   - Show sync status in UI
   - Warn before losing unsaved work

3. **E-commerce Sites**

   - Prevent checkout when offline
   - Allow browsing cached products
   - Show availability may not be current

4. **Educational Apps**
   - Let students access downloaded course materials
   - Indicate when live features unavailable
   - Queue quiz submissions for later sync

### Review Questions

Test your understanding:

1. **Why show an offline message if the service worker makes the app work offline?**

   - Users need to know they're viewing cached content
   - Sets expectations (e.g., can't submit forms)
   - Professional UX - transparency builds trust

2. **What's the difference between offline detection and service worker?**

   - **Offline detection** - Tells users about network status
   - **Service worker** - Actually makes offline functionality work
   - Both needed for complete PWA experience

3. **How would you test that service worker caching works?**

   - Load page while online (populates cache)
   - Go offline in DevTools
   - Refresh page - should load from cache
   - Check DevTools → Application → Cache Storage

4. **What other browser APIs could enhance offline experience?**
   - **IndexedDB** - Store data locally
   - **Background Sync** - Queue actions to sync when online
   - **Push Notifications** - Alert users when back online
   - **Local Storage** - Simple key-value storage

---

## Extension 6: Add App Installation Prompt

### Learning Objectives

- Understand PWA installation and the `beforeinstallprompt` event
- Learn how to trigger app installation programmatically
- Practice handling browser install prompts
- Understand installability criteria for PWAs

### Understanding PWA Installation

**What is PWA Installation?**

PWAs can be installed on a user's device just like native apps from an app store. Once installed:

- App appears on home screen/desktop with your custom icon
- Opens in standalone window (no browser UI)
- Launches faster from home screen
- Works offline via service worker

**Real-World Analogy:**

Think of bookmarking vs installing:

- **Bookmark**: Quick link in browser - still opens in browser tab
- **Installed PWA**: Full app on device - opens like a native app

**Current Behavior:**

Your PWA is already installable! Browsers show install prompts automatically when criteria are met. But we can make it more obvious by adding our own "Install App" button.

### How Browser Installation Works

**Installability Criteria:**

Browsers check these requirements before allowing installation:

1. ✅ Valid `manifest.json` with icons (you have this!)
2. ✅ Service worker registered (you have this!)
3. ✅ Served over HTTPS or localhost (development uses localhost)
4. ✅ User has engaged with the site

**The `beforeinstallprompt` Event:**

When a PWA meets installation criteria, the browser fires this event. We can:

1. Capture the event
2. Prevent the automatic browser prompt
3. Show our own custom install button
4. Trigger installation when user clicks our button

### Step-by-Step Implementation

> [!Note]
> This extension only modifies frontend files (HTML, CSS, JavaScript). No backend changes needed!

#### 1. Commit Your Current Code

```bash
# Ensure you're in the project root
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source

# Check current status
git status

# Add and commit all changes
git add .
git commit -m "Working PWA before adding installation prompt feature"

# Verify commit
git log --oneline -3
```

#### 2. Add Install Button to HTML

```bash
cd templates
code layout.html
```

**Understanding What We're Adding:**

A simple button that appears only when the app is installable. The button will be hidden by default and shown by JavaScript when appropriate.

**Current Code (Before):**

```html
<body>
  <div id="offline-banner" class="offline-banner" style="display: none;">
    <p>⚠️ You are currently offline. Showing cached content.</p>
  </div>

  <main>
    {% include "partials/menu.html" %} {% block content %}{% endblock %}
  </main>
  <script src="static/js/app.js"></script>
</body>
```

**New Code (After):**

Replace the entire `<body>` section with:

```html
<body>
  <div id="offline-banner" class="offline-banner" style="display: none;">
    <p>⚠️ You are currently offline. Showing cached content.</p>
  </div>

  <button id="install-button" class="install-button" style="display: none;">
    📱 Install App
  </button>

  <main>
    {% include "partials/menu.html" %} {% block content %}{% endblock %}
  </main>
  <script src="static/js/app.js"></script>
</body>
```

**Understanding the Changes:**

1. **`<button id="install-button">`** - The installation trigger button
2. **`style="display: none;"`** - Hidden by default (shown by JavaScript when installable)
3. **`📱 Install App`** - Clear call-to-action with icon
4. Positioned after offline banner but before main content

#### 3. Style the Install Button

```bash
cd ../static/css
code style.css
```

Add this CSS to the end of your `style.css` file:

```css
/* Install App button */
.install-button {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: #14e6dd;
  color: #106d69;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 25px;
  font-weight: 700;
  font-size: 1rem;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(20, 230, 221, 0.4);
  transition: all 0.3s ease;
  z-index: 999;
  animation: pulse 2s infinite;
}

.install-button:hover {
  background: #106d69;
  color: #14e6dd;
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(20, 230, 221, 0.6);
}

.install-button:active {
  transform: translateY(0);
  box-shadow: 0 2px 8px rgba(20, 230, 221, 0.4);
}

@keyframes pulse {
  0%,
  100% {
    box-shadow: 0 4px 12px rgba(20, 230, 221, 0.4);
  }
  50% {
    box-shadow: 0 4px 20px rgba(20, 230, 221, 0.7);
  }
}
```

**Understanding the CSS:**

- `position: fixed; bottom: 20px; right: 20px` - Floating button in bottom-right corner
- `background: #14e6dd` - Cyan colour matching your app theme
- `border-radius: 25px` - Rounded pill shape
- `z-index: 999` - Appears above content (but below offline banner at 1000)
- `animation: pulse` - Gentle pulsing effect to draw attention
- `:hover` - Inverted colours on hover for feedback
- `box-shadow` - Elevated appearance (material design)

#### 4. Add Installation JavaScript

```bash
cd ../js
code app.js
```

**Current Code (Before):**

```js
// Register service worker
if ("serviceWorker" in navigator) {
  window.addEventListener("load", function () {
    navigator.serviceWorker
      .register("static/js/serviceworker.js")
      .then((res) => console.log("service worker registered"))
      .catch((err) => console.log("service worker not registered", err));
  });
}

// Online/Offline detection
const offlineBanner = document.getElementById("offline-banner");

function updateOnlineStatus() {
  if (navigator.onLine) {
    offlineBanner.style.display = "none";
    console.log("App is online");
  } else {
    offlineBanner.style.display = "block";
    console.log("App is offline");
  }
}

window.addEventListener("load", updateOnlineStatus);
window.addEventListener("online", function () {
  updateOnlineStatus();
  console.log("Connection restored");
});
window.addEventListener("offline", function () {
  updateOnlineStatus();
  console.log("Connection lost");
});
```

**New Code (After):**

Replace the entire file with:

```js
// Register service worker
if ("serviceWorker" in navigator) {
  window.addEventListener("load", function () {
    navigator.serviceWorker
      .register("static/js/serviceworker.js")
      .then((res) => console.log("service worker registered"))
      .catch((err) => console.log("service worker not registered", err));
  });
}

// Online/Offline detection
const offlineBanner = document.getElementById("offline-banner");

function updateOnlineStatus() {
  if (navigator.onLine) {
    offlineBanner.style.display = "none";
    console.log("App is online");
  } else {
    offlineBanner.style.display = "block";
    console.log("App is offline");
  }
}

window.addEventListener("load", updateOnlineStatus);
window.addEventListener("online", function () {
  updateOnlineStatus();
  console.log("Connection restored");
});
window.addEventListener("offline", function () {
  updateOnlineStatus();
  console.log("Connection lost");
});

// PWA Installation
let deferredPrompt;
const installButton = document.getElementById("install-button");

// Capture the install prompt event
window.addEventListener("beforeinstallprompt", (event) => {
  // Prevent the default browser install prompt
  event.preventDefault();

  // Store the event so we can trigger it later
  deferredPrompt = event;

  // Show our custom install button
  installButton.style.display = "block";

  console.log("App is installable - showing install button");
});

// Handle install button click
installButton.addEventListener("click", async () => {
  if (!deferredPrompt) {
    console.log("Install prompt not available");
    return;
  }

  // Show the install prompt
  deferredPrompt.prompt();

  // Wait for the user's response
  const { outcome } = await deferredPrompt.userChoice;

  console.log(`User response: ${outcome}`);

  if (outcome === "accepted") {
    console.log("User accepted the install prompt");
  } else {
    console.log("User dismissed the install prompt");
  }

  // Clear the deferred prompt
  deferredPrompt = null;

  // Hide the install button
  installButton.style.display = "none";
});

// Detect when app is successfully installed
window.addEventListener("appinstalled", () => {
  console.log("PWA was installed successfully");

  // Hide install button (app is now installed)
  installButton.style.display = "none";

  // Clear the deferred prompt
  deferredPrompt = null;
});
```

**Understanding the JavaScript:**

**1. Capture the install prompt:**

```js
let deferredPrompt; // Store the event for later

window.addEventListener("beforeinstallprompt", (event) => {
  event.preventDefault(); // Stop automatic browser prompt
  deferredPrompt = event; // Save event
  installButton.style.display = "block"; // Show our button
});
```

- Browser fires `beforeinstallprompt` when PWA is installable
- We prevent default behavior to control the timing
- Store the event to trigger installation later
- Show our custom button

**2. Handle button click:**

```js
installButton.addEventListener("click", async () => {
  deferredPrompt.prompt(); // Show browser's install dialog
  const { outcome } = await deferredPrompt.userChoice; // Wait for response

  if (outcome === "accepted") {
    // User installed the app
  }

  installButton.style.display = "none"; // Hide button
});
```

- When user clicks our button, trigger the browser's native install dialog
- Wait for user to accept or dismiss
- Hide button after interaction

**3. Detect successful installation:**

```js
window.addEventListener("appinstalled", () => {
  console.log("PWA was installed successfully");
  installButton.style.display = "none";
});
```

- Fires when app is actually installed
- Hide button since app is now installed

**Why This Approach?**

- Users have control over when to see the install prompt
- Better UX than random browser popups
- Button only appears when app is actually installable
- Follows PWA best practices

#### 5. Test the Installation Feature

```bash
# Start Flask application
cd /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source
python3 main.py
```

**Manual Testing Checklist:**

**Test 1: Check Installability Criteria**

1. Open `http://localhost:5000` in Chrome
2. Press `F12` to open DevTools
3. Go to **Application** tab
4. Click **Manifest** in left sidebar
5. Check if manifest loads correctly
6. Look for "Installability" section - should show green checkmarks
7. If there are issues, they'll be listed here

**Test 2: Verify Install Button Appears**

1. With the app open in Chrome
2. Look for the floating "📱 Install App" button in bottom-right corner
3. Button should have a pulsing glow effect
4. If button doesn't appear immediately:
   - Close and reopen the browser
   - Navigate between pages
   - Wait a few seconds (browser checks installability)

**Test 3: Install the PWA**

1. Click the "📱 Install App" button
2. Browser's native install dialog should appear
3. Click "Install" in the dialog
4. App should open in standalone window
5. Check your desktop/start menu - app icon should be there
6. Install button should disappear (app is now installed)

**Test 4: Test Installed App**

1. Close the installed app
2. Open it from desktop/start menu icon
3. Should open in standalone window (no browser UI)
4. Should work normally with all features
5. Offline functionality should still work

**Test 5: Console Logging**

```bash
# Open DevTools → Console tab
# You should see these messages:

"service worker registered"
"App is online"
"App is installable - showing install button"

# After clicking install button:
"User response: accepted" (or "dismissed")
"PWA was installed successfully"
```

**What You Should See:**

```
✅ Install button appears when PWA is installable
✅ Button has pulsing animation to draw attention
✅ Clicking button shows browser's install dialog
✅ After installation, button disappears
✅ App appears on desktop/start menu with custom icon
✅ Installed app opens in standalone window
```

#### 6. Test on Different Browsers

**Chrome/Edge (Chromium-based):**

- Full support for `beforeinstallprompt`
- Install button should work perfectly

**Firefox:**

- Limited PWA support on desktop (as of 2025)
- Better support on Android
- Install button may not appear (browser doesn't fire event)

**Safari:**

- iOS/macOS support "Add to Home Screen"
- Doesn't fire `beforeinstallprompt` event
- Install button won't appear (different installation flow)
- iOS users: Share button → Add to Home Screen

**Testing Browser Support:**

```js
// Add this temporarily to test browser support
if ("beforeinstallprompt" in window) {
  console.log("✅ Browser supports beforeinstallprompt");
} else {
  console.log("❌ Browser doesn't support beforeinstallprompt");
}
```

#### 7. Commit Your Changes

```bash
# Stop Flask (Ctrl+C)

# Check what files changed
git status

# View the specific changes
git diff templates/layout.html
git diff static/css/style.css
git diff static/js/app.js

# Stage all changes
git add templates/layout.html static/css/style.css static/js/app.js

# Commit with descriptive message
git commit -m "Add custom PWA installation prompt with floating button"

# View commit history
git log --oneline -5
```

#### ✅ Checkpoint: PWA Installation Complete

**Verification Steps:**

```bash
# 1. Verify all files were updated
git log -1 --stat
# Should show: layout.html, style.css, app.js

# 2. Test Flask application starts
python3 main.py
# Should start without errors

# 3. Test installation in Chrome
# - Open http://localhost:5000
# - Look for floating "Install App" button
# - Click button and install
# - Verify app appears on desktop
# - Open installed app - should work standalone

# 4. Check console for proper logging
# - Open DevTools → Console
# - Should see installation-related messages
# - No JavaScript errors
```

### Understanding Check: Key Concepts

Before moving forward, ensure you can answer:

1. **What is the `beforeinstallprompt` event?**

   - Fired by browser when PWA meets installation criteria
   - Allows developers to control when/how to prompt installation
   - Can be prevented and triggered later

2. **Why capture and defer the install prompt?**

   - Better UX - user chooses when to see prompt
   - Prevents intrusive automatic prompts
   - Can show custom UI instead of browser's default
   - Increases installation rate (user-initiated)

3. **What happens when user clicks the install button?**

   - Calls `deferredPrompt.prompt()` to show browser's install dialog
   - User chooses to install or dismiss
   - `appinstalled` event fires if user accepts
   - Button hides after interaction

4. **Why does the install button only appear sometimes?**
   - PWA must meet installability criteria
   - Browser must support `beforeinstallprompt`
   - App must not already be installed
   - User must engage with site first

### Enhancing the Feature: Optional Improvements

**Optional Enhancement A: Installation Instructions for Safari**

Help iOS/Safari users install the app:

Modify the install button in `layout.html`:

```html
<div id="install-container" style="display: none;">
  <button id="install-button" class="install-button">📱 Install App</button>
  <div id="ios-instructions" class="ios-instructions" style="display: none;">
    <p>To install on iOS:</p>
    <ol>
      <li>Tap the Share button <span style="font-size: 1.2rem;">⎋</span></li>
      <li>Select "Add to Home Screen"</li>
      <li>Tap "Add"</li>
    </ol>
  </div>
</div>
```

Add JavaScript to detect iOS:

```js
// Detect iOS devices
const isIOS = /iPhone|iPad|iPod/.test(navigator.userAgent);

if (isIOS && !window.navigator.standalone) {
  // User is on iOS and hasn't installed the app
  document.getElementById("install-container").style.display = "block";
  document.getElementById("ios-instructions").style.display = "block";
}
```

**Optional Enhancement B: Track Installation Analytics**

Log when users install or dismiss:

```js
installButton.addEventListener("click", async () => {
  deferredPrompt.prompt();
  const { outcome } = await deferredPrompt.userChoice;

  // Send to analytics
  if (outcome === "accepted") {
    console.log("Analytics: User installed PWA");
    // Could send to Google Analytics, etc.
  } else {
    console.log("Analytics: User dismissed install prompt");
  }

  installButton.style.display = "none";
});
```

**Optional Enhancement C: Show Install Success Message**

Confirm installation with visual feedback:

```js
window.addEventListener("appinstalled", () => {
  // Show temporary success message
  const successMessage = document.createElement("div");
  successMessage.className = "install-success";
  successMessage.innerHTML = "<p>✅ App installed successfully!</p>";
  document.body.appendChild(successMessage);

  // Remove after 3 seconds
  setTimeout(() => {
    successMessage.remove();
  }, 3000);

  installButton.style.display = "none";
  deferredPrompt = null;
});
```

Add CSS:

```css
.install-success {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: #4caf50;
  color: white;
  padding: 1rem 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  z-index: 1002;
  animation: slideDown 0.3s ease-out;
}
```

**Optional Enhancement D: Dismissible Install Button**

Let users hide the button if not interested:

```html
<button id="install-button" class="install-button" style="display: none;">
  📱 Install App
  <span class="install-close">×</span>
</button>
```

Add JavaScript:

```js
// Close button on install prompt
document.querySelector(".install-close")?.addEventListener("click", (event) => {
  event.stopPropagation(); // Don't trigger install
  installButton.style.display = "none";

  // Remember user dismissed (use localStorage)
  localStorage.setItem("installPromptDismissed", "true");
});

// Check if user previously dismissed
window.addEventListener("beforeinstallprompt", (event) => {
  const wasDismissed = localStorage.getItem("installPromptDismissed");

  if (wasDismissed === "true") {
    console.log("User previously dismissed install prompt");
    return; // Don't show button
  }

  event.preventDefault();
  deferredPrompt = event;
  installButton.style.display = "block";
});
```

### Common Pitfalls

⚠️ **Mistake 1: Testing Without HTTPS**

```bash
# WRONG - Testing on HTTP (not localhost)
http://192.168.1.100:5000  # Won't be installable

# CORRECT - Use localhost for development
http://localhost:5000      # Installable
https://yourdomain.com     # Installable in production
```

PWAs require HTTPS in production. Localhost is exempt for development.

⚠️ **Mistake 2: Expecting Button to Appear Immediately**

```js
// Browser needs time to check installability
// User must also engage with site first
// Button may not appear on first visit
```

**Solution:** Navigate around the site, wait a few seconds, then check.

⚠️ **Mistake 3: Not Handling Missing `beforeinstallprompt`**

```js
// WRONG - Assumes event will fire
installButton.addEventListener("click", async () => {
  deferredPrompt.prompt(); // Could be null!
});

// CORRECT - Check if prompt is available
installButton.addEventListener("click", async () => {
  if (!deferredPrompt) {
    console.log("Install prompt not available");
    return;
  }
  deferredPrompt.prompt();
});
```

⚠️ **Mistake 4: Forgetting Manifest Icon Paths**

```json
// WRONG - Absolute URL won't work locally
"src": "/static/icons/icon-512x512.png"

// CORRECT - Relative to manifest.json location
"src": "icons/icon-512x512.png"
```

### Debugging Tips

**Problem: Install button never appears**

```bash
# Check DevTools → Application → Manifest
# Look for errors in manifest.json
# Check "Installability" section for issues

# Common issues:
# - Manifest not loading
# - Icons missing
# - Service worker not registered
# - Not using localhost/HTTPS
```

**Problem: "beforeinstallprompt" event not firing**

```js
// Add debugging
let eventFired = false;

window.addEventListener("beforeinstallprompt", (event) => {
  eventFired = true;
  console.log("✅ beforeinstallprompt event fired!");
  // rest of code
});

// Check after a few seconds
setTimeout(() => {
  if (!eventFired) {
    console.log("❌ beforeinstallprompt never fired");
    console.log("Browser:", navigator.userAgent);
    console.log("Is HTTPS/localhost?", location.protocol);
  }
}, 5000);
```

**Problem: App installs but doesn't work correctly**

```bash
# Check service worker is active in installed app
# DevTools → Application → Service Workers
# Should show "activated and running"

# Check manifest scope
# Make sure start_url matches your app structure
```

**Problem: Icons not showing after install**

```bash
# Verify icon files exist:
ls -la static/icons/

# Check manifest icon paths are correct
# Should be relative to manifest.json location

# Try hard refresh after fixing:
# Uninstall app → Clear cache → Reinstall
```

### Real-World Considerations

**When to Show Install Prompt:**

1. **Good times:**

   - After user completes a task successfully
   - When user returns to app (shows value)
   - After user browses multiple pages
   - When user saves something important

2. **Bad times:**
   - Immediately on first visit
   - During critical user flow
   - Before user sees app value
   - Multiple times if dismissed

**Installation Best Practices:**

- Don't be pushy - respect user's choice
- Explain benefits of installing
- Make button dismissible
- Don't show again if user dismisses multiple times
- Consider user engagement level before prompting

**Measuring Success:**

Track these metrics:

- Install button shown (impressions)
- Install button clicked (engagement)
- Installation completed (conversion)
- Time from first visit to install
- User retention after installation

### Review Questions

Test your understanding:

1. **What's required for a PWA to be installable?**

   - Valid manifest.json with name, icons, start_url
   - Registered service worker
   - Served over HTTPS (or localhost for development)
   - User engagement with the site

2. **Why use `event.preventDefault()` on `beforeinstallprompt`?**

   - Stops browser's automatic install prompt
   - Gives control over timing and presentation
   - Allows custom UI instead of browser default
   - Improves user experience

3. **How do installed PWAs differ from browser bookmarks?**

   - **PWA**: Standalone window, custom icon, offline support, feels like native app
   - **Bookmark**: Opens in browser tab, browser icon, no offline support

4. **What happens if user declines installation?**

   - `userChoice` returns "dismissed"
   - App continues working normally in browser
   - Can show prompt again later (with delay)
   - User can still install via browser menu

5. **How would you test installation on mobile devices?**
   - Deploy to HTTPS server or use ngrok for localhost
   - Test on Android Chrome (best support)
   - Test on iOS Safari (different install flow)
   - Use Chrome DevTools device emulation for initial testing

---

## 🔧 COMPREHENSIVE TROUBLESHOOTING GUIDE

This section provides solutions to common issues encountered when working through the extension activities. Issues are organised by extension and include step-by-step debugging procedures.

### Table of Contents - Troubleshooting

- [General Issues Affecting All Extensions](#general-issues-affecting-all-extensions)
- [Extension 1: Contact Form Issues](#extension-1-contact-form-issues)
- [Extension 2: Duplicate Email Issues](#extension-2-duplicate-email-issues)
- [Extension 3: Context Manager Issues](#extension-3-context-manager-issues)
- [Extension 4: Filtering Issues](#extension-4-filtering-issues)
- [Extension 5: Online/Offline Detection Issues](#extension-5-onlineoffline-detection-issues)
- [Extension 6: Installation Issues](#extension-6-installation-issues)
- [Git and Version Control Issues](#git-and-version-control-issues)
- [Performance and Optimization Tips](#performance-and-optimization-tips)

---

## General Issues Affecting All Extensions

### Issue: Flask Server Won't Start After Making Changes

**Symptoms:**

- Error messages when running `python3 main.py`
- Syntax errors or import errors
- Server starts but pages return 500 errors

**Diagnostic Steps:**

```bash
# Check for syntax errors
python3 -m py_compile main.py
python3 -m py_compile database_manager.py

# Check Python version
python3 --version
# Should be 3.8 or higher

# Verify Flask is installed
pip3 show flask

# Check for detailed error messages
python3 main.py
# Read the full error traceback carefully
```

**Common Causes and Solutions:**

**1. Indentation Errors (Python is indentation-sensitive)**

```python
# WRONG - Mixed tabs and spaces
def index():
    data = dbHandler.listExtension()
	return render_template('/index.html', content=data)  # Tab used here!

# CORRECT - Consistent spacing (4 spaces recommended)
def index():
    data = dbHandler.listExtension()
    return render_template('/index.html', content=data)
```

**Solution:**

```bash
# Check your editor settings
# VSCode: Bottom bar should show "Spaces: 4"
# Convert tabs to spaces: Ctrl+Shift+P → "Convert Indentation to Spaces"
```

**2. Import Errors**

```python
# Error: ModuleNotFoundError: No module named 'database_manager'

# Check the import statement matches the filename
import database_manager as dbHandler  # File must be database_manager.py

# Check you're in the correct directory
pwd  # Should show: /workspaces/Flask_PWA_Programming_For_The_Web_Task_Source
```

**3. Port Already in Use**

```bash
# Error: OSError: [Errno 98] Address already in use

# Solution 1: Kill the process using port 5000
lsof -ti:5000 | xargs kill -9

# Solution 2: Use a different port in main.py
app.run(debug=True, host='0.0.0.0', port=5001)
```

### Issue: Changes Not Appearing in Browser

**Symptoms:**

- Made code changes but browser shows old version
- CSS/JavaScript changes not visible
- Database changes not reflected

**Solutions:**

**1. Hard Refresh the Browser**

```bash
# Windows/Linux: Ctrl + F5
# Mac: Cmd + Shift + R
# Or: Ctrl + Shift + Delete → Clear cached images and files
```

**2. Flask Caching (Debug Mode)**

```python
# Ensure debug mode is enabled in main.py
app.run(debug=True, host='0.0.0.0', port=5000)

# Debug mode automatically reloads when files change
# Look for this message in terminal: "Restarting with stat"
```

**3. Service Worker Caching**

```bash
# Service worker may be serving cached files
# Open DevTools (F12) → Application → Service Workers
# Click "Unregister" next to your service worker
# Then: Application → Storage → Clear site data
# Refresh the page
```

**4. Browser Cache vs Service Worker Cache**

```bash
# Clear both types of cache:
# 1. Browser cache: Ctrl + Shift + Delete
# 2. Service worker cache: DevTools → Application → Cache Storage → Delete
# 3. Unregister service worker
# 4. Close all browser tabs with your app
# 5. Restart Flask
# 6. Open fresh browser tab
```

### Issue: Database File Locked or Access Denied

**Symptoms:**

- `sqlite3.OperationalError: database is locked`
- `sqlite3.OperationalError: unable to open database file`
- Changes not saving to database

**Solutions:**

**1. Close All Database Connections**

```bash
# Check if database file is open elsewhere
lsof | grep data_source.db

# Close any SQLite editor windows
# Close any terminal sessions running sqlite3
```

**2. Verify File Permissions**

```bash
# Check database file permissions
ls -la database/data_source.db

# Should show: -rw-r--r-- (readable/writable)
# If not, fix permissions:
chmod 644 database/data_source.db
```

**3. Check for Unclosed Connections in Code**

```python
# WRONG - Connection not closed
def listExtension():
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    data = cur.execute("SELECT * FROM extension").fetchall()
    # Missing: con.close()
    return data

# CORRECT - Using context manager (Extension 1)
def listExtension():
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        data = cur.execute("SELECT * FROM extension").fetchall()
    return data
```

### Issue: Template Not Found Errors

**Symptoms:**

- `jinja2.exceptions.TemplateNotFound: index.html`
- 404 errors when accessing pages

**Solutions:**

**1. Verify Template File Structure**

```bash
# Check templates directory exists and contains files
ls -la templates/
# Should show: index.html, layout.html, add.html

ls -la templates/partials/
# Should show: menu.html
```

**2. Check Template Path in Flask Route**

```python
# Flask looks for templates in 'templates/' folder by default

# WRONG - Absolute path
render_template('/home/user/templates/index.html')

# WRONG - Missing leading slash
render_template('index.html')  # Will work, but inconsistent

# CORRECT - Path relative to templates folder
render_template('/index.html')  # Best practice for this project
```

**3. Verify Flask Template Configuration**

```python
# In main.py, Flask automatically looks in 'templates/' folder
app = Flask(__name__)  # Correct

# If you've changed the template folder:
app = Flask(__name__, template_folder='my_templates')  # Would need different paths
```

---

## Extension 1: Contact Form Issues

### Issue: Form Submits But No Data Saved

**Symptoms:**

- Form shows success message
- No data appears in database
- No errors in terminal

**Diagnostic Steps:**

```bash
# Check if table exists
cd database
sqlite3 data_source.db
.tables
# Should show: contact_list  extension

# Check table has data
SELECT * FROM contact_list;
# Should show rows if submissions worked

# Check table structure
.schema contact_list
# Should match: id INTEGER PRIMARY KEY, email TEXT UNIQUE, name TEXT
```

**Common Causes:**

**1. Missing `con.commit()`**

```python
# WRONG - No commit
def insertContact(email, name):
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
    con.close()  # ❌ Changes are lost!

# CORRECT - With commit
def insertContact(email, name):
    con = sql.connect("database/data_source.db")
    cur = con.cursor()
    cur.execute("INSERT INTO contact_list (email, name) VALUES (?, ?)", (email, name))
    con.commit()  # ✅ Saves changes
    con.close()
```

**2. Wrong database path**

```python
# Check you're connecting to the right database
con = sql.connect("database/data_source.db")  # Correct
# NOT:
con = sql.connect("data_source.db")  # Wrong - file not found
con = sql.connect("/database/data_source.db")  # Wrong - absolute path
```

### Issue: Form Input Fields Don't Match Python Variables

**Symptoms:**

- KeyError: 'email' or KeyError: 'name'
- Form submits but crashes

**Cause:**
HTML `name` attribute doesn't match `request.form['key']`

**Solution:**

```html
<!-- HTML form -->
<input type="email" name="email" />
<!-- name="email" is the key -->
<input type="text" name="name" />
<!-- name="name" is the key -->
```

```python
# Python route - keys must match HTML name attributes
email = request.form['email']  # Matches name="email"
name = request.form['name']    # Matches name="name"
```

### Issue: Success Message Not Appearing

**Symptoms:**

- Form submits successfully
- Still shows form instead of success message

**Cause:**
Template conditional not working or variable not passed

**Solution:**

```python
# Flask route - Make sure to pass is_done=True
if request.method == 'POST':
    email = request.form['email']
    name = request.form['name']
    dbHandler.insertContact(email, name)
    return render_template('/add.html', is_done=True)  # ✅ Must pass this!
```

```html
<!-- Template - Check spelling exactly -->
{% if is_done %}
<!-- Must match variable name exactly -->
<div class="success-message">...</div>
{% else %}
<form>...</form>
{% endif %}
```

---

## Extension 2: Duplicate Email Issues

### Issue: Exception Not Being Caught

**Symptoms:**

- App still crashes on duplicate email
- See `IntegrityError` on screen
- Try-except block seems ignored

**Diagnostic Steps:**

```bash
# Check terminal output when you submit duplicate email
# Should see the POST request and any print statements
```

**Common Causes:**

**1. Missing `import sqlite3`**

```python
# At top of main.py - MUST import this
import sqlite3  # ✅ Required to catch sqlite3.IntegrityError
from flask import Flask, request, render_template
import database_manager as dbHandler
```

**2. Wrong exception name**

```python
# WRONG - Typo in exception name
except sql.IntegrityError:  # ❌ Should be sqlite3, not sql

# CORRECT
except sqlite3.IntegrityError:  # ✅ Full module name
```

**3. Exception raised in wrong place**

```python
# If insertContact() is catching the error and NOT re-raising it:
def insertContact(email, name):
    try:
        # database code
    except sql.IntegrityError:
        print("Error!")  # ❌ Exception consumed here, route won't catch it

# Solution: Let exception propagate to route, OR return error status
```

### Issue: Error Message Not Displaying

**Symptoms:**

- No crash, but no red error box appears
- Form just reloads blank

**Cause:**
Template not checking for `error` variable

**Solution:**

```html
<!-- Add this BEFORE the form in add.html -->
{% if error %}
<div class="error-message">
  <p>⚠️ {{ error }}</p>
</div>
{% endif %}
```

### Issue: Form Fields Not Repopulating

**Symptoms:**

- After error, user has to retype everything
- Fields are blank

**Cause:**
Not passing data back to template, or missing `value` attribute

**Solution:**

```python
# Flask route - Pass form data back
except sqlite3.IntegrityError:
    error_message = "Email already subscribed."
    return render_template('/add.html',
                         error=error_message,
                         email=email,  # ✅ Pass data back
                         name=name)    # ✅ Pass data back
```

```html
<!-- Template - Use value attribute -->
<input type="text" name="name" value="{{ name or '' }}" />
<input type="email" name="email" value="{{ email or '' }}" />
```

---

## Extension 3: Context Manager Issues

### Issue: IndentationError After Refactoring

**Symptoms:**

- `IndentationError: expected an indented block`
- Code inside `with` block not executing

**Cause:**
Content inside `with` block must be indented one level.

**Solution:**

```python
# WRONG - No indentation
def listExtension():
    with sql.connect("database/data_source.db") as con:
    cur = con.cursor()  # Error: Not indented!
    data = cur.execute("SELECT * FROM extension").fetchall()
    return data

# CORRECT - Proper indentation
def listExtension():
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()  # Indented 4 spaces
        data = cur.execute("SELECT * FROM extension").fetchall()
    return data  # Back to original indentation
```

### Issue: Data Not Available After `with` Block

**Symptoms:**

- `NameError: name 'data' is not defined`
- Variables from inside `with` block not accessible

**Cause:**
Trying to use cursor after connection closes.

**Solution:**

```python
# WRONG - Trying to use cursor after connection closes
def listExtension():
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
    data = cur.execute("SELECT * FROM extension").fetchall()  # Error: Connection closed!
    return data

# CORRECT - Fetch data before connection closes
def listExtension():
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        data = cur.execute("SELECT * FROM extension").fetchall()  # Inside with block
    return data  # Data is safe to use here
```

### Issue: Changes Not Committing to Database

**Symptoms:**

- INSERT/UPDATE/DELETE queries run without errors
- But data not saved to database
- Query results show old data

**Cause:**
Forgot to call `commit()` for write operations.

**Solution:**

```python
# WRONG - Missing commit()
def insertContact(email, name):
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        cur.execute("INSERT INTO contact_list (email,name) VALUES (?,?)", (email, name))
    # Missing: con.commit()

# CORRECT - Commit before connection closes
def insertContact(email, name):
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        cur.execute("INSERT INTO contact_list (email,name) VALUES (?,?)", (email, name))
        con.commit()  # Must commit for INSERT/UPDATE/DELETE
```

**Verification:**

```bash
# Check if data was actually saved
sqlite3 database/data_source.db "SELECT * FROM contact_list;"

# If empty or missing records, commit() was not called
```

---

## Extension 4: Filtering Issues

### Issue: Filter Buttons Not Working

**Symptoms:**

- Clicking filter buttons shows all extensions
- URL changes but displayed content doesn't
- No filtering occurs

**Diagnostic Steps:**

```bash
# 1. Check if new database function exists
grep "def listExtensionByLanguage" database_manager.py
# Should show the function definition

# 2. Check Flask route is updated
grep "request.args.get" main.py
# Should show parameter handling in index() function

# 3. Check browser console for errors
# F12 → Console tab
# Look for JavaScript or network errors
```

**Common Solutions:**

**1. Database Function Not Called**

```python
# Check your index() route in main.py

# WRONG - Not using the filtered function
@app.route("/", methods=["POST", "GET"])
def index():
    language = request.args.get('language')
    data = dbHandler.listExtension()  # Always returns all data!
    return render_template("/index.html", content=data)

# CORRECT - Use filtered function when parameter present
@app.route("/", methods=["POST", "GET"])
def index():
    language = request.args.get('language')
    if language:
        data = dbHandler.listExtensionByLanguage(language)  # Filtered data
    else:
        data = dbHandler.listExtension()  # All data
    return render_template("/index.html", content=data)
```

**2. Flask Not Restarted**

```bash
# Flask must be restarted after changes to main.py or database_manager.py
# Stop Flask: Ctrl + C
# Start Flask: python3 main.py

# With debug=True, Flask auto-restarts, but check terminal for:
# "Restarting with stat" - Confirms reload
```

### Issue: URL Encoding Problems with Special Characters

**Symptoms:**

- Filter for `#BASH` doesn't work
- `#` character breaks the URL
- Browser jumps to page anchor instead of filtering

**Cause:**
`#` is a special character in URLs (marks anchors/fragments).

**Solution:**

```html
<!-- WRONG - # breaks the URL -->
<a href="/?language=#BASH">
  <!-- CORRECT - URL encode # as %23 -->
  <a href="/?language=%23BASH"></a
></a>
```

**Testing:**

```bash
# When you click the filter, check the URL bar:
# Wrong: http://localhost:5000/#BASH
# Correct: http://localhost:5000/?language=%23BASH

# Python automatically decodes %23 back to #
# So your database query receives "#BASH" correctly
```

### Issue: Filter Shows No Results (Empty Page)

**Symptoms:**

- Clicking filter makes all cards disappear
- No error messages
- Page is just empty

**Diagnostic Steps:**

```bash
# 1. Check database has matching data
sqlite3 database/data_source.db
SELECT DISTINCT language FROM extension;
# Note the exact spelling/capitalisation

# 2. Check filter button matches database exactly
# Compare button href with database values

# 3. Add debug logging to Flask route
```

**Debug Logging:**

```python
@app.route("/", methods=["POST", "GET"])
def index():
    language = request.args.get('language')
    print(f"DEBUG: Filter language = '{language}'")  # Add this

    if language:
        data = dbHandler.listExtensionByLanguage(language)
        print(f"DEBUG: Found {len(data)} results")  # Add this
    else:
        data = dbHandler.listExtension()
        print(f"DEBUG: Showing all {len(data)} results")  # Add this

    return render_template("/index.html", content=data)
```

**Check Flask terminal output:**

```bash
# Should see:
DEBUG: Filter language = 'Python'
DEBUG: Found 0 results

# This tells you:
# - Parameter is being received correctly
# - But no matching data in database
```

**Solutions:**

**1. Case Sensitivity**

```sql
-- Database has: "python" (lowercase)
-- Filter searches for: "Python" (capitalized)
-- Result: No match!

-- Solution A: Match case exactly in filter buttons
<a href="/?language=python">Python</a>

-- Solution B: Use case-insensitive SQL query
SELECT * FROM extension WHERE LOWER(language) = LOWER(?)
```

**2. Extra Spaces in Database**

```bash
# Check for hidden spaces
sqlite3 database/data_source.db
SELECT language, length(language) FROM extension;

# If "Python " has length 7 instead of 6, there's a trailing space
# Clean the data:
UPDATE extension SET language = TRIM(language);
```

### Issue: SQL Injection Vulnerability Warning

**Symptoms:**

- Security scanner flags your code
- Linter shows warning about SQL queries

**Check Your Code:**

```python
# DANGEROUS - DO NOT USE
def listExtensionByLanguage(language):
    query = f"SELECT * FROM extension WHERE language = '{language}'"
    cur.execute(query)  # VULNERABLE TO SQL INJECTION!

# SAFE - Always use parameterised queries
def listExtensionByLanguage(language):
    cur.execute("SELECT * FROM extension WHERE language = ?", (language,))
```

**Why This Matters:**

```python
# Malicious user could send:
# /?language=Python' OR '1'='1

# With string formatting (UNSAFE):
# Query becomes: SELECT * FROM extension WHERE language = 'Python' OR '1'='1'
# This returns ALL records, bypassing the filter!

# With parameterised query (SAFE):
# The database treats the entire input as a string value
# Looking for language literally equal to "Python' OR '1'='1"
# This returns no results (safe)
```

---

## Extension 5: Online/Offline Detection Issues

### Issue: Offline Banner Never Appears

**Symptoms:**

- Going offline doesn't show banner
- No visual feedback when network disconnects
- Browser console shows no errors

**Diagnostic Steps:**

```bash
# 1. Verify banner element exists in HTML
# View page source: Ctrl+U
# Search for: id="offline-banner"

# 2. Check JavaScript is loaded
# F12 → Console tab
# Type: document.getElementById("offline-banner")
# Should return the element, not null

# 3. Test offline detection manually
# F12 → Console tab
# Type: navigator.onLine
# Should return true (online) or false (offline)
```

**Common Solutions:**

**1. JavaScript Not Running**

```bash
# Check browser console for errors (F12 → Console)
# Common errors:

# "Uncaught TypeError: Cannot read properties of null"
# → Element ID doesn't match JavaScript

# "Uncaught ReferenceError: offlineBanner is not defined"
# → Variable declared inside wrong scope
```

**Fix:**

```js
// Make sure this line is OUTSIDE any function
const offlineBanner = document.getElementById("offline-banner");

// Not inside:
window.addEventListener("load", function () {
  const offlineBanner = document.getElementById("offline-banner"); // Wrong!
});
```

**2. Element ID Mismatch**

```html
<!-- HTML has: -->
<div id="offline-banner">
  <!-- JavaScript looks for: -->
  const offlineBanner = document.getElementById("offline-banner"); ✅ Match

  <!-- Common typos: -->
  const offlineBanner = document.getElementById("offlineBanner"); ❌ Wrong case
  const offlineBanner = document.getElementById("offline_banner"); ❌ Underscore
</div>
```

**3. Inline Style Prevents CSS**

```html
<!-- Inline style takes precedence -->
<div id="offline-banner" style="display: none;">
  <!-- JavaScript must use inline style to override -->
  <script>
    // Wrong - CSS class won't override inline style
    offlineBanner.classList.add("visible"); // Won't work!

    // Correct - Set inline style directly
    offlineBanner.style.display = "block"; // Works!
  </script>
</div>
```

### Issue: Service Worker Typo Breaks Everything

**Symptoms:**

- Console error: "serviceWorker is not defined"
- Service worker not registering
- Both offline detection and caching broken

**Cause:**
Typo in `serviceWorker` (must be camelCase with capital W).

**Solution:**

```js
// WRONG - Multiple typos
if ("serviceworker" in navigator) {
  // All lowercase
  navigator.serviceworker.register(); // All lowercase
}

// WRONG - Wrong capitalisation
if ("serviceWorker" in navigator) {
  // Correct
  navigator.ServiceWorker.register(); // Wrong - capital S
}

// CORRECT - Proper camelCase
if ("serviceWorker" in navigator) {
  // Capital W
  navigator.serviceWorker.register(); // Capital W (both places)
}
```

### Issue: Online Event Not Firing

**Symptoms:**

- Going online doesn't hide banner
- Banner appears but never disappears
- `console.log` in online event never shows

**Debug:**

```js
// Add extensive logging
window.addEventListener("online", function () {
  console.log("🟢 ONLINE EVENT FIRED!"); // Add this
  updateOnlineStatus();
});

window.addEventListener("offline", function () {
  console.log("🔴 OFFLINE EVENT FIRED!"); // Add this
  updateOnlineStatus();
});

// Test manually in console:
window.dispatchEvent(new Event("online"));
// Should trigger event and show log message
```

**Common Causes:**

**1. Event Listener Added After Event Fires**

```js
// WRONG - Events registered too late
function someFunctionCalledLater() {
  window.addEventListener("online", updateOnlineStatus); // Misses early events
}

// CORRECT - Register immediately when script loads
window.addEventListener("online", updateOnlineStatus); // Top level
```

**2. Function Not Defined**

```js
// WRONG - Typo in function name
window.addEventListener("online", updateOnLineStatus); // Capital L!

// CORRECT - Match function name exactly
window.addEventListener("online", updateOnlineStatus); // lowercase L
```

---

## Extension 6: Installation Issues

### Issue: Install Button Never Appears

**Symptoms:**

- No install button visible
- `beforeinstallprompt` event not firing
- Browser console shows no errors

**Diagnostic Steps:**

```bash
# 1. Check PWA installability
# F12 → Application → Manifest
# Look for "Installability" section
# Should show green checkmarks or list issues

# 2. Check if browser supports feature
# F12 → Console
if ('beforeinstallprompt' in window) {
    console.log("✅ Browser supports beforeinstallprompt");
} else {
    console.log("❌ Browser doesn't support this API");
}

# 3. Check if app is already installed
# If installed, event won't fire
# Check: chrome://apps or edge://apps
```

**Common Causes and Solutions:**

**1. PWA Not Meeting Installability Criteria**

```bash
# Check DevTools → Application → Manifest
# Common issues:

# ❌ Manifest not loading
# Fix: Check manifest.json path in layout.html
<link rel="manifest" href="static/manifest.json">

# ❌ Service worker not registered
# Fix: Check app.js loads and registers service worker
<script src="static/js/app.js"></script>

# ❌ Icons missing
# Fix: Verify icon files exist
ls -la static/icons/

# ❌ Not HTTPS or localhost
# Fix: Use localhost:5000 for development
```

**2. Browser Doesn't Support Installation**

```bash
# Safari (iOS/macOS) - Different installation method
# Doesn't fire beforeinstallprompt
# Users must use: Share → Add to Home Screen

# Firefox - Limited desktop PWA support
# May not fire event on desktop

# Best support: Chrome, Edge, Opera (Chromium-based)
```

**3. App Already Installed**

```bash
# If app is already installed, event won't fire

# Check if installed:
# Chrome: chrome://apps
# Edge: edge://apps
# Look for your app icon

# To test again:
# 1. Uninstall the app
# 2. Clear site data (DevTools → Application → Storage)
# 3. Close all tabs with your app
# 4. Open fresh tab to localhost:5000
```

**4. User Dismissed Prompt Previously**

```bash
# Browser may temporarily block prompt if user dismissed it

# Solutions:
# - Wait 3-5 minutes and reload
# - Clear browsing data
# - Use incognito/private mode
# - Test on different browser profile
```

### Issue: Install Prompt Shows But Installation Fails

**Symptoms:**

- Button appears and user clicks it
- Browser shows install dialog
- User clicks "Install"
- But app doesn't actually install

**Check:**

```js
// Add logging to track installation process
installButton.addEventListener("click", async () => {
  console.log("1. Install button clicked");

  if (!deferredPrompt) {
    console.log("ERROR: deferredPrompt is null");
    return;
  }

  console.log("2. Showing prompt...");
  deferredPrompt.prompt();

  console.log("3. Waiting for user choice...");
  const { outcome } = await deferredPrompt.userChoice;

  console.log(`4. User chose: ${outcome}`);

  if (outcome === "accepted") {
    console.log("5. User accepted - waiting for appinstalled event");
  } else {
    console.log("5. User dismissed");
  }
});

window.addEventListener("appinstalled", () => {
  console.log("6. ✅ APP INSTALLED SUCCESSFULLY");
});
```

**Common Issues:**

**1. Manifest JSON Invalid**

```bash
# Validate manifest.json
# Copy content to: https://jsonlint.com/

# Common JSON errors:
# - Missing comma
# - Extra comma at end of array/object
# - Unquoted property names
# - Wrong quote types (' instead of ")
```

**2. Icon Files Missing or Wrong Format**

```bash
# Verify all icon files exist
ls -la static/icons/
# Should show: icon-128x128.png, icon-192x192.png, icon-384x384.png, icon-512x512.png

# Check file sizes are reasonable
ls -lh static/icons/
# Should be: 1-50KB each (web-optimised)

# Verify they're actual PNG files
file static/icons/icon-512x512.png
# Should show: PNG image data, 512 x 512
```

**3. Start URL Incorrect**

```json
// In manifest.json

// WRONG - Absolute URL won't work
"start_url": "http://localhost:5000/"

// WRONG - Missing leading slash
"start_url": "index.html"

// CORRECT - Root-relative path
"start_url": "/"
```

### Issue: Installed App Opens in Browser Tab

**Symptoms:**

- App installs successfully
- But opens in browser tab instead of standalone window
- Still shows browser UI (address bar, tabs)

**Cause:**
Manifest `display` property not set correctly.

**Solution:**

```json
// In manifest.json

// Check this property
"display": "standalone",  // Correct

// If you have:
"display": "browser",     // Opens in browser tab
"display": "minimal-ui",  // Shows minimal browser UI

// Options:
// "standalone" - No browser UI (recommended for apps)
// "fullscreen" - No browser UI, fills screen
// "minimal-ui" - Minimal browser UI
// "browser" - Regular browser tab
```

### Issue: Icons Not Showing After Installation

**Symptoms:**

- App installs
- But shows default browser icon instead of custom icon
- Desktop shortcut has wrong icon

**Solutions:**

**1. Check Icon Paths in Manifest**

```json
// In manifest.json

// WRONG - Absolute path
"src": "/static/icons/icon-512x512.png"

// WRONG - Missing directory
"src": "icon-512x512.png"

// CORRECT - Relative to manifest.json location
"src": "icons/icon-512x512.png"
```

**2. Verify Icon Sizes Match**

```json
// File must actually be 512x512 pixels

{
  "src": "icons/icon-512x512.png",
  "sizes": "512x512", // Must match actual file dimensions
  "type": "image/png"
}
```

**Check actual dimensions:**

```bash
# Install imagemagick if needed
file static/icons/icon-512x512.png
# Should show: 512 x 512

# Or use Python
python3 << EOF
from PIL import Image
img = Image.open('static/icons/icon-512x512.png')
print(f"Dimensions: {img.size}")
EOF
```

**3. Cache Issues**

```bash
# Browser may have cached old icon

# Solution:
# 1. Uninstall app completely
# 2. Clear browser cache (Ctrl+Shift+Delete)
# 3. DevTools → Application → Storage → Clear site data
# 4. Close browser completely
# 5. Reopen and reinstall
```

---

## Git and Version Control Issues

### Issue: Git Commands Not Working

**Symptoms:**

- `git: command not found`
- `fatal: not a git repository`

**Solutions:**

**1. Git Not Installed**

```bash
# Check if Git is installed
git --version

# If not installed:
# Windows: Download from https://git-scm.com/
# Mac: xcode-select --install
# Linux: sudo apt-get install git
```

**2. Not in Git Repository**

```bash
# Check if you're in a git repository
git status

# If error: "fatal: not a git repository"
# Initialize git:
git init

# Or clone the template:
git clone https://github.com/TempeHS/Flask_PWA_Programming_For_The_Web_Task_Template.git
```

### Issue: Git Commit Fails

**Symptoms:**

- `nothing to commit, working tree clean` (but you made changes)
- `Changes not staged for commit`

**Solutions:**

```bash
# Check what files changed
git status

# Stage all changes
git add .

# Or stage specific files
git add database_manager.py main.py

# Then commit
git commit -m "Your commit message"

# If commit still fails, check git config:
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

### Issue: Accidentally Committed Wrong Files

**Symptoms:**

- Committed database file (should be in .gitignore)
- Committed environment files or secrets
- Want to undo last commit

**Solutions:**

```bash
# Undo last commit but keep changes
git reset --soft HEAD~1

# Undo last commit and discard changes
git reset --hard HEAD~1

# Remove file from git but keep locally
git rm --cached database/data_source.db

# Add to .gitignore
echo "database/*.db" >> .gitignore
git add .gitignore
git commit -m "Add database files to gitignore"
```

---

## Performance and Optimisation Tips

### Issue: App Feels Slow or Unresponsive

**Diagnostic Steps:**

```bash
# 1. Check Flask debug mode (slower but helpful in development)
# In main.py:
app.run(debug=True)  # Slower, but auto-reloads

# For production (faster):
app.run(debug=False)

# 2. Check database query performance
# Add timing to queries:
```

```python
import time

def listExtension():
    start = time.time()
    with sql.connect("database/data_source.db") as con:
        cur = con.cursor()
        data = cur.execute("SELECT * FROM extension").fetchall()
    end = time.time()
    print(f"Query took {end - start:.4f} seconds")
    return data
```

**Optimization Tips:**

**1. Database Indices**

```sql
-- If filtering is slow, add an index
CREATE INDEX idx_language ON extension(language);

-- Check query plans
EXPLAIN QUERY PLAN SELECT * FROM extension WHERE language = 'Python';
```

**2. Limit Database Queries**

```python
# WRONG - Multiple queries in loop
@app.route("/")
def index():
    extensions = dbHandler.listExtension()
    for ext in extensions:
        details = dbHandler.getExtensionDetails(ext[0])  # N+1 query problem!
    return render_template("/index.html", content=extensions)

# CORRECT - Single query with JOIN
@app.route("/")
def index():
    data = dbHandler.listExtensionWithDetails()  # One query
    return render_template("/index.html", content=data)
```

**3. Optimise Images**

```bash
# Check image sizes
ls -lh static/images/
ls -lh static/icons/

# Icons should be:
# - Web-optimised (TinyPNG)
# - Correct dimensions (no larger than needed)
# - 10-50KB each maximum

# If images are large:
# 1. Use https://tinypng.com/
# 2. Or install ImageMagick:
convert logo.png -quality 85 -resize 1080x1080 logo-optimised.png
```

**4. Service Worker Caching Strategy**

```js
// In serviceworker.js
// Only cache essential files for faster loading

const assets = [
  "/",
  "static/css/style.css",
  "static/js/app.js",
  // Don't cache large images unless necessary
];
```

### Issue: Service Worker Not Updating

**Symptoms:**

- Made changes to service worker
- But browser still uses old version
- New caching rules not applying

**Solution:**

```bash
# 1. Change service worker version in serviceworker.js
const CATALOGUE_ASSETS = "catalogue-assets-v2";  // Increment version

# 2. Update activate event to clear old caches
self.addEventListener("activate", function (evt) {
    evt.waitUntil(
        caches.keys().then((keyList) => {
            return Promise.all(
                keyList.map((key) => {
                    if (key !== CATALOGUE_ASSETS) {  // Clear old versions
                        console.log("Removed old cache:", key);
                        return caches.delete(key);
                    }
                })
            );
        })
    );
});

# 3. Force update in browser
# DevTools → Application → Service Workers → Click "Update"
# Or: Check "Update on reload"
# Or: Click "Unregister" and refresh page
```

---

## Getting Additional Help

### Before Asking for Help

Complete this checklist:

```bash
# ✅ 1. Read error messages carefully
# - Copy the FULL error message
# - Note which file and line number

# ✅ 2. Check browser console
# F12 → Console tab
# Look for red error messages

# ✅ 3. Check Flask terminal
# Look for Python errors and tracebacks

# ✅ 4. Try the troubleshooting steps above
# - Matched your error?
# - Tried the solutions?

# ✅ 5. Test in incognito/private mode
# Eliminates browser cache/extension issues

# ✅ 6. Verify your environment
python3 --version
pip3 show flask
git --version
pwd
```

### Information to Include When Asking

When seeking help, provide:

1. **What extension are you working on?**

   - Extension 1, 2, 3, or 4?

2. **What were you trying to do?**

   - Step-by-step what you did

3. **What did you expect to happen?**

   - Describe expected behavior

4. **What actually happened?**

   - Include error messages (full text)
   - Include screenshots if relevant

5. **What have you tried?**

   - List troubleshooting steps attempted

6. **Your environment:**
   ```bash
   python3 --version
   pip3 list | grep -i flask
   git --version
   ```

### Useful Resources

- **Flask Documentation**: https://flask.palletsprojects.com/
- **MDN PWA Guide**: https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps
- **SQLite Documentation**: https://www.sqlite.org/docs.html
- **W3C Web Standards**: https://www.w3.org/standards/
- **Python Documentation**: https://docs.python.org/3/

---

<p xmlns:cc="http://creativecommons.org/ns#" xmlns:dct="http://purl.org/dc/terms/"><a property="dct:title" rel="cc:attributionURL" href="https://github.com/TempeHS/Flask_PWA_Programming_For_The_Web_Task_Source">Flask PWA Programming For The Web - Extension Activities</a> by <a rel="cc:attributionURL dct:creator" property="cc:attributionName" href="https://github.com/benpaddlejones">Ben Jones</a> is licensed under <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/?ref=chooser-v1" target="_blank" rel="license noopener noreferrer" style="display:inline-block;">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International</a></p>
