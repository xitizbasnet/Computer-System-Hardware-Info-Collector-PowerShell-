### 📁 **Project Title**: System Hardware Info Collector (PowerShell)

This PowerShell script gathers detailed hardware and network information from a Windows machine, including serial numbers, model details, RAM specifications, MAC addresses, and GPU details.

---

## 🔧 Features

* Computer Name and Username
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

# Basic Info
$computerName = $env:Computername
$userName = $env:Username

# System Info
$system = Get-WmiObject -Class Win32_ComputerSystem

# System Category (1=Desktop, 2=Laptop, 3=Workstation, etc.)
switch ($system.PCSystemType) {
    1 { $systemCategory = "Desktop" }
    2 { $systemCategory = "Laptop" }
    3 { $systemCategory = "Workstation" }
    4 { $systemCategory = "Enterprise Server" }
    5 { $systemCategory = "SOHO Server" }
    6 { $systemCategory = "Appliance PC" }
    7 { $systemCategory = "Performance Server" }
    default { $systemCategory = "Unknown" }
}

# Computer Manufacturer
$manufacturer = $system.Manufacturer

# Laptop Serial Number
$serialNumber = (Get-WmiObject -Class Win32_BIOS).SerialNumber

# Laptop Model Number
$modelNumber = $system.Model

# Processor Detail
$processorDetail = (Get-WmiObject -Class Win32_Processor).Name

# Get all disks
$disks = Get-WmiObject -Class Win32_DiskDrive

# HDD Info (exclude SSDs)
$hdd = $disks | Where-Object { $_.MediaType -ne 'SSD' } | Select-Object -First 1
if ($hdd) {
    $hardDiskSerialNumber = $hdd.SerialNumber
    $hardDiskModelNumber = $hdd.Model
    $hardDiskSize = "{0:N2} GB" -f ($hdd.Size / 1GB)
} else {
    $hardDiskSerialNumber = "Not Detected"
    $hardDiskModelNumber = "Not Detected"
    $hardDiskSize = "Not Detected"
}

# SSD Info
$ssd = $disks | Where-Object { $_.MediaType -eq 'SSD' } | Select-Object -First 1
if ($ssd) {
    $ssdSerialNumber = $ssd.SerialNumber
    $ssdModelNumber = $ssd.Model
    $ssdSize = "{0:N2} GB" -f ($ssd.Size / 1GB)
} else {
    $ssdSerialNumber = "Not Detected"
    $ssdModelNumber = "Not Detected"
    $ssdSize = "Not Detected"
}

# Ethernet MAC Address - ORIGINAL CODE PRESERVED
$ethernetMAC = (Get-WmiObject -Class Win32_NetworkAdapter | Where-Object { $_.MACAddress -ne $null -and $_.PhysicalAdapter -eq $true }).MACAddress
if (-not $ethernetMAC) { $ethernetMAC = "Not Detected" }

# WiFi MAC Address - ORIGINAL CODE PRESERVED
$wifiMAC = (Get-WmiObject -Class Win32_NetworkAdapter | Where-Object { $_.MACAddress -ne $null -and $_.NetEnabled -eq $true }).MACAddress
if (-not $wifiMAC) { $wifiMAC = "Not Detected" }

# RAM Info
$physicalMemory = Get-WmiObject -Class Win32_PhysicalMemory
$ramCapacity = ($physicalMemory | Measure-Object -Property Capacity -Sum).Sum / 1GB
$ramSpeed = ($physicalMemory | Select-Object -ExpandProperty Speed | Sort-Object -Unique) -join ", "

# Get first non-empty RAM Manufacturer string
$ramManufacturer = $physicalMemory | Where-Object { $_.Manufacturer -and $_.Manufacturer.Trim() -ne "" } | Select-Object -ExpandProperty Manufacturer -First 1
if (-not $ramManufacturer) { $ramManufacturer = "Not Detected" }

# Format RAM strings with units
$ramCapacityStr = "{0:N2} GB" -f $ramCapacity
$ramSpeedStr = $ramSpeed + " MHz"

# Graphics Card Detail and VRAM
$graphicsCards = Get-WmiObject -Class Win32_VideoController
$graphicsCardDetail = ($graphicsCards | Select-Object -ExpandProperty Caption) -join ", "

# Calculate total graphics memory (sum if multiple GPUs) in GB
$totalGraphicsMemoryBytes = ($graphicsCards | Measure-Object -Property AdapterRAM -Sum).Sum
if ($totalGraphicsMemoryBytes -and $totalGraphicsMemoryBytes -gt 0) {
    $graphicsMemoryGB = "{0:N2} GB" -f ($totalGraphicsMemoryBytes / 1GB)
} else {
    $graphicsMemoryGB = "Not Detected"
}

# Display Size (inches)
$monitorSizeInfo = Get-CimInstance -Namespace root\wmi -ClassName WmiMonitorBasicDisplayParams | Select-Object -First 1

if ($monitorSizeInfo) {
    $horizontalSizeCm = $monitorSizeInfo.MaxHorizontalImageSize
    $verticalSizeCm = $monitorSizeInfo.MaxVerticalImageSize
    $diagonalSizeInches = [math]::Round( [math]::Sqrt(($horizontalSizeCm * $horizontalSizeCm) + ($verticalSizeCm * $verticalSizeCm)) / 2.54, 1)
} else {
    $diagonalSizeInches = "Not Detected"
}

# Prepare output array
$attributes = @(
    'Computer Name:', $computerName,
    'Username:', $userName,
    'Computer Manufacturer:', $manufacturer,
    'System Category:', $systemCategory,
    'Laptop Serial Number:', $serialNumber,
    'Laptop Model Number:', $modelNumber,
    'Processor Detail:', $processorDetail,
    'HDD Model:', $hardDiskModelNumber,
    'HDD Size:', $hardDiskSize,
    'HDD Serial:', $hardDiskSerialNumber,
    'SSD Model:', $ssdModelNumber,
    'SSD Size:', $ssdSize,
    'SSD Serial:', $ssdSerialNumber,
    'Ethernet MAC Address:', $ethernetMAC,
    'WiFi MAC Address:', $wifiMAC,
    'RAM Capacity:', $ramCapacityStr,
    'RAM Speed:', $ramSpeedStr,
    'RAM Manufacturer:', $ramManufacturer,
    'Graphics Card Detail:', $graphicsCardDetail,
    'Graphics Memory (GB):', $graphicsMemoryGB,
    'Display Size (inches):', $diagonalSizeInches
)

# Output the attribute-value pairs side by side
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
Computer Name:  GGN215
Username:       Administrator
Computer Manufacturer:  LENOVO
System Category:        Laptop
Laptop Serial Number:        ABC1234567
Laptop Model Number:         Dell Latitude 7490
Processor Detail:            Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz
HDD Model:      SAMSUNG MZAL4256HBJD-00BL2
HDD Size:       238.47 GB
HDD Serial:     0025_38E5_3107_6655.
SSD Model:      Not Detected
SSD Size:       Not Detected
SSD Serial:     Not Detected
Ethernet MAC Address:        00-1A-2B-3C-4D-5E
WiFi MAC Address:            00-1B-63-84-45-E6
RAM Capacity (GB):           16 GB
RAM Speed (MHz):             2400 MHz
RAM Manufacturer:       Samsung
Graphics Card Detail:        Intel(R) UHD Graphics 620
Graphics Memory (GB):   0.50 GB
Display Size (inches):  13.9
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

