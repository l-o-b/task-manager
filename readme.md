# Task Manager Service

## POST /tasks

```
{ 
  title : 'email me some positive thoughts. I am feeling down!'
      title or short description of the task
      displayed prominently in task listings and such
      some listing display shorter portions if you put a long title
      expect to be shown in many places with ellipsis

      required 
      
  description  :  ''
      longer/full description of the task
      typically displayed when showing a single task

      optional 
      empty 

  deadline  : 
      deadline for the task
      datetime in whatever is the standard form an API

      optional  
      default to 24 hrs from submission time
      negative or 0 means asap and changes into now (plus + estimate)

  
  estimate : 300  //assuming that the units is "seconds"
      submitter expectation of time that it would take to finish the task
      in whatever is the standard unit for time apis

      optional
      default => 1 hr

     
  owner :  "odysseas@ezhome.com"
      who this task is for
      using email because it makes the API more easy to use..everyone has the email
      quite possibly internally the app should always be recording user-id  but allow
      incoming user references to be email based and outgoing user references are also 
      converted into the "user's current email" for api friendlyness

      optional
      default : the currently authenticated user
      
  tags : [ tag1 tag2 ] # list of tags
      optional 
      tags are used as the catch-all instead of structured filtering (list-ing) for now
      

}
```

In response the POST call returns the newly created object
which includes filled in defaulted empty attributes, (equiv to GET /tasks/5768)

The above should be support the use case of the add command.

## GET /tasks/5768

```
{   
  title : 'email me some positive thoughts. I am feeling down!'
  description  :  
  deadline  : '2015-11-20T17:31:33Z'
  estimate : 300
  owner :  'odysseas@ezhome.com'
  tags : [ tag1 tag2 ]

  /* system attrs */
  _id : 5768
  _created_by : 'odysseas@ezhome.com'    
  _created_on : '2015-11-20T17:26:33Z' 
  _last_modified_on : '2015-11-20T17:26:33Z'
  _state : 'queued'
  _worked_by : 'mary@ezhome.com'            # if the task is under the control of a worker which worker that is
  _worked_on : '2015-11-20T17:26:33Z'       # if the task is under the control of a worker .. when it was last grabbed */
  _work_status : suspended or active        # if the task is under the control of a worker 
  _completion_status : success or failure   # if the test is completed 
  _completed_on : '2015-11-20T17:26:33Z'    # if the test is completed 

}
```

The above should be able to support the use case of show command

Updating an existing task 
with PUT or whatever is the right convention for the update of an existing object

## PUT /tasks/5768     
```
{   
  _id : 5768
  title : 'email me some positive thoughts. I am feeling down!'
  description  :  
  deadline  : '2015-11-20T17:31:33Z'
  estimate : 300
  owner :  'odysseas@ezhome.com'

}
```
=> returns the updated object same as GET /tasks/5768

Given the minimal need for this I am trying to keep the implementation as simple as possible.

For simplicity, the complete object is being replaced, 
ie, if a parameter is missing we do not keep existing values, we simply follow the same defaulting 
logic that we follow on `add` to generate defaults for the missing parameters.
Also, note that for now we do not allow tag update (more complex will need tag add and tag remove functionality)

The only "business logic" is that we only allow a task to be updated if it is in `queued` state.
Descriptive error messages are generated otherwise, e.g.
```
Cannot Update. Task is  (being worked on|completed|deleted)
Task does not exist.
```
The above should be able to support the use case of update command

## DELETE /tasks/5768     
```
{   
  _id : 5768
}
```

It sets the state to "deleted"  - which is considered a final state
Only queued tasks are allowed to be deleted

=> returns the deleted object same as GET /tasks/5768
Descriptive error messages are generated on error
```
Cannot Delete. Task is  (being worked on|completed)
Task does not exist.
```




## GET /tasks/5768/history

