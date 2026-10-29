# Aspiro Project Diagrams

This document contains PlantUML source for report sections 4.3 and 4.4. Paste each source block into a PlantUML editor or save it as a `.puml` file to render it as an image.

The diagrams are based on the current application code: React frontend, Express backend, MongoDB/Mongoose models, and the Flask ML service.

## 4.3 E-R Diagram

The application stores users, job postings, applications, resumes, saved jobs, and notifications. `userId`, `jobId`, and similar fields below are MongoDB ObjectId references. `Application(userId, jobId)` and `SavedJob(userId, jobId)` have unique compound indexes.

```plantuml
@startuml
title Aspiro - Entity Relationship Diagram
hide circle
skinparam linetype ortho

entity "User" as User {
  * _id : ObjectId <<PK>>
  --
  name : String
  email : String <<unique>>
  password : String
  role : student | recruiter | admin
  phone : String
  location : String
  skills : String[]
  experience : Embedded[]
  education : Embedded[]
  resumeAnalysis : Embedded
  uploadcareResume : Embedded
  isEmailVerified : Boolean
  isActive : Boolean
  createdAt : Date
  updatedAt : Date
}

entity "Job" as Job {
  * _id : ObjectId <<PK>>
  --
  title : String
  description : String
  location : String
  type : String
  salaryMin : Number
  salaryMax : Number
  experience : String
  skills : String[]
  requirements : String
  benefits : String
  status : String
  visibility : String
  recruiter : ObjectId <<FK User>>
  company : String
  applications : ObjectId[] <<FK Application>>
  applicationDeadline : Date
  isExpired : Boolean
  createdAt : Date
  updatedAt : Date
}

entity "Application" as Application {
  * _id : ObjectId <<PK>>
  --
  user : ObjectId <<FK User>>
  job : ObjectId <<FK Job>>
  resume : ObjectId <<FK Resume, optional>>
  status : String
  coverLetter : String
  appliedAt : Date
  lastUpdated : Date
  UNIQUE(user, job)
}

entity "Resume" as Resume {
  * _id : ObjectId <<PK>>
  --
  user : ObjectId <<FK User>>
  filename : String
  originalName : String
  filePath : String
  fileSize : Number
  mimeType : String
  analysis : Embedded
  isActive : Boolean
  isLatest : Boolean
  createdAt : Date
  updatedAt : Date
}

entity "SavedJob" as SavedJob {
  * _id : ObjectId <<PK>>
  --
  user : ObjectId <<FK User>>
  job : ObjectId <<FK Job>>
  savedAt : Date
  UNIQUE(user, job)
}

entity "Notification" as Notification {
  * _id : ObjectId <<PK>>
  --
  user : ObjectId <<FK User>>
  application : ObjectId <<FK Application>>
  type : email | sms | whatsapp
  status : pending | sent | failed
  message : String
  recipient : String
  applicationStatus : String
  sentAt : Date
  error : String
}

User ||--o{ Job : recruits
User ||--o{ Application : submits
Job ||--o{ Application : receives
User ||--o{ Resume : owns
Resume o|--o{ Application : attached to
User ||--o{ SavedJob : saves
Job ||--o{ SavedJob : bookmarked as
User ||--o{ Notification : receives
Application ||--o{ Notification : status updates
@enduml
```

## 4.4 User Modeling Language Diagrams

### 4.4.1 Use Case Diagram

```plantuml
@startuml
title Aspiro - Use Case Diagram
left to right direction
skinparam packageStyle rectangle

actor Visitor
actor Student
actor Recruiter
actor Admin
actor "ML Service" as ML
actor "Email Service" as Email

Student --|> Visitor
Recruiter --|> Visitor

rectangle "Aspiro Career Guidance Platform" {
  usecase "Register / Sign in" as UCAuth
  usecase "Browse and search jobs" as UCBrowse
  usecase "Get career recommendations" as UCCareer
  usecase "Analyze resume" as UCResume
  usecase "Use career chatbot" as UCChat
  usecase "Manage profile" as UCProfile
  usecase "Save job" as UCSave
  usecase "Apply for job" as UCApply
  usecase "Track / withdraw\napplications" as UCTrack
  usecase "View career / resume history" as UCHistory
  usecase "Create / manage job postings" as UCManageJobs
  usecase "Review applicants" as UCReview
  usecase "Update application status" as UCStatus
  usecase "Manage applications" as UCAdmin
  usecase "Send application/status email" as UCEmail
}

Visitor --> UCBrowse
Visitor --> UCCareer
Visitor --> UCResume
Visitor --> UCChat
Visitor --> UCAuth

Student --> UCProfile
Student --> UCSave
Student --> UCApply
Student --> UCTrack
Student --> UCHistory

Recruiter --> UCManageJobs
Recruiter --> UCReview
Recruiter --> UCStatus

Admin --> UCAdmin

UCCareer --> ML : requests recommendation
UCResume --> ML : requests resume analysis
UCChat --> ML : requests career advice / skill-gap analysis
UCApply --> UCEmail : application confirmation
UCStatus --> UCEmail : status update
ML --> UCCareer
ML --> UCResume
ML --> UCChat
Email --> UCEmail
@enduml
```

