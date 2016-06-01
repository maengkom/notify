## Installation

1. Copy folder `packages\notice` to the laravel root folder.
2. Copy config file from package to folder config in root laravel and rename it as `notice.php` 
3. Copy migration file and create the table

##### Config

Configuration is simple for now. Just for enable or disable the feature. Also to tell package which on is notification table should be used.

```
return [

    /*
    |--------------------------------------------------------------------------
    | Notice table
    |--------------------------------------------------------------------------
    |
    | This table is used when persistent all notification into database
    | the default is 'notifications' table, but you may use different table,
    | but don't forget to change the migration file to create the new table
    |
    */

    'table' => 'notifications',

    /*
    |--------------------------------------------------------------------------
    | Notice Type
    |--------------------------------------------------------------------------
    |
    | You can set which type to run in the system. When a type set to false
    | that type will not send any notification to receiver.
    |
    */

    'text'  => true,
    'email' => true,
    'flash' => true,

    /*
    |--------------------------------------------------------------------------
    | Email Sender Info
    |--------------------------------------------------------------------------
    |
    | You can set email sender information to display in the recipients
    |
    */

    'subject'   => 'Shokse Notification System',
    'sender'    => [
        'email' => 'admin@shokse',
        'name'  => 'Administrator'
    ],

];
```

## Usage

Put the code below in the controller method or model suitable to listen the activities.

For example `Notice::project()->created($data);` can be put in `ProjectController` in method `store()` when creating a new project.

##### Project Notification

1. Notice::project()->created($data);
2. Notice::project()->taken($data);
3. Notice::project()->breakdown($data);
4. Notice::project()->delivered($data);
5. Notice::project()->validated($data);

##### Task Notification

1. Notice::task()->created($data);
2. Notice::task()->applied($data);
3. Notice::task()->applicationApproved($data);
4. Notice::task()->applicationRejected($data);
5. Notice::task()->delivered($data);
6. Notice::task()->validated($data);

##### Doc Notification

1. Notice::doc('project')->edited($data);
2. Notice::doc('task')->edited($data);

##### Usage Example

`$data` parameter will need a fix structure ike this below. For example in `ProjectController` after creating a new Project

```
// Recipients id who will get notified from query based on match skills
$ids = collect([2]); 

$data = [
    'receiver_ids'  => $ids,	
    'content'       => [
        'type'      => 'project',			// Type : project, task or doc
        'id'        => $project->id, 		// Project id 
        'title'     => $project->title		// Project title
    ]
];

// Notify in the action
Notice::project()->created($data);
```

## Output

##### How to test

Note : Flash notification and badge in top right corner, appear when there are some unread notification. To mark as read, currently just click link "Mark all as read" in notification icon (top right corner).

1. Open 3 different browsers or perhaps can in 3 private tabs. So it will have 3 different sessions.
2. Each login with path : /loginascompany, /loginasexpert, /loginasdeveloper
3. Try to trigger the activities in the menu, such as "Project created", login as Company.
4. Refresh the expert page, it will get notified. Also email send to the expert email.
5. Next try to trigger 'Project taken' by expert page.
6. Refresh company page, then it will get notified.
7. So on, and for text notification in top right corner, can check 'Mark all as read', the current user will not get notified again.

##### Flash Notification
To display notification in views just put this code before </body> tag or anywhere should be ok. This is generated a script block to check unread.

```
{!! html_entity_decode(\Notice::flash()->get()) !!}
```

##### Text Notification

To display list notification, include read and unread : 

```
$notif = \Notice::text()->all();
```

Then loop to display the list, get the object properties with column name.


##### Email Notification

Email notification is directly send to the email of recipients (receiver_id) after listener called.

## Others

##### Table

Notification table will store these data :

| Column      | Description                                                   |
|-------------|---------------------------------------------------------------|
| id          | increment                                                     |
| message     | generated in package per object (Project, Task or Doc)        |
| receiver_id | single receiver id                                            |
| sender_id   | come from Auth::user()->id                                    |
| type        | type of object (project, task or doc)                         |
| type_id     | id of object (project id, task id or wiki id)                 |
| type_title  | title of object (project title, task title, or article title) |
| status      | 0 = unread, 1 = read                                          |
| created_at  | builtin                                                       |
| updated_at  | builtin                                                       |
| deleted_at  | builtin                                                       |

##### Email

Email to send notification will use the setting in .env. For message template located in /resources/views/notify/email.blade.php. The package already pass the record in notification table.

To test you can use mailtrap.io, its free.

```
MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=null
```
