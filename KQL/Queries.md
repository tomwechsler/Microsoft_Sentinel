## Sample KQL Queries

**Single User Query**  

```
SigninLogs
| where AlternateSignInName contains "Alexw@tomscloud.ch"
```

**Break Glass Admin Account Query**  

```
SigninLogs
| project UserPrincipalName 
| where UserPrincipalName == "admin365@yourdomain.onmicrosoft.com"
```

**Risky Users Query**  

```
SigninLogs
| where RiskState contains "atRisk"
| extend City = tostring(LocationDetails.city)
| extend Region = tostring(LocationDetails.countryOrRegion)
| extend Latitude = tostring(parse_json(tostring(LocationDetails.geoCoordinates)).latitude)
| extend Longitude = tostring(parse_json(tostring(LocationDetails.geoCoordinates)).longitude)
| extend MaliciousIP = IPAddress
```

**Global Administrator Role Assignment Query**  

```
AuditLogs
| where Category == "RoleManagement"
| where Result == "success"
| where OperationName == "Add member to role"
| where (TargetResources has "Company" or TargetResources has "Tenant" or TargetResources has "Global Administrator")
| project TargetUser = tostring(TargetResources.[0].["userPrincipalName"]), InitiatedBy = tostring(InitiatedBy.user.["userPrincipalName"])
```

**Users with at least 5 failed login attempts**  

```
SigninLogs
| where ResultType != "0"  // Filters only failed login attempts (ResultType ≠ 0)
| summarize count() by Identity, bin(TimeGenerated, 10m)  // Groups by user and 10-minute time slots and counts the events
| where count_ >= 5  // Only shows users with at least 5 failed login attempts
```

**This query identifies administrative users with specific login errors**  
// who register outside certain regions, and counts their activity in 5-minute intervals.

```
SigninLogs
| where ResultType in ("50126", "50053")  // Filters specific login error codes (e.g. invalid login information)
| where UserPrincipalName has "admin"  // Searches for user names containing “admin”
| where Location !in ("Switzerland")  // Excludes applications from Switzerland
| summarize Count = count() by UserPrincipalName, bin(TimeGenerated, 5m)  // Groups the results by user and 5-minute intervals and counts the events
```

**This query identifies failed logins with high risk in the last 7 days**  
// and projects relevant fields such as timestamp, identity, location, IP address and error description

```
SigninLogs
| where TimeGenerated >= ago(7d)  // Only considers events in the last 7 days
| where ResultType != "0" and ResultType != "50125"  // Filters failed login attempts (ResultType ≠ 0)
| where RiskLevel == "high"  // Refers to high-risk events
| project TimeGenerated, Identity, Location, IPAddress, ResultDescription  // Displays selected fields for analysis
```

**Search for Intune Devices**  

```
IntuneDevices
| where todatetime(CreatedDate) > ago(21d)
| distinct DeviceName, UserName, CreatedDate, SerialNumber, Model
```