### 4.4.2 Activity Diagram

This activity shows the job-application workflow from the student dashboard through the backend.

```plantuml
@startuml
title Aspiro - Activity Diagram: Apply for a Job

|Student|
start
:Browse public jobs;
:Select a job;
:Enter cover letter (optional);
:Submit application;

|Frontend|
:POST /api/applications/apply/{jobId};

|Express API|
:Authenticate request;
if (User authenticated?) then (Yes)
  :Check permitted role;
else (No)
  :Return authentication error;
  stop
endif

:Find selected job;
if (Job exists?) then (Yes)
else (No)
  :Return "Job not found";
  stop
endif

:Check for existing\nuser-job application;
if (Already applied?) then (Yes)
  :Return existing application;
  :Trigger application email notification;
else (No)
  :Find user's latest resume;
  if (Latest resume exists?) then (Yes)
    :Attach resume reference;
  else (No)
    :Continue without resume;
  endif
  :Create pending application;
  :Trigger application email notification;
endif

|Student|
:View application result;
stop
@enduml
```

### 4.4.3 Sequence Diagram

This sequence follows the implemented `POST /api/applications/apply/:jobId` route.

```plantuml
@startuml
title Aspiro - Sequence Diagram: Job Application
autonumber

actor Student
participant "React Frontend" as Frontend
participant "Express API\n(Auth middleware)" as API
participant "Application Controller" as Controller
database "MongoDB" as DB
participant "Email Notification\nService" as Email

Student -> Frontend : Choose job and submit
Frontend -> API : POST /api/applications/apply/{jobId}\nAuthorization + coverLetter
API -> API : Verify token and allowed role
API -> Controller : applyForJob(req, res)
Controller -> DB : Job.findById(jobId)
DB --> Controller : Job or not found

alt Job does not exist
  Controller --> Frontend : 404 Job not found
  Frontend --> Student : Show error
else Job exists
  Controller -> DB : Application.findOne(userId, jobId)
  DB --> Controller : Existing application or null
  alt Application already exists
    Controller -> DB : User.findById(userId)
    DB --> Controller : User details
    Controller ->> Email : Send application email (async)
    Controller --> Frontend : 200 Existing application
    Frontend --> Student : Show existing application
  else New application
    Controller -> DB : Resume.findOne(userId, isLatest=true)
    DB --> Controller : Latest resume or null
    Controller -> DB : Application.create(user, job,\ncoverLetter, optional resume, pending)
    DB --> Controller : Created application
    Controller -> DB : User.findById(userId)
    DB --> Controller : User details
    Controller ->> Email : Send application email (async)
    Controller --> Frontend : 201 Application submitted
    Frontend --> Student : Show confirmation
  end
end
@enduml
```

### 4.4.4 Collaboration Diagram

This communication (collaboration) diagram presents the same application workflow, emphasizing the numbered messages between objects.

```plantuml
@startuml
title Aspiro - Collaboration Diagram: Job Application
left to right direction

object "student:Student" as Student
object "ui:ReactFrontend" as Frontend
object "api:ExpressAPI" as API
object "controller:ApplicationController" as Controller
object "db:MongoDB" as DB
object "email:EmailNotificationService" as Email

Student --> Frontend : 1: select job and submit
Frontend --> API : 2: POST /api/applications/apply/{jobId}
API --> Controller : 3: authenticate and call applyForJob
Controller --> DB : 4: find Job by id
DB --> Controller : 5: return Job
Controller --> DB : 6: find existing Application(user, job)
DB --> Controller : 7: return existing / null

Controller --> DB : 8a: find latest Resume (if no existing application)
DB --> Controller : 8b: return Resume / null
Controller --> DB : 9: create pending Application (if new)
DB --> Controller : 10: return created application
Controller --> Email : 11: send application email asynchronously
Controller --> Frontend : 12: return application result
Frontend --> Student : 13: display confirmation or error
@enduml
```

