#Breaking the Build
Pushing code into a branch watched by Continuous Integration, will sometimes break the build. A build could fail because of poor configuration, compilation errors, broken tests, missing dependencies, etc. A broken build means that another developer cannot pull from the repository without causing their local build to fail. While this is a common enough occurrence when developing code, it does need immediate attention. 

The author of the change that broke the build is responsible for fixing the build as a top-priority. If the failure was the result of a merge froma pull-request, the requester and the merger are responsible for working together to fix the build.

Leaving builds broken overnight is anti-social.

###Further Reading
* [Breaking the Build: Why is it a Bad Thing?](http://stackoverflow.com/questions/3290702/breaking-the-build-why-is-it-a-bad-thing)
