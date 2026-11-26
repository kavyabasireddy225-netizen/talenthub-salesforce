TalentHub – Salesforce Recruiting App

A recruiting and applicant-tracking app built on the Salesforce Platform. It covers data modeling, security, declarative automation and data quality.

Portfolio project built in a Salesforce Developer Edition org with fictional data.

Business problem

A company tracks hiring in spreadsheets. Recruiters lose track of candidates, hiring managers can't see progress, and salary data isn't protected. TalentHub manages positions, candidates, job applications and interviews in one place, with the right security and automation.

Data model
Position (1) ──< Job Application >── (1) Candidate
                      │
                      └──< Interview
Object	Purpose	Key relationships
Position	An open job	Master of Job Application
Candidate	A person applying	Master of Job Application
Job Application	Junction object (many-to-many)	Master-detail to Position and Candidate
Interview	One interview round	Required lookup to Job Application

Design decision: a junction object can't be the master in another master-detail relationship. Interview therefore uses a required lookup with "Don't allow deletion", and the interview roll-ups are rebuilt with record-triggered Flows.

Key fields

Position: Status, Department, Location, Hiring Manager, Min/Max Pay, Open/Close Date, Days Open (formula), Total Applications and Hired Count (roll-up summaries), record types Technical / Non-Technical
Candidate: Auto-number name (CAND-0000), unique Email, Phone, Source, Years of Experience
Job Application: Status, Rejection Reason, Offer Amount, Applied Date, Interview Count, Average Rating, Candidate Full Name (cross-object formula)
Interview: Interviewer, Date, Type, Rating, Rating Value (formula), Feedback, Recommendation
Features
User experience
TalentHub Lightning app with custom tabs
Custom Lightning record page for Position, with Highlights Panel, Tabs and Dynamic Forms field sections
Path on Position Status, with key fields and Guidance for Success for each stage (separate paths for each record type)
Conditional field visibility: Max Pay appears only when Status = Open
Custom list views (e.g. All Job Applications with Position, Candidate and Status columns)
Data quality
Validation rule	Object	Rule
Close_Date_After_Open_Date	Position	Close Date can't be before Open Date
Min_Pay_Not_Above_Max_Pay	Position	Min Pay can't exceed Max Pay
Rejection_Reason_Required	Job Application	Reason required when Status = Rejected
Offer_Amount_Required	Job Application	Offer Amount required when Status = Offer Extended
No_Apply_To_Closed_Position	Job Application	No new applications to Closed/Filled positions

Duplicate management: a matching rule on Candidate (Email OR (Last Name AND Phone)) and a duplicate rule that blocks likely duplicates. Exact email duplicates are blocked separately by the unique Email field.

Security model
Object	Org-wide default	Reason
Position	Public Read Only	Everyone can see open jobs
Candidate	Private	Personal data
Job Application	Controlled by Parent	Inherits access from Position and Candidate
Interview	Private	Interview feedback is sensitive

Access is opened up through the role hierarchy, sharing rules and permission sets (least privilege).

Automation
Automation	Type	What it does
Position Approval	Approval Process	A New position with pay set can be submitted. The record locks and Status becomes Pending Approval. On approval → Open and Open Date = today. On rejection or recall → New, and the owner gets an email alert
Candidate Hired	Record-triggered Flow (after save)	When an application changes to Hired: sets the Position to Filled with a Close Date, rejects all other open applications (Position Filled) and emails the hiring manager. Fault paths email the admin
Interview Totals	Record-triggered Flows (create/update and delete)	Keeps Interview Count and Average Rating on Job Application up to date, replacing roll-ups that a lookup can't have

Flow design notes

Uses a single conditional Update Records element, not DML inside a loop, so it's bulk-safe
Runs only when a record is updated to meet the condition, which stops repeat runs
Divide-by-zero guard on the average calculation
Fault paths send {!$Flow.FaultMessage} to the admin
Testing
Scenario	Expected result	✓
Mark an application Hired	Position → Filled, other open applications → Rejected, email sent	✅
Already-rejected application on the same position	Left unchanged	✅
Edit an unrelated field on a hired application	Flow doesn't run again	✅
Submit a position without Max Pay	Blocked by the approval entry criteria	✅
Approve / reject a position	Status → Open / New; record unlocked	✅
Create a candidate with the same last name and phone as an existing one	Blocked by the duplicate rule	✅
Reject an application without a reason	Validation error	✅



Deploying to another org
bash
sf org login web --alias target-org
sf project deploy start --manifest manifest/package.xml --target-org target-org

Some settings must be configured by hand after deployment: users and managers, Deliverability (All email), and approval process activation.

Skills demonstrated

Data modeling (master-detail, junction objects, lookups, roll-ups, formulas) · Lightning App Builder · Dynamic Forms · Path · Validation rules · Duplicate management · Org-wide defaults and sharing · Approval processes · Record-triggered Flows · Fault handling · Salesforce CLI and Git source control
