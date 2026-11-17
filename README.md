# Power BI

Power BI projects saved as PBIP (TMDL semantic model + PBIR report), one folder per project.

| Project | Description |
|---|---|
| [First AI Report](First%20AI%20Report/) | Financial Dashboard: star-schema model built in Power Query from `financial_accounting.csv` |

## Working with a project

1. Open the `.pbip` file in Power BI Desktop.
2. Source data files are not committed. Place the CSV next to the `.pbip` file and update the path in the model's staging query if needed.
3. Commit changes from the repo root: `git add -A`, `git commit -m "..."`, `git push`.
