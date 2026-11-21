# BudgetCash – Assignment 4 Testing Prototype

**Course:** CSCI 5801 Software Engineering, Youngstown State University, Fall 2026
**Project team:** Sonam Dorje Lama

This repository contains a working subset of BudgetCash (BudgetWise), the personal budget and expense tracking app proposed in Assignments 2 and 3. It implements two features from the Assignment 3 design:

- **Add an expense and check the budget.** The expense is validated and saved, the category's monthly spending is compared with its budget, and an alert is raised only when the budget moves to a higher level (green → amber at 80% → red at 100%).
- **Monthly summary report.** Total income, total expenses, balance, the top three spending categories and the change from the previous month.

It also implements the business rules behind them: exact money handling in whole paisa, transaction validation, budget status levels and alert state changes. This is a testable backend prototype, not a claim that the full BudgetCash application is finished.

## Requirements

- Python 3.9 or newer (on Windows, tick **"Add python.exe to PATH"** when installing)
- Nothing else: the tests use only the Python standard library (`unittest`, `sqlite3`)

No `pip install`, internet connection, database server, API key or live external service is needed to run these tests.

## Download and run the tests

Open a terminal in the repository root (the folder containing `run_tests.bat`, `budgetcash` and `tests`):

```
git clone https://github.com/this-is-shet/csci_assignment4.git
cd csci_assignment4
python --version
python -m unittest discover -s tests -t . -v
```

Expected after a successful run:

```
Ran 45 tests in ...s

OK
```

The lines above are the expected outcome, not a substitute for the real output. Use your own run and screenshots as evidence.

**Without git:** on the GitHub page click **Code → Download ZIP**, right-click the downloaded file and choose **Extract All** (the tests cannot run from inside the zip), then open a terminal in the extracted folder.

**Windows shortcut:** double-click `run_tests.bat`. It runs the unit tests, then the component tests, and keeps the window open until you press a key.

## Run each test category separately

```
# Four isolated units, 28 tests total
python -m unittest tests.test_unit -v

# Two components, 17 tests total
python -m unittest tests.test_component -v
```

If `python` is not recognised on Windows, use `py` instead (for example `py -m unittest tests.test_unit -v`).

## What is tested?

| Category | Test class | Tests | Scope |
| --- | --- | --- | --- |
| Unit | `U1MoneyUtilTests` | 7 | Exact paisa conversion, 2 vs 3 decimals, lowest values, every error message, very large values, display format |
| Unit | `U2TransactionValidatorTests` | 7 | Amount boundaries (0/1 and max/max+1), today vs tomorrow, 200 vs 201-character notes, wrong or missing category, all errors at once |
| Unit | `U3BudgetStatusTests` | 7 | Green/amber/red at exactly 80% and 100%, overspending, nothing spent, invalid limits |
| Unit | `U4AlertServiceTests` | 7 | State changes NONE → APPROACHING → EXCEEDED, no repeated alerts, drop back after delete, storage failure, missing category |
| Component | `C1AddExpenseComponentTests` | 9 | TransactionService + validator + BudgetService + AlertService + repositories + SQLite |
| Component | `C2MonthlySummaryComponentTests` | 8 | ReportService + repository SQL queries + SQLite, with data entered through TransactionService |
| **Total** | | **45** | **28 unit and 17 component tests** |

Test cases were chosen with the Chapter 8 techniques: partition and boundary testing, testing guidelines, object state testing, and interface testing (misuse, misunderstanding and timing errors).

Each unit test calls the unit directly, without a database. AlertService's repositories are replaced by small hand-written fakes. Each component test uses the real services and repositories on a **new temporary SQLite database file**, which is deleted after the test. "Today" is fixed at 19 Nov 2026, so results are the same on every run, and no real data is touched.

## Relationship to Assignment 3 and honest implementation changes

The Assignment 3 design proposed a layered application (presentation, application logic, data access) with TransactionService, BudgetService, AlertService and ReportService. This implementation keeps the application-logic and data-access layers and the rules from the "Add expense transaction" sequence diagram and the budget state diagram, but intentionally narrows the first working scope:

- **Implemented:** transaction add/edit/delete in one database transaction, validation, budget status levels, alerts without repeats, monthly summary with previous-month comparison, per-user data separation, automated tests.
- **Language and storage:** Python with SQLite, the same approach as the course's WeatherReportDemo. Money is stored as whole paisa (`int`), as the design required.
- **Added rules:** maximum amount Rs 10,000,000 per transaction, 200-character notes, no future dates.
- **Not implemented:** user interface, log-in and password hashing (AuthService), category management screens, data export.

This restricted scope is intentional and consistent with Assignment 4, which requires tested working features rather than the entire system.

## Bugs found and fixed

- **Budget not saved (found by C1-05).** `BudgetService.set_budget()` did not commit the new budget, so if the next expense failed, the budget was lost too. Fixed by running `set_budget()` and `roll_over_budgets()` in their own database transaction.
- **Alert text (found by U4-07).** When a category was missing, the alert read "Budget budget at 80%". Fixed so it reads "Budget at 80%".

## Remaining limitations

- No system, user-interface or performance tests yet, because there is no user interface.
- `roll_over_budgets()` is not yet covered by a test.
- Setting a budget below what is already spent raises no alert until the next expense (found by C1-06); the design does not say whether that is wanted.
- Tests use SQLite files; a production database server and many simultaneous users were not tested.

## Source layout

```
budgetcash/
  domain.py               Transaction, Category, Budget, Alert, MonthlySummary, levels
  money_util.py           Unit 1 – paisa conversion and display
  validation.py           Unit 2 – transaction validation rules
  budget_service.py       Unit 3 calculate_status, plus BudgetService
  alert_service.py        Unit 4 – AlertService
  transaction_service.py  Component 1 – add / edit / delete expense and check budget
  report_service.py       Component 2 – monthly summary
  repositories.py         SQLite data access (every query filtered by user)
  db.py                   Database schema and new-user setup
tests/
  test_unit.py            28 isolated unit tests
  test_component.py       17 component tests on temporary SQLite databases
run_tests.bat             Windows: run both test levels
.vscode/settings.json     VS Code test explorer settings
```
