
### Definition Of Done
Our scrum development process is built around the concepts of stories, commitments, and delivery on these commitments within timeboxes.  A clear definition of what __*done*__ means is crucial for establishing clear delivery expectations between Developers, QA Engineers, Product Owners, and the business.

#### The Developer's Definition of Done
_For a story to be considered ready QA, the developer needs to have completed all of the following:_
##### All Tickets
* Developer has verified the desired functionality works as desired in the developer integration environment.
* Any manual config steps, dataloads, database changes, etc. that cannot be committed and deployed have been manually done in the dev and appropriate testing environments and noted on the [configuration wiki](https://sungevity.assembla.com/spaces/sfun-software-engr-wiki/wiki/Config_Requirements_Tracking)
  * _This step is absolutely crucial in preventing differences between the integration and QA environments and in making sure that all production deployment requirements are tracked in one place._
* Testing tips (including areas of focus, possible regressions, and areas already covered by unit and functional tests) have been added to the ticket description to help and guide the QA Engineer.
* Ticket description is clear, fully up to date, accurate, and only covers what was delivered.
* Any ticket comments that are no longer valid, misleading, or contradictory are deleted or clearly called out as not applicable in later comments. 

##### Tickets Involving Commits to Git
* Changes are committed and [pushed to the correct branch on GitHub](../git/readme.md), with the commit referencing the Assembla Space and ticket number.
  * _A commit message for ticket 893 in the SalesForce space would reading something like: **SalesForce#893 - Added a new doohicky to the thingamabob**_
* The change has been successfully deployed to the appropriate testing environment. (Check with [Jenkins](http://jenkins.uat.sungevity.com) to see which branches are deploying to which environments, and the status of the builds)

##### Tickets Involving Changes to Code
* All error conditions have been handled with clear messaging and/or logging, avoiding uncaught exceptions.
* Code Cleaned up:
  * All extraneous debugging statements removed
  * Commented out code removed. (Old code will always be available via checking out old git commits, after all)
  * Removal of trailing spaces, extra returns, whitespace in general
* Code clearly and concisely commented (docblox and/or inline, depedent on the code style being used for the given repository)
* All cross-repository dependencies have been kept to a minimum. (Where possible and reasonable, try to avoid the need to deploy two repositories at the same time)
* When dependencies are unavoidable, deployment of other dependent commits has been coordinated with other developers and clearly noted in ticket and commit message
* Unit tests covering both success and error conditions have been written and are passing.
* Developer has verified that commit has resulted in an equal or greater code coverage percentage for the repository as a whole.
* All Functional/API tests passing

