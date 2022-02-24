# Content Directory

## \*\*Warning: documentation of the content directory is stale, new documentation is forthcoming.\*\*

## Introduction

The content directory is an on-chain index of all content and metadata, and related concepts - such as channels and playlists. The data model was conceived to facilitate publishing and curation objectives inherent to the platform within the constraints of the blockchain environment, as a result, it has the following four major design traits:

* **Versioned**: Entities can have multiple simultaneous representations, known as _schemas_. Having this flexibility of multiple representations per entity allows us to avoid having to migrate our entire content directory whenever we want to alter the representation of some category. For example, adding a metadata field to videos. This is extremely valuable, because such a migration - even if feasible, will not only require freezing substantial portions of on-chain state across multiple blocks but also will incur the same high-security risk as doing any runtime upgrade at all. As a result, it can only happen infrequently, and only after substantial community coordination. This is extremely costly for the platform because the content directory has to be able to evolve quickly to allow new user-facing features.
* **Structured**: Representations are structured. This structuring not only allows for integrity guarantees on the content but also is the foundation for having permissions in the context of a multiple writer scenario. Without structure, you cannot selectively give different actors to write access to different properties.
* **Linked**: Representations are linked allowing us to build realistic domain models where entities are reused in different relationships.
* **Owned**: Entities are owned, initially be the actor responsible for creating them, and the write access permission model is aware of this ownership status.
* **Bespoke write access model**: There is a write access model that attempts to capture the common access control rules one would want to enforce over this type of data model. The model has to accommodate an evolving set of subject matter domain concepts without being updated itself, by the very assumption mentioned prior. As a result, it has a bespoke structure that attempts to strike a balance between simplicity and expressivity.

## Working Group

The content directory subsystem has a working group. The purpose of the group is to allocate curation responsibilities, and to performed highly privlidged alterations to the state of the directory, done by the lead, and to do actual curation, done by both workers and the lead. Workers are referred to as _content curators_.

## Data Model

To aid in the explanation of the data model we will develop a model of the following domain: people and the places they have lived.

### Classes

At the top of the conceptual hierarchy, we have the idea of a `class`. It represents a conceptual category, similar - but not identical, to tables in relational databases and classes in object-oriented programming. Examples of classes in our example could be

* `Person`: represents the concept of a person.
* `Place`: represents the concept of a place.
* `PersonLivedInPlace`: represents the concept that a person lived in a place.

There are a finite number of classes in the content directory at any given time, and each is identified by a unique positive integer called the `class id`. They start at `0`, and are incremented by one for each new class created. This is the identifier the blockchain uses, however, there is also a unique name and description property for each class, just for convenience.

### Entities

An `entity` represents a particular instance of a class, hence each entity corresponds to exactly one class. It represents a conceptual category, similar - but not identical, to rows in relational databases and objects in object-oriented programming. Following on from our example, they could be

* `Bob`: is an entity, representing a person with that first name, of the `Person` class.
* `London`: is an entity, representing the city of that name, of the `Place` class.
* `BobLivedInLondon`: is an entity, representing the fact that the person `Bob` has lived in place `London`, of the `PersonLivedInPlace` class.

There are a finite number of entities of each class in the content directory at any given time, and each is identified (across all classes) by a unique positive integer called the `entity id`. They start at `0`, and are incremented by one for each new entity is created. This is the identifier the blockchain uses for entities, however, there is also a unique name and description property for each class, just for convenience. Importantly, the name uniqueness for entities is only within the corresponding class.

### Schemas

A `schema` represents a standard for what information must be associated with any entity of a given class. There may be multiple such schemas associated with a given class at any given time. Each schema is identified with a unique integer within the given class, called the `schema id`, and it starts at `0`, and increases with each new schema introduced. Importantly, each entity may only admit to a subset of available schemas of their class, hence different entities may very well submit different schema sets simultaneously.

### Property Type

A `property type` represents a value constraint in four different ways. First, it requires that the value has a particular data type, of which there are the following available

