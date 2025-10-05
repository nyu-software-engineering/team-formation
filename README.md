# Team Formation

This script takes a CSV file with student project preferences and groups the students into project teams. The output with all students' team assignments is saved to a new CSV file.

## Input data

The input CSV file should have the following fields:

- `Timestamp`
- `Email Address`
- `Are any of these choices your own project proposal?`
- `First choice`
- `Second choice`
- `Third choice`

Example header row:

```csv
Timestamp,Email Address,Are any of these choices your own project proposal?,First choice,Second choice,Third choice
```

## Output data

The resulting output CSV file with student team assignments will have the following fields:

- `email`
- `project`
- `own` (indicates if the project is the student's own proposal)

Example header row:

```csv
email,project,own
```

## Grouping logic

Unique project names are identified. For each project, students who selected it as one of their choices are sorted according to the following priority list:

1. The project is their first choice and is their own proposal.
2. The project is their first choice.
3. The project is their second choice and is their own proposal.
4. The project is their second choice.
5. The project is their third choice and is their own proposal.
6. The project is their third choice.

Students in this list are then assigned to the project until the maximum group size is reached. The process is repeated for all subsequent projects. Any students who are not assigned to a project by the end are placed into random groupings.

## Setup

Install dependencies into a virtual environment with [pipenv](https://pipenv.pypa.io/en/latest/), e.g. `pipenv shell`. Place the input data file into the `data` directory. Open the `project-groups.ipynb` file in a Jupyter notebook environment and execute all code.