### 4.4.5 Class Diagram

The class diagram shows the persisted domain models and the main request-processing/integration components.

```plantuml
@startuml
title Aspiro - Class Diagram
skinparam classAttributeIconSize 0

class User <<Mongoose Model>> {
  +name: String
  +email: String
  +password: String
  +role: student | recruiter | admin
  +phone: String
  +location: String
  +skills: String[]
  +experience: Embedded[]
  +education: Embedded[]
  +resumeAnalysis: Embedded
  +uploadcareResume: Embedded
  +isEmailVerified: Boolean
  +isActive: Boolean
  +getSignedJwtToken()
  +matchPassword(enteredPassword)
  +getResetPasswordToken()
}

class Job <<Mongoose Model>> {
  +title: String
  +description: String
  +location: String
  +type: String
  +experience: String
  +skills: String[]
  +status: String
  +visibility: String
  +recruiter: ObjectId
  +company: String
  +applicationDeadline: Date
  +isExpired: Boolean
}

class Application <<Mongoose Model>> {
  +user: ObjectId
  +job: ObjectId
  +resume: ObjectId?
  +status: String
  +coverLetter: String
  +appliedAt: Date
  +lastUpdated: Date
}

class Resume <<Mongoose Model>> {
  +user: ObjectId
  +originalName: String
  +filePath: String
  +fileSize: Number
  +mimeType: String
  +analysis: Embedded
  +isActive: Boolean
  +isLatest: Boolean
}

class SavedJob <<Mongoose Model>> {
  +user: ObjectId
  +job: ObjectId
  +savedAt: Date
}

class Notification <<Mongoose Model>> {
  +user: ObjectId
  +application: ObjectId
  +type: email | sms | whatsapp
  +status: pending | sent | failed
  +message: String
  +recipient: String
  +applicationStatus: String
  +sentAt: Date
}

class ApplicationController <<Controller>> {
  +applyForJob(req, res)
  +getMyApplications(req, res)
  +updateApplication(req, res)
  +withdrawApplication(req, res)
}

class CareerController <<Controller>> {
  +getCareerSuggestions(req, res)
  +generateCareerReport(req, res)
  +getCareerHistory(req, res)
}

class ResumeController <<Controller>> {
  +analyzeResume(req, res)
  +getLatestResume(req, res)
  +getResumeHistory(req, res)
}

class CareerRecommender <<ML Service>> {
  +predict(userProfile)
}

class ResumeAnalyzer <<ML Service>> {
  +analyze(resumeFile, jobRole)
}

class ChatbotML <<ML Service>> {
  +getCareerRecommendations(message)
  +analyzeSkillsGap(skills, career)
}

class EmailNotificationService <<Service>> {
  +sendJobApplicationEmail(user, job)
  +sendStatusUpdateEmail(user, job, status)
}

User "1" -- "0..*" Job : recruiter
User "1" -- "0..*" Application : applicant
Job "1" -- "0..*" Application : receives
User "1" -- "0..*" Resume : owns
Resume "0..1" -- "0..*" Application : attached resume
User "1" -- "0..*" SavedJob
Job "1" -- "0..*" SavedJob
User "1" -- "0..*" Notification
Application "1" -- "0..*" Notification

ApplicationController ..> Application : creates / updates
ApplicationController ..> Job : looks up
ApplicationController ..> Resume : selects latest
ApplicationController ..> EmailNotificationService : sends email
CareerController ..> CareerRecommender : HTTP request
ResumeController ..> ResumeAnalyzer : HTTP request
ResumeController ..> Resume : persists analysis
CareerController ..> User : reads profile/history
@enduml
```

## Notes for the Report

- The database is MongoDB, so the diagram labels ObjectId references as foreign-key-like references; they are not relational SQL constraints.
- `Application` and `SavedJob` prevent duplicate user/job pairs through unique compound indexes.
- Resume analysis and career recommendation requests are forwarded by the Express backend to the Flask ML service.
- Resume analysis and career suggestions can be invoked without authentication in the current route setup; results are associated with a user only when the request has an authenticated user.
- The `Notification` model is included because it exists in the backend data model. The application email flow is handled by the email notification service.
- The job-application route is described for students, although its current authorization middleware also permits recruiter accounts.
