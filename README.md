An Unofficial Google+ Circle and Followers API for Google Chrome
================================================

It has been 4 months since we have seen any Circle/Posts/Followers
Write/Read API for Google+. Since Google+ is by nature asynchronous
we could tap into their XHR calls and imitate the requests.

I provide you a very basic asynchronous Google+ API, in the current
release, you can do the following:

- Create, Modify, Sort, Query, Remove Circles
- Query, Modify your Profile Information
- Add, Remove, Move People from and to Circles

This is a fully read and write API.

Native Examples:

    // Create an instance of the API.
    var plus = new GooglePlusAPI();

    // Refresh your circle information.
    plus.refreshCircles(function() {

       // Let us see who added me to their circle.
       plus.getPeopleWhoAddedMe(function(people) {
         console.log(people);
       });

       // Let us see who is in my circles.
       plus.getPeopleInMyCircles(function(people) {
         console.log(people);
       });
       
       // Let us see who is in our circles but didn't add us to theirs.
       plus.getDatabase().getPersonEntity().eagerFind({in_my_circle: 'Y', added_me: 'N'}, function(people) {
         console.log(people);
       });
    });
    
As you see, it is pretty easy to query everything. The possibilities are inifinite
since the data is backed up in a WebSQL DataStore, so querying, reporting, etc, would
be super quick.

If you want to place that in an extension, I have created a bridge, so you can use
this in a content script context and extension context safely. To do so, you send a
message as follows:

    // Initialize the API so we get the authorization token.
    chrome.extension.sendRequest({method: 'PlusAPI', data: {service: 'Plus', method: 'init'}}, function(initResponse) {
      chrome.extension.sendRequest({method: 'PlusAPI', data: {service: 'Plus', method: 'refreshCircles'}}, function() {
        // etc ... The method is the same method we defined previously in the raw example.
      });
    });


A full blown example will be released by the end of this week showing how powerful this could be.
As you already know, I am creating a simple circle management utility so you can manage your circles.

API Documentation
----

AbstractEntity Members:

- `String getName()` - The table name that this entity holds.
- `void tableDefinition()` - Abstract method that you override to describe the table.
- `void initialize()` - Private method that creates the DDL to execute from tableDefinition
- `void drop(Function:doneCallback)` - Drops the table from cache including the definition.
- `void clear(Function:doneCallback)` - Removes all rows from the table, keeps the definition.
- `void create(Object[]:obj, Function<Object{status, data}>:callback)` - Inserts object(s) into the table.
- `void destroy(String[]:id, Function<Object{status, data}>:callback)` - Deletes object(s) into the table.
- `void update(Object[]:obj, Function<Object{status, data}>:callback)` - Updates object(s) into the table.
- `void find(Object:obj, Function<Object{status, data}>:callback)` - Find a specific object(s) in the table.
- `void findAll(Function<Object{status, data}>:callback)` - Queries for everthing, all the data.
- `void count(Object:obj, Function<Object{status, data}>:callback)` - Counts the number of rows in the table.
- `void save(Object[]:obj, Function<Object{status, data}>:callback)` - Updates otherwise it creates.

PlusDB Entities:

- `void open()` - Opens the database
- `void clearAll(Function:doneCallback)` - Drops all tables from the database.
- `AbstractEntity getPersonEntity()` - Returns the PersonEntity
- `AbstractEntity getCircleEntity()` - Returns the CircleEntity
- `AbstractEntity getPersonCircleEntity()` - Returns the PersonCircleEntity

Native querying:

- `PlusDB getDatabase()` - Returns the native Database to do advanced queries

Initialization, fill up the database: 

- `void init(Function:doneCallback)` - Initializes session and data, you can call it at app start.
- `void refreshCircles(Function:doneCallback)` - Queries G+ Service for all circles and people information.
- `void refreshFollowers(Function:doneCallback)` - Queries G+ Service for everyone who is following me.
- `void refreshFindPeople(Function:doneCallback)` - Queries G+ Services for discovering similar people like me.
- `void refreshInfo(Function:doneCallback)` - Refresh my information. Rarely used.

Persistence:

- `void addPeople(Function:doneCallback, String:circleName, Array<String>:usersToAdd)` - Adding people to a circle.
- `void removePeople(Function:doneCallback, String:circleName, Array<String>:usersToRemove)` - Removing people from a circle
- `void createCircle(Function:doneCallback, String:circleName, String:optionalDescription)` - Creating a circle.
- `void removeCircle(Function:doneCallback, String:circleID)` - Removing a circle.
- `void modifyCircle(Function:doneCallback, String:circleID, String:optionalName, String:optionalDescription)` - Modifying circle meta.
- `void sortCircle(Function:doneCallback, String:circleID, Number:index)` - Sort a circle to the given index, G+ will deal with the order
- `void saveProfile(Function:doneCallback, String:introduction)` - Save a new introduction.

Read:

- `void getProfile(Function({introduction}):callback, String:googleProfileID)`
- `void getInfo(Function({id, name, email, acl}:callback)`
- `void getCircles(Function(CircleEntity[]):callback)`
- `void getCircle(Function(String:circleID, CircleEntity):callback)`
- `void getPeople(Function(PersonEntity[]):callback)`
- `void getPerson(Function(String:googleProfileID, PersonEntity):callback)`
- `void getPeopleInMyCircles(Function(PersonEntity[]):callback)`
- `void getPersonInMyCircles(String:googleProfileID, Function(PersonEntity):callback)`
- `void getPeopleWhoAddedMe(Function(PersonEntity[]):callback)`
- `void getPersonWhoAddedMe(String:googleProfileID, Function(PersonEntity):callback)`


Private Members (only for internal API):
- `Object _parseJSON(String:input)` - Parses the Google Irregular JSON
- `void _requestService(Function:callback, String:url, String:postData` - Sends an XHR request to Google Service
- `String _getSession()`- Unique user session that authenticates to persist to your account.

Watch this space!

