# Member 3 Presentation Script — Mitigation, Bugs & Flaws
**Secure Task Management System (IKB21503 – Secure Software Development)**
Target length: 5–7 minutes (within the 4–10 min requirement)

> Fill in `[Your Name]` before recording. Lines in *italics* are stage directions / screen-recording cues, not spoken.

---

### 1. Intro (15–20 sec)

"Hi everyone, I'm **[Your Name]**, and I'm Member 3 on this team. My part of the project was applying mitigations against the issues, bugs, and flaws we identified in our Secure Task Management System — basically, taking each weakness we found and showing how we fixed it."

---

### 2. Setting up the problem (20–30 sec)

"Before I show the fixes, let's quickly recall the risks we had to deal with: users seeing or editing each other's tasks, no way to trace who changed what, and the usual web attack surface — SQL injection, cross-site scripting, and CSRF. My job was making sure each of these had a concrete mitigation in the system, not just a mention in the report."

---

### 3. Mitigation walkthrough (3–4 min — the core of the presentation)

*Recommend screen-recording each demo live as you talk.*

**a) Broken Access Control / Data Isolation**
"The first issue was access control — could one user reach another user's task? We mitigated this by filtering every task query by the logged-in user: `Task.objects.filter(owner=request.user)`. We also built a reusable helper, `get_object_or_404(Task, pk=id, owner=request.user)`, so every view — edit, delete, view — enforces ownership the same way instead of repeating the check inconsistently."
*Demo: try opening another user's task URL directly → show 403 or redirect.*

**b) CSRF (Cross-Site Request Forgery)**
"Next, CSRF. Every form in our app includes Django's `{% csrf_token %}`, and it's validated on every POST request."
*Demo: strip the CSRF token using browser dev tools and submit → show the 403 Forbidden response.*

**c) SQL Injection**
"Because we use Django's ORM for all database queries instead of raw SQL, injection attempts typed into form fields are automatically sanitized before they ever reach the database."
*Demo (optional): type a classic injection string like `' OR 1=1` into a task field and show it's stored as plain text, not executed.*

**d) Cross-Site Scripting (XSS)**
"For XSS, Django's template engine auto-escapes HTML in any variable it renders. So if someone enters a script tag as a task title, it shows up as literal text on the page instead of running."
*Demo: enter `<script>alert(1)</script>` as a task title and show it displays as text, no alert pop-up.*

**e) Weak Authentication**
"On the authentication side, passwords are never stored in plaintext — Django hashes them with PBKDF2-SHA256 using 600,000 iterations and a random salt per password. On top of that, our password validators reject anything too short, too common, or fully numeric."
*Demo: try setting a weak password like "12345678" and show the validation error.*

**f) Audit Log Tampering**
"One flaw we specifically caught was that an admin could, in theory, edit or delete audit log entries — which would defeat the whole point of having a log. We fixed this by overriding `has_add_permission`, `has_change_permission`, and `has_delete_permission` to return `False` in our `AuditLogAdmin` class, making the logs strictly read-only, even for superusers."
*Demo: open the audit log in the admin panel and show there's no edit/delete option available.*

---

### 4. Bug → Fix highlight (30–40 sec)

"One challenge we ran into during development: it was easy to forget the ownership check on a *new* view we added later, which would have reopened the access-control flaw. Instead of relying on remembering to add it every time, we centralized it into that one helper function I mentioned earlier, so any new view automatically inherits the protection. That turned a recurring bug risk into a one-time fix."

---

### 5. Proof it works — test results (30–40 sec)

"To confirm these mitigations actually hold up, we ran targeted security tests: submitting forms without a CSRF token returned 403 as expected, trying to access another user's task by manipulating the URL was blocked, and accessing protected pages while logged out redirected straight to the login page. All of these passed."

---

### 6. Closing — what's next (15–20 sec)

"Going forward, the team has identified further hardening, like adding two-factor authentication, as a future enhancement. That wraps up the mitigation side from my end — back to [next member]'s part."

---

## Quick checklist before recording
- [ ] Replace `[Your Name]` and `[next member]`
- [ ] Have the app running locally and logged in as two different test users for the demos
- [ ] Practice the CSRF-token-removal demo once beforehand (dev tools step can be fiddly)
- [ ] Keep total time under 10 minutes — trim demos if running long