1. **Bool:** True or false.
2. **Integer:** Either a signed or an unsigned integer of bit depth 16, 32 or 64.
3. **Text:** A byte vector no longer than some specified length.
4. **Reference:** Entity identifier of some specified class.
5. **Vector:** A vector, no longer than some specified length, of elements of some type defined in points 1-4 in this list, where all elements have the same parameter values for their constraints, so texts will all have the same upper bound, and references will all have the same class.

Second, whether the value can optionally be set to undefined or null value, in addition to its normal value range. Third, whether it is locked from being mutated by certain roles, the details of which are fully explained in the permissions model section. Lastly, fourth, whether entity references, should respect ownership, the details of which are fully explained in the permissions model section.

A property type corresponds to exactly one class, and is identified with unique positive integer (within that class), called a `property type id`. They start at `0`, and are incremented by one for each new type added for a given class. In addition, they also have a unique name (within that class), and a description. Following on from our example, some natural property types for the `Person` class, identified here by their name, could be

1. **firstName:** Text up to 20 bytes
2. **isMarried:** Bool
3. **isMale:** Bool
4. **bestFriendId:** Optional reference to another `Person` instance

The previously mentioned concept of a schema is really a list of property types. In fact, the only way to introduce new property type is to introduce a schema that describes a new property type. A new schema can both introduce new ones, and reference existing ones. For example, the three following schemas could exist for the `Person` class simultaneously

1. `[firstName, isMarried]`
2. `[firstName, isMarried, isMale]`
3. `[firstName, bestFriendId]`

Assume that they were introduced in order, and notice that a schema may both reuse old property types, or omit them entirely. Each schema introduced should be understood as an incremental change to the desired representation of the concept of a `Person`. However, the introduction of a new schema does not mean that old ones are no longer valid.

### Property Values

A `property value` is a value that corresponds to a property type in a given class, and respects the constraints imposed by that type. Each entity is said to support one or more of the schemas associated with the corresponding class, and for each unique property type in those schemas, the entity has a property value. The way property values are added to an entity is in-fact only in the context of introducing support for some schema that references a property type for which the entity does not already have a value. Building further upon our example, let's assume `Bob` supports schemas 1, in which case he could have property values

1. **firstName (due to 1):** Robert
2. **isMarried (due to 1):** false
3. **isMale (due to 1):** true

Then support for schema 2 is introduced for `Bob`, in which case he could now have property values

1. **firstName (due to 1 and 2):** Robert
2. **isMarried (due to 1):** false
3. **isMale (due to 1):** true
4. **bestFriendId (due to 2):** 3

where 3 is entity id of some other `Person` entity. Notice here the benefit that schemas that share property types will share values, avoiding duplication.

#### Vectors

Vector values additionally maintain a non-negative integer nonce value that starts at zero. The purpose of this value is to support different actors possibly simultaneously operating on the vector, as a vector. By this we mean adding or removing a vector element in an arbitrary position. These kinds of operations require a nonce value so as to ensure that the operation has the outcome the actor intends, even as others may be performing other contradictory operations. Normal account nonces are not a substitute, as they only avoid replaying the same operation by the same actor.

## Permission Model

### Overview

The permission model is purely a _write-access_ model, as all data is publicly readable by anyone. It is a role-based model, which means that a given subject may or may not perform some write operation in the context of some specific role, hence a given write operation may be available under one role, but not under another. It is the responsibility of the subject to manage the information about what is available to them under which role they occupy at any given time.

### Roles

There are four roles in the content directory, namely

* **Council:** The council can directly act within the content directory through its proposal system. It can change very high-level permissions.
* **Lead:** This role, which is only ever occupied by at most one actor, is the highest authority in the directory. The role is rewarded and staked. The lead can both directly update content and permissions, and also introduce curators with a narrower set of related responsibilities.
* **Curator:** This role, which may be occupied by many actors simultaneously, is a curation and publishing authority in the directory. The role is rewarded and staked, and an actor enters and is evicted and slashed, at the discretion of the lead. Each curator is identified by a unique positive integer called a _curator id_.
* **Member:** This is the normal membership role on the Joystream platform, and recall that each member is identified by a unique positive integer called a _member id_.

Beyond these explicit roles, there are three types of status that can apply to subsets of these roles.

