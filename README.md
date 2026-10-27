# Cheat Sheet: How to Finish the School Schedule Manager

You already did the hardest part: **all six models are ready** (`Subject`, `Teacher`, `SchoolClass`, `Student`, `Schedule`, `Grade`). 🎉

Now we build the **console program** on top of them, test it, and put the project on GitHub.

> 🧭 **Work in small steps.** Write **one** function → run the program → test it → make a Git commit. Don't write everything at once!

---

## The Plan (check each box when done)

- [ ] **Step 1.** Check that the models and the database are OK
- [ ] **Step 2.** Create the folders and the file for the console program
- [ ] **Step 3.** Write the menu (the "skeleton")
- [ ] **Step 4.** Mandatory part: `add_subject`, `add_teacher`, `add_class`, `add_student`
- [ ] **Step 5.** Extra part: `add_schedule`, `add_grade`
- [ ] **Step 6.** Test everything
- [ ] **Step 7.** Put the project on GitHub (commits + Pull Request)
- [ ] **Step 8.** Prepare a short presentation

---

## Step 1. Check the models and the database

Open the terminal in your project folder (the one with `manage.py`) and make sure the virtual environment is on (you see `(venv)` at the start of the line).

```bash
python manage.py makemigrations
python manage.py migrate
```

- If you see `No changes detected` — **that is fine**, it means everything is already saved.
- If you see an error — read the **last line** of it. Most often: a typo in `models.py`, or `'school'` is missing in `INSTALLED_APPS` (file `config/settings.py`).

> ℹ️ In this cheat sheet the app is called `school`. If **your** app has another name, replace `school` with your name everywhere (in folder names and in `from school.models import ...`).

**Quick test in the shell** (to be sure the tables work):

```bash
python manage.py shell
```

```python
>>> from school.models import Subject
>>> Subject.objects.create(name="Math")
<Subject: Math>
>>> Subject.objects.all()
>>> exit()
```

---

## Step 2. Create the files for the console program

We use a Django **custom command**. Django has a rule: *a file inside `management/commands/` becomes a command* that you run with `python manage.py <file name>`.

📁 Create **exactly** this structure inside the `school` folder:

```
school/
├── management/
│   ├── __init__.py              ← empty file
│   └── commands/
│       ├── __init__.py          ← empty file
│       └── school_menu.py       ← our program
```

**Why `__init__.py`?** An empty file with this name tells Python: *"this folder is a package"*. Without it Django will say `Unknown command: 'school_menu'`.

---

## Step 3. The menu (skeleton)

📁 **File:** `school/management/commands/school_menu.py`

Start with this. The functions don't exist yet, so for now we only make the menu and **one** function:

```python
from django.core.management.base import BaseCommand

from school.models import Subject, Teacher, SchoolClass, Student


def add_subject():
    print("TODO: add subject")     # we will replace this in Step 4


class Command(BaseCommand):
    help = "Console menu for the School Schedule Manager"

    def handle(self, *args, **options):
        while True:
            print("\n=== SCHOOL MANAGER ===")
            print("1. Add subject")
            print("0. Exit")

            choice = input("Choose an option: ").strip()

            if choice == "1":
                add_subject()
            elif choice == "0":
                print("Goodbye! 👋")
                break
            else:
                print("❌ Unknown option, please try again.")
```

**What does it mean?**

| Line | Meaning |
|---|---|
| `from django.core.management.base import BaseCommand` | import the base class for commands |
| `from school.models import ...` | import our models so we can use the database |
| `class Command(BaseCommand):` | the class **must be called exactly `Command`** — Django looks for this name |
| `def handle(self, *args, **options):` | the method Django **runs** when you start the command |
| `while True:` | an endless loop — the menu appears again and again |
| `choice = input(...)` | the user's choice (it is **text**, so we compare with `"1"`, not `1`) |
| `if / elif / else` | choose what to do depending on the choice |
| `break` | leave the loop → the program ends |

**Run it:**

```bash
python manage.py school_menu
```

Type `1` → you should see `TODO: add subject`. Type `0` → the program stops. ✅

> 💾 **Make your first commit now** (see Step 7 for the commands).

