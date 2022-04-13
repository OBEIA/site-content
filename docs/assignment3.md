# Apollo & the Vehicle State Provider: A Suggestion for Enhancement

---

pdf

slides

video link

---

## Abstract

Apollo is an open-source autonomous driving platform supporting the development of systems for self-driving vehicles. Having been built and continually refined over many years, Apollo’s architecture has understandably developed idiosyncrasies. As an improvement to the software system’s architecture, we propose introducing a new module to be solely responsible for vehicle state.

As well as proposing this new module and its functionality, we outline two potential implementations for realizing it following paradigms already present in the Apollo system. We then assess these potential implementations according to SAAM, as outlined by Kazman, Bass, Abowd, and Webb, to determine which is the most desirable approach. Our final determination is to continue with the primarily publish-subscribe architecture present in Apollo for the implementation of the new module. We finally review the risks that the selected implementation introduces and propose strategies, including a testing strategy, to mitigate these risks.

## Introduction

Apollo is a platform dedicated to the development of autonomous vehicles, which continues to be in development itself. The complexity of autonomous driving combined with vehicle hardware brings intricate detail that has been, and must continue to be, considered as the system develops. To implement the system, Apollo’s developers have utilized a largely publish-subscribe-based approach, creating components that are mainly decoupled, for functionality and enhancement. These components include a Planning module to plan a trajectory based on various factors such as perceived obstacles, traffic light status, and the route created by the system’s Routing module; a Control module that transforms the Planning module’s trajectory into commands for the vehicle to execute; and a Map module that holds various maps to be utilized in the system, including a “relative map” that is relative to the vehicle.

Apollo is a system with a variety of valued non-functional, quality-based attributes; these include performance, modifiability, and reliability. As a platform for autonomous driving, performance and reliability of utmost importance: if the system does not perform well, such as within a certain timeframe, or is not reliable enough in its results, it can be unsafe for a vehicle to use on the road, meaning that it must be refined. The system must also be modifiable to suit other developer’s needs, such as to create new scenarios to plan for, or to utilize in implementations for specific vehicles. This suggests that Apollo’s architecture and design needs to be easy for other developers in this field to understand and modify.

Apollo includes a variety of utilities and objects within its system’s Common module, with one being the aptly named vehicle state provider. It provides information relating to the vehicle’s state to modules that utilize this data in calculations, such as for planning a trajectory or generating control commands. However, the vehicle state is updated by other modules as they receive or process appropriate input. The vehicle state provider’s strange placement as a Common utility should be reviewed to determine if an enhancement can be made to improve the system, such as to improve modularity and testability. It is the subject of our proposed enhancement, described in the following in more detail.

## Proposed Enhancement

-

### Motivation

-

### Current State

FIGURE 1, 2, 3

### Effects on Architectures

-

### Interactions with Other Features

The vehicle state provider is an essential utility for many modules, each holding an important role, as noted previously. For example, the Map module’s relative map is created using the vehicle state data, and the Planning module stages use the data in their planning processes. However, each module updates the vehicle state as well to suit their needs; for example, both the Map’s relative map and the Control component update the vehicle state upon receiving their appropriate inputs to ensure that further processing is done on fresh data and not a stale vehicle state that is no longer accurate.

Therefore, features that interact with the Vehicle State Provider, and thus would be affected directly by its refactoring, include: i) relative map creation, as the vehicle state is utilized and updated; ii) stage planning for various scenarios; iii) navigation (Navi) planning; and iv) control command creation, as the vehicle state is updated with its input. In summary, features such as map creation, trajectory planning, and control command creation—all being relative to the vehicle state—interact with the vehicle state provider.

## Implementation 1: Direct Code Dependencies

-

### Impact

-

### Architectural Styles

The main architectural style utilized in this implementation would be an object-oriented style. This is in a way obvious, as a singleton object would have to be defined, meaning the implementation is oriented around an object. In this style, other objects—in this case, the modules making use of the vehicle state provider—would need to know the identities of this new object in question. Further, this style is also used to identify and protect specific data; the vehicle state provider is designed for this purpose, protecting and identifying a variety of information on the vehicle’s state. In fact, it would be further protected and isolated from how it currently is, as stricter patterns for accessing it are enforced, rather than letting each consuming module choose its own approach.

## Implementation 2: Publish-Subscribe

-

### Impact

As discussed previously, the impacted modules other than the newly created vehicle state provider would be Common, Planning, Control, Task Manager, and Map. The Task Manager’s dependency on the provider in Common would be reviewed and likely removed, and Common’s original vehicle state provider would be extracted to the dedicated module. This would remove Common’s only inter-module dependency, on Localization. Other modules would see little change other than to their dependencies and potentially to the manner the state provider is accessed or acquired.

However, while the architecture of the system would be affected, modules themselves would be affected minimally. The means of acquiring and accessing the vehicle state provider may change, but it would have little effect on the internal logic and operation of the modules.

### Architectural Styles

