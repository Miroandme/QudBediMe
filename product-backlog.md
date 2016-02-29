#Product Backlog

The Product Backlog is a description of the product being built by the Scrum Team, in the form of a prioritized list of  User Stories. The Product Owner is responsible for ensuring the Product Backlog contains stories with sufficient detail to be implemented, tested and deployed by the Scrum Team. The Product Owner is also responsible for ensuring the stories in the backlog are ordered by implementation priority.

###Futher Reading
* Mike Cohn's [thoughts on the Product Backlog](http://www.mountaingoatsoftware.com/agile/scrum/product-backlog/).

##User Stories
User Stories are features to be implemented that are described as a refutable statement of need. We use the very standard form of:

"As a {role or persona or actor} I want to {perform some action} so that I can {realize some benefit or achieve some goal}"

Identifying the role or persona or actor that initiates the story is a powerful technique for identifying who is going to be performing the action and typically who is the originator and/or recipient of the value of this story. This is also helpful mechanism to understand authorization requirements. In general the actor should not be another system as this skews stories towards implementation and obscures who is really receiving the benfeit. The action performed should be described with as little technical detail as possible so as not to dictate implementation. The final goal clause is optional but encouraged as this can help qualify and quantify the value of the story in relation to others.

###Adding Detail with Behavior Driven Development
At Sungevity, we add detail to our User Stories by defining scenarios using [Behavior Driven Development](http://en.wikipedia.org/wiki/Behavior-driven_development) (BDD). The intention is to describe pre-conditions, trigger events and post-conditions to aid implementation and testing by identifying normal and exceptional secnarios. We use the "Given (pre-conditions), When (triggers), Then (post-conditions)" format. For example:

"Given the authenticated user is a Customer
When the Customer clicks or presses the logout icon
Then the customer is logged out and needs to login before using features requring authentication"

###Futher Reading
* Mike Cohn's [thoughts on User Stories](http://www.mountaingoatsoftware.com/agile/user-stories/).

##Epics
Epics are User Stories that are too substantial or insufficiently defined for the Scrum Team to complete in any given sprint. Experience has taught us never to size Epics, and more importantly never to attempt to work on Epics until they can be decomposed into more specific User Stories.

##Sub-Tasks
Sub-tasks are the technical to-do list for implementing User Stories. It is the responsibility of the Scrum Team to decompose stories into a set of practical sub-tasks so that work can be assigned and tracked within the team. Sub-tasks are never sized. The value of the sub-task is realized through it's parent story when the story is completed.

##Story Points
Story Points are an abstract measure of the effort required to complete a story, as well as indicator of complexity, uncertainty and risk. At Sungevity we use Fibonacci-scale numbers of 0, 1, 2, 3, 5, 8 & 13. Stories with zero-points are generally unsized; stories with 13 points are considered to be very risky to complete within a single sprint and suggest decomposition into smaller, more manageable stories.

##Bugs & Emerging Requirements
Bugs represent work that was not completed either due to missing or contradictory information in the User Story or some other technical/implementation defect. Ideally bugs are simply dealt with during a sprint so that completed User Stories are tested and bug-free. However, bugs will be found at later times. When this happens, bugs should be entered directly into the backlog alongside User Stories. They will be tagged as "bug" in Assembla. Bugs are not sized. Fixing bugs that are identified on User Stories being on worked on in the Sprint is a priority that Sprint as the story is not complete until known bugs are resolved.   

Sometimes, when implementing a User Story, it becomes clear that additional or changed requirements emerge; these will be recorded as additional User Stories in the Product Backlog and sized appropriately. 