---

## Step 4. The mandatory part

We write the four functions. **Every function has the same plan:**

```
1. Ask the user (input)
2. Check the data           ← if something is wrong: print ❌ and return
3. Save to the database     ← Model.objects.create(...)
4. Print ✅
```

### 🔑 The `return` trick

`return` **stops the function immediately**. So we write: *check for a problem → print the error → `return`*. If all the checks pass, the code reaches the saving part.

### 4.1 `add_subject` (full code)

Replace the `TODO` version with this:

```python
def add_subject():
    name = input("Subject name: ").strip()
    description = input("Description: ").strip()

    if not name:
        print("❌ The name cannot be empty.")
        return

    if Subject.objects.filter(name=name).exists():
        print("❌ A subject with this name already exists.")
        return

    Subject.objects.create(name=name, description=description)
    print(f"✅ Subject '{name}' added.")
```

| Line | Meaning |
|---|---|
| `input("Subject name: ")` | show a question, wait for the answer (returns text) |
| `.strip()` | remove spaces at the beginning and the end: `"  Math "` → `"Math"` |
| `if not name:` | an empty text counts as `False`, so this means "the name is empty" |
| `Subject.objects.filter(name=name).exists()` | "is there already a subject with this name?" → `True` / `False`. **This is the uniqueness check** from the assignment |
| `Subject.objects.create(...)` | save the new subject (SQL `INSERT`) |
| `f"... '{name}' ..."` | an f-string: puts the value of `name` inside the text |

### 4.2 `add_teacher` (full code)

```python
def add_teacher():
    first_name = input("Teacher first name: ").strip()
    last_name = input("Teacher last name: ").strip()
    subject_name = input("Subject name: ").strip()

    if not first_name or not last_name:
        print("❌ First name and last name cannot be empty.")
        return

    try:
        subject = Subject.objects.get(name=subject_name)
    except Subject.DoesNotExist:
        print("❌ This subject does not exist. Add it first.")
        return

    Teacher.objects.create(
        first_name=first_name,
        last_name=last_name,
        subject=subject,
    )
    print(f"✅ Teacher {first_name} {last_name} added ({subject.name}).")
```

| Line | Meaning |
|---|---|
| `if not first_name or not last_name:` | error if **at least one** of them is empty |
| `try: ... except Subject.DoesNotExist:` | "try this; if `get()` finds nothing, do the `except` part instead of crashing" |
| `Subject.objects.get(name=subject_name)` | find **one** subject by its name. This is **the existence check** |
| `subject=subject` | the teacher's `ForeignKey` needs a **`Subject` object**, not a text. After the check we have it in the variable `subject` |

### 4.3 `add_class` — fill in the blanks ✏️

The structure is the same as `add_subject`. Replace every `___`:

```python
def add_class():
    name = input("Class name (for example 7-A): ").strip()
    study_year_text = input("Year of study (number): ").strip()

    if not name:
        print("❌ The name cannot be empty.")
        return

    if not study_year_text.___():        # hint: a text method that checks "only digits"
        print("❌ The year must be a number.")
        return

    if SchoolClass.objects.___(name=name).___():     # same check as in add_subject
        print("❌ A class with this name already exists.")
        return

    SchoolClass.objects.___(name=name, study_year=___(study_year_text))
    print(f"✅ Class {name} added.")
```

**Hints:**
- The text method for digits is `.isdigit()`.
- `input()` always gives **text**. Your model needs a **number**, so convert it with `int(...)`.
- Look at `add_subject` for the other two blanks.

### 4.4 `add_student` — fill in the blanks ✏️

The structure is the same as `add_teacher`:

```python
def add_student():
    first_name = input("Student first name: ").strip()
    last_name = input("Student last name: ").strip()
    class_name = input("Class name: ").strip()

    if not first_name or not last_name:
        print("❌ First name and last name cannot be empty.")
        return

    try:
        school_class = SchoolClass.objects.___(name=class_name)
    except SchoolClass.___:
        print("❌ This class does not exist. Add it first.")
        return

    Student.objects.create(
        first_name=first_name,
        last_name=last_name,
        school_class=___,
    )
    print(f"✅ Student {first_name} {last_name} added to {class_name}.")
```

