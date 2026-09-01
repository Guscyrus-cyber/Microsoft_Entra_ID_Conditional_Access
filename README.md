**Microsoft Entra ID Conditional Access Investigation for SOC Analysts**

Introduction

This SOC-focused lab covers how a security analyst investigates Microsoft Entra ID Conditional Access activity. The emphasis is not on designing a complex enterprise Conditional Access architecture. Instead, the lab will focus on examining sign-in events, determining whether Conditional Access policies were applied, identifying why authentication was allowed, interrupted, or blocked, and correlating relevant identity activity with Microsoft Sentinel.

### **Step 1:** Opening my Microsoft Entra admin center at entra.microsoft.com (Image 1)\
\
\
Step 2 — Find Conditional Access\
In the top search bar (“Search resources, services, and docs”), I type: Conditional access. (Image 2)\
\
\
**Step 3 — Open Sign-in logs**

Clicking on Sign-in logs\
I examine the existing events first and see whether they contain **Conditional Access status/results**.\
I now have actual **interactive sign-in events** to investigate. I can see four successful events, including **SOC Test User → Azure Portal → Success** at **9:22:43 PM**. (Image 3)

### \
\
Step 4 — Opening the SOC Test User event

I click on **8/22/2026, 9:22:43 PM → SOC Test User → Azure Portal → Success**

A details panelopens.

From there, I start reading the event: **Basic info → Location/Device → Authentication Details → Conditional Access** and determining whether a CA policy was evaluated or applied.\
From **Basic info**, I can already establish:

- **User:** SOC Test User

- **Application:** Azure Portal

- **Resource:** Azure Resource Manager

- **Status:** **Success**

- **Authentication requirement:** **Multifactor authentication**

- **Additional details:** **“MFA requirement satisfied by claim in the token”**

- **Client:** Browser

- **User agent:** Safari on macOS

- **Flagged for review:** No

The important SOC question now is: **Why was MFA required, and did Conditional Access cause it? (Images 4, 5, and 6)**

\
\

### \
\
Step 5 — Conditional Access tab

I click on: Conditional Access

That screen is particularly important for this lab because it should tell whether a Conditional Access policy was **Applied, Not applied, Report-only, or otherwise evaluated**.

Conditional Access tab shows:

- **Policy Name:** Security defaults

- **Grant Control:** Require multifactor authentication

- **Result:** **Success**

So, by looking at the image, The SOC Test User's Azure Portal sign-in was subject to **Security defaults**, which required MFA. The sign-in succeeded because the MFA requirement was satisfied; the Basic Info tab told me that the MFA claim was already present in the token.

This is especially useful because although my **Entra ID Free** tenant cannot create custom Conditional Access policies, **Security defaults is still enforcing identity protections**, giving me a real event to investigate. (Image 7)

### \
\
Step 6 — Investigating how MFA was satisfied

I click on: **Authentication Details**.\
I want to examine: **Authentication method → authentication result → requirement → details.\**
The important fields are:

- **Authentication Policies Applied:** Security Defaults

- **Authentication method:** Previously satisfied

- **Succeeded:** true

- **Result detail:** MFA requirement satisfied

### \
\
What does “Previously satisfied” mean?

It means Microsoft **did not need to ask the SOC Test User for MFA again during this particular Azure Portal sign-in**. The existing authentication session/token already contained evidence that the user had successfully completed the required MFA.

So the investigation chain is now: **SOC Test User signs in → Security Defaults evaluates the sign-in → MFA is required → previous MFA claim satisfies the requirement → access succeeds.** (Image 8)

### Step 7 — Investigating the source

I click on: **Location** tab at the top.

I want to examine the sign-in's **IP address, geographic location, and related network information**, just as a SOC analyst would when determining whether the successful login looks legitimate or suspicious.\
\
\
The questions:\
**Was a security policy involved?** → **Yes — Security defaults**\
**What control did it require?** → **Require multifactor authentication**\
**What was the result?** → **Success**

I have already established a real example: **SOC Test User → Azure Portal → Security Defaults applied → MFA required → MFA previously satisfied → Sign-in Success.**

