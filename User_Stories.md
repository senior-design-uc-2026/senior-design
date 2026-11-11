# User Stories and Use Cases

CS 5001 Senior Design I, Assignment 4

**Project:** Tiny Cloud

**Team members:** Pratham Prabhu, Phani Bharadwaj Jarugumilli

**Advisor:** Pratyush Srivastava

## 1. Stakeholder Map

We sorted stakeholders by how directly they touch Tiny Cloud. The hidden ones are the people we would only notice after launch if we did not look for them on purpose.

| Category | Stakeholder | What they care about | Story |
| --- | --- | --- | --- |
| Primary | App owner, for example a club officer or a student who built an app | Getting the app to the right people without setting up hosting, login or a database, and cutting people off when they leave | US-01, US-04 |
| Primary | Viewer, an invited person with no account anywhere | Getting into the app within a minute, without making a password | US-03 |
| Secondary | The coding agent that works for the owner (Claude Code, Cursor, Codex) | Tools it can call to deploy and share, with clear errors it can act on | US-02 |
| Hidden | Owner of a different app on the same server | Not having their club's saved data read or broken by someone else's app | US-05 |
| Hidden | A person whose email gets typed into a login page by someone else | Not receiving login codes they never asked for | AC-02.4 |

## 2. User Stories

**US-01 (primary: app owner)** As a club member who just had a coding agent build a sign-up sheet, I want to deploy the app with one command and get back a private link, with saved data and secrets handled for me, so that my club can start using it today without me setting up hosting, login or a database.

**US-02 (secondary: coding agent acting for the owner)** As a student working in Claude Code, I want my coding agent to deploy my app and invite people through tool calls, so that I can go from "build a poll" to a shared link without leaving the chat or typing a command.

**US-03 (primary: viewer)** As a club member with no account on any hosting service, I want to prove who I am with a code sent to my email, so that I can open an app I was invited to in under a minute without making a password.

**US-04 (primary: app owner)** As an app owner whose member just left the club, I want to remove that person and have them locked out right away, so that they cannot keep voting or reading results.

**US-05 (hidden: owner of another app on the same server)** As the owner of one app on a shared server, I want every other app to be unable to read my app's files or reach it directly, so that a badly written or hostile app running next to mine cannot leak my club's saved data.

### INVEST self-check

Columns: I is Independent, N is Negotiable, V is Valuable, E is Estimable, S is Small, T is Testable.

| Story | I | N | V | E | S | T | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| US-01 | Yes | Yes | Yes | Yes | Borderline | Yes | Deploy, saved data and secrets are one goal for the owner. We split them into separate requirements later. |
| US-02 | Yes | Yes | Yes | Yes | Yes | Yes | Needs US-01 working underneath but can be tested on its own with a deploy. |
| US-03 | Yes | Yes | Yes | Yes | Yes | Yes | The one-minute goal is tested in AC-02.1. |
| US-04 | Yes | Yes | Yes | Yes | Yes | Yes | Needs an invite to exist first, which is set up in the test. |
| US-05 | Yes | Yes | Yes | Yes | Borderline | Yes | Covers files, data and network. We test each one separately. |

## 3. Use Cases

### UC-01: Deploy an app and invite a viewer

**Expands:** US-02 (and the deploy part of US-01)

**Primary actor:** App owner

**Secondary actors:** Coding agent, Tiny Cloud

**Preconditions:**

- The owner has run tiny login, so a login is saved on their computer.
- The app folder has a start script and follows Tiny Cloud’s app rules (how app must be written to run on Tiny Cloud).
- The Tiny Cloud server is running.
- The owner knows the email to invite, for example sam@uc.edu.

**Main success flow**

| Step | Who | What happens |
| --- | --- | --- |
| 1 | Owner | Asks the coding agent to share the finished poll app with sam@uc.edu |
| 2 | System | Receives the app, checks the app rules, builds and starts it, and returns a link with status “running”. |
| 3 | Coding Agent | Asks Tiny Cloud to invite sam@uc.edu to the poll app. |
| 4 | System | Records the invite and confirms. |
| 5 | Owner | Reads the link from the agent and sends it to Sam. |

**Alternate flow A1: redeploy an app that already exists (at step 1)**

2a. The owner deploys a name that is already running. The system replaces the running app with the new build and keeps its saved data. The flow continues at step 3.

