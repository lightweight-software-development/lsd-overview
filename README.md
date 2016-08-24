#Lightweight Software Development


##Introduction



Lightweight Software Development (LSD) is a way for programmers to deliver web applications that are
- Quicker and easier to develop and maintain
- Cheaper to run
- Better for users

It does this by
- Using simple tools and components instead of complex frameworks and practices
- Taking advantage of modern hardware and software
- Reducing the "implementation gap" between describing a system and coding it

LSD combines an approach to development with actual implementations of tools and components that support it

##The Problems
###The implementation gap
The code for a system is usually much larger than a plain language description of the business requirement.
One reason for this is that the code mixes many implementation details with the business logic - things like 
explicitly storing and retrieving data, and low-level view markup.  
  
###Building it twice
Most modern web applications end up being developed twice - one application to run on the server, and another for the browser.
Both are substantial programs with overlapping functionality, and they need even more code to communicate with each other.

###Running the back-end
Apart from the effort to develop and deploy server applications, they are expensive to run, 
especially if they need a database to go with them

###Framework-itis
Developers seem to love frameworks that take over the overall structure of the application,
leaving them to poke details into holes at various places.  They also use complex build tools that have to be persuaded
to produce the desired result with magic configurations.
All of these take time to learn and remove power from the developer.
 
###Working offline
Applications have moved over the last 20 years or so from being all on a PC, to all on the web.  
Now people want a mix - to work on the web when it is available, but to carry on working entirely on the client device when it is not.
Few applications support this well at the moment.


##The Opportunities
###Powerful user devices
Modern PCs have far more memory and CPU speed than most people need.  Even phones can have 1Gb of RAM and fast processors.  
The client device in a web application often has more memory and CPU available than the virtual machine running the back end application.
So we can move away from the idea of a powerful server feeding titbits to an underpowered browser, and regard
the user's device as the main place where the application runs, maybe with support from a back end.

###Ample memory and bandwidth
Computers have gigabytes of memory available, but in many applications all the data will fit into 100Mb or so.
With fast internet connections, 100Mb can be transferred in seconds.  So why worry about picking data out of a database
one piece at a time?  Just copy all the data 
(that the user is allowed to see) to the device where it is needed, and hold it all in memory.

###Browser storage
Modern browsers can store applications and large amounts of data locally (with local storage and service workers).
If all the data is copied to the user's device, and updates are managed in the right way, 
they can work offline, and synchronise with a central store
when the internet is available again.

###Universal programming language
With JavaScript 2015 standard on browsers and servers with Node.js, it is now possible to write software
in a powerful language that can be run on both servers and user devices.  There is no need to have separate front end
and back end - just one application running wherever it is needed.

###On-demand storage and server functions
With AWS Lambda, it is possible to run JavaScript code whenever it is needed without running a server continuously.
With AWS S3, data can be stored directly from a browser without having to run an application server or database. These two
facilities together allow much simpler web applications with fewer moving parts.


##The Techniques

LSD does not *require* all of these techniques: they could be used individually.  But they help each other,
so the best results will usually come from combining them.

###Functional models
In most applications the business model contains a mixture of data about things 
that exist or have happened in the outside world, and data that is calculated from that source data.
A functional model has an **unopinonated data model** with pure source data (just the facts), with functions
to calculate everything else.

In the simplistic bank account model often used in coding examples, the Account object would have just
things like the account number and holder's name, but *not* the balance.  The deposits and withdrawals
are also held as primary source data, and there is a function to calculate the balance from these when it is needed.

Inefficient?  It could be, but there are many ways to ensure the calculation is cached and done as few times as possible.
And if the calculation is running on the client device, with lots of spare processor power, is efficiency really a problem?

Some objects in a functional model will be all calculation in any case, with no source data of their own.  An example
would be a report based on all the Account objects in the system.

####Event sourcing
By just recording all the source facts and events, instead of using them to update cached calculations in the model,
you are building a system in the event sourcing style, with the various advantages that brings.
All the events are recorded as standard **actions** - create, update or delete objects in the data model.