The main architectural style utilized in this implementation would, of course, be the publish-subscribe style. By becoming a component in Cyber RT, the vehicle state provider is then able to publish and subscribe to topics, like many other modules in the system; Apollo is largely based on this style. The current vehicle state provider of the system is, as noted in how the architecture and features would be affected, often making use of these publish-subscribe messages such as in the Control module; thus, the vehicle state could instead make use of published topics as its own component. Various modules also request data from the provider; modules could instead receive the vehicle state as soon as it is published.

## SAAM Analysis

The SAAM Analysis, created by Kazman, Bass, Abowd, and Webb, is a method for studying software with respect to their quality attributes, features, and architecture. By considering how non-functional requirements are affected by a new or existing feature, software architects and developers can determine how well the system supports the feature in its current state. [\[Kazman, Bass, Abowd, and Webb\]](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.65.8786&rep=rep1&type=pdf) By repeating this process for two different implementations of a feature, they can further determine which would better suit the software. To determine non-functional requirements, Apollo’s major stakeholders and requirements are identified.

### Stakeholders

The stakeholders for the proposed enhancement are all parties whose experience may be affected by implementation of the enhancement. Perhaps the most direct stakeholders are the developers assigned to improve and maintain Apollo software as the location of source code blocks and dependencies (either code or publish-subscribe) between modules would be changed. Another stakeholder with a similar perspective to developers is the technician. Technicians are those who would be fixing Apollo-equipped vehicles, and their domain may not be limited to just the software.

The next stakeholder for the enhancement is the end user. While the developer may care about the structure of the software, the user only cares about the impact of changes on the experience of being a passenger in an Apollo vehicle, including factors like safety and convenience.

Car manufacturers are another stakeholder that should be considered. Interestingly, manufacturers have overlapping interests with users and developers. In order to be successful in selling Apollo-equipped vehicles, manufacturers must be interested with providing the best user experience possible. On the development side, manufacturers must be interested in the costs associated with maintaining and implementing Apollo in vehicles.

### Non-Functional Requirements

By analyzing the interests of each stakeholder, we can identify the most important non-functional requirements to be met by the enhancement. From the perspective of the end user, the software must have good performance, accuracy, reliability and security. Performance can be broken into several areas including smoothness of ride, energy efficiency and speed of vehicle response to road conditions. Another requirement of end users is accuracy. The software should be accurate in its interpretation of nearby objects, selection of routes, and its creation of control inputs to the physical hardware of the car. Furthermore, the system must be reliable. If the software is reliable this means that it is safe to use, and that maintenance is not required often. Finally, end users of Apollo require the system to be secure. Since Apollo is responsible for operating cars with passengers, a system that could be hacked would pose huge threats to the passengers of the Apollo-controlled vehicle as well as those in other vehicles. All these non-functional requirements, which are be influenced by the software’s architecture, impact the experience of the user.

The development team working on Apollo requires the code to be modifiable, maintainable and testable. Modifiability and maintainability allow improvements to be made to the software more easily. Good testability allows these modifications to be debugged. Technicians are in a similar category to developers in that they are responsible for maintaining vehicles, although their domain may be focused on the entire car as a whole, rather than just the software. They also would not be responsible for improving the software system, only maintaining it. Nonetheless, technicians that work on Apollo-equipped vehicles require manageability and supportability.

Car manufacturers require good portability and testability. Good portability refers to the ability to easily implement the software system on different hardware platforms, specifically, different vehicles. This would matter to car manufacturers because they may have several different models within their lineup that will all need Apollo software. Testability is important because the manufacturer is responsible to ensure that their vehicles operate correctly.

### Analysis

Table 1 presents the results of analyzing each attribute and how they may be affected by each implementation. It should be noted that certain attributes would be unaffected by either implementation, such as portability; however, as the vehicle state is utilized in various key features of the system, they remain of note.

TABLE 1

### Evaluation

Both implementations proposed improve supportability, modifiability, and testability, due to the structure of the enhancement and its intended purposes. The vehicle state provider’s separation and refactoring to a dedicated module improve the development process by improving vehicle state-related testing and increasing modularity within the system; this eases the development and testing of features and components, both present and not yet present within the system. Supportability is also improved as the system’s deployment including this enhancement may make it more straightforward for those working with the system to utilize and understand vehicle state-related information. Security may also be improved with respect to the vehicle state provider in both implementations, albeit differently.

The direct code implementation brings with it many of the same qualities as the current vehicle state provider implementation. For example, responsiveness and maintainability would not be strongly affected when compared to the current system, as methods of how the vehicle state is accessed and updated remain similar. However, system reliability may be at risk as developers must ensure the state is updated. While modifiability is improved by separating the vehicle state from the Common module, it could be improved further in this regard with another implementation. Finally, while the vehicle state is currently accurate for development purposes, other modules have control in updating it; as development continues, it should be studied to see if the state’s accuracy is affected in the future.

