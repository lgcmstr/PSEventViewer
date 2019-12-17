<center>

[![PowerShellGallery Version](https://img.shields.io/powershellgallery/v/PSEventViewer.svg)](https://www.powershellgallery.com/packages/PSWinReportingV2)
[![Build Status Azure DevOps](https://dev.azure.com/evotecpl/PSEventViewer/_apis/build/status/EvotecIT.PSEventViewer?branchName=master)](https://dev.azure.com/evotecpl/PSEventViewer/_build/latest?definitionId=1)

[![PowerShellGallery Platform](https://img.shields.io/powershellgallery/p/PSEventViewer.svg)](https://www.powershellgallery.com/packages/PSWinReportingV2)
[![PowerShellGallery Preview Version](https://img.shields.io/powershellgallery/vpre/PSEventViewer.svg?label=powershell%20gallery%20preview&colorB=yellow)](https://www.powershellgallery.com/packages/PSEventViewer)

![Top Language](https://img.shields.io/github/languages/top/evotecit/PSEventViewer.svg)
![Code](https://img.shields.io/github/languages/code-size/evotecit/PSEventViewer.svg)
[![PowerShellGallery Downloads](https://img.shields.io/powershellgallery/dt/PSEventViewer.svg)](https://www.powershellgallery.com/packages/PSEventViewer)

</center>

###### Description

This module was built for a project of Events Reporting. As it was a bit inefficient I've decided to rewrite it and split reading events to separate module. While underneath it's just a wrapper over `Get-WinEvent` it does add few tweaks here and there...

Project was split into 2 parts:

- `PSEventViewer` - this module.
- `PSWinReporting` - reporting on Active Directory Events, Windows Events... generally reporting (separate project on GitHub)

###### Links

- [Documentation for PSEventViewer (overview)](https://evotec.xyz/hub/scripts/pseventviewer-powershell-module/)
- [Documentation for PSEventViewer (examples and how things are different)](https://evotec.xyz/working-with-windows-events-with-powershell/)

###### Updates

- 1.0.10 - 17.12.2019
  - Path is back. Not sure why it was gone. Need improvments.
- 1.0.9 - 12.11.2019
  - Removed dependency on PSSharedGoods on published module
    - PSSharedGoods is still dependency but building process makes it possible to compile it and push to PSGallery/Releases without that dependency.

- 1.0.7 - 12.09.2019
  - Small changes to Get-EventsInformation

- 0.62 - 11.01.2019
  - Fix for Member Name with comma inside

- 0.61 - 2.1.2019
  - Multiple new parameters, some new functionality

- 0.51
  - Added -RecordID parameter (currently it only works with LogName + RecordID, you can't use any other parameters with RecordID as it will take LogName + RecordID anyways and crash if it's not there)

- 0.50
  - Version that worked fine :-)

###### Example for -RecordID (added in 0.51)

There is huge difference difference if you ask for `-RecordID` in `FilterXML` and when you do post-processing of it via Where { }. And by huge difference I mean really huge one. Depending on amount of Event ID's stored that you query for... it maye take minutes or even hours to get a single RecordID. Since -FilterHashTable doesn't allow `RecordID` as parameter, nor `Get-WinEvent` doesn't have the `-RecordID` directly ... one has to use `FilterXML` which as you can see below speed up search from `6+ minutes to 141 miliseconds`.

```powershell
Clear-Host
Import-Module PSEventViewer -Force
Get-Events -LogName 'Security' -ID 5379 -RecordID 19626 -Verbose | Format-Table TimeCreated, ProviderName, Id, Message # takes 380 miliseconds

VERBOSE: Get-Events - Overall events processing startVERBOSE: Get-Events - Events to process in Total: 1VERBOSE: Get-Events - Events to process in Total ID: 5379
VERBOSE: Get-Events - Processing computer EVO1 for Events ID: 5379
VERBOSE: Get-Events - Processing computer EVO1 for Events ID Count: 1
VERBOSE: Get-Events - Processing computer EVO1 for Events LogName: Security
VERBOSE: Get-Events - Processing computer EVO1 for Events ProviderName:
VERBOSE: Get-Events - Processing computer EVO1 for Events Keywords:
VERBOSE: Get-Events - Processing computer EVO1 for Events StartTime:
VERBOSE: Get-Events - Processing computer EVO1 for Events EndTime:
VERBOSE: Get-Events - Processing computer EVO1 for Events Path:
VERBOSE: Get-Events - Processing computer EVO1 for Events Level: 0
VERBOSE: Get-Events - Processing computer EVO1 for Events UserID:
VERBOSE: Get-Events - Processing computer EVO1 for Events Data:
VERBOSE: Get-Events - Processing computer EVO1 for Events MaxEvents: 0
VERBOSE: Get-Events - Processing computer EVO1 for Events Path:
VERBOSE: Get-Events - Processing computer EVO1 for Events UserSID:
VERBOSE: Get-Events - Processing computer EVO1 for Events Oldest: False
VERBOSE: Get-Events - Processing computer EVO1 for Events RecordID: 19626
VERBOSE: Get-Events - Running query with parallel enabled...
VERBOSE: Get-Events - Verbose from runspace: Get-Events - preparing to run
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 executing on: EVO1
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events ID: 5379
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events ID: Security
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events RecordID: 19626
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events Oldest: False
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events Max Events: 0
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 - FilterXML:
                <QueryList>
                    <Query Id="0" Path="Security">
                        <Select Path="Security">
                        *[
                            (System/EventID=5379)
                            and
                            (System/EventRecordID=19626)
                         ]
                        </Select>
                    </Query>
                </QueryList>
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 Events founds 1
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 - Processing events...
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 - Time to generate 0 hours, 0 minutes, 0 seconds, 141 milliseconds
VERBOSE: Get-Events - Verbose from runspace: Get-Events - finished run
VERBOSE: Get-Events - Overall events processed in total for the report: 1
VERBOSE: Get-Events - Overall time to generate 0 hours, 0 minutes, 0 seconds, 260 milliseconds
VERBOSE: Get-Events - Overall events processing end

TimeCreated         ProviderName                          Id Message
-----------         ------------                          -- -------
17.07.2018 19:24:58 Microsoft-Windows-Security-Auditing 5379 Credential Manager credentials were read....
```

```powershell
Get-Events -LogName 'Security' -ID 5379 -Verbose | Where { $_.RecordID -eq 19626 } | Format-Table  TimeCreated, ProviderName, Id, Message  # takes 4-6 minutes depending on amount of events there are.

VERBOSE: Get-Events - Overall events processing start
VERBOSE: Get-Events - Events to process in Total: 1
VERBOSE: Get-Events - Events to process in Total ID: 5379
VERBOSE: Get-Events - Processing computer EVO1 for Events ID: 5379
VERBOSE: Get-Events - Processing computer EVO1 for Events ID Count: 1
VERBOSE: Get-Events - Processing computer EVO1 for Events LogName: Security
VERBOSE: Get-Events - Processing computer EVO1 for Events ProviderName:
VERBOSE: Get-Events - Processing computer EVO1 for Events Keywords:
VERBOSE: Get-Events - Processing computer EVO1 for Events StartTime:
VERBOSE: Get-Events - Processing computer EVO1 for Events EndTime:
VERBOSE: Get-Events - Processing computer EVO1 for Events Path:
VERBOSE: Get-Events - Processing computer EVO1 for Events Level: 0
VERBOSE: Get-Events - Processing computer EVO1 for Events UserID:
VERBOSE: Get-Events - Processing computer EVO1 for Events Data:
VERBOSE: Get-Events - Processing computer EVO1 for Events MaxEvents: 0
VERBOSE: Get-Events - Processing computer EVO1 for Events Path:
VERBOSE: Get-Events - Processing computer EVO1 for Events UserSID:
VERBOSE: Get-Events - Processing computer EVO1 for Events Oldest: False
VERBOSE: Get-Events - Processing computer EVO1 for Events RecordID: 0
VERBOSE: Get-Events - Running query with parallel enabled...
VERBOSE: Get-Events - Verbose from runspace: Get-Events - preparing to run
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 executing on: EVO1
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events ID: 5379
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events ID: Security
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events RecordID: 0
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events Oldest: False
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 for Events Max Events: 0
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 Data in FilterHashTable LogName Security
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 Data in FilterHashTable Id 5379
VERBOSE: Get-Events - Verbose from runspace: Constructed structured query:
<QueryList><Query Id="0" Path="security"><Select Path="security">*[((System/EventID=5379))]</Select></Query></QueryList>.
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 Events founds 9041
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 - Processing events...
VERBOSE: Get-Events - Verbose from runspace: Get-Events - Inside EVO1 - Time to generate 0 hours, 4 minutes, 55 seconds, 627 milliseconds
VERBOSE: Get-Events - Verbose from runspace: Get-Events - finished run
VERBOSE: Get-Events - Overall events processed in total for the report: 9041
VERBOSE: Get-Events - Overall time to generate 0 hours, 4 minutes, 55 seconds, 751 milliseconds
VERBOSE: Get-Events - Overall events processing end

TimeCreated         ProviderName                          Id Message
-----------         ------------                          -- -------
```