###Metadata
In many applications there is much hand-coding that follows predictable patterns with slight variations.  
This is especially true in defining views, where the layout and elements used are tied to the underlying data,
with a few bits of additional knowledge about that data.
So why not encode that knowledge as metadata
and use it to generate views and business model validation automatically? 
 
The knowledge held in metadata for each data item includes data type, validation rules, help text, localised name for display,
and whether it is normally displayed.  If the data item is a list, or a reference to another object, the metadata also has
 the type of the objects contained in the list or reference.
 
Metadata is also held for items calculated by functions, and objects that are all calculation.


###Powerful components
Instead of handing over control to a framework, the programmer builds the application around a functional model
using a hierarchy of nested high and low-level components (think building blocks rather than painting by numbers).
The main components in most applications would be for data storage and views.

####Data storage
The data storage component works with **actions** - the source facts or events that update the data model.
It is connected to the data model and both listens for new actions from the user,
and also feeds in actions to the model from elsewhere.  
It saves new actions in local storage, and if online, to
a remote store.  On startup, it feeds in all locally stored actions to the model to bring it to the current state,
and if online, loads actions sent to the remote store by other users.  By looking for updates in the remote store at intervals,
it can keep the model up to date.

This technique helps both the end user, by allowing offline working and automatic synchronisation, and the developer,
by making data persistence just a matter of plugging in the storage component.

####View components
These components range from low-level components for individual input elements, up to high-level components
that create the view for the whole application.  The high-level components are configured by a combination
of specific settings from the developer and the business model metadata.  They can have other components
"plugged in" to them by configuration.

Many views can be generated given just the metadata.  For the rest,
it is important to have fine-grained overrides - changing only the details you need to - to combine the benefits
of metadata-driven code with full control.


###Serverless
The application is written to run primarily in the browser.  Some applications may not any server component, for example 
if each user has their own separate data and validation in the browser application is sufficient.  In other cases it may be necessary to have a server function
to validate changes from a user before they are copied to a shared central store, or to filter the data
being sent out to each user if fine-grained permissions are needed.  Server functions may also be needed for things
that can't be done directly from a browser, like sending emails, providing a web API or providing fixed HTML pages.

Server functions run in on-demand services like AWS Lambda, and they use the *same business model code* as the browser application.
For providing fixed HTML pages, they would also use the same view code as the browser application.

AWS is complex to set up, mainly because of the permissions that are applied to every object and action.  
Configuring AWS should be done by a repeatable setup script that is part of the application code.
 
####The end of "environments"
So long as the AWS setup script can take a parameter to give unique names to each component, you can
deploy as many instances of the application as you want, at any time, and discard them when they are no longer needed.
This makes development and testing very flexible - each developer can deploy a unique instance for each build, for example.

####Switchable data sets
Within a deployed instance, it should also be possible to start the application with a different data set.  
This allows a deployed instance to be tested without destroying production data, or for a user to run it 
with demo data to try something out.

###Testing

####Built-in tests
The tests are included with the deployed application and can be run at any time.  Smoke tests that do not change data can
run with the main data set, others can only be run if the application is running against a test data set.

####Simpler functional tests
Most tests will test the functional model by setting up a known set of data, and checking the output.
The tests are simpler to write because they only involve "given" and "then".

####Simpler browser tests
Some tests may check the view in a real browser.  As they are built into the application and served from the 
same origin, that browser can be an iframe in the test page, and the test can be written in JavaScript
acting directly on the application to set up data and inspect the state of the page.  jQuery does most of what
current browser test tools do in terms of selecting and inspecting page elements
 
####Tests double as help or demo
As the tests can be run in the browser at any time, some of them could be used as documentation and help demonstrations.
Tests on the functional model could be presented as input-output tables.  Tests using the browser window
could be slowed down and display comments at each step to provide an animated guide to the application.
 