* **Curator group membership:** There is also the concept of a _curator group_, which is a group with one or more member curators that can rotate in and out. The number of curator groups can change over time, as the lead and add and remove them, and the lead also manages the presence of curators in such groups. All actions accessible to a curator is by virtue of membership in one such group. The reason for this extra level of indirection is not only that curators are likely to work in groups, where each group corresponds to permissions that reflect some coherent set of responsibilities, but also so that it is easy to withdraw permissions from an individual curator, without having to update a lot of state. Lastly, a group also has a status, which is either enabled or disabled, where the latter status makes it impossible to act under that status at that time.
* **Class maintainer:** This is a status that a curator group can occupy for a given class, and a class can have none or multiple such maintainer groups. A maintainer is automatically granted the ability to create entities of that class, and is also set as the controller when doing so (see next point). Individual property types can also be configured to prevent or allow corresponding property values to not be mutable by the maintainer.
*   **Entity controller:** This is a status that can be held by either

    * the lead
    * a specific member
    * maintainers of the class

    for a specific entity, hence each entity will have an independent controller. Individual property types can also be configured to prevent or allow corresponding property values to not be mutable by the controller.

### Personae

When making any transaction, an actor must specify what role they are acting under, and authenticate with the account currently associated with that role. Some transactions are inherently only open to a particular role, but where multiples roles can act an explicit role representation called a _personae_ is provided, and it is one of the following

* The lead
* A member
* A curator in some curator group

Importantly, a lead and a member can only have the status as a controller, not a maintainer, while a curator in a curator group can be both a controller and a maintainer. Hence, transactions that are sensitive to the distinction between the maintainer and controller status, the runtime will automatically work out whether a curator in a group satisfies the required status. This is specifically the case with all operations on existing entities.

### Property Level Permissions

For property types there are the following two permissions available

* **Locking:** Each property type can independently constrain the entity maintainer and entity controller from mutating a property value in an entity corresponding to the class of the type. The typical use case for this is that there may be sensitive information that should not be mutable by the given party. For example, information about where to make payments should not be mutable by the maintainer, while information on the curation status should not be mutable by the controller.
* **Same-controller references:** Referential property types can require that the destination entity must have the same controller as the source entity. The use case for this is that certain types of relationships one may want to model should only be allowed to established by the controller. For example, only the controller of a channel should be able to associated videos under that channel, or only the controller of a music album should be allowed to associate tracks as being singles in that album.

### Class Level Permissions

Associated with each class there are a set of class permission settings which describes the following

1. Whether everyone is currently blocked from creating entities.
2. Whether all entity property values are currently locked from everyone.
3. Whether members can create entities.
4. The current set of maintainers.
5. An upper bound on the number of entities allowed to exist.
6. Default upper bound on the number of entities a single controller can create.
7. The number of entities created by each possible controller is tracked, and there is a separate limit for each that can be changed by the lead.

### Entity Level Permissions

Associated with each entity there is a set of permission settings

1. The current controller
2. Whether the entity can have any of its property values updated by anyone at all.
3. Whether the entity is currently referencable by any new reference.

### General Permissions

The maximum number of

1. operations in an batching transaction.
2. classes.
3. schemas in any class.
4. curator groups.
5. entities the lead can manually set a controller to be allowed to create.

## Operations

In this section the main operations available are described. The _permissions_ section on each operation is not meant to be an exhaustive list of conditions, just to highlight the permissions constraints.

### Council operations

They can update the values found under the general permissions section, but there are hard coded upper bounds that prevent new values from being abusive. Changing these upper bounds requires a runtime upgrade.

### Lead operations

This section groups all operations that are exclusive to the lead.

#### Create class

* **Parameters**
  * Name of class.
  * Description of class.
* **Permissions**
  * Only lead can perform.
  * Respect maximum number of classes limit.
* **Effect**
  * Introduces a new class with given name and description.

#### Add curator group

* **Parameters**
  * Name of group.
  * Description of the group.
* **Permissions**
  * Only lead can perform.
  * Respect maximum number of groups limit.
* **Effect**
  * Introduces a new group with given name and description, which is enabled and has no members.

#### Update curator group status

* **Parameters**
  * Group identifier.
  * New status value.
