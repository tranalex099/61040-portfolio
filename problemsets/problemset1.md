# Problem Set 1: Reading and Writing Concepts

## Exercise 1: Reading a Concept

### Questions

1. **Invariants**. What are two invariants of the state? (*Hint*: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

- One invariant concerns the count which cannot be negative. The other invariant concerns the Purchase which must have a corresponding Request with the same Item. The second invariant with the Purchases and Requests is more important because if purchases are not tied to a request and registry we can't track who bought what and determine how much is remaining. The purchase (purchaser: User, registry: Registry, item: Item, count: Number) action requires a request for this item with at least count before decrementing which preserved the first invariant. It also requires there to be a corresponding and active registry and request. However, it does not guarantee that this will still exist after the purchase has been made.

2. **Fixing an action**. Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

- The removeItem (registry: Registry, item: Item) action can break the invariant that a Purchase must have corresponding Registry and Request if the removeItem action occurs after a purchase has been made. It currently can remove items from the registry after purchases are made by only removing the Request. This would now leave the Purchase with an attached request. One way to fix this would be to require no Purchase to exist with the same Item before deleting. Another way would be to attach a Registry to the Purchase so that if the Request is deleted it is still attached to a Registry.

3. **Inferring behavior**. The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

- A registry can be repeatedly opened and closed since it just flips the active Flag and there is no constraint on how many times this can be done. Some reasons to allow this would be if you wanted to use the same Registry for multiple events, so you would close it at the end of one event and reopen it before the next event. It it also useful to temporarily close it to edit the items on the Registry.

4. **Registry deletion**. There is no action to delete a registry. Would this matter in practice?

- No, since you can close it to the public and purchases can no longer be made. The history is also useful for returns, warranties, and audits. A delete registry would require there to be no purchases associated with that registry.

5. **Queries**. What are two common queries likely to be executed against the concept state? (*Hint*: one is executed by a registry owner, and one by a giver of a gift.)

- The registry owner would likely query for purchases made with the registry and would likely want a list of how many items were purchased by who. The gift giver would likely want to query for available items to pruchase (i.e. items with non-zero Request counts in active Registrys).

6. **Hiding purchases**. A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

- To augment the concept to support surprises, we can add a surprise Flag to the Registry which would be set to False during the create action. We can add a new setSurprise(registry: Registry, surprise: Flag) action to turn on and off the surprise. When the surprise is True, only allow owner queries to show remaining counts.

7. **Generic types**. The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

- Names, descriptions, prices are subject to change over time and can vary based on seller. By using generic parameters, you are also not constrained to a specific type allowing the code to be more versatile. Since SKUs also don't vary across sellers, you can connect easily this concept to other ones, including externally.

## Exercise 2: Extending a familiar concept

### Questions

1. Complete the definition of the concept state.

  **state**  
    a set of Users with...  
      a username String  
      a password String  

    a set of usernames
      a user User

2. Write a requires/effects specification for each of the two actions. (*Hints*: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)

register (username: String, password: String): (user: User)  
  **requires** username to not already exist  
  **effects** creates and returns a new user User with username and password, add username to set of usernames  

authenticate (username: String, password: String): (user: User)
  **requires** user User to exist with matching username and password  
  **effects** returns user User

3. What essential invariant must hold on the state? How is it preserved?

- The essential invariant that must hold is that usernames must be unique. We preserve this by requiring the username to not already exist in our set of Users before we can create a new user User.

4. One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality. (*Hints*: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)

  **state**  
    a set of Users with...  
      a username String  
      a password String  
      a confirmed Flag
      a token Token

    a set of usernames
      a user User

register (username: String, password: String): (user: User, token: Token)  
  **requires** username to not already exist  
  **effects** creates and returns a new user User with username, password, confirmed to False, and a token Token, add username to set of usernames  

confirm (username: String, token: Token): (user: User)
  **requires** user User to exist with matching username and token  
  **effects** sets confirmed Flag to True

## Exercise 3: Comparing concepts

**concept** PersonalAccessToken[User, Permission]

**purpose** to allow for alternative authentication

**principle**  
a user creates a token with a scope and expiration;  
the system records the token and displays its value to the user;  
then the user and their clients can present the token when accessing GitHub and the system authenticates requests if the token is active and valid;  
the user or GitHub can revoke or expire the token, after which it can no longer be used for authentication.

**state**  
  a set of Users with  
    a username String  
    a Token

  a set of Tokens with  
    a Value  
    an owner User  
    a scope Permission  
    an expiration Expiration  
    an status Flag

  a set of Values with  
    a Token

**actions**  
createToken (owner: User, scope: Permission, expiration: Expiration): (token: Token)  
  **effects** generates a new token Token with random value, associates it with owner, stores its scope and expiration, marks status active, and adds value to set of Values

authenticate (username: String, value: Value): (user: User)  
  **requires** a User with username exists and has a token with matching value, not expired, and an active status  
  **effects** returns user User

revokeToken (token: Token)  
  **requires** token Token is Active  
  **effects** changes token status to Revoked, removes value from set of token values

expireToken (token: Token)  
  **requires** token Token has passed expiration date  
  **effects** changes token status to Expired, removes value from set of token values

A password is tied to a user’s entire account. A personal access token (classic) is a separate, generated secret that can be created, scoped, expired, and revoked independently. A personal access token changed without affecting the main account password. However, they are broader than passwords in scope. They cover all repositories the user can access unless restricted by organization policy. The documentation could be clearer if it included an quick comparison table near the top, showing side by side differences and use cases.

## Exercise 4: Defining familiar Concepts

### URL Shortener

**concept** URLShortener[User, URL]

**purpose** to create shorter URLs that point to a longer URL

**principle**  
a user creates an auto-generated or user-defined shorter URL to point to a longer URL;  
the user can input the shorter URL to see the longer URL, see all of their shorter URLs, and delete unneeded shorter URLs

**state**  
  a set of ShortURLs  
    an owner User  
    a longURL URL  
    a suffix Suffix  

  a set of Suffixes  
    an owner User  

  a set of owner Users
    a suffix Suffix

**actions**  
createGeneratedSuffix (owner: User, longURL: URL): (shortURL: ShortURL, suffix: Suffix)  
  **effects** create new ShortURL, suffix Suffix, and owner User with longURL

createDefinedSuffix (owner: User, longURL: URL, suffix: Suffix): (shortURL: ShortURL, suffix: Suffix)  
  **requires** suffix Suffix to not already exist  
  **effects** create new ShortURL, suffix Suffix, and owner User with longURL

deleteSuffix (owner: User, suffix: Suffix)  
  **requires** suffix Suffix and shortURL ShortURL to exist with matching owner User  
  **effects** removes shortURL and suffix

accessURL (owner: User, suffix: Suffix): (longURL: URL)  
  **requires** suffix Suffix and shortURL ShortURL to exist with matching owner User  
  **effects** returns associated longURL

userSuffixes (owner: User): (suffix: Suffix)  
  **requires** owner User to exist  
  **effects** returns associated suffixes

### Conference Room Booking

**concept** ConferenceRoomBooking[User, Room]

**purpose** to let users reserve conference rooms at specific times

**principle**  
a user creates

**state**  
  a set of Slots  
    a Room  
    a Time  

  a set of Bookings  
    a User  
    a Slot  

**actions**  
createSlot (room: Room, time: Time)  
**effects** add a new slot for room at time

deleteSlot (slot: Slot)  
**requires** no bookings exist for this slot  
**effects** remove s from the set of slots

book (u: User, r: Room, t: Time): Booking  
**requires** some unbooked slot for r at t  
**effects** add new booking for slot with user u

cancel (b: Booking)  
**requires** b is an existing booking  
**effects** remove b from the set of bookings