Now investigating a **failed/blocked/interrupted sign-in** and compare it with this successful one. That will teach to how a SOC analyst recognizes when access controls actually interfere with authentication. I generate/select an appropriate **failed or interrupted SOC Test User sign-in** and investigate its Conditional Access result. That's the core hands-on exercise for this lab.\
\
So, My Microsoft Entra administrator account **(liberty566@yahoo.com)** was used to access the Microsoft Entra admin center and perform the SOC investigation. This administrator account served as the analyst/investigator account for viewing and analyzing the sign-in logs.

The authentication activity being investigated, however, was primarily generated using the separate SOC Test User **(testuser01@...onmicrosoft.com)** account. This separation simulated a realistic SOC workflow in which an administrator or security analyst investigates authentication activity associated with another organizational user.

During the lab, the administrator account was therefore used to observe and investigate**,** while the SOC Test User was used to generate authentication events**,** including successful, failed, and interrupted sign-in activity. (Image 9)

### \
\
\
Step 8 —Sign-in my SOC Test User for failed or interrupted

I look for a **SOC Test User** event whose **Status is Failure or Interrupted**.

\
\
Then, I Click Sign out. This signs out SOC Test User only from this browser session.

Once Microsoft takes me to the sign-in/account-selection page.

\
\
This screen is asking **which account to sign out of**, but it is showing two entries for my personal liberty566@yahoo.com account rather than the SOC Test User.

### Instead, I click the browser **Back (←)** button once. Then, the page went blank after going back which means indication of interrupting. (Images 10, 11, and 12)\
\
\
\
Step 9**— opening a Private Safari window**

**myaccount.microsoft.com**

Because the private window has a separate session, Microsoft should ask me to sign in.

When I reach the Microsoft **Sign in / Pick an account** screen, I used **SOC Test User (testuser01@...)** there and intentionally I generate the failed authentication event. My administrator Entra window stays untouched.\
\
I Click **testuser01@liberty566yahoo.onmicrosoft.com**. Microsoft ask for the password. I entered an **intentionally incorrect password once**, and I Submit it.

### \
**\
\**
Microsoft displayed the incorrect password/error message. “My account or password is incorrect.” The failed sign-in has been generated. (Image 13 and 14)\
\
\
\
Step 10— investigating it in Entra

I Return to my **administrator Entra window** where **Conditional Access → Sign-in logs** is open.I Click **Refresh**. I look for **SOC Test User** event. It would appear as **Failure** with a non-zero sign-in error code.Then I open that event and answer main SOC question again:

**Was the security/Conditional Access policy involved, what control was evaluated, and why did this sign-in fail?\**
\
The new events are very useful:

- **9:50:29 PM — SOC Test User — My Profile — Failure — Error 50126**

- **9:22:39 PM — SOC Test User — Azure Portal — Interrupted — Error 50140**

- **9:22:43 PM — SOC Test User — Azure Portal — Success — Error 0**

The **50126 Failure** is the failed-password attempt I deliberately generated. (Image 15)

**\
\**

### \
\
Step 11 A— Investigating the failed sign-in

I click on the row: **9:50:29 PM → SOC Test User → My Profile → Failure → 50126\**
Conditional access: not applicable

\
These are important SOC evidence for the **failed sign-in**.

From the event we can conclude:

- **Status:** Failure

- **Error code:** 50126

- **Failure reason:** Invalid username or password

- **Authentication method:** Password

- **Password validation:** false

- **Source:** Lorton, Virginia, US

- **IP:** 108.28.79.19

- **Device:** macOS / Safari

- **Managed:** No

- **Compliant:** No

- **Conditional Access:** **Not applicable**

- **Report-only:** **Not applicable**

The important SOC lesson is that this particular attempt **failed at the credential authentication stage before Conditional Access became applicable**. Therefore, this event does **not** demonstrate Conditional Access blocking the user.

Step 11 B— Investigating the failed sign-in\
I click on the row: 9:22:39PM → SOC Test User → My Profile → Interrupted → 50140\
(Image 16)\
**\**
**\
\**
Question: Was Conditional Access/security policy involved in this sign-in, what control did it require, and what was the result?

Answer**:** Yes. The Security defaults security policy was involved. It required multifactor authentication (MFA), and the result was Success.

Now we can compare the two events:

| **Event** | **Authentication** | **Security policy / CA** | **Result** |
|----|----|----|----|
| Failed attempt | Password failed | Not applicable | **Failure** |
| Successful Azure Portal sign-in | Credentials succeeded | Security defaults → Require MFA | **Success** |

### **\**
