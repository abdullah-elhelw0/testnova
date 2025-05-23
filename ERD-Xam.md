## Final ERD

### Entities and Attributes

#### 1. User

- **user_id** (Primary Key, integer)
- **username** (string)
- **password** (string)
- **role** (enum: 'instructor', 'admin')
- **is_approved** (boolean)  
  _Description_: Represents all users (instructors and admins). The `is_approved` field ensures only validated instructors can create exams.

#### 2. Instructor

- **instructor_id** (Primary Key, integer, Foreign Key to User.user*id)  
  \_Description*: Sub-entity for instructors, linked to the `User` table via `user_id`.

#### 3. Admin

- **admin_id** (Primary Key, integer, Foreign Key to User.user*id)  
  \_Description*: Sub-entity for admins, linked to the `User` table via `user_id`.

#### 4. Exam

- **exam_id** (Primary Key, integer)
- **exam_name** (string)
- **description** (text, optional)
- **start_date** (datetime)
- **end_date** (datetime)
- **duration** (integer, in minutes)
- **instructor_id** (Foreign Key to Instructor.instructor*id)  
  \_Description*: Stores exam details, created by an instructor.

#### 5. Question

- **question_id** (Primary Key, integer)
- **exam_id** (Foreign Key to Exam.exam_id)
- **question_type** (enum: 'multiple-choice', 'true/false', 'short-answer', 'essay')
- **question_text** (text)
- **score** (integer)  
  _Description_: Represents questions within an exam, supporting various types.

#### 6. Option

- **option_id** (Primary Key, integer)
- **question_id** (Foreign Key to Question.question_id)
- **option_text** (string)
- **is_correct** (boolean)  
  _Description_: Stores options for multiple-choice or true/false questions.

#### 7. AllowedStudents

- **allowed_id** (Primary Key, integer)
- **exam_id** (Foreign Key to Exam.exam_id)
- **uni_id** (string)  
  _Description_: Lists UNI IDs of students allowed to take a specific exam, uploaded by the instructor.

#### 8. ExamAttempt

- **attempt_id** (Primary Key, integer)
- **exam_id** (Foreign Key to Exam.exam_id)
- **uni_id** (string)
- **start_time** (datetime)
- **end_time** (datetime)
- **score** (integer)  
  _Description_: Tracks each student’s exam attempt using their UNI ID.

#### 9. Answer

- **answer_id** (Primary Key, integer)
- **attempt_id** (Foreign Key to ExamAttempt.attempt_id)
- **question_id** (Foreign Key to Question.question_id)
- **answer_text** (text)
- **is_correct** (boolean, for auto-graded questions)  
  _Description_: Stores student responses for each question in an attempt.

---

### Relationships

- **Instructor creates Exam**:  
  One instructor can create many exams (one-to-many).  
  _Link_: `Exam.instructor_id` → `Instructor.instructor_id`.

- **Exam has Question**:  
  One exam can have many questions (one-to-many).  
  _Link_: `Question.exam_id` → `Exam.exam_id`.

- **Question has Option**:  
  One question can have multiple options (one-to-many).  
  _Link_: `Option.question_id` → `Question.question_id`.

- **Exam has AllowedStudents**:  
  One exam has a list of allowed UNI IDs (one-to-many).  
  _Link_: `AllowedStudents.exam_id` → `Exam.exam_id`.

- **ExamAttempt belongs to Exam**:  
  Many attempts can belong to one exam (many-to-one).  
  _Link_: `ExamAttempt.exam_id` → `Exam.exam_id`.

- **ExamAttempt has Answer**:  
  One attempt can have many answers (one-to-many).  
  _Link_: `Answer.attempt_id` → `ExamAttempt.attempt_id`.

---

### Key Design Notes

- **No Student Entity**:  
  Since students don’t have accounts, there’s no `Student` table. Instead, their UNI IDs are stored in:

  - `AllowedStudents`: Validates which students can take an exam (based on instructor-uploaded lists).
  - `ExamAttempt`: Tracks who took the exam by UNI ID.

- **Instructor Validation**:  
  The `User.is_approved` field ensures only approved instructors can create and manage exams.

- **Student Access**:  
  The `AllowedStudents` table links each exam to a list of valid UNI IDs, allowing the system to check eligibility before granting access.

- **Result Export**:  
  Results can be generated by querying `ExamAttempt` and `Answer`, grouped by `uni_id`, and exported as a CSV.
