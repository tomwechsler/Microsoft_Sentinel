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

**Search for Intune Devices**  

```
IntuneDevices
| where todatetime(CreatedDate) > ago(21d)
| distinct DeviceName, UserName, CreatedDate, SerialNumber, Model
```