# Lab 6 — Answers

## Exercise 1

- (a) Messages **3** and **5** are returns.
- (b) `Course` must have `removeStudent(student)` (message **2**) and `getCredits()` (message **4**). `UndergraduateStudent` must have `removeCredits(6)` (message **6**).
- (c) As drawn, the service still gets the course's credits and removes 6 credits from the student, even though the student was not enrolled. Add an `alt` fragment after message 3: put messages **4–6** in the `[student was enrolled]` branch and leave the `[student was not enrolled]` branch empty.

## Exercise 2

```mermaid
sequenceDiagram
    participant S as service: EnrollmentService
    participant P as printer: Printer
    create participant E as enrollment: Enrollment
    S->>E: new Enrollment(student, course)
    S-)P: printConfirmation(enrollment)
```

- (a) The enrollment box starts lower because the enrollment object is created during the interaction.
- (b) Message 2 is an asynchronous message, drawn with a solid line and an open arrowhead.

## Exercise 3

```mermaid
sequenceDiagram
    participant S as service: EnrollmentService
    participant C as course: Course
    participant L as lab: LabSession
    S->>C: hasLab()
    C-->>S: has lab?
    opt course has a lab
        S->>L: reserveBench(student)
    end
```

## Exercise 4

```mermaid
sequenceDiagram
    actor User
    participant S as service: EnrollmentService
    participant C as course: Course
    participant W as waitlist: Waitlist
    User->>S: enroll(student, course)
    S->>C: hasSeatLeft()
    C-->>S: seat left?
    alt seat is left
        S->>C: addStudent(student)
        S-->>User: "enrolled"
    else no seat is left
        S->>W: add(student)
        S-->>User: "waitlisted"
    end
```

## Exercise 5

```mermaid
sequenceDiagram
    participant T as transcript: Transcript
    participant E as enrollment: Enrollment
    loop for each enrollment
        T->>E: getGrade()
        E-->>T: grade
        T->>E: getCredits()
        E-->>T: credits
    end
```

## Exercise 6

```mermaid
sequenceDiagram
    actor User
    participant S as service: EnrollmentService
    participant C as course: Course
    participant St as student: UndergraduateStudent
    User->>S: enroll(student, course)
    S->>C: getPrerequisite()
    C-->>S: prereq
    S->>St: hasPassed(prereq)
    St-->>S: passed?
    break student has not passed the prerequisite
        S-->>User: "missing prerequisite"
    end
    S->>C: isOpen()
    C-->>S: open?
```

## Exercise 7

```mermaid
sequenceDiagram
    participant G as grades: GradeService
    participant St as student: Student
    participant A as advisor: Advisor
    par notify student
        G-)St: notify(grade)
    and notify advisor
        G-)A: notify(grade)
    end
```

## Exercise 8

| Sentence from the requirements | Fragment |
|---|---|
| "For each course in the student's list, …" | `loop` |
| "If the student asked for a printed timetable, also print it." | `opt` |
| "The student pays the fees either by card or by bank transfer." | `alt` |
| "If registration is closed, refuse and stop." | `break` |
| "Email the student and the advisor at the same time." | `par` |

## Exercise 9

```mermaid
sequenceDiagram
    actor Registrar
    participant S as service: WaitlistService
    participant C as course: Course
    participant W as waitlist: Waitlist
    Registrar->>S: promote(course, waitlist)
    S->>C: hasSeatLeft()
    C-->>S: seat left?
    alt no seat is left
        S-->>Registrar: undefined
    else a seat is left
        S->>W: next()
        W-->>S: next?
        alt waitlist is empty
            S-->>Registrar: undefined
        else a student is next
            S->>C: addStudent(next)
            S-->>Registrar: next
        end
    end
```

## Exercise 10

```typescript
enrollAll(student: Student, courses: Course[]): number {
  let accepted = 0;
  for (const course of courses) {
    if (course.isOpen()) {
      const credits = course.getCredits();
      if (student.addCredits(credits)) {
        accepted++;
      }
    }
  }
  return accepted;
}
```

## Exercise 11

| # | Line of the diagram | What is wrong | Correction |
|---|---|---|---|
| 1 | `S->>E: new Enrollment(student, course)` | The enrollment is created before checking whether the course is open, so the closed-course path would create one too. | Move the creation into the open-course branch, after checking `isOpen()`. |
| 2 | `C->>S: true` | A return is drawn as a call from the course, and `true` does not express either possible result. | Use the return arrow `C-->>S: open?`. |
| 3 | `S->>C: seats = seats - 1` | The service directly changes the course's internal state instead of asking the course to add the student. | Use `S->>C: addStudent(student)`. |

```mermaid
sequenceDiagram
    actor User
    participant S as service: EnrollmentService
    participant C as course: Course
    User->>S: enroll(student, course)
    S->>C: isOpen()
    C-->>S: open?
    alt course is open
        S->>C: addStudent(student)
        create participant E as enrollment: Enrollment
        S->>E: new Enrollment(student, course)
        S-->>User: enrollment
    else course is closed
        S-->>User: undefined
    end
```
