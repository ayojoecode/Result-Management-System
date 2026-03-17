# School Result Management Portal
## Setup & Installation Guide

---

## Tech Stack
- PHP 7.4 (PDO)
- MySQL 5.7+
- Bootstrap 4 (Melody Admin Template)
- JavaScript (vanilla)

---

## 1. File Structure

```
school_portal/
├── index.php                  ← Root redirect
├── login.php                  ← Staff login page
├── logout.php
├── database.sql               ← Import this first!
├── includes/
│   ├── config.php             ← DB credentials + grading logic
│   ├── auth.php               ← Login/session/role helpers
│   ├── layout_header.php      ← Navbar + sidebar partial
│   └── layout_footer.php      ← JS includes partial
├── admin/
│   ├── dashboard.php          ← Super Admin dashboard
│   ├── students.php           ← Add/Edit/Delete students
│   ├── classes.php            ← Manage classes + subjects
│   ├── users.php              ← Manage staff accounts
│   ├── sessions.php           ← Academic sessions & terms
│   ├── pins.php               ← Generate & manage PINs
│   ├── print_pins.php         ← Printable scratch cards
│   └── school_settings.php    ← School name/logo/address
├── headteacher/
│   ├── dashboard.php          ← Head of School dashboard
│   ├── approve_results.php    ← Approve/reject per class (radio buttons)
│   └── view_results.php       ← View class results by term
├── teacher/
│   ├── dashboard.php          ← Teacher dashboard
│   ├── enter_results.php      ← Enter CA + Exam scores
│   └── my_class.php           ← View class students & subjects
├── public/
│   └── check_result.php       ← Student/parent PIN result checker
├── uploads/                   ← Logo uploads
└── assets/                    ← CSS, JS, vendors (from template)
```

---

## 2. Database Setup

1. Create a MySQL database:
   ```sql
   CREATE DATABASE school_portal CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

2. Import the schema:
   ```bash
   mysql -u root -p school_portal < database.sql
   ```

3. Default Super Admin login:
   - Username: `admin`
   - Password: `admin123` ← **Change this immediately after first login!**

---

## 3. Configuration

Edit `includes/config.php`:
```php
define('DB_HOST', 'localhost');
define('DB_NAME', 'school_portal');
define('DB_USER', 'your_db_user');
define('DB_PASS', 'your_db_password');
define('APP_URL', 'http://yourdomain.com/school_portal');
```

---

## 4. Web Server Setup

### Apache (.htaccess in root):
```apache
Options -Indexes
DirectoryIndex index.php

<FilesMatch "\.php$">
    # PHP already handled
</FilesMatch>
```

### Nginx:
```nginx
root /var/www/school_portal;
index index.php;
location ~ \.php$ {
    fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

### XAMPP / WAMP (local):
- Place the `school_portal` folder in `htdocs/` (XAMPP) or `www/` (WAMP)
- Access via: `http://localhost/school_portal`

---

## 5. Folder Permissions

```bash
chmod 755 school_portal/
chmod 777 school_portal/uploads/
chmod 777 school_portal/assets/images/  # For logo upload
```

---

## 6. First-Time Setup Steps

1. **Login** as Super Admin (`admin` / `password`)
2. **Change password** immediately in Users → Edit
3. **School Settings** → Set your school name, address, logo
4. **Sessions & Terms** → Create the current academic session (e.g. 2024/2025) and set active term
5. **Classes** → Verify classes exist, add any missing ones
6. **Classes → Subjects** → Add subjects to each class
7. **Users** → Add Head of School and Class Teacher accounts
8. **Classes** → Assign teachers to their classes
9. **Students** → Add students with admission numbers
10. **PINs** → Generate a batch of PINs for result checking

---

## 7. Workflow

### Teacher:
1. Login → Enter Results page
2. Fill in CA scores and Exam scores for each student
3. Save as Draft (auto-calculates grade + remark)
4. When ready, click **Submit for Approval**

### Head of School:
1. Login → Approve Results page
2. Select a class from the left panel
3. Review each student's scores
4. Use **radio buttons** (Approve / Reject) per student
5. Or click **Approve All Submitted** for the whole class
6. Positions are automatically computed after approval

### Student / Parent:
1. Go to `public/check_result.php`
2. Enter Admission Number + PIN from scratch card
3. Select Session and Term
4. View and print the result sheet

---

## 8. Grading System

### KG & Nursery (percentage-based):
| Score | Grade | Remark     |
|-------|-------|------------|
| 80–100 | A    | Excellent  |
| 65–79  | B    | Very Good  |
| 50–64  | C    | Good       |
| 40–49  | D    | Fair       |
| 0–39   | F    | Fail       |

### Primary / JSS / SSS (Nigerian standard):
| Score | Grade | Remark      |
|-------|-------|-------------|
| 75–100 | A    | Distinction |
| 65–74  | B    | Credit      |
| 55–64  | C    | Credit      |
| 45–54  | D    | Pass        |
| 40–44  | E    | Pass        |
| 0–39   | F    | Fail        |

CA Maximum: **30 marks** (Primary/JSS/SSS), **40 marks** (KG/Nursery)
Exam Maximum: **70 marks** (Primary/JSS/SSS), **60 marks** (KG/Nursery)

---

## 9. PIN System

- PINs are generated in batches (any quantity up to 2,000)
- Format: `XXXX-XXXX-XXXX` (12 alphanumeric chars + 2 dashes)
- Each PIN is unique and can only be used **once**
- Export any batch as CSV or print as scratch cards
- Used PINs show the admission number that consumed them

---

## 10. Security Notes

- All passwords are hashed with `password_hash()` (bcrypt)
- Sessions use `httponly` cookies and 1-hour timeout
- All user inputs are sanitized with `htmlspecialchars()` and PDO prepared statements
- Role-based access control on every page
- For production: set `'secure' => true` in config.php session cookie params (requires HTTPS)

---

## 11. Customisation

- To change grading thresholds → edit `getGrade()` in `includes/config.php`
- To change CA/Exam split → edit `getMaxScores()` in `includes/config.php`
- To add more levels → add to the `ENUM` in `database.sql` and update grading functions
- School branding → Admin → School Settings (name, logo, motto, address)