**Hints:**
- To find **one** object: `.get(...)`.
- The error name has the form `ModelName.DoesNotExist`.
- The last blank is the **object** you found in the `try` part (not the text `class_name`!).

### 4.5 Update the menu

After each new function, add it to the menu. The final menu for Step 4:

```python
def handle(self, *args, **options):
    while True:
        print("\n=== SCHOOL MANAGER ===")
        print("1. Add subject")
        print("2. Add teacher")
        print("3. Add class")
        print("4. Add student")
        print("0. Exit")

        choice = input("Choose an option: ").strip()

        if choice == "1":
            add_subject()
        elif choice == "2":
            add_teacher()
        elif choice == "3":
            add_class()
        elif choice == "4":
            add_student()
        elif choice == "0":
            print("Goodbye! 👋")
            break
        else:
            print("❌ Unknown option, please try again.")
```

> ✅ After each function: run `python manage.py school_menu`, test the function (a good case **and** a bad case), then commit.

---

## Step 5. The extra part: schedule and grades

These functions have **more questions** and **more checks**, but no new ideas. Everything comes from Step 4 plus two new tools below.

### 5.0 Two new tools

**Tool 1 — `.first()` (another way to check "does it exist?")**

```python
subject = Subject.objects.filter(name=subject_name).first()
if subject is None:
    print("❌ Subject not found.")
    return
```

`.first()` returns the first object found, or **`None`** if nothing was found. It is the same as `try/except`, just shorter.

**Tool 2 — turning text into a time or a date**

At the **top** of the file add:

```python
from datetime import datetime
```

Then:

```python
start_time = datetime.strptime(time_text, "%H:%M").time()        # "09:00"  → time
grade_date = datetime.strptime(date_text, "%Y-%m-%d").date()     # "2026-10-26" → date
```

If the user types something in a wrong format, Python raises a `ValueError`. We catch it:

```python
try:
    start_time = datetime.strptime(time_text, "%H:%M").time()
except ValueError:
    print("❌ The time must look like 09:00.")
    return
```

| Code | Meaning |
|---|---|
| `%H:%M` | hours:minutes (`09:00`) |
| `%Y-%m-%d` | year-month-day (`2026-10-26`) |
| `.time()` / `.date()` | keep only the time / only the date part |

Also don't forget to update the import of models at the top:

```python
from school.models import Subject, Teacher, SchoolClass, Student, Schedule, Grade
```

### 5.1 `add_schedule` — template ✏️

**Ask for:** day (`1`–`5`), start time, subject name, class name, teacher last name.

```python
def add_schedule():
    weekday_text = input("Day of the week (1 = Monday ... 5 = Friday): ").strip()
    time_text = input("Start time (HH:MM, for example 09:00): ").strip()
    subject_name = input("Subject name: ").strip()
    class_name = input("Class name: ").strip()
    teacher_last_name = input("Teacher last name: ").strip()

    # CHECK 1: the day is one of "1", "2", "3", "4", "5"
    if weekday_text not in ["1", "2", "3", "4", "5"]:
        print("❌ The day must be a number from 1 to 5.")
        return

    # CHECK 2: the time has the right format
    # TODO: use try / except ValueError with datetime.strptime (see Tool 2)

    # CHECK 3: the subject exists
    # TODO: use .filter(...).first() and `if subject is None` (see Tool 1)

    # CHECK 4: the class exists
    # TODO: the same, with SchoolClass and class_name

    # CHECK 5: the teacher exists (search by last_name)
    # TODO: the same, with Teacher and teacher_last_name

    # SAVE
    Schedule.objects.create(
        weekday=int(weekday_text),
        start_time=start_time,
        subject=subject,
        school_class=school_class,
        teacher=teacher,
    )
    print("✅ Lesson added to the schedule.")
```

**Remember:**
- `weekday` in the model is a **number**, but `input()` gives **text** → use `int(weekday_text)`.
- `subject`, `school_class`, `teacher` must be **objects** (the ones you found), not text.

### 5.2 `add_grade` — template ✏️