####No more CI
As any version of the application can be deployed with a unique name (like a Git commit id) at any time, and can test itself, 
the traditional CI server isn't really relevant any more.  If each deployed instance saves the test results to its own S3 data area,
a simple app running in the browser can collect those results and display them.
When a particular instance is judged ready for release, 
it can be made live just by copying in the latest data to its main data set and switching the DNS CNAME to point to it.

###Agile++ (or --)

The increase in development velocity allows a few changes to the way an agile project works.

####Daily iterations
With practice using the LSD techniques, developers should be able to deliver meaningful increments in functionality
in a single day.  So every day can be an iteration starting with a short planning meeting instead of a standup,
and ending with a short demo and review of the help material generated from the tests.

####User pairing
With a practised developer it should be possible to take requirements directly from the user and build a prototype
in front of them.  Much more efficient than writing up detailed stories or 
trying to remember who said what in the planning meeting days before.  And very often users find it difficult
to imagine what they want - having a working prototype in front of them makes it much easier.

####Experiment to learn
AKA less conversation, more action.

Rather than debating how a feature should work, or whether it is worthwhile, or trying to ~~guess~~ estimate how long it will take, 
just build it and find out. 






##The Tools

The following tools and components to support LSD are currently in development.

###Immutable business models
The state for the whole application is a single immutable object, of a class that represents the whole business model.
It contains immutable collections
of other immutable objects, which are instances of classes representing the business entities.  
The Immutable.js library is very useful for defining these classes.  The application class and the business object classes
contain primary source data, and have functions for the calculated parts of the data model.

Each update action applied to any class in the model follows the immutable object style: return a new instance 
with the changes.  Actions from the view all go to the application state, never directly to entities within the model.
The methods called return a whole new instance of the application state with the changes applied.  Using Immutable.js
ensures all of the state except the parts changed is copied from one state instance to the next.  A state controller
keeps track of the current instance and the view subscribes to changes.

Each function on the business model objects is memoized automatically, so as long as the 
object is not changed, the function does not need to be re-run.

###Automatic data storage
The browser storage component listens for actions applied to the business model, 
and stores the method name and arguments with a unique id as an action.
The actions go first to browser local storage to ensure they are persisted.  
When the application is (next) online they are copied to S3 in a data area for that user.

If data is shared between users, another storage component running in Lambda reads
the updates from the user areas, applies them to an instance of the business model to validate them, and copies them
to the shared data area. 

The browser component continually polls the shared area for new updates, stores them locally and applies them to 
the local instance of the application.

To reduce the number and size of updates transferred and stored, 
another Lambda component periodically consolidates multiple actions
into one update, and at longer intervals creates a snapshot of the application state which replaces all the updates
up to that point.  The updates are still archived as an audit trail and to allow regenerating the snapshot if 
the application functionality changes.




###Metadata driven views
This is a set of React components designed to work with LSD metadata to automatically create complex views.  They include
components for:
- Displaying an edit form for a business object
- Displaying lists or tables of business object instances
- Editable lists of business object instances
- Automatic form items that shows the best element for the data type
- Combined business object list and edit view for the one selected
- Report objects 
- Routing to match the browser location with the view shown

The highest level view is the one that represents the whole application.  It subscribes to application state changes,
and re-renders the whole application with the new data.  Because React only re-renders parts that have changed,
and with immutable objects you only need to compare instances to see whether they have changed, this is very efficient.

###Supporting acts

####Observables
A very simple observer/listener mechanism for values and events.  It is used by the other LSD components,
and allows a "data wiring" style of programming, where components are built and tested without knowing what
their collaborators are (no mocks required), and then "assembled" by a container component.

####Metadata
Simple classes to support defining business model metadata.

####Memoization
A simple tool for memoizing functions efficiently where 'this' and the arguments are all immutable objects, so it is easy
to know whether they have changed or not.

###More to come
Among further tools planned:
- Browser test-to-help/demo tools
- Application builder tool: gather the business model metadata in a worksheet style application, then generate the code from it