* **Permissions**
  * Only lead can perform.
* **Effect**
  * Changes group to have new status value.

#### Remove curator group

* **Parameters**
  * Group identifier.
* **Permissions**
  * Only lead can perform.
  * Only if group is not maintainer of any class.
* **Effect**
  * Removes group.

#### Add curator to curator group

* **Parameters**
  * Group identifier.
  * Curator identifier.
* **Permissions**
  * Only lead can perform.
* **Effect**
  * Curator is member of group.

#### Remove curator from curator group

* **Parameters**
  * Group identifier.
  * Curator identifier.
* **Permissions**
  * Only lead can perform.
* **Effect**
  * Curator is no longer member of group.

#### Add schema to class

* **Parameters**
  * Class identifier.
  * Schema description: a vector where each element is either the id of an existing property type in the class or the description of a new property type.
* **Permissions**
  * Only lead can perform.
  * Respect the maximum number of schemas per class limit.
* **Effect**
  * Introduces a new schema, as described, on the given class.

#### Update controller entity creation limit for class

* **Parameters**
  * Class identifier.
  * Controller.
  * New limit.
* **Permissions**
  * Only lead can perform.
  * Respect maximum global limit for entity for any controller.
* **Effect**
  * Controller can at most create the new number of entities

### Public operations

These operations are public in the sense that they can in principle be open to either member, curator groups, or leads, depending on how permissions are currently set.

#### Create entity of class

* **Parameters**
  * Class identifier.
  * Personae.
* **Permissions**
  * Class level entity creation is allowed.
  * Total entity count for class is respected.
  * Respect personae specific entity creation budget for class.
* **Effect**
  * Introduce a new entity of class, which supports no schemas, and thus has no property values. It also increments the entity creation counter for personae on class.
  * Importantly, the personae are set as the controller, and specifically, a curator results in the class maintainers as the controller, not that specific group.

#### Batch operation

This operation allows a batch operation, which is atomic, over the following operations

* Entity creation
* Adding schema support to the entity
* Update property value of the entity

Additionally, entities created within the a batch can be referenced in later operations in the same batch, sidestepping the problem that final entity ids are not knowable when transactions are issued, due to possible competing write operations.

Lastly, all operations must be performed under the same personae.

### Entity controller and maintainers

Be aware that a curator group member personae is automatically granted the most extensive access if it is both controller and maintainer.

#### Add schema support to entity

* **Parameters**
  * Personae.
  * Entity identifier.
  * Class identifier.
  * Schema identifier.
  * New property values.
* **Permissions**
  * Entity is not frozen.
  * Personae is maintainer or entity controller.
* **Effect**
  * Introduce schema support on entity, and storing new property values. Importantly, even values that are locked from mutation by personae can be provided initial value.

#### Update property value on entity

* **Parameters**
  * Personae.
  * Entity identifier.
  * Property value identifier.
  * New value.
* **Permissions**
  * Entity is not frozen.
  * Class level entity value mutation not locked.
  * Personae is not locked from mutating.
  * New value respects property type constraints.
* **Effect**
  * Updates property value.

#### Update vector property value element

* **Parameters**
  * Personae.
  * Entity identifier.
  * Property value identifier.
  * Element identifier.
  * New value.
* **Permissions**
  * Entity is not frozen.
  * Class level entity value mutation not locked.
  * Personae is not locked from mutating.
  * New value respects property type constraints.
* **Effect**
  * Updates property value.

#### Add element to vector property

* **Parameters**
  * Personae.
  * Entity identifier.
  * Property value identifier.
  * New element position identifier.
  * New value.
* **Permissions**
  * Entity is not frozen.
  * Class level entity value mutation not locked.
  * Personae is not locked from mutating.
  * New value respects property type constraints.
* **Effect**
  * Updates property value.

#### Remove element from vector property

* **Parameters**
  * Personae.
  * Entity identifier.
  * Property value identifier.
  * Element identifier.
* **Permissions**
  * Entity is not frozen.
  * Class level entity value mutation not locked.
  * Personae is not locked from mutating.
* **Effect**
  * Vector element is removed.

## Examples

WIP.

## Tooling

WIP.

## Current Model

WIP.
