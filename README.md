### 📁 **Project Title**: System Hardware Info Collector (PowerShell)

This PowerShell script gathers detailed hardware and network information from a Windows machine, including serial numbers, model details, RAM specifications, MAC addresses, and GPU details.

---

## 🔧 Features

* Laptop serial and model number
* Processor name
* HDD & SSD serial/model numbers
* MAC addresses for Ethernet and WiFi
* RAM capacity and speed
* Graphics card details
* Outputs all information in a readable format

---

## 🖥️ Script

```powershell
# ===========================
# System Hardware Info Collector
# ===========================

# Laptop Serial Number
$serialNumber = (Get-WmiObject -Class Win32_BIOS).SerialNumber

# Laptop Model Number
$modelNumber = (Get-WmiObject -Class Win32_ComputerSystem).Model

# Processor Detail
$processorDetail = (Get-WmiObject -Class Win32_Processor).Name

# Hard Disk Serial Number (first detected drive)
$hardDiskSerialNumber = (Get-WmiObject -Class Win32_DiskDrive)[0].SerialNumber

# Hard Disk Model Number
$hardDiskModelNumber = (Get-WmiObject -Class Win32_DiskDrive)[0].Model

# Solid State Drive Details (first detected SSD)
$ssd = Get-WmiObject -Class Win32_DiskDrive | Where-Object { $_.MediaType -eq 'SSD' }
$ssdSerialNumber = $ssd.SerialNumber
$ssdModelNumber = $ssd.Model

# Ethernet MAC Address
$ethernetMAC = (Get-WmiObject -Class Win32_NetworkAdapter | Where-Object {
    $_.MACAddress -ne $null -and $_.PhysicalAdapter -eq $true
}).MACAddress

# WiFi MAC Address
$wifiMAC = (Get-WmiObject -Class Win32_NetworkAdapter | Where-Object {
    $_.MACAddress -ne $null -and $_.NetEnabled -eq $true
}).MACAddress

# RAM Capacity (in GB)
$ramCapacity = (Get-WmiObject -Class Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum).Sum / 1GB

# RAM Speed (MHz)
$ramSpeed = (Get-WmiObject -Class Win32_PhysicalMemory | Select-Object -ExpandProperty Speed | Sort-Object -Unique) -join ", "

# Graphics Card Details
$graphicsCardDetail = (Get-WmiObject -Class Win32_VideoController | Select-Object -ExpandProperty Caption) -join ", "

# Output
$attributes = @(
    'Laptop Serial Number:', $serialNumber,
    'Laptop Model Number:', $modelNumber,
    'Processor Detail:', $processorDetail,
    'Hard Disk Serial Number:', $hardDiskSerialNumber,
    'Hard Disk Model Number:', $hardDiskModelNumber,
    'Solid State Drive Serial Number:', $ssdSerialNumber,
    'Solid State Drive Model Number:', $ssdModelNumber,
    'Ethernet MAC Address:', $ethernetMAC,
    'WiFi MAC Address:', $wifiMAC,
    'RAM Capacity (GB):', [math]::Round($ramCapacity, 2),
    'RAM Speed (MHz):', $ramSpeed,
    'Graphics Card Detail:', $graphicsCardDetail
)

# Display output
for ($i = 0; $i -lt $attributes.Count; $i += 2) {
    $attribute = $attributes[$i]
    $value = $attributes[$i + 1]
    Write-Output "$attribute`t$value"
}
```

---

## 📦 How to Use

1. Open PowerShell with administrative privileges.
2. Copy and paste the script into your PowerShell session or save it as `Get-SystemInfo.ps1`.
3. Run the script to see the hardware information printed in the console.

---

## 📋 Output Example

```
Laptop Serial Number:        ABC1234567
Laptop Model Number:         Dell Latitude 7490
Processor Detail:            Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz
Hard Disk Serial Number:     XYZ9876543
Hard Disk Model Number:      ST500LM034-2E717D
Solid State Drive Serial Number:    S5DZNF0R123456H
Solid State Drive Model Number:     Samsung SSD 860 EVO
Ethernet MAC Address:        00-1A-2B-3C-4D-5E
WiFi MAC Address:            00-1B-63-84-45-E6
RAM Capacity (GB):           16
RAM Speed (MHz):             2400
Graphics Card Detail:        Intel(R) UHD Graphics 620
```

---

## 🛡️ Notes

* Script is built using WMI, so it's compatible with most modern Windows systems.
* Admin privileges may be required to access some hardware details.
* Ensure PowerShell Execution Policy allows running scripts:

  ```powershell
  Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
  ```

---