The publish-subscribe implementation improves maintainability further, both from a development perspective in the maintaining of code, as well as an operation perspective in the maintaining of the vehicle state itself. Modifiability is also further increased, as developers now have ensured that the vehicle state is updated upon new data, in addition to the transformation to a dedicated module easing state-related development. As with the other implementation, reliability and accuracy are coupled; the vehicle state would be updated as soon as new data is found, increasing accuracy, but it must be ensured that reliant modules have received the updated vehicle state before calculations begin, or otherwise the system’s reliability may be at risk. Finally, while concurrency in the system is improved, there is a risk due to added overhead; thus, responsiveness is not necessarily improved by this implementation.

## Chosen Implementation

-

### Effects on System NFR's

As discussed in the SAAM analysis of this implementation, maintainability may be positively affected by this implementation. From a development perspective, its separation makes the vehicle state provider and the features utilizing it easier to maintain and update; there is less burden on ensuring the vehicle state is at its most recent. From an operational perspective, the publish-subscribe implementation also improves maintainability of the system by ensuring that as soon as data is published suggesting that the state has changed, the vehicle state is updated.

The system’s evolvability may also be positively affected, as modifiability is increased due to the vehicle state provider’s separation into a new module. By making the system easier to modify, it also becomes easier to evolve in more major regards, provided that no major rehauls occur. Finally, by isolating the vehicle state provider and transforming it into its own dedicated module, the quality of the system’s testability may also be improved. By becoming its own module, it becomes easier to isolate for testing purposes.

However, responsiveness may be affected negatively by this implementation. While concurrency may be increased to some degree, there may also be added overhead as a consequence of the Vehicle State Provider becoming a publish-subscribe component rather than a directly accessed singleton object. This potential decrease in response time is a risk for the system.

## Use Cases

### Use Case 1 - Planning (Reading Vehicle State)

FIGURE 4

### Use Case 2 (Updating Vehicle State)

FIGURE 5

## Plans for Testing

Whenever modifying a systems architecture, it is crucial to test the enhancement. In this case, we will be adding a new Vehicle State component as a Cyber RT component. This neat separation of vehicle state into its own publish-subscribe module allows for much easier testing.

Currently the logic for the vehicle state is spaced out among various parts of the architecture. This means that all components that have a dependency on vehicle state will have to be refactored to use the new Vehicle State module. Such a large restructuring can lead to many hard to predict errors. Thus, a full integration test is required, to ensure the other components work correctly with the new Vehicle State module. This testing would likely first take form of a dry run wherein the test isolates key functions that access vehicle state and ensure there are performing as before. Since this enhancement is exclusively a change to the architecture, we can test the results of the functions before and after the enhancement to ensure they are the same. The final form of testing would be a real-life scenario, with a real autonomous vehicle running the Apollo software. The test would ensure that the vehicle operates as normal. This test can use the Monitor and Dreamview modules to view real time information and ensure it is as expected.

## Potential Risks

One of the main risks identified is the responsiveness of the software. Information about the vehicle state is critical and is required to be accessed quickly and frequently. This is best pictured by imagining a human driver. The driver, while not necessarily consciously aware, is continually assessing the state of the vehicle. If the engine were to stall and the gas pedal becomes unresponsive, the driver is immediately aware. If the brakes are applied, but perhaps due to ice, the vehicle does not decelerate as quickly as anticipated, the driver is immediately aware and able to respond. This responsiveness to vehicle state is perhaps even more critical for the Apollo autonomous software. Adding the Vehicle State as a pub sub module could add overhead in response time relative to directly accessing the data as is done presently. When enacting this enhancement, there must be clear bounds set on the response time of various queries to vehicle state.

## Lessons Learned

Various complications occurred in the process of creating and determining implementations for a new feature or enhancement. When coming up with possible new features, we found that their implementations would be straightforward due to how Apollo is currently implemented. As we wanted to perform SAAM Analysis and determine an implementation for a less straightforward enhancement, this lengthened the process of choosing a new feature or enhancement.

Once an enhancement was determined, we discovered that tasks would have to be done in order; we couldn’t divide up some of the work until previous work was completed. For example, the SAAM Analysis was more serial than parallel, and the evaluation had to be completed before use cases for a chosen implementation could be worked on in detail.

## Glossary

**SAAM** – Software Architecture Analysis Method; a method for describing architectures and how well they meet their non-functional requirements, or quality attributes, by studying these attributes with respect to various tasks. [\[Kazman, Bass, Abowd and Webb\]](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.65.8786&rep=rep1&type=pdf) 

## References

ApolloAuto. “ApolloAuto/apollo: An open autonomous driving platform.” GitHub. Last accessed April 11, 2022. Retrieved from https://github.com/ApolloAuto/apollo. 

Kazman, Rick et al. “SAAM: A Method for Analyzing the Properties of Software Architectures,” ICSE, 1994. Retrieved from https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.65.8786&rep=rep1&type=pdf. 

OBEIA. “The Concrete Architecture of Apollo.” March 21, 2022. Retrieved from https://obeia.github.io/assignment2/. 