**Exception flow E1: the app has no start script (at step 2)**

2e. The system stops the deploy and returns an error saying the start script is missing. No app is left running under that name and nobody is invited. The owner fixes the app and starts again at step 1.

**Postcondition:** The app is running at its link, only the invited emails can open it, and no other app on the server was changed.

### UC-02: Log in with an emailed code and open an app

**Expands:** US-03

**Primary actor:** Viewer (Sam)

**Secondary actors:** Router, email service (Resend)

**Preconditions:**

- The app is deployed and sam@uc.edu is invited to it.
- Sam is not logged in.
- Sam can read email sent to sam@uc.edu.

**Main success flow**

| Step | Who | What happens |
| --- | --- | --- |
| 1 | Sam | Opens the app link in a browser. |
| 2 | System | Sees no login and shows the login page. |
| 3 | Sam | Types his email, sam@uc.edu. |
| 4 | System | Checks that the email is invited to an app, then emails a 6-digit code. |
| 5 | Sam | Reads the email and types the code. |
| 6 | System | Checks the code, logs Sam in, wakes the app if it is asleep, and shows it with Sam’s email passed to the app. |
| 7 | Sam | Votes in the poll |
| 8 | System | The app records the vote as coming from sam@uc.edu |

**Alternate flow A1: Sam already has a session (at step 1)**

1a. The system finds a valid login and skips steps 2 to 5. The flow picks up at step 6.

**Exception flow E1: wrong code (at step 6)**

6e. The code does not match. The system shows an error and lets Sam try again. After 5 wrong tries the code stops working and Sam ask for a new one.

**Exception flow E2: expired code (at step 6)**

6f. The code was sent more than 10 minutes ago. The system refuses it and offers to send a new code.

**Exception flow E3: email not invited to any app (at step 4)**

4e. The email is not invited to any app. The system sends no email and shows the same "check your email" page it shows for everyone, so the page does not reveal who is invited. If that person later opens an app link they are not invited to, the system shows "access denied" (see AC-02.5).

**Postcondition:** Sam has a session, the code he used cannot be used again, and the app received his email in the identity header on every request.

## 4. Acceptance Criteria (Given / When / Then)

### UC-01

**AC-01.1 (main flow)** Given the owner is logged in and the poll app follows the app rules, When the owner's agent deploys it, Then a link of the form https://\<app\>-\<owner\>.\<domain\> comes back within 90 seconds, and the link opens for an invited viewer.

**AC-01.2 (exception E1)** Given an app folder with no start script, When the agent deploys it, Then the deploy fails with an error that says the start script is missing, no link is returned, and tiny list shows no running app under that name.

**AC-01.3 (alternate A1)** Given the poll app is running with 3 saved votes, When the owner deploys the same app name again, Then the new version is running within 90 seconds and all 3 votes are still there.

### UC-02

**AC-02.1 (main flow)** Given sam@uc.edu is invited and is not logged in, and the code email arrives within 10 seconds, When he opens the link, types his email, and then types the correct code less than 10 minutes after it was sent, Then he sees the app within 60 seconds of opening the link, and the app is told his email, sam@uc.edu, on every request he makes.

**AC-02.2 (exception E1)** Given a code was sent to sam@uc.edu, When he types a different 6-digit code, Then he is not logged in and sees an error, and after the 5th wrong try even the correct code is refused.

**AC-02.3 (exception E2)** Given a code that was sent 10 minutes and 1 second ago, When Sam types it correctly, Then it is refused as expired and no login is created.

**AC-02.4 (exception E3)** Given stranger@example.com is not invited to any app, When that email is typed into the login page, Then the page shows the same "check your email" message as for any other email, and zero emails are sent to that address.

**AC-02.5 (exception E3, second half)** Given Sam is logged in but is not invited to app X, When he opens the link for app X, Then he sees "access denied", and app X receives no request from him.

### Stories without a use case yet

**AC-04.1 (US-04, revoke)** Given Sam is logged in with a page open to the app, When the owner removes sam@uc.edu, Then within 1 second Sam's open page is disconnected and his next request gets "access denied".

**AC-05.1 (US-05, isolation)** Given app A and app B are both running, When app A tries to read app B's saved data or connect to app B directly, Then both attempts fail, and app B's saved data is unchanged.
