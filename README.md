# Web-Programming-Chat

Instance messaging web app

Description: web application for users to communicate in chennels and groups to other participants in real time.

## Run

Enter the webpage directory
```cmd
cd Chatty-Chat
```

### Build Client
```cmd
npm install
ng build
```

### Run Database Server

Note: The database folder can be located anywhere
```cmd
mkdir database
mongod --dbpath database
```
Runs on mongodb://localhost:27017/

### Run Web Server

```cmd
node server/index.js
```
Runs on http://localhost:3000/

## Git (Version Control)

### Layout

- Chatty-Chat:      Chat angular and server application source code
  - src:              client source code (angular)
  - server:           server source code (nodejs)
  - database:         database file locations (mongo)
- prototyping:      mainly screen design prototyping
- media:            assest and corresponding production files

### Version Control Approach

The project is commited with every significant change. Such change may include addition of a significant feature, bug fix or a number of little addition. Each commit should be functional and the features it introduces should be complete such that the project as a whole can run as correctly as can be expected from the stage the project is at.

## Requirements / appication model

- Groups
  - Channels
- Users
  - Details
    - Username
    - Email
    - Password
    - Initial user is called super
  - Group admins
    - Manage groups and channels
    - Manage users in channel
    - Elevate users to group admin
  - Super admin
    - System wide access
    - All of group admins
    - Create admin user + group pair
    - Remove users
    - Elevate users to super admins
- Login - authentification
  - login with username/password
  - super admin creates user
- Pages
  - Landing page
    - Signed out
      - Login page
      - Remeber login details locally
    - Signed in
    - List of groups
      - List of channels
    - Channel selected
      - Chat history - messages
      - Text box to send messages
      - Image selectior
      - Send text and image messages
      - Connect to socket for instant messaging
- Dialogs
  - Delete item dialogues
    - delete a channel
    - delete a group
  - New item dialogues
    - new channel
    - new group
  - Manage members dialogues
    - manage members of channels
    - manage members of groups
    - manage members of the system
  - Error dialogues
    - Login errors
      - user doesn not exists
      - incorrect password
    - Server request errors
      - invalid requiest
      - user not autherised
      - internal error
      - requested items dont exists
      - action could not be completed by server

## Functions

- Server
  - Check if autherised for all actions
    - Return error structures to unautherisd requests
  - Recieve login data and return status
  - Serve website
  - Serve list of groups of user
    - Super admin gets all groups
  - Serve list of channels of group and user
    - Super admin and group admin gets all channels
  - Serve list of messages for a channel
  - Serve message hash for update checking
  - Recieve message from user to channel
  - Process users added to group
  - Process users added to channel
  - Process users removed from group and channels
  - Process users removed from channels
  - Process users elevated to group admin
  - Process users elevated to super admin
  - Store groups, channels and messages
  - Store user data
  - Add\Remove groups
  - Add\Remove channels
  - Add\Remove users
  - Upload user avatar images
  - Upload image messages
  - Create sockets for chats and currently active users
  - Provide testing methods
- Browser
  - If user details saved
    - Check autherisation with server
      - If incorrect, delete local details and show login screen
    - Get user priviledge
    - Get groups list for user
    - Get channels for user for group
    - Get messages for channel
      - Periodicially check if messages available
  - If user details not saved
    - Show login screen
    - authenticate user details
      - If correct, save them
  - If Group admin or super admin, show group, channel and participants managment options

## Data structures

Key:

- { name1: type1 => name2: type2 } denotes a mapping of name1: type1 onto name2: type2
- ( fiel1: type1, ... ) denotes an object with fields file1: type1, ...
- [ name: type ]  denotes an array of elements of type type

### Server representation:

In the server representation users, groups and channels are all doubly linked
This is to improve access and writing efficiency

#### Users

---
User objects contain basic information and references to the groups and channels thay are part of.

```
users: (
  userID: int (indexed),
  active: boolean,
  superadmin: boolean,
  groupadmin: boolean,
  username: string,
  useremail: string,
  password: string,
  colo: int,
  groups: { groupID:int => [channelID: int ]}
)}
```

```
usernames: { username: string => userID }
```

#### Groups

---
Groups contain their name and references to its participants and channels.

```
groups: ( 
  groupID: int (indexed),
  groupName: string (indexed),
  participants: [ userName: string ],
  channels: [ channelName: string ],
)
```

#### Channels

---
Channels contain name and references to the group they belong to and its participants.

```
channels: (
  channelID: int (indexed),
  groupName: string (indexed),
  channelName: string (indexed),
  participants: [ (
    userName: string,
    color: int,
  ) ],
)
```

#### Messsages

---
Messages are indexed by they channel ID (map from ID to message history). Each message contains a reference to the sender, the content and datetime in seconds from epoch format. Messages are stored seperate from channels to improve write and access performance. 

```
messages: (
  messageID: int (indexed),
  groupName: string (indexed),
  channelName: string (indexed),
  userName: string,
  color: int,
  content: string,
  datetime: int,
)
```

### Database

The data is stored in a single document of the collection 'chattychat' in database the database 'chattychat', it holds the above listed server objects.

