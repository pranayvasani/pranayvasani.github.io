---
tags: [AI, Productivity, Personal]
---

# The PTM I Missed Was Hiding in a PDF

I missed my son's parent-teacher meeting today. I'd marked it for tomorrow. The school had moved it, and the notice was in my inbox, in a PDF attached to an email whose body said only "Please find attached circular."

That's the real problem with school email. The subject lines are vague, the bodies are empty, and everything that matters (dates, fees, deadlines, reschedules) sits in attachments nobody opens on a busy day.

## Trying Claude first

I started with Claude, which already had access to my Gmail. It found every school email in seconds and summarised what was in the email bodies. Then it hit a wall: the Gmail connector could see the attachments but couldn't open them. Most of what mattered stayed unread.

It offered two workarounds. One was a script in my Google account to convert PDFs to text and email them back to me. The other was having it open each PDF in a browser on my laptop every morning. Both would have worked, but I didn't want to maintain plumbing just to read my child's school circulars.

## Gemini just did it

Gemini lives inside Google's own ecosystem, so I tried it there. I pointed it at the latest circular and asked for a detail that only existed in the PDF, and it answered correctly. That settled it. I set it up as a daily scheduled action.

## The prompt

```text
Every day at 7:30 AM: check my Gmail for emails from <school-sender-address>.
Mark anything from the last 24 hours as NEW. Carry forward older emails from
the last 14 days only if they still have an open action or upcoming date.
Open every PDF attachment and use its contents, not just the email body.
If nothing is new and nothing is pending, say "Nothing new from school today."

Context: my child is in Grade <X>. Skip items only for other grades, but list
them briefly at the end.

Summarise in this order:
1. ACTION NEEDED: fees, forms, consent slips, registrations, things to send.
   Deadline, cost, how to do it, source email. Earliest first. Flag overdue.
2. MEETINGS & DATES FOR ME: PTMs, parent sessions, fairs. Date, time, venue,
   mandatory or not, whether my child attends too.
3. OPPORTUNITIES: competitions, olympiads, bootcamps, events. Deadline, date,
   cost, eligibility, and one honest line on whether it's worth the time.
   Call out paid upsells.
4. RESULTS & UPDATES: results, reschedules, holidays, schedule changes.
5. FYI ONLY: one line each.

Rules: merge reminders with originals and use the latest details. Don't invent
dates or amounts. End with a checklist of what I need to do this week.
```

Section 4 is the one that would have saved me today.

## The takeaway

Pick the tool that sits closest to the data. Claude was the better conversation, but Gemini already lived where the attachments were, and for this job that mattered more than anything else.
