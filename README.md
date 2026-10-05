# Team Formation

This notebook takes a CSV file with student project preferences and groups the students into project teams. The output with all students' team assignments is saved to a new CSV file.

## Input data

### Student preferences

A CSV export of the Google Form where students choose their projects. It should have the following fields:

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

The export may include responses from past semesters, and duplicate columns left over from earlier versions of the form (which pandas reads as `First choice.1`, `First choice.2`, etc.). Only responses submitted on or after `TERM_START` are used, and for each choice the column with the most answers is used. If a student responded more than once, their latest response is used.

### Project proposals

Each project proposal is a fork of the [proposal repository](https://github.com/nyu-software-engineering/project-proposal), with the project name in its README. On the first run, the notebook fetches the forks from GitHub and saves them to the proposals file, guessing each project's name from its README:

```csv
owner,repo,created,project
```

Review and edit this file by hand, then run the notebook again. In particular:

- fix project names the README parser got wrong (the notebook lists suspicious ones),
- give forks of the same project (e.g. one per teammate) exactly the same `project` name,
- leave `project` blank for forks that are not proposals.

Set the `GITHUB_TOKEN` environment variable if GitHub's rate limit is reached. See [data/proposals.csv.example](data/proposals.csv.example).

### Aliases (optional)

Students type project names by hand. The notebook matches votes to proposals by link to the fork, or by name ignoring case, spacing, punctuation, and anything after ` - `, `:`, or `(`. Votes that still do not match are listed by the notebook, with a suggested proposal. Add any that are misspellings to the aliases file:

```csv
vote,project
```

See [data/aliases.csv.example](data/aliases.csv.example). Votes that do not match any of this semester's proposals (e.g. projects from a previous semester) are ignored.

### Roster (optional)

A CSV with an `email` column. Students on the roster who did not fill out the form are placed in teams too.

## Output data

The resulting output CSV file with student team assignments will have the following fields:

- `email`
- `project` (project name in snake case)
- `own` (indicates if the project is the student's own proposal)
- `project_title` (project name as written)
- `choice` (`1st`, `2nd`, `3rd`, `none of their choices`, or `no valid vote`)

Example header row:

```csv
email,project,own,project_title,choice
```

## Grouping logic

1. **Choose teams.** Which projects run, and who is on each, is chosen all at once as an integer program: every student is on at most one of the projects they voted for, every project that runs has between `MIN_TEAM_SIZE` and `MAX_TEAM_SIZE` members, and the total score is as high as possible. A first, second, or third choice is worth `CHOICE_WEIGHTS` points, plus `OWN_PROPOSAL_BONUS` if it is the student's own proposal.
2. **Group students who could not be placed.** Students who voted but whose choices could not run are grouped into new teams of about `TARGET_TEAM_SIZE`. Any 3 or more who voted for the same project start a team together, and the rest are added so that teammates have as many votes in common as possible. Each new team works on the project its members voted for most.
3. **Place students with no valid votes.** Each joins one of the smallest teams, chosen at random (with a fixed seed, so results are reproducible).

## Setup

Install dependencies into a virtual environment with [pipenv](https://pipenv.pypa.io/en/latest/), e.g. `pipenv install` and `pipenv shell`. Place the input data file into the `data` directory. Open the `project-groups.ipynb` file in a Jupyter notebook environment, update the file names and dates in the **Settings** cells for the current semester, and execute all code. Review the output of the *Review* cells, update the proposals and aliases files as needed, and run again.
