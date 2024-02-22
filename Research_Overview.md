# Independent Research Overview

## General

1.  Should we use three environments (i.e., DEV, Production, and Staging) per application?

    - The staging environment would be a copy of Production’s resources to properly test features before they are put into Production.

```
Absolutely. Let's first suppose we don't use three environments. Then what? Dev and Prod? Dev then becomes our testing branch which puts holds on completed feature/bug PRs.
```

2.  Should we keep the frontend and backend in the same repository or separate repositories?

```
I loved working with the front and back in the same repo... Not only is it just nice to have for stuff like searching variable names and endpoints between the two, but this allows for the bicep files to be included in the same repo, AND runs the deployments at the same time for github actions.
```

3.  Should we ever delete any database entry?

- Set the entry as inactive or update its status to cancelled/inactive.
- We would need to ensure that all of our GET calls that pull lists of items (i.e., load board) don’t include these inactive database entries, unless specifically requested by the user via a search (maybe admin only).

- Why lose the data?

- If we don’t delete, should we ever use a RESTful delete call?

```
I don't have a strong enough opinion either way but I think it's best to err on the side of caution and opt for never deleting anything at all. We can always see how our users interact with the app and choose to delete entries later for the sole purpose of db optimization (fewer rows to search through)... This feels like a later problem.
```

4. What authentication provider should we use? Azure B2C, Auth0, or other?

```
Azure B2C for its cheap cost, the potential for easier interaction between our other resources in Azure, and Graph API. We've confirmed B2C has the tools we need.
```

5. What Portal should we build first? Customer or Management?

```
First thoughts were build Customer portal first. Should go by faster.
```

## Bicep / Pipeline

6. Should we use Azure’s sample Bicep Template as a starting point for our Bicep Template?

```
MS wrote it... It's going to follow all the best practices for Bicep. We can replace the front and backend code with whatever we'd like. It manages setting up everything from appServicePlan to database secrets in the keyvault AND github actions workflow secrets. It seems too good to pass up on.
```

7. What Azure resources will we need for each portal (i.e., Customer, Management, and Carrier Portals)?

```
For each environment we should have a separate resource group. Each environment should have atleast the following resources:

- App Service Plan
- 2 App Services (frontend/backend)
- Key vault
- SQL Server
- SQL Database

Optionally we should also use:

- Application Insights
- Log Analytics workspace
- Shared dashboard

```

8. How should we ensure the hosted production backend and frontend are in sync? Version checking?

```
Etag, refreshing client when mismatched.
```

9. Should we use blue green deployment?

```
https://learn.microsoft.com/en-us/azure/container-apps/blue-green-deployment?pivots=azure-cli

Would be another copy of production resouces, helps us achieve instantaneous deployments (flipping the router) or we can gradually send more and more users to the new deployment. Built in rollback strategy.

Downsides are now we have 3 copies of production resources per portal: Customer, Shipper, and Management. BUT we can take down the inactive copy of prod after a successfull deployment that we know we won't need to rollback on.
```

10. How do we effectively use GitHub Actions to manage our pipeline?

```
Azure Sample App starting point!
```

## Backend

11. Should we use Onion Archiecture?

- Should we put our layers in different solutions within the same project?

- How should we approach the Repository layer? What are benefits/drawbacks of multiple Repositories?

```
The repository layer is where we are actually interacting with the database.
We pass querries to the repo layer abstracting away our LINQ. Why abstract away our LINQ unless we plan on using the same expressions for several handlers. But lets go with

My biggest thing with this is I want to be able to have full control over my LINQ querries on a per call basis in order to make the querry which best fits for the call. If we start abstracting away our LINQ from our handlers then we'll be tempted to use abstracted functions which get the job done, but might not be most efficient for the use case.

Cartesian explosion & Split Queries...
```

12. What backend file structure should we use?

```
Completely depends on if onion or not...

Controllers, Handlers, Models, View Models, DTOs.
```

13. What backend testing framework should we use? Live database, repository, or other?

```
some other way to avoid having to seed a test db for ever test case.
```

14. What backend testing tool should we use? MSTest, xUnit, or other?

```
some other way to avoid having to seed a test db for ever test case.
```

15. How should we mock data for our backend testing data?

```
Live database! Or some other way to avoid having to seed a test db for ever test case.
```

16. Should we queue backend calls?

```
idk

Performing parrallel querries to a SQL DB:

var task1 = dbContext.Table1.ToListAsync();
var task2 = dbContext.Table2.ToListAsync();
await Task.WhenAll(task1, task2);
var result1 = task1.Result;
var result2 = task2.Result;

```

17. How should we handle auditing user input and changes to the database?

- External library? Audit.NET?

```
idk
```

## Frontend

18. What React framework should we use? Vite, Next.js, or other?

```
Prob Vite prob. I don't have a strong preference here. Next < Vite ≤ Other.
```

19. Should we mockup every page in Figma prior to finishing a page?

