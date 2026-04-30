# 🏕️ Camping Planning System — Architecture & Design

## High-Level Architecture

```
┌─────────────────────────────────────────────────┐
│              React Frontend (HTML/JS)            │
│  Schedule | Shopping | Expenses | Signups        │
└────────────────────┬────────────────────────────┘
                     │ fetch() — JSON over HTTPS
┌────────────────────▼────────────────────────────┐
│        Google Apps Script Web App               │
│     doGet(e) / doPost(e) — REST-style API       │
└────────────────────┬────────────────────────────┘
                     │ SpreadsheetApp SDK
┌────────────────────▼────────────────────────────┐
│           Google Sheets (6 tabs)                │
│  Schedule | Shopping | Volunteers | Families    │
│  Expenses | Signups                             │
└─────────────────────────────────────────────────┘
```

**Why Apps Script as middleware?**
- Zero infrastructure to manage (runs on Google's servers)
- No OAuth friction for small teams — deploy as "Anyone" access
- Read + Write to Sheets via native SDK
- Free, instant deployment with one URL

---

## Google Sheet Schema

### Tab 1: `Schedule`
| Column | Type | Description |
|--------|------|-------------|
| Day | Number | 1, 2, 3... |
| Meal | String | Breakfast / Lunch / Dinner / Activity |
| Item | String | e.g. "Pancakes", "Hiking", "BBQ" |
| Details | String | Notes, ingredients, location |
| Assigned Volunteer | String | Who's responsible |
| Status | String | Planned / Done |

### Tab 2: `ShoppingList`
| Column | Type | Description |
|--------|------|-------------|
| ID | String | Unique row ID (auto) |
| Meal | String | Breakfast / Lunch / Dinner / Supplies |
| Category | String | Groceries / Produce / Dairy / Supplies |
| Item | String | e.g. "Eggs" |
| Quantity | String | e.g. "2 dozen" |
| Store | String | Costco / Trader Joe's / etc. |
| Volunteer | String | Who is buying it |
| Status | String | Pending / Purchased |
| Cost | Number | Actual cost entered by volunteer |

### Tab 3: `Volunteers`
| Column | Type | Description |
|--------|------|-------------|
| Name | String | Volunteer name |
| Family | String | Family they belong to |
| Assigned Store | String | Which store they're responsible for |
| Phone | String | Optional contact |

### Tab 4: `Families`
| Column | Type | Description |
|--------|------|-------------|
| Family Name | String | e.g. "The Smiths" |
| Members | Number | Total headcount |
| Contact | String | Primary contact name |
| Email | String | Optional |

### Tab 5: `Expenses`
| Column | Type | Description |
|--------|------|-------------|
| Volunteer | String | Who made the purchase |
| Store | String | Where it was purchased |
| Item | String | What was bought |
| Amount | Number | Cost in dollars |
| Receipt | String | Optional photo URL or note |
| Date | String | Date of purchase |

### Tab 6: `Signups`
| Column | Type | Description |
|--------|------|-------------|
| Name | String | Participant name |
| Family | String | Family group |
| Members | Number | Number in their party |
| Email | String | Contact email |
| Dietary Notes | String | Restrictions/allergies |
| Signed Up At | String | Timestamp |

---

## API Endpoints (Apps Script)

All requests go to one URL: `https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec`

### GET Requests
| Action | URL Params | Returns |
|--------|-----------|---------|
| Get schedule | `?action=getSchedule` | All schedule rows |
| Get shopping list | `?action=getShopping` | All shopping items |
| Get volunteers | `?action=getVolunteers` | All volunteers |
| Get families | `?action=getFamilies` | Families + headcounts |
| Get expenses | `?action=getExpenses` | All expenses |
| Get signups | `?action=getSignups` | All signups |
| Get summary | `?action=getSummary` | Totals: cost, headcount, split |

### POST Requests
| Action | Body | Effect |
|--------|------|--------|
| Add signup | `{action:"addSignup", data:{...}}` | Appends row to Signups |
| Update item status | `{action:"updateStatus", id, status}` | Updates ShoppingList row |
| Add expense | `{action:"addExpense", data:{...}}` | Appends to Expenses |
| Update schedule item | `{action:"updateSchedule", row, data}` | Updates Schedule row |
| Add shopping item | `{action:"addShoppingItem", data:{...}}` | Appends to ShoppingList |

---

## Frontend Component Structure

```
App
├── Header (nav tabs)
├── ScheduleView
│   ├── DayCard (repeats per day)
│   │   ├── MealRow (Breakfast/Lunch/Dinner)
│   │   └── ActivityRow
│   └── AddMealModal
├── ShoppingView
│   ├── FilterBar (by meal, category, volunteer)
│   ├── ShoppingItem (item card with status toggle)
│   └── AddItemModal
├── ExpensesView
│   ├── TotalSummaryCard
│   ├── StoreBreakdownTable
│   ├── FamilySplitTable
│   └── AddExpenseModal
├── SignupsView
│   ├── HeadcountBanner
│   ├── FamilyCard (repeats per family)
│   └── SignupForm
└── VolunteersView
    ├── VolunteerCard (name, store, items assigned)
    └── AssignmentModal
```

---

## Cost Splitting Logic

```
Total Cost = SUM(Expenses.Amount)
Total Participants = SUM(Signups.Members)
Cost Per Person = Total Cost / Total Participants

Per Family:
  Family Cost = Family.Members × Cost Per Person
```

---

## Deployment Steps

1. **Set up Google Sheet** — Create the 6 tabs above with headers
2. **Add Apps Script** — Tools → Apps Script → paste Code.gs
3. **Deploy as Web App** — Execute as "Me", access "Anyone"
4. **Copy the Web App URL** into the frontend config
5. **Open index.html** in a browser or host on GitHub Pages / Netlify

---

## Security Notes

- Deploy Apps Script with "Anyone can access" for simplicity (suitable for small private groups)
- Share the Sheet URL only with organizers
- For production use, switch to OAuth + service accounts
