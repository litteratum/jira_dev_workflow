# Development workflow with Jira

## General rules
1. Use Jira's projects. One project in reality - one project in Jira ([why?](#why-to-use-jira-projects))
2. Regardless who you are and what are you working on, if you find a bug in any project, you must create a task for it. The description may be hight level, a more qualified specialist will improve it later if needed

## Task fields
  * Description. Obvious
  * Priority. Obvious
  * Due date. Obvious
  * Time tracking. Obvious
  * [Contributors](#contributors)
    * Reporter. It is a standard field. Obvious
    * Assignee. It is a standard field. Obvious
    * [Manager](#manager-task-field)
    * [Executor](#executor-task-field)
    * [Reviewer](#reviewer-task-field)
    * [QA](#qa-task-field)
  * RC. Defines the RC number this task belongs to (1...N)

## Task statuses
  * `Draft`: the task is just created. Not ready for the development
  * `Approved`: the task was approved. We may start working on it according to priorities
  * `Rejected`: the task was rejected. Still may be approved if the reporter improves the description or brings new arguments
  * `Under Review:` the task is solved and awaits for the code review
  * `Reviewed`: the task was reviewed
  * `Under QA`: the task awaits for the QA
  * `QA OK`: the task passed the QA
  * `Returned`: the task was returned back to the development (either from CR or testing)
  * `Done`: the task is completely solved

## Task types
  * Development tasks. There is no functional difference between them. It is only for visualization or statistics
    * `Bug`: some issue, something to fix
    * `Feature`: something new to add
    * `Refactoring`: something to improve
  * Special tasks
    * [Question](#question-task-type)
    * [Step](#step-task-type)
    * [Release](#release-task-type)

### Question task type
Question task type is for something to research, discuss, or ask.
This type does not have "QA" field. The "Reviewer" field is empty by default.

### Step task type
Step is a task within another task. Use it only when creating a separate task is an overkill.
Steps are usually used in automation to define some pre-defined tasks. For example, you should usually follow the same steps to release a package, so the Release task will always be created with a predefined set of steps.

### Release task type
This is a task type for release preparation.
This task must always be created with predefined [steps](#step-task-type) (using automation). For example:
  * Test Release 1.1.0
  * Publish Release 1.1.0

This type does not have "QA" and "Reviewer" fields.


## Task transitions
  * Draft: Auto reassigned to the [Manager](#manager-task-field)
    * Approved. We will solve it
    * Rejected. We will not solve it or the reporter must describe the task better
  * Approved. The Assignee is cleared
    * Draft. there is something wrong with the task. A manager's attention required
    * Under Review. The task is solved by an executor. Now it is waiting code review
  * Rejected. Auto reassigned to the reporter:
    * Draft. The reporter improved the description or brought new arguments
  * Under Review. Auto reassigned to the [Reviewer](#reviewer-task-field):
    * Reviewed
    * Returned
  * Reviewed. Auto reassigned to the [Executor](#executor-task-field):
    * Under Review. If more work from reviewers is required
    * Under QA
    * Done. The task transitions to this status automatically if the "QA" field is empty or absent
  * Under QA:
    * QA OK
    * Returned
  * QA OK. Auto reassigned to the [Executor](#executor-task-field):
    * Under QA. If more work from QA is required
    * Done
  * Returned. Auto reassigned to the [Executor](#executor-task-field):
    * Under review
  * Done: the final task's state

## Why to use Jira projects
The reason is simple - it is designed this way. It allows a flexible configuration per project.
The most useful thing - setting default [contributors](#contributors) for each project.
Also, give access to the project only to certain people.

## Contributors
Contributors are people contributing to a task. Those people transition a task from one state to another until it is finally done.
There are different contributor roles like executor, QA, reviewer, etc.

**Important note**: if you are assigned as a contributor to a task, it does not necessarily mean you will contribute to it, but **you will always be responsible for the contribution** ([why?](#why-am-i-responsible-for-contribution)).

### Why am I responsible for contribution
So let's say you are assigned as a reviewer for a task. The task's executor solved the task and made a pull request asking Bob and Ann for the review. But you are assigned to the task, why?
There are two main reasons:
1. Jira does not allow multiple assignees
2. Even if Jira allowed multiple assignees, only one of them must transition a task to the "Reviewed" state

### Executor task field
This field may seem redundant since we already have the "Assignee" field. But "Assignee" is a completely different thing.
Each task will go from one person to another, so it means the assignee will constantly change. But the "Executor" field almost never gets changed.

If you are an "Executor", it almost always means you will be responsible for the development of the task. But not always ([why?](#why-am-i-responsible-for-contribution)).

Work on a task always starts from the development, so the "Executor" is a person who will be assigned to the task via automatization (or manually). It is also a person who will get the task once it is reviewed, tested, or returned (automatization).

### Manager task field
A manager is a person who is responsible for managing tasks.
Usually, it means the following:
1. Decide whether the task is needed. Approve it or reject it
2. Ensure that other contributors understand the task's requirements
3. Control the task's progression (set deadlines, detect hanging tasks, answer questions, etc.)

When a task is moved to the "Draft" status, the task is automatically reassigned to the "Manager".

A manager must constantly review "Draft" tasks and either approve or reject them.

I have one **cool automatization suggestion**: if a task is assigned to somebody but it has not been touched for N days, reassign it to the "Manager". This way, the manager will automatically be aware of the problem and may take further action (set deadline, close the task, reassign to another person, fire the executor :D, and so on).

### Reviewer task field
A reviewer is a person who is responsible for reviewing tasks.

The field is good for automatization: when a task is solved and the status is changed to "Under Review", the task is reassigned to the "Reviewer".

When the review is done and the task's status is changed, the task is automatically reassigned to the executor.

### QA task field
QA is a person who is responsible for the quality assurance of the task.
QA reads the task description and ensures that the application works according the description.

The field is good for automatization: when an executor prepares an RC and changes the task's status to "Under test", the task is reassigned to the "QA".

When the QA is done and the task's status is changed, the task is automatically reassigned to the executor.

## Development workflow
1. Sprint planning. The team meets and decide what will be accomplished within the next sprint (1-4 weeks). Move tasks from the backlog to the sprint. I would auto assign a task to the "Executor" when it is moved to the sprint. The team may also create the next release version if not done yet
2. Contributors solve the tasks within the sprint. See [task solving](#task-solving)
3. After the sprint is finished, the team gather for retrospection meeting and work on the next sprint (go to [1])
4. At some point we release the changes

### Task solving
1. The Executor solves the task in a separate git branch
2. The Executor pushes the changes. It is either a pull request (preferred) or a direct push to the branch. In case of a pull request, the Executor selects the reviewers
3. The Executor changes the task's status to "Under Review". The task is automatically reassigned to the "Reviewer"
4. Reviewers review the task. The task's Reviewer is responsible for ensuring that all the reviewers have reviewed the task (easier to achieve with pull requests). Once the task is reviewed, it is merged to the "master" and the task's status is changed to "Reviewed" or "Returned" (see [task is returned from CR](#task-is-returned-from-cr))
5. Once the [RC is prepared](#rc-preparation), the Executor changes the task's status to "Under QA". The task is automatically reassigned to the "QA". The QA's job must be clear from the task's description or the last comment. Once the QA is done, the task's status is changed to "QA OK" or "Returned" (see [task is returned from QA](#task-is-returned-from-qa))
6. The Executor checks the QA output and changes the task's status to "Done"

### Task is returned from CR
If a task is returned from CR, the Executor must decide if it is justified. If not, the task may be returned to the CR again.
Otherwise, the Executor fixes the issues found during CR and asks for the CR again. Usually, the fixes are pushed to the same git branch.

### Task is returned from QA
The same as with code review, the task may be assigned back to QA if the QA's conclusion is not justified (e.g. QA did something wrong).
Otherwise, the [task solving](#task-solving) cycle repeats.

### RC preparation
At some point, when there is something ready to be tested, the release "Executor" may decide to build a release candidate (RC):
  * The Executor creates "X.Y.ZrcN" git tag, where "X.Y.Z" - current release version, and "N" - the sequential RC number
  * The CI builds the package and makes it available for testing
  * The Executor sets the "RC" field for each task that belongs to the RC
  * The Executor changes the tasks' status to "Under QA"
