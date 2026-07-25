# Investigation: What a Failed Logon Actually Told Me

## The moment

I wanted a clean example of a brute force pattern for this lab, so I locked my machine and typed the wrong password four times in a row before logging in correctly. A few minutes later I pulled the logs expecting an easy win: four 4625 failed logon events, back to back, seconds apart. On the surface, that is exactly what a brute force attempt looks like.

I almost took it at face value. Then I actually read the fields.

## What the search showed

```spl
index=main host="DESKTOP-HJSIADG" EventCode=4625
| table _time, Account_Name, Logon_Type, Logon_Process, Workstation_Name, Source_Network_Address, Failure_Reason, Status, Sub_Status
| sort -_time
```

![Initial 4625 table view](../screenshots/analysis-01-initial-4625-table.png)

| _time | Account_Name (Subject) | Logon_Type | Logon_Process | Source_Network_Address | Status / Sub Status |
|---|---|---|---|---|---|
| 13:16:42.103 | DESKTOP HJSIADG$ | 2 | User32 | 127.0.0.1 | 0xC000006D / 0xC0000380 |
| 13:16:40.418 | DESKTOP HJSIADG$ | 2 | User32 | 127.0.0.1 | 0xC000006D / 0xC0000380 |
| 13:16:39.186 | DESKTOP HJSIADG$ | 2 | User32 | 127.0.0.1 | 0xC000006D / 0xC0000380 |
| 13:16:37.430 | DESKTOP HJSIADG$ | 2 | User32 | 127.0.0.1 | 0xC000006D / 0xC0000380 |

Four events, seconds apart, same host. It matched the story I already had in my head, so it would have been easy to just screenshot this and call it a brute force detection.

## Where it fell apart

A 4625 event actually carries two separate identity fields, and it is easy to confuse them.

Subject is the account reporting the failure. This is often just a local system process checking something in the background, not the person sitting at the keyboard.

Account For Which Logon Failed is the actual username that was typed and rejected. This is the field that should show me.

The table above only surfaces the Subject, which is why every row shows a machine account instead of my name. So I pulled the raw event to see the rest of it.