```
We should absolutely have a scaffold mockup of both the desktop and mobile view. This first draft doen't need to include colors or styles, but should show the position of components and how they should change between mobile and desktop view. This way we can set up the containers and components correctly the first time.
```

20. How should we solve client-side data concurrency? React Query, PubSub, or other?

```
I found working with PubSub to be pretty straightforward and is an amazing choice if we want to prioritize guaranteeing data concurrency, HOWEVER, one of the biggest problems that Darwin is going to run into is managing the different subscription groups... Which subscribers should receive updates after publishing? This adds another layer of complexity that we need to consider. Assuming we can get around this problem then pub/sub is a super low cost solution with huge benefits.

But for now, I think it makes the most sense to let Darwin find solutions to these problems first... That being said React Querry provides a middle ground of ensuring data concurrency on a per client basis through invalidating query keys. Ian walked me through his demo and I'm a huge fan of the simplicity.
```

21. What frontend file structure should we use?

```
React docs say file structure won't help / hurt preformance so it's all personal preferense.
It's popular to group by features or routes...
Key point is to avoid too much nesting.

Maybe something like this:
```

```
api/
    apiUtil1.js
    apiUtil1.js
    apiUtil1.js
    apiUtil1.js
types/
    type.js
    type.js
    type.js
pages/
    load/
        details/
            page.js
            page.js
        docs/
            page.js
            page.js
        page.js
        page.js
    customer/
        page.js
        page.js
        page.js
    shipper/
        page.js
        page.js
        page.js
helper/
    formatters/
        page.js
        page.js
        page.js

components/
    containers/
        comp.js
        comp.js
        comp.js
    util/
        comp.js
        comp.js
        comp.js
    inputs/
        comp.js
        comp.js
        comp.js
```

- Should we separate the API calls from the view components?

```
This isn't a terible idea. Might be nice to structure all of the frontend calls in a simlar way to the backend.
```

22. How do we ensure we don’t make <div> soup?

```
React fragments </>... idk
```

23. Should we adhere to ARIA guidelines?

```
Tailwind handles most ARIA guidelines besides a few that require javascript.
```

24. What frontend style pattern / CSS methodology should we follow? CSS libraries, modular CSS, Tailwind, or other?

- Should we use global style?

```
Global styles still makes a lot of sense as long as the styles we put in are truely global... Things like changing the behavior of ALL buttons in the app.

I think a combination of modular CSS and Tailwind works great together. Prioritize using tailwind when you can and then use modular css for outlier cases.
```

25. How do we keep the style of the application consistent?

- We need to put together a style guide for buttons, inputs, tables, cards, etc.

```
Create components for each of these elements. Or use CSS ui/ux library that has these for us.
```

26. What frontend unit testing framework should we use?

- What should we test via unit tests?

- What shouldn’t we test via unit tests?

```
We should unit test formatters like phone number formatter, or currency formatter.
Nothing else?
```

27. What frontend end-to-end testing framework should we use? Playwright, Cypress, or other?

- What should we test via end-to-end tests?

- What shouldn’t we test via end-to-end tests?

```
Playwright seems super convinient and should be used for high traffic flows like creating a load, as well as checking each page for white screens.
```

28. What level of typescript should we implement?

- Types vs. Interfaces.

- Where should we put our types/interfaces?

```
No clue whatsoever on this one...
```

29. Should we use a library for our tables or create our own?

- Tables, like the load board, are one of the most important parts of our application, even though it seems simple.

```
I havn't worked enough with table libraries to understand the tradeoffs.
```

30. Do we need to implement global state management?

- To what extent? Just to hold a user object?

```
MSAL hook will have the user object already... But does not include stuff about the user from the database.
```

## General Part II

31. What kind of documentation should we maintain?

- Where should we put the documentation? Readme, SharePoint, Notion, or a combination?

- What shouldn’t we document?

```
I think we should document every single backend endpoint. A short description of what it does, what paramateres if any are optional, and an example of a call. Think to something like Graph API which is a pain but would be so much more of a pain if we didn't have their docs.

It would also be nice to atleast have a place for documenting complex frontend components similar to UI library documentation.

Typescript documentation on types/interfaces?
```

32. What is our tolerance for warnings on the frontend and backend?

```
...ZERO TOLERANCE!!!! But there will simply be warning that we might not be able to avoid... For a warning to be acceptable it must be approved by team?
```

33. Should we create user guides?

- When in the process?

```
FAQ would be simple and fast to put together so why not. Is it worth running the user through a tutorial once signed up? This would be a pop up in the frontend which the user clicks through or skips where as they click through the app automatically walks them through a certain task.
```

34. What considerations should we have for performance? Backend, Database, Frontend, Lighthouse metrics?

```
No clue
```

35. What is our definition of done?

```
For frontend a full demo in fullscreen and mobile view.

For backend let somebody try and break the call in postman.
```