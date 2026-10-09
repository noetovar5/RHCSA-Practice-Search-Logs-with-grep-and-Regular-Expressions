# RHCSA-Practice-Search-Logs-with-grep-and-Regular-Expressions
RHCSA Practice — Search Logs with grep and Regular Expressions

Today’s focused objective: Use grep, pipes, and regular expressions to locate useful information in configuration files and application logs.
Estimated time: 45 minutes
Lab system: RHEL 9 or RHEL 10
Privileges required: None
Safety: Everything stays under /tmp/rhcsa-grep-lab. This lesson does not change networking, storage, boot, authentication, SELinux, firewall rules, or system services.
This is a core RHCSA skill. Red Hat’s current EX200 objectives specifically include using grep and regular expressions to analyze text. Official RHCSA EX200 objectives
1. The concept in plain language — 8 minutes
grep searches text and prints lines matching a pattern.
Basic format:
grep OPTIONS PATTERN FILE

Example:
grep ERROR application.log

This displays lines containing the exact uppercase text ERROR.
Common options
Option	Purpose
-i	Ignore uppercase and lowercase differences
-n	Display matching line numbers
-v	Display lines that do not match
-c	Count matching lines
-w	Match a complete word
-r	Search directories recursively
-l	Display only filenames containing matches
-E	Use extended regular expressions
-A 2	Show two lines after each match
-B 2	Show two lines before each match
-C 2	Show two lines before and after each match


Regular expressions
Regular expressions describe text patterns:
Pattern	Meaning
^ERROR	Line begins with ERROR
failed$	Line ends with failed
.	Any single character
[0-9]	Any one digit
[A-Z]	Any one uppercase letter
*	Previous element occurs zero or more times
+	Previous element occurs one or more times with grep -E
ERROR|WARN	Either pattern with basic grep
`ERROR	WARN`


Quotation marks are a good habit because they prevent the shell from interpreting special characters:
grep -E 'ERROR|WARN' application.log

2. Prepare the lab — 5 minutes
Safety warning
The first command recursively removes only an earlier copy of this temporary lab. Confirm the path is exactly /tmp/rhcsa-grep-lab.
rm -rf /tmp/rhcsa-grep-lab
mkdir -p /tmp/rhcsa-grep-lab/{logs,config,reports}
cd /tmp/rhcsa-grep-lab
pwd

Expected location:
/tmp/rhcsa-grep-lab

Create a realistic application log:
printf '%s\n' \
'2026-09-17 18:00:01 INFO  InventoryApp startup initiated' \
'2026-09-17 18:00:02 INFO  Loading configuration' \
'2026-09-17 18:00:03 WARN  Database response time 850ms' \
'2026-09-17 18:00:04 ERROR Database connection failed' \
'2026-09-17 18:00:05 INFO  Retrying database connection' \
'2026-09-17 18:00:06 INFO  Database connection successful' \
'2026-09-17 18:05:10 INFO  User 1042 authenticated' \
'2026-09-17 18:05:14 ERROR User 1042 transaction failed' \
'2026-09-17 18:06:20 warn  Disk usage reached 78 percent' \
'2026-09-17 18:07:30 INFO  Health check passed' \
'2026-09-17 18:08:40 ERROR API request returned HTTP 503' \
'2026-09-17 18:09:50 INFO  InventoryApp running' \
> logs/inventory.log

Create two configuration files:
printf '%s\n' \
'application=InventoryApp' \
'environment=production' \
'database_server=db01.example.test' \
'database_port=5432' \
'debug=false' \
> config/application.conf

printf '%s\n' \
'monitoring_enabled=true' \
'warning_threshold=75' \
'critical_threshold=90' \
'notification_email=linuxadmin@example.test' \
> config/monitoring.conf

Inspect the starting data:
cat logs/inventory.log
cat config/application.conf
cat config/monitoring.conf

3. Guided practice: basic searching — 8 minutes
Find uppercase ERROR entries:
grep 'ERROR' logs/inventory.log

Expected: three matching lines.
Display their line numbers:
grep -n 'ERROR' logs/inventory.log

Why this matters: line numbers help you locate the problem quickly when opening a large configuration or log file in an editor.
Count the errors:
grep -c 'ERROR' logs/inventory.log

Expected result:
3

Search without caring about capitalization:
grep -in 'warn' logs/inventory.log

Expected: both the uppercase WARN and lowercase warn entries.
Search for everything except informational entries:
grep -v ' INFO ' logs/inventory.log

This displays lines not containing the exact text INFO.
Combine case-insensitive matching with inverse matching:
grep -iv ' INFO ' logs/inventory.log

Search for complete words
grep -w 'failed' logs/inventory.log

-w prevents a search word from matching as part of a longer word.
Compare:
printf '%s\n' 'fail' 'failed' 'failure' | grep 'fail'
printf '%s\n' 'fail' 'failed' 'failure' | grep -w 'fail'

The first command matches all three lines. The second matches only the complete word fail.
4. Guided practice: pipes and multiple patterns — 7 minutes
A pipe sends one command’s output into another command:
command1 | command2

Find error entries and then search those results for Database:
grep 'ERROR' logs/inventory.log | grep 'Database'

Only the database error should remain.
Count error and warning entries with an extended regular expression:
grep -Ei 'ERROR|WARN' logs/inventory.log

Options:
- -E enables extended regular expressions.
- -i ignores capitalization.
- ERROR|WARN means ERROR or WARN.
Count those entries:
grep -Eic 'ERROR|WARN' logs/inventory.log

Expected result:
5

Save the results to a report:
grep -Ein 'ERROR|WARN' logs/inventory.log > reports/incidents.txt

This combines skills from your earlier redirection practice:
- grep finds the entries.
- -n adds original line numbers.
- > writes the results into a file.
Verify:
cat reports/incidents.txt
wc -l reports/incidents.txt

Expected line count:
5 reports/incidents.txt

5. Guided practice: anchors and character classes — 7 minutes
Match the beginning of a line
Search for configuration lines beginning with database:
grep '^database' config/application.conf

Expected:
database_server=db01.example.test
database_port=5432

The caret ^ anchors the pattern to the beginning.
Compare:
grep 'database' config/application.conf
grep '^database' config/application.conf

The first searches anywhere in each line. The second searches only at the beginning.
Match the end of a line
Find configuration values ending in false:
grep 'false$' config/application.conf

Expected:
debug=false

The dollar sign $ anchors the pattern to the end.
Match digits
Find lines containing at least one digit:
grep -E '[0-9]+' logs/inventory.log

Because every log entry contains a date and time, all log lines should match.
Find four-digit user numbers:
grep -E 'User [0-9]{4}' logs/inventory.log

Expected: the two entries containing User 1042.
The pattern means:
- User  — literal text and a space
- [0-9] — one digit
- {4} — exactly four occurrences
Match a precise HTTP status pattern
grep -E 'HTTP [45][0-9]{2}' logs/inventory.log

This looks for:
- HTTP
- A first digit of 4 or 5
- Exactly two additional digits
Expected result:
2026-09-17 18:08:40 ERROR API request returned HTTP 503

6. Guided practice: recursive searches and context — 5 minutes
Search every file beneath config:
grep -rn 'threshold' config

Expected matches come from config/monitoring.conf.
Search recursively while showing only filenames:
grep -rl 'production' .

Expected output includes:
./config/application.conf

Display context around the database failure:
grep -n -B 2 -A 2 'Database connection failed' logs/inventory.log

You should see two lines before and two lines after the error. Context is useful because the surrounding events often explain what happened immediately before and after a failure.
The shorter equivalent is:
grep -n -C 2 'Database connection failed' logs/inventory.log

7. Understand the exit status — 2 minutes
Run:
grep 'ERROR' logs/inventory.log > /dev/null
echo $?

Expected:
0

Now search for something absent:
grep 'KERNEL_PANIC' logs/inventory.log > /dev/null
echo $?

Expected:
1

For grep:
- 0 means a match was found.
- 1 means no match was found.
- 2 generally means an actual error occurred.
/dev/null discards normal output when you only need to test whether a match exists.
8. Independent challenge — 5 minutes
Complete this without copying commands from the guided section.
Your manager asks for a quick incident report from logs/inventory.log.
Create:
reports/critical-events.txt

It must meet all these requirements:
1. Include uppercase and lowercase error or warning entries.
2. Include original line numbers.
3. Exclude every informational entry.
4. Contain exactly five lines.
5. Do not manually type or copy the matching log entries.
6. Use grep and redirection to generate the report.
7. Verify the report’s line count.
8. Verify that it contains both the database failure and HTTP 503 event.
Then perform these searches without saving their output:
- Find lines containing a four-digit user number.
- Find configuration entries beginning with critical.
- Find configuration entries ending in true.
- Identify which files beneath the lab contain the word production.
- Display two lines of context surrounding the HTTP 503 event.
9. Independent verification
Use appropriate tests to confirm:
test -s reports/critical-events.txt && echo "PASS: report exists and is not empty"

Determine whether the line count is five:
test "$(wc -l < reports/critical-events.txt)" -eq 5 \
  && echo "PASS: report contains five incidents" \
  || echo "CHECK: incorrect incident count"

Confirm the required events are present:
grep -q 'Database connection failed' reports/critical-events.txt \
  && echo "PASS: database failure included"

grep -q 'HTTP 503' reports/critical-events.txt \
  && echo "PASS: HTTP 503 included"

Confirm no informational entries slipped into the report:
if grep -q ' INFO ' reports/critical-events.txt; then
    echo "CHECK: INFO entries were incorrectly included"
else
    echo "PASS: no INFO entries included"
fi

These commands verify the outcome without revealing the command you should use to create the report.
10. Knowledge check
Answer these without looking back:
1. What does grep -i do?
2. What does grep -v do?
3. Which option displays line numbers?
4. What does ^database mean?
5. What does failed$ mean?
6. What does [0-9]{4} represent when using grep -E?
7. Why should regular expressions normally be enclosed in quotes?
8. What does the pipe symbol do?
9. How do you search a directory recursively?
10. How do you display only filenames containing matches?
11. What does grep exit status 1 mean?
12. How would you search for either ERROR or WARN?
13. What is the difference between grep 'fail' and grep -w 'fail'?
14. How can you show three lines before and after a match?
I would consider this objective mastered when you can search a large log for multiple severities, use anchors and character classes, send results through a pipe, save them using redirection, and verify the report without assistance.
Safe cleanup
Safety warning
The following command recursively removes the temporary lab. Confirm the complete path is exactly /tmp/rhcsa-grep-lab.
cd /tmp
rm -rf /tmp/rhcsa-grep-lab

Verify cleanup:
test ! -e /tmp/rhcsa-grep-lab \
  && echo "PASS: grep lab removed safely" \
  || echo "CHECK: grep lab still exists"

Expected result:
PASS: grep lab removed safely
