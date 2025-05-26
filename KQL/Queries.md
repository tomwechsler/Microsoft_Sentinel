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
IdentityRiskEventsSigninLogs
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

**Search for Intune Devices**  

```
IntuneDevices
| where todatetime(CreatedDate) > ago(21d)
| distinct DeviceName, UserName, CreatedDate, SerialNumber, Model
```