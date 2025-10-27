# 5-Layer Architecture

## Term Definitions

Definitions of terms used in this document:

| Term | Meaning |
|------|---------|
| Horizontal Tapping (HTG'83) | Implementing system integration between components in the same layer |
| Component | A system divided by technical roles and composed of domain aggregates |
| Business Logic | The purpose processing of functions provided on the service |

## Reasons for Adopting 5-Layer Architecture

## Benefits Achieved by Adopting 5-Layer Architecture

The system divides roles into 5 layers to achieve the following:

- **Improve maintainability** by dividing layers according to technical capabilities (separation of technical concerns), clarifying code understanding and modification locations
- **Enable expansion of technology stack** for each layer
- **Increase reusability of each component** by clarifying the role of each layer
- **Reduce organization and achieved coupling** by limiting the impact of specific layer changes
  - **Contain the impact of data store changes to the Data Access layer** by abstracting DB connection processing from business logic (service layer) through the Data Access layer
- **Improve maintainability** by separating mutual dependencies and circular dependencies through limiting system integration directions
- **Improve individual scalability** by arranging layers according to access characteristics

## Achievable Quality Characteristics

| Characteristic | Sub-characteristic | System Design Quality Requirements | Quality Achievable Through This Architecture |
|----------------|-------------------|-----------------------------------|---------------------------------------------|
| **Compatibility** | Interoperability | Whether it can appropriately exchange information and connect with other systems | |
| **Reliability** | Maturity | The system is sufficiently tested. Has sufficient operational track record and operates normally. For new systems, it is not considered mature yet (Bugs/code line) | |
| | Availability | The degree to which the system is available, requiring fault tolerance and recoverability | ✅ |
| | Fault Tolerance | Handling abnormal data detection and system malfunction detection to continue service | |
| | Recoverability | How easily the system can recover when problems occur | |
| **Maintainability** | Modularity | Whether the system is well-structured and configured, and modules are independent | ✅ |
| | Reusability | Whether general-purpose systems or software components can be used in various situations | ✅ |
| | Analyzability | Easy identification of causes when failures occur | |
| | Modifiability | Can make corrections quickly and accurately without degradation during modification. Limited dependencies, limited modification locations, etc. | |
| | Testability | Ease of testing. Whether functions supporting testing monitoring, profiling are comprehensive | |
| **Portability** | Adaptability | Effectiveness and efficiency of system adaptation to different hardware or software environments | |
| | Installability | Whether procedures for adding or removing from the operating environment are easy | |
| | Replaceability | Whether the system/software operating environment can be used continuously | |
| **Performance Efficiency** | Resource Efficiency | Whether the system uses resources efficiently | ✅ |

---

# 5-Layer Architecture Details

## Roles of Each Layer and Rules for Each Layer [MUST]

The 5-layer architecture clarifies each role by separating concerns.

| Layer | Layer Name | Role | Rules |
|-------|-----------|------|-------|
| - | **Special Layer** | • Enable realization of unique business logic using main application API, Service GW Layer, Service Layer as platform foundation, and using it via Service GW Layer<br>• Backend-oriented tools and customer support tools special layer plugin<br>• Layer that combines APIs to build GUI<br>• Mainly implements template processing and JavaScript<br>• BFF for apps and frontend also corresponds to this layer<br><br>**About BFF Layer:**<br>Backend For Frontend provides data capabilities needed when building UI. This function exists as part of the User Interface Layer without being separated as an independent layer. | • Only Service Layer can be used via Service GW Layer<br>• Direct access to Data Access Layer is prohibited<br>• Data stored within Special Layer can directly access data store |
| 5 | **User Interface Layer** | • Specific roles:<br>• Cookie resolution<br>• Login ID conversion<br>• UI logic<br>• Multi-application mashup<br>• Cross-domain mashup<br>• BFF basically cannot have service logic<br>• Reasons:<br>• Service logic distribution makes layer and functional unit roles ambiguous<br>• BFF code becomes bloated and difficult to develop<br>• External partners also correspond to this layer | • Only APIs provided from Service Layer via Service GW can be used<br>• Direct access to Data Access Layer is prohibited |
| 4 | **Service GW Layer** | • Expose APIs to User Interface and external partners (external exposure)<br>• Acts as Proxy for each Service Layer<br>• Handles traffic control for Service Layer origins | • All APIs provided by each Service Layer are used through this Service GW Layer<br>• Access to Data Access Layer is not permitted |
| 3 | **Service Layer** | • Provide business logic<br>• Business processing rules<br>• Judgment and formatting processing requiring multiple data sources<br>• Know dependencies and order between Data Access Layers<br>• Know Data Access Layer API specifications<br>• Batch processing also corresponds to Service Layer when it relates to business logic<br><br>• Perform Data Storage Layer abstraction<br>• Perform O/R mapping<br>• Hold logic completed with single data source<br>• Manage scaling<br>• Farming and sharding<br>• ID and farm mapping judgment, etc.<br>• Validate data types, string lengths, etc.<br>• Contain Data Storage replacement impact only to Data Access Layer | • Must be accessed via Service GW Layer<br>• Use abstract APIs provided by Data Access Layer for information reference and updates (do not directly access Data Storage Layer)<br>• Output trace logs to facilitate bug and report-based investigations<br>• Must not create dependencies where Service Layer calls other Service Layers |
| 2 | **Data Access Layer** | • Perform Data Storage Layer abstraction<br>• Perform O/R mapping<br>• Hold logic completed with single data source<br>• Manage scaling<br>• Farming and sharding<br>• ID and farm mapping judgment, etc.<br>• Validate data types, string lengths, etc.<br>• Contain Data Storage replacement impact only to Data Access Layer | • Only allow access from Service Layer, do not permit direct access through GW Layer |
| 1 | **Data Storage Layer** | • Layer that stores data such as DB and storage | • Data updates and operations are performed through Data Access Layer<br>• Organization-wide APIs providing specific logic are also defined as Data Storage Layer and accessed through Data Access Layer |

---

## Constraints on Integration Direction Between Layers [MUST]

Each layer in the 5-layer architecture is a closed layer. Closed layers must go through the immediately lower layer without skipping any layers when accessing between layers.

The reason is to arrange into layers with specific functional roles and integrate components.

Maintaining this state keeps dependencies in one direction, reduces coupling, and makes the entire system resistant to changes in specific components.

When this constraint is ignored there are situations such as the need to go through layers that have no actual meaning, causing latency deterioration, increased failure points, and resource waste.

These should be judged case-by-case regarding how much compliance should be maintained.

**[Diagram: Integration Direction Constraints]**

### Note about Message Queue Path:

In the overview integration diagram, certain message queue paths may have reverse request flow, but this is a permitted path on the premise of only performing low-coupling information integration.

Details: Event Integration (MQ/Webhook) Feasibility

---

## Overview Integration Diagram

**[Diagram: 5-Layer Architecture Overview]**

**[Diagram: System Integration Overview]**