```
chattychat: (
  users: Object,
  usernames: Object,
  groups: Object,
  channels: Object,
  messages: Object,
  id: int,
)

```

Each field of this document is updated as needed individually whenever the server changes state. 

### Client representation

In the client representation objects contain the data directly rather than linking to other strscture.

#### User Token

USed to identify a signed in user

```
userID: int
```

#### User

---
A single user object with basic information about the signin user as well as references to the users groups and channels.

```
user: {
  userID: int (indexed),
  active: boolean,
  superadmin: boolean,
  groupadmin: boolean,
  username: string,
  useremail: string,
  colo: int,
  groups: { groupID:int => [ channelID: int ]}
)
```

#### User's Groups and Channels

List of groups the user is part of. Each group has lists of channels that the user is part of. This is used to build the group and channel list.

```
usersGroups: [ 
  group: (
    groupID: int,
    name: string,
    channels: [
      channel: (
        channelID: int,
        name: string,
      )
    ],
  )
]
```

#### Channel's participants

The participant list contained in this structure is dependent on teh channel currently selected. Each participant contains the username, color and isadminstatus. Note that isadmin is true if the user is either groupadmin or superadmin.

```
participants: { userName: string => color: int }
```

#### Messages

The message histpry contained in this structure depends on the channel currently selected. Each message contains the senders username, content, datetime in a readable format and user's color.

```
messages: [
  message: (
    userName: string,
    color: int,
    content: string,
    datetime: string,
  )
]
```

#### Managing Users

This strcture is used to manage user. It can be used to add users to and remove them from groups, channels and the service. availableUsers is a list of all users that qualify given the context (ie only users is the parent group can be added to a channel). selectedUserID is a list of ids that is included in the to be managed entiry (ie users part of a channel).

```
manageUsers: {
  availableUsers: [ userName: string ],
  selectedUserNames: [ userName: string ],
}
```

## REST API - Routes

### Client routes

Routes created by the client locally in the borwser

| Route  | Purpose |
|--------|---------|
| /      | default route, redirects to login or dash depending on signed in status  |
| /login | ligin screen                                                             |
| /dash  | signin dashboard with all functionality dependign on user                |

### Server post routes

#### Get requests (static hosting)

The static get hosting serves the webpage client files to the server as generated by angular.

| Route         | Action                                       |
|---------------|----------------------------------------------|
| /[sourcepath] | returns appropriate file contents to browser |

#### Post requests

Post routes are used to request and transfere data bestween client and server. Note user token is shorted to UT, UT's are used for most request to verify authority to perform actions.

| Route           | Action | In | Out |
|-----------------|--------|----|-----|
| /login          | verifies a login request  | username, password  | User token or error                                      |
| /user           | provides user information | user token          | user information, group and channel lists                |
| /channel        | open channel              | UT, channel ID      | refactored messages history, refactored participaUT list |
| /new-group      | create a group            | UT (creator), name                                        | succes status |
| /new-channel    | create a channel          | UT (creator), parent group, name                          | succes status |
| /new-user       | creates a new user        | UT (creator), name, email, permissions, color             | succes status |
| /delete-group   | delete a group and all its channels                       | UT, groups ID             | succes status |
| /delete-channel | delete a channel                                          | UT, channel ID            | succes status |
| /manage-group   | serve list of users relevant to a group to be governed    | UT, group ID     | list of relevant users |
| /manage-channel | serve list of users relevant to a channel to be governed  | UT, channel ID   | list of relevant users |
| /manage-users   | serve list of user in the system to be governed           | UT               | list of relevant users |
| /update-group   | add and remove user from a group and its channels | UT, users to add, users to remove | succes status |
| /update-channel | add and remove users from a channel               | UT, users to add, userd to remove | succes status |
| /update-users   | remove people from the system                             | UT, users to be removed   | succes status |
| /send-message   | registered a new message and saves it                 | UT, sender, content, datetime | succes status |

## Angular Architecture

### Custom Components

| Name                | Purpose |
|---------------------|---------|
| login               | Login page, presents the user with a simple layout that allows them to sign in. If sign in fails approriate errors are shown |
| dashboard           | Main page onced signed in. Allows users to message each other and admins to mannage groups, channels and users |
| new-dialog          | Dialog that asks for a name for a new group or channel to be entered. It then returns this name to its caller. |
| new-user-dialog     | Dialog that queries for new user details. Details include username, email, permissions and color. the details are returned to the caller. |
| delete-dialog       | Dialog that asks for comfirmation before deleting groups and channels. |
| manage-user-dialog  | Dialog that allows users to be added to and removed from groups, channels and the system |

### Hierarchy

```
      app
     /   \
    /     \
login    dashboard
        /   / \   \
       /   /   \   \
      /   /     \   \
     /   |       |   \
    /    |       |    \
   /     |       |     \
new-   delete-  new-   manage-
dialog dialog   user-  user-
                dialog dialog
```

## Compilation (Tested)

- Client compiles on
  - Angular CLI: 6.1.2
  - Node: 8.11.1
  - Angular: 6.1.1
- Server runs on
  - Node: 8.11.1