# Wazuh Failed Login Investigation

## Overview

I used my Home SOC lab to investigate failed Windows login attempts in Wazuh.

The goal was to generate a controlled failed login, confirm that Wazuh detected it, and review the event details like the username, source IP, logon type, and failure reason.

## What I Did

I created a temporary Windows account called `soclab` and generated two failed network login attempts by entering the wrong password.

```powershell
net use \\127.0.0.1\IPC$ /user:soclab *
```

The login attempts failed with:

```text
System error 1326 has occurred.
The user name or password is incorrect.
```

![Failed login generated](01-failed-login-generated.png)

## Wazuh Detection

I searched for `soclab` in Wazuh Threat Hunting.

Wazuh detected:

- 2 authentication failures
- 0 authentication successes
- Windows endpoint: `Windows11-Endpoint`

![Wazuh authentication failures](02-wazuh-authentication-summary.png)

I then reviewed the individual alerts. Wazuh identified both events as:

`Logon Failure - Unknown user or bad password`

The alerts were generated under Wazuh rule `60122` with rule level `5`.

![Failed login events](03-wazuh-failed-login-events.png)

## Event Details

I opened one of the failed login events and found:

- Target user: `soclab`
- Windows Event ID: `4625`
- Source IP: `127.0.0.1`
- Logon Type: `3`
- Logon Process: `NtLmSsp`
- Status: `0xC000006D`
- SubStatus: `0xC000006A`

![Failed login event details](04-failed-login-event-details.png)

The event showed that the login attempt came from the same Windows machine and failed because an incorrect password was used.

## Conclusion

The failed login attempts were successfully detected and recorded by Wazuh.

This project helped me practice using a SIEM to search for authentication activity, review Windows security events, and investigate the details behind a failed login alert.