```
{
  events : [         # complete chronoligical history of events
    { when : '2015-11-20T17:31:33Z', who : 'joe@ezhome.com', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', who : 'joe@ezhome.com', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', who : 'joe@ezhome.com', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', who : 'joe@ezhome.com', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', who : 'joe@ezhome.com', what : 'grab' },
  ]

  # some aggregates on top of the history of the events
  time_elapsed       # time since first submitted
  time_til_deadline  # 
  time_worked_on     # time the task has spent being the active task of the current worker (only if in-progress)
  time_suspended     # time the task has spent suspended by the current worker (only if in-progress)
  time_queued        # total time in the queue
  time_rejected      # total time under the control of users that eventually released the task without completing it
  times_rejected     # how many times the current task has been rejected by prior users
}
```

The above should be able to support the use case of show command

## GET /workers/xxx/long=true
```
{ 
  _id : xxx
  userid : odysseas@ezhome.com

  current: {}
  suspended: [ ] #lifo first in list is last suspended

  events : [         # complete chronoligical history of events since login
    { when : '2015-11-20T17:31:33Z', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', what : 'grab' },
    { when : '2015-11-20T17:31:33Z', what : 'grab' },
  ]

}
```
the task  detail inside each is same as GET /tasks/56  except if long=true in which case the returned tasks 
contain a history section (without events)
The above should be able to support the use case of finger,jobs,ps command


## GET /tasks?state=queued&owner=u&n=N&tag=tag1

returns a list of up to N tasks 
    [id1,id2,id3]
Primary use case is the support of the list command
If a tag is provided only the tasks that have this tag is returned.

## GET /tasks/available?n=N
returns
    [id1,id2,id3,...]

Primary use case is the support of the peek command
Returns a list of tasks ordered (1st most urgent ) up to N
if N missing returns just 1
In the simplest form it can return tasks based on order requested

## POST /workers/
```
{
  id : odysseas@ezhome.com
}
```
returns
```
{ 
  _id : 4567  # a worker-id
  userid : odysseas@ezhome.com
  loggedin_on : 
}
```
This is to support the `login` functionality
has the biz logic of the prototype

## GET /workers
returns
```
[
  { 
    _id : 4567  # a worker-id
    userid : odysseas@ezhome.com
    loggedin_on : ..
  },
  { 
    _id : 1234  # a worker-id
    userid : jim@ezhome.com
    loggedin_on : ...
  },
  {}, {}...
]
```

This is to support the `who` functionality

## DELETE /workers/xxx
This is to support the logout functionality
has the biz logic of the prototype


## POST /workers/xxx/tasks
```
{
  task : 4567  # this is to support the grab xxx functionality
}
```
returns
has the biz logic of the prototype

## PUT /workers/xxx/tasks/xx
```
{
  status : fg or bg or release/complete-success/complete-failure
}
```
this is to support fg xx or bg xx
has the biz logic of the prototype 

## GET /events?user=xx&task=tt&type=loginout&type=tasks&N=&
returns in chronological order the last N events for the particular selection

Supports the use case of last lastcomm finger
    time_since_login   # if still logged in
    time_since_logout  # if logged out


## GET /worker-stats?from=&to&worker=
```
{
  worker1 : {
    avg_session_length :
    num_of_sessions :
    time_loggedin :
    time_active :
    tasks_grabbed :
    tasks_completed :
    tasks_completed_successfully :
    rejection_ratio :
    failure_ratio :
    avg_tat :
  },
  worker2 : {

  },
  ..
}
```
supports the use case of getting performace stats for a particular user


## GET /owner-stats?from=&to&owner=
```
{
  owner1 : {
    tasks_requested :
    tasks_completed :
    tasks_completed_successfully :
    time_active :
    num_of_workers
    rejection_ratio :
    failure_ratio :
    
    avg_tat
    avg_time_worked_on  
    avg_time_suspended  
    avg_time_queued    
    avg_time_rejected  
    avg_times_rejected 

  },
  owner2 : {

  },
  ..
}
```
supports the use case of getting costs amount of work given by specific requestor

## GET /task-stats?from=&to&task-pattern=xxx
```
{
  avg_tat
  avg_time_worked_on  
  avg_time_suspended  
  avg_time_queued    
  avg_time_rejected  
  avg_times_rejected 
}
```
supports the use case of getting stats for a given type of work/ or just a batch


