
# README.md

## Team Table Allotment Generator (Google Sheets + Apps Script)

This script processes **hackathon/team registration data** and generates a **clean team-level table** from individual member entries.

It groups rows by **team number** and produces a structured sheet for **table allotment or logistics management**.

---

# Features

* Groups members by **Team Number**
* Calculates **team size automatically**
* Supports **up to 6 members per team**
* Fills missing members with **`nil`**
* Detects **Internal / External participants**
* Creates a formatted output sheet named:

```
table alottment data
```

---

# Required Input Format (Sheet Template)

Your sheet must contain the following columns **in this exact order**.

| Column | Header                     |
| ------ | -------------------------- |
| A      | Team Name                  |
| B      | Team Number                |
| C      | Number of Members          |
| D      | Member Name                |
| E      | Registration Number        |
| F      | Email                      |
| G      | College                    |
| H      | Phone Number               |
| I      | Gender                     |
| J      | Residence/Schooling Status |
| K      | Overnight Stay             |
| L      | Problem Statement          |

Example:

| Team Name | Team Number | Members | Member Name   | Reg No       | Email | College                     | Phone | Gender | Residence | Stay | Problem |
| --------- | ----------- | ------- | ------------- | ------------ | ----- | --------------------------- | ----- | ------ | --------- | ---- | ------- |
| Novastra  | 1           | 6       | Ajai Seelan R | 310624106007 | email | Easwari Engineering College | phone | Male   | External  | Yes  | GKM1    |

Each **member should be in a separate row**.

---

# How to Run

## Step 1 — Upload CSV

Import your CSV into **Google Sheets**.

```
File → Import → Upload CSV
```

---

## Step 2 — Open Apps Script

Go to

```
Extensions → Apps Script
```

---

## Step 3 — Add Script

Create a file called:

```
Code.gs
```

Paste your script:

```javascript
function formatTeams() {

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sourceSheet = ss.getActiveSheet();
  const data = sourceSheet.getDataRange().getValues();

  const rows = data.slice(1);
  const teams = {};

  rows.forEach(r => {

    const teamName = r[0];
    const teamNumber = r[1];
    const memberName = r[3];
    const college = r[6];
    const problem = r[11];

    if (!teams[teamNumber]) {
      teams[teamNumber] = {
        team_name: teamName,
        team_number: teamNumber,
        problem_statement: problem,
        members: [],
        type: "internal"
      };
    }

    teams[teamNumber].members.push(memberName);

    if (!college.toLowerCase().includes("vit")) {
      teams[teamNumber].type = "external";
    }

  });

  const output = [];

  output.push([
    "team_number",
    "team_name",
    "team_size",
    "problem_statement",
    "member1_name",
    "member2_name",
    "member3_name",
    "member4_name",
    "member5_name",
    "member6_name",
    "participant_type"
  ]);

  Object.values(teams).forEach(t => {

    const members = t.members;

    output.push([
      t.team_number,
      t.team_name,
      members.length,
      t.problem_statement,
      members[0] || "nil",
      members[1] || "nil",
      members[2] || "nil",
      members[3] || "nil",
      members[4] || "nil",
      members[5] || "nil",
      t.type
    ]);

  });

  let outputSheet = ss.getSheetByName("table alottment data");

  if (!outputSheet) {
    outputSheet = ss.insertSheet("table alottment data");
  } else {
    outputSheet.clear();
  }

  outputSheet.getRange(1,1,output.length,output[0].length).setValues(output);

}
```

---

## Step 4 — Run Script

Click

```
Run ▶ formatTeams
```

You will be asked to **authorize Google permissions** once.

---

## Step 5 — Output

The script will generate a new sheet:

```
table alottment data
```

Example Output:

| team_number | team_name    | team_size | problem_statement | member1  | member2  | member3    | member4 | member5 | member6 | participant_type |
| ----------- | ------------ | --------- | ----------------- | -------- | -------- | ---------- | ------- | ------- | ------- | ---------------- |
| 1           | Novastra     | 6         | GKM1              | Ajai     | Harini   | Dhanasekar | Ameera  | Akash   | Harini  | external         |
| 2           | Syntax Error | 4         | GKM1              | Poojitha | Sanjayan | Jerita     | Mukesh  | nil     | nil     | internal         |

---

# Folder Structure (if shared in repo)

```
team-table-generator/
│
├── README.md
├── Code.gs
└── sample_registration.csv
```

---

# Notes

* The script assumes **VIT is internal**.
* Any other college is marked **external**.
* Maximum **6 members per team**.

---

