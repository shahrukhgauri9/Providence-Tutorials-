# Integration Guide - User Management & Password Reset Features

## 📌 Overview
This guide explains how to integrate the complete user management system into your dashboard.html file with:
- ✅ User authentication
- ✅ Add/Edit/Delete users
- ✅ Admin password reset for other users
- ✅ Admin reset own password
- ✅ Users can change their own password
- ✅ Role-based UI visibility

---

## 🔧 INTEGRATION STEPS

### Step 1: Add User Management Script
**Location**: In `<script>` tag after line 793

Copy the entire contents of `USER_MANAGEMENT_SYSTEM.js` into your dashboard.html script section.

### Step 2: Update the Sidebar Navigation
**Location**: Lines 319-327 in dashboard.html

Add Settings link if not present:
```html
<div class="nav-item" onclick="showPage('settings')"><span class="icon">⚙️</span> Settings</div>
```

### Step 3: Add New Panel for Admin Reset Own Password
**Location**: Add this in Settings page (around line 639)

Insert BEFORE the "Change My Password" panel:
```html
<div class="panel" id="admin-own-reset-panel" style="display:none;">
  <div class="panel-head"><h3>🔐 Admin: Reset Your Own Password</h3></div>
  <div class="panel-body">
    <p style="font-size:13px;color:var(--muted);margin-bottom:18px;">
      As the admin, you can reset your own password here. This is useful if you forget it.
    </p>
    <button class="btn btn-gold" onclick="showAdminResetOwnPassword()">🔑 Reset My Password</button>
  </div>
</div>
```

### Step 4: Update applyRoleUI() Function
**In USER_MANAGEMENT_SYSTEM.js**

Add this code to hide the "Admin: Reset Your Own Password" panel for staff:
```javascript
// Hide admin-specific password reset
if (user.role !== 'admin') {
  const adminOwnResetPanel = document.getElementById('admin-own-reset-panel');
  if (adminOwnResetPanel) adminOwnResetPanel.style.display = 'none';
}
```

---

## 🎯 KEY FEATURES

### 1. User Management Table
- Shows all users with ID, Name, Role, and Status
- Admin can Edit, Reset Password, or Delete users
- Actions: Edit, Reset Password (for any user), Delete (except admin)

### 2. Add/Edit User Form
- **Add User**: Create new staff or admin accounts
- **Edit User**: Modify user name, role, and password
- **Fields**:
  - User ID (unique, lowercase)
  - Full Name
  - Password (min 6 characters)
  - Role (Admin or Staff)

### 3. Password Reset Options
| Who | What | How |
|-----|------|-----|
| Admin | Reset any user's password | Click "🔑 Reset" button in users table |
| Admin | Reset own password | "Admin: Reset Your Own Password" panel |
| Any User | Change own password | "Change My Password" panel |

### 4. Default Admin Account
```
User ID: admin
Password: admin123
Role: Admin
```
**⚠️ Change this in production!**

---

## 💾 DATA STORAGE

All user data is stored in `localStorage`:

### Storage Keys
- `pt_users` - Array of all users
- `pt_currentUser` - Currently logged-in user

### User Object Structure
```javascript
{
  uid: "teacher1",           // Unique user ID
  name: "John Teacher",      // Display name
  password: "hashedpw",      // Password (should be hashed!)
  role: "admin" | "staff",   // User role
  created: "2026-04-30...",  // Creation timestamp
  lastLogin: "2026-04-30...", // Last login timestamp
  status: "active"           // active or inactive
}
```

---

## 🔒 SECURITY ISSUES & FIXES

### ⚠️ CRITICAL - Passwords Are Not Hashed
**Current**: Passwords stored as plain text
**Fix**: Use bcryptjs library or implement server-side hashing

```javascript
// Example with bcryptjs (add to HTML head):
// <script src="https://cdn.jsdelivr.net/npm/bcryptjs@2.4.3/dist/bcrypt.js"></script>

// Hash password before saving
const hashedPassword = bcryptjs.hashSync(password, 10);

// Verify password on login
const isMatch = bcryptjs.compareSync(inputPassword, hashedPassword);
```

### ⚠️ Session Management
**Current**: No automatic logout
**Recommendation**: Add logout timer for security

```javascript
// Add auto-logout after 30 minutes of inactivity
let logoutTimer;

function resetLogoutTimer() {
  clearTimeout(logoutTimer);
  logoutTimer = setTimeout(() => {
    alert('Session expired due to inactivity');
    doLogout();
  }, 30 * 60 * 1000); // 30 minutes
}

// Reset timer on any user activity
document.addEventListener('click', resetLogoutTimer);
document.addEventListener('keypress', resetLogoutTimer);
```

### ⚠️ Input Validation
**Current**: Basic validation
**Recommendation**: Enhanced validation

```javascript
// Validate user ID format
function isValidUserID(uid) {
  return /^[a-z0-9_]{3,20}$/.test(uid);
}

// Validate email (if added)
function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

---

## 🧪 TESTING CHECKLIST

- [ ] Admin can add a new user
- [ ] New user can be edited (name, role)
- [ ] New user can be deleted (except admin)
- [ ] Admin can reset any user's password
- [ ] Admin can reset their own password
- [ ] Staff users can change their password
- [ ] Staff users cannot see "Admin Reset" panel
- [ ] Staff users cannot see user management table
- [ ] Users can log out
- [ ] Current user displays in topbar
- [ ] User role displays in topbar

---

## 📝 MIGRATION FROM EXISTING SYSTEM

If you have an existing system:

1. **Export existing users** (if any) to JSON
2. **Initialize pt_users** in localStorage with your data:
```javascript
const existingUsers = [
  { uid: 'admin', name: 'Admin', password: 'admin123', role: 'admin', 
    created: new Date().toISOString(), lastLogin: null, status: 'active' },
  { uid: 'teacher1', name: 'John', password: 'pass123', role: 'staff',
    created: new Date().toISOString(), lastLogin: null, status: 'active' }
];
localStorage.setItem('pt_users', JSON.stringify(existingUsers));
```

---

## 🚀 NEXT STEPS

### Phase 2: Audit Logging
Track who did what and when:
```javascript
function logAction(action, user, details) {
  const log = {
    timestamp: new Date().toISOString(),
    user: user.uid,
    action: action,
    details: details
  };
  // Save to localStorage or server
}
```

### Phase 3: Two-Factor Authentication (2FA)
Add SMS or email verification for admin login

### Phase 4: Server-Side Implementation
Move from localStorage to database:
- Node.js + Express backend
- MongoDB for user storage
- JWT for session management
- Password hashing with bcryptjs

---

## 🐛 TROUBLESHOOTING

### Users table shows "No users found"
- Check if `renderUsersTable()` is being called
- Verify localStorage has `pt_users` key
- Open DevTools Console and check `users` variable

### Password reset not working
- Ensure `doAdminReset()` is properly defined
- Check if user object exists in array
- Verify password confirmation matches

### Current user not displaying
- Check if `setCurrentUser()` is called on page load
- Verify `currentUser` object in localStorage
- Check topbar IDs: `topbar-user`, `topbar-role`

### Settings page not visible
- Ensure Settings nav item onclick calls `showPage('settings')`
- Check if `applyRoleUI()` is hiding it for staff
- Verify page element ID is `pg-settings`

---

## 📞 SUPPORT

For issues or questions about this implementation, refer to:
1. REVIEW_AND_IMPROVEMENTS.md - Detailed bug analysis
2. USER_MANAGEMENT_SYSTEM.js - Complete function implementations
3. Dashboard inline comments - Code explanations