**Ask for:** student's first name, last name, subject name, grade, date.

```python
def add_grade():
    first_name = input("Student first name: ").strip()
    last_name = input("Student last name: ").strip()
    subject_name = input("Subject name: ").strip()
    value_text = input("Grade (1-10): ").strip()
    date_text = input("Date (YYYY-MM-DD): ").strip()

    # CHECK 1: the student exists (search by first_name AND last_name)
    # TODO: Student.objects.filter(first_name=..., last_name=...).first()

    # CHECK 2: the subject exists
    # TODO

    # CHECK 3: the grade is a number from 1 to 10
    if not value_text.isdigit() or not 1 <= int(value_text) <= 10:
        print("❌ The grade must be a number from 1 to 10.")
        return

    # CHECK 4: the date has the right format
    # TODO: try / except ValueError with datetime.strptime(date_text, "%Y-%m-%d").date()

    # SAVE
    Grade.objects.create(
        student=student,
        subject=subject,
        value=int(value_text),
        date=grade_date,
    )
    print("✅ Grade added.")
```

> ⚠️ **Why do we check the grade by hand?** The validators in the model (`MinValueValidator`, `MaxValueValidator`) are **not** checked by `.create()`. Without our own check, `Grade.objects.create(value=99, ...)` would save a wrong grade!
>
> 💡 `1 <= int(value_text) <= 10` is a chain of comparisons: "1 is less than or equal to the grade, and the grade is less than or equal to 10". Python allows this.
>
> ⚠️ **Order of checks:** we check `.isdigit()` **before** `int(...)`. The word `or` stops as soon as the first part is `True`, so `int("abc")` never runs.

### 5.3 The final menu

```python
print("\n=== SCHOOL MANAGER ===")
print("1. Add subject")
print("2. Add teacher")
print("3. Add class")
print("4. Add student")
print("5. Add lesson to schedule")
print("6. Add grade")
print("0. Exit")
```

and two new branches:

```python
elif choice == "5":
    add_schedule()
elif choice == "6":
    add_grade()
```

---

## Step 6. Test everything

Go through this table. Tick each row. If something doesn't work as expected — fix it.

| # | What to test | Expected result | ✔ |
|---|--------------|-----------------|---|
| 1 | Add subject "Math" | ✅ success | |
| 2 | Add subject "Math" again | ❌ already exists | |
| 3 | Add subject with an empty name | ❌ cannot be empty | |
| 4 | Add teacher with a subject that does not exist | ❌ subject does not exist | |
| 5 | Add teacher with an existing subject | ✅ success | |
| 6 | Add class "7-A", then "7-A" again | ✅, then ❌ already exists | |
| 7 | Add class with year "abc" | ❌ year must be a number | |
| 8 | Add student to a class that does not exist | ❌ class does not exist | |
| 9 | Add student to an existing class | ✅ success | |
| 10 | Add lesson with day "7" | ❌ wrong day | |
| 11 | Add lesson with time "25:99" | ❌ wrong time format | |
| 12 | Add lesson with everything correct | ✅ success | |
| 13 | Add grade "15" | ❌ grade must be 1–10 | |
| 14 | Add grade with everything correct | ✅ success | |
| 15 | Choose menu option "9" | ❌ unknown option | |

**Want to see what is saved?** Use the shell:

```bash
python manage.py shell
```

```python
>>> from school.models import Schedule, Grade
>>> Schedule.objects.all()
>>> Grade.objects.all()
```

Thanks to `__str__` you will see readable lines like `Monday - 09:00 - 7-A - Math`.

---

## Step 7. GitHub: commits and a Pull Request

The assignment asks for a GitHub repository with **commits** and **Pull Requests**. Then you send the **link** to your mentor.

### 7.1 Prepare two files in the project root (next to `manage.py`)

**`.gitignore`** — files that must **not** go to GitHub:

```
venv/
__pycache__/
*.pyc
db.sqlite3
.vscode/
.idea/
```

**`requirements.txt`** — the list of libraries:

```bash
pip freeze > requirements.txt
```

### 7.2 The first commit and the first push

1. On github.com click **New repository**. Give it a name (for example `school-manager`). Leave it **empty** (no README). Click **Create repository**.
2. In the terminal:

