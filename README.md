# daily-tracker
Daily tracking app with correct/wrong buttons and analytics dashboard

## Usage

Open `index.html` in any browser (desktop or phone).

- **Correct** (green) and **Wrong** (red): press to count a click.
- Clicks are saved per calendar day in the browser's local storage. Each day's totals are kept, and the counters start again at 0 at midnight.
- The stats panel shows all-time totals for each button, the change from your first tracked day to today, and today vs. yesterday.
- **Daily results** shows a line chart of correct vs. wrong for each day (last 30 days), plus a table with totals and accuracy.
- **Undo last** removes the most recent click, **Export CSV** downloads all daily totals, and **Clear data** wipes everything (after a confirmation).

There are no dependencies and no build step.
