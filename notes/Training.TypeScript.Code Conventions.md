---
id: isa78jdkt5e6ex5c2ewl48o
title: Code Conventions
desc: ''
updated: 1745512336648
created: 1745509513580
---
[Language guidelines in the architecture docs](https://architecture-docs.apak.io/design/tech_stack/backend/languages/typescript/language_guidelines/)

### Code Style
- In our TS IL code use a mix of pure OO and file-scoped functions, and functions as first class citizens
- Allows more flexibility for programming solutions, effectively functions can  be passed as arguments, returned as values and assigned to variables.
- So generally less around objects, and therefore less side effects

### Starting point for a service
A service can consist of multiple lambda functions, but generally in IL they only have one.
- Term *lambda, handler, function, lambda function* used interchangeably to mean the function that's called when the service is invoked by AWS' service (Lambda) which lets code run functions in the cloud
- root `index.ts` file is lambda's entry point. Must export a `constant` called `handler`, of type `Handler`. First parameter to `handler` is `event` and is the one we use in our services. Generally being `SQSEvent` as it comes from the service's queue.
- once a `Handler` has been defined, before exporting it, we pass it along with the `service's id (name)` to the function `lambdaWithContext` from `serverless-logger` module. Wraps the Handler up with logging context to make its logs traceable. This is necessary because we're executing code in a serverless way, so we can't rely on accessing logs from a pre-defined server
- **error handling**
    - a single event being passed to handler can contain multiple records
    - don't want a service to abort because of a single record failing, as other records from the even could get missed
        - to prevent this we wrap the logic that processes a single record in a `try-catch` and push every `Error` onto an array and then move on to process the next
    - once all records have been processed, there is a `CompositeError` we create from the error array and throw that [see here](https://bitbucket.apak.delivery/projects/SLS/repos/serverless-logger/browse/src/lambda.ts#54)

As an entry point, the `index.tx` file containing the handler should be at a high level of abstraction. Preferably only containing set-up of variables for the hanlder function, the function itself and the `handler` export statement. The handler function should pretty much solely be delegating off to logic in other modules, and should **not** contain any low level logic itself, it should read as a high level outline of the lambda, each step being a call to a well-named function.

### Testing
- **integration testing**
    - root index file should have an integration/end-to-end test suite that runs Lambda from start to finish. They're verifying that modules work together correctly but shouldn't integrate external systems, these should be mocked
    - naming convention is `index.itest.ts`
    - should assert on final output of handler
        -  e.g. A typical output of an IL handler is that a decrypted, transformed file is uploaded to S3. The integration test for such a handler should mock the S3 client alone, with it's mock implementation outputting the file content to a variable in memory (or a local file if necessary). Then the assertion should compare the content of the created file (& any relevant metadata, e.g. file name) with the expected value. This implicitly confirms that the decryption, transformation & upload have occurred successfully.
- **unit testing** 
    - all logic in other files should be tested by unit test suites
    - if the root `index.ts` has its own logic, aside from top level lambda flow, it should be unit tested
> all of this should be done with the [Jest](https://jestjs.io/docs/getting-started) framework
- **running tests**
    - in `package.json` there's 2 scripts for running tests
    - one runs all tests once and the other runs them and watches the relevant files and runs them again upon code change
    - to run, open a terminal and execute `yarn <scriptName>`
    - you can optionally append a parameter to the command, specifying a full or partial file name or path to specify certain tests to run

### General IL info
[General IL info sf-wiki](https://sf-wiki.atlassian.net/wiki/spaces/WIKI/pages/41275160/General+IL+Info)

### Development Procedure
- After checking out code
    - run `yarn install` locally
- After making changes, before push and pr
    - run build and tests with `yarn build` and `yarn run-tests-once` 
- After merging
    - check build success in CodeBuild
- Assigned tickets should:
    - Be *In Progress* when being worked on
    - Be moved back into *Backlog* if parked for some reason
    - Be moved into *Verification* once all code changes are deployed
    - *Closed* once verified 
- Ticket should have a `Remaining Estimate` that reflects dev's true impression of how much more time
    - log against it daily
    - frequently review estimate
        - if no longer accurate speak with team lead to suggest adjustment
        - if not available then make adjustment and tag them in a comment
        - add a comment with adjustment and reason