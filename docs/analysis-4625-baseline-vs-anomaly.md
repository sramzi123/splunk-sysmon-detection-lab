# Investigation: What a Failed Logon Actually Told Me

## The moment

I wanted a clean example of a brute force pattern for this lab, so I locked my machine and typed the wrong password four times in a row before logging in correctly. A few minutes later I pulled the logs expecting an easy win: four 4625 failed logon events, back to back, seconds apart. On the surface that is exactly what a brute force attempt looks like.

I almost took it at face value. Then I actually read the fields.

## What the search showed

```spl
index=main host="DESKTOP-HJSIADG" EventCode=4625
| table _time, Account_Name, Logon_Type, Logon_Process, Workstation_Name, Source_Network_Address, Failure_Reason, Status, Sub_Status
| sort -_time
```

![Initial 4625 table view](../screenshots/analysis-01-initial-4625-table.png)

Four events, seconds apart, same host. It matched the story I already had in my head, so it would have been easy to just screenshot this and call it a brute force detection.

## Where it fell apart

A 4625 event actually carries two separate identity fields, and it is easy to confuse them.

Subject is the account reporting the failure. This is often just a local system process checking something in the background, not the person sitting at the keyboard.

Account For Which Logon Failed is the actual username that was typed and rejected. This is the field that should show me.

The table above only surfaces the Subject, which is why every row shows a machine account instead of my name. So I pulled the raw event to see the rest of it.

![Raw event showing the blank target account](../screenshots/analysis-02-raw-event-blank-target.png)

The target account field was blank, with a Null SID. That stopped me. Even a wrong password still requires Windows to know which username was being attempted. A blank field with a Null SID means the system never got that far. Whatever this was, it was not someone typing my password incorrectly at the lock screen.

## Ruling myself out

I sign in with a PIN through Windows Hello, not a typed password, and that distinction turned out to matter a lot. PIN validation happens locally against the device's TPM and does not travel through the same Negotiate and NTLM pipeline that a password based logon uses, which happens to be the exact pipeline responsible for populating that target account field in a standard 4625 event. So my real failed PIN attempts were never going to show up the way I expected them to.

I also found other people online reporting this identical fingerprint, the same Subject account, the same svchost.exe and User32 combination, the same Status and Sub Status codes, with no real explanation from Microsoft either. It looks like a routine background authentication check, not an actual credential failure.

So the four events I was ready to screenshot as my brute force example were not my attempts. They were something else entirely that just happened to land in the same window.

## Trying again, and getting a real answer

I repeated the test using a typed password instead of a PIN. This time the Sub Status changed to 0xC000006A, which specifically means wrong password rather than the earlier unresolved code. That was a real signal.

Searching more broadly around that timestamp, I found a successful 4624 logon two and a half seconds later. I initially assumed the account name on that success was mine, but running whoami on the actual machine showed my real username was Shaza, not the name that showed up in the result. That one turned out to be a different local placeholder account entirely, unrelated to me.

![Sequence showing the failed attempt and the logons that followed](../screenshots/analysis-03-4624-success-sequence.png)

I have not yet pinned down the exact record showing Shaza succeeding right after a failure. Rather than force a conclusion the data has not earned, I am leaving that open. This document reflects what the evidence actually supports right now, not what would make the tidiest story.

## What this actually taught me

An event code alone does not tell you what happened. I looked at this same 4625 code from four different angles and it meant something different almost every time.

The Subject field and the target account field are answering two different questions, and I definitely used to just conflate them without realizing it. Get that wrong and you either miss something real or chase something that was never there in the first place.

A blank target account with a Null SID is its own thing, not just a normal wrong password. I hadn't thought about that distinction before this.

And the authentication method piece is the one I keep coming back to. If an environment leans on PIN or biometric sign in, there's a real gap in 4625 based brute force detection that I would not have known to look for otherwise.

## Where I left it

At some point I just decided to stop chasing a perfectly clean before and after pair, since it was pretty clear the data was not going to hand me one easily. What I ended up with instead is honestly more useful anyway, a real investigation where my first read of the logs was wrong and I actually caught it, instead of writing it down as fact and moving on.