```bash
git init
git add .
git commit -m "Add models for school schedule project"
git branch -M main
git remote add origin https://github.com/YOUR-NAME/school-manager.git
git push -u origin main
```

| Command | Meaning |
|---|---|
| `git init` | turn the folder into a Git repository |
| `git add .` | choose **all** changed files for the next commit |
| `git commit -m "..."` | save a snapshot with a short message |
| `git branch -M main` | name the main branch `main` |
| `git remote add origin URL` | connect your folder to the GitHub repository |
| `git push -u origin main` | upload the commits to GitHub |

### 7.3 Work in a branch and make a Pull Request

Do **not** write new features directly in `main`. Use a branch:

```bash
git switch -c feature/console-menu          # create a branch and switch to it
```

Now write your functions. After **each** finished function:

```bash
git add .
git commit -m "Add add_subject and add_teacher functions"
```

When the work in the branch is done:

```bash
git push -u origin feature/console-menu
```

Then:

1. Open your repository on GitHub.
2. Click the green button **Compare & pull request**.
3. Write a short description: *what did you add?*
4. Click **Create pull request**, then **Merge pull request**.

Next feature → new branch: `git switch main`, `git pull`, then `git switch -c feature/schedule-and-grades` and repeat.

### 7.4 Good commit messages

| ✅ Good | ❌ Bad |
|---|---|
| `Add Student model` | `fix` |
| `Add teacher validation` | `aaa` |
| `Fix grade range check` | `final` |

Make **many small commits** (after every function), not one giant commit at the end.

---

## Step 8. Prepare a short presentation (2–3 minutes)

Be ready to tell your mentor:

1. **What the project does** — in one sentence.
2. **Which models you made** and how they are connected (`Teacher → Subject`, `Student → SchoolClass`, ...).
3. **Why `PROTECT` and `CASCADE`** — for example: *"You cannot delete a subject while teachers use it, but if a student is deleted, their grades are deleted too."*
4. **A live demo**: one successful action and one error check.
5. **What was the hardest part** and what you learned.

---

## 🆘 Common Errors and How to Fix Them

| Error message | What it means | How to fix |
|---|---|---|
| `Unknown command: 'school_menu'` | Django can't find your command | Check the folders `management/commands/`, the two `__init__.py` files, and the file name |
| `Command must be a subclass of BaseCommand` / `has no attribute 'Command'` | the class has a wrong name | The class must be called exactly `Command` and inherit from `BaseCommand` |
| `No changes detected` | Django doesn't see model changes | Save `models.py`; check `'school'` in `INSTALLED_APPS` |
| `ModuleNotFoundError: No module named 'school'` | wrong app name in the import | Use the real name of your app folder |
| `NameError: name 'Schedule' is not defined` | model not imported | Add it to `from school.models import ...` |
| `NameError: name 'datetime' is not defined` | forgot an import | Add `from datetime import datetime` at the top |
| `ValueError: invalid literal for int()` | you called `int()` on a non-number | Check with `.isdigit()` **before** `int(...)` |
| `Cannot assign "'Math'": "Teacher.subject" must be a "Subject" instance` | you passed text to a `ForeignKey` | Pass the **object** you found, not the text |
| `ProtectedError` | you tried to delete something that is used | That's `PROTECT` doing its job — delete the related rows first |
| `IndentationError` | wrong spaces at the start of a line | Use 4 spaces for each level; don't mix tabs and spaces |

---

## ✅ Final Checklist (before you send the link)

- [ ] `python manage.py migrate` works without errors
- [ ] `python manage.py school_menu` starts and shows the menu with 6 options + exit
- [ ] All 15 tests from Step 6 pass
- [ ] The repository is on GitHub and has **several** commits
- [ ] There is at least **one merged Pull Request**
- [ ] `.gitignore` and `requirements.txt` are in the repository (and `venv/` and `db.sqlite3` are **not**)
- [ ] I can explain my models and the arrows between them
- [ ] I sent the link to my mentor

Good luck! 🚀 If you get stuck — read the **last line** of the error, check the table above, and write to your teacher.
