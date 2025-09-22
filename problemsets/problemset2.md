# Problem Set 2: Composing Concepts  

## Concept Questions  

1. **Contexts**. The *NonceGeneration* concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?  

The short string generated only needs to be unique in its context. The context provided holds all of the short strings for that specific context which is what we care about since the short string can be duplicated across contexts. In the URL shortening app, the context would be the base URL such as tinyurl.com or yellkey.com. For example, the short string "apple" can be used for both domains (tinyurl.com/apple and yellkey.com/apple are ok to exist at the same time). However, we do not want two of yellkey.com/apple. Context would allow that.  

2. **Storing used strings**. Why must the *NonceGeneration* store sets of used strings? One simple way to implement the *NonceGeneration* is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an *abstraction function*.)  

*NonceGeneration* needs to store used strings so that the generated string is unique to the ones in the context. It can only do that if it knows what other strings are being used, thus we have to store the used strings. Our invariant is that the size of the set of used strings should be equal to the counter. Every time the counter increments, generate was called and should have given us a unique string. *NonceGeneration* should update its set of used strings to reflect that. However, if the size of the set of used strings did not increase and is not equal to the counter, then it means a duplicate was added to the set.  

3. **Words as nonces**. One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the *NonceGeneration* concept to realize this idea?  

One advantage for the user is that the shortened URL is easier to remember and share with others. One disadvantage for the user is that it can be easy to mistype or mishear the word due to homophones and different spellings (e.g. gray vs. grey or knight vs. night).  

To modify *NonceGeneration*, we can pass a dictionary generic type which holds a set of valid words into generate and require that the nonce is also in the dictionary. Below shows the following modifications (only changes are shown).  

- **concept** NonceGeneration [Context, DictionaryWords]  
- **state**  
    - a set of DictionaryWords with  
        - a commonword String  
- **actions**  
    - generate (context: Context, dictionary: DictionaryWords) : (nonce: String)  
        - **effect** returns a nonce that is not already used by this context and is in the dictionary  

## Synchronization Questions  

1. **Partial matching**. In the first sync (called *generate*), the *Request.shortenUrl* action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called *register*) both appear. Why is this?  

In *generate*, we only need the shortUrlBase argument for *NonceGeneration.generate* so it does not matter what targetUrl is. Since we don't need it, we can omit it. In *register*, *UrlShortening.register* requires both the shortUrlBase and targetUrl as arguments so we need both which needs to be named and comes from *Request.shortenUrl*.  

2. **Omitting names**. The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?  

You can only omit the name when they are the same. While it provides more succint specification, you don't always want this. Using different names allows for more flexibility such as using Context instead of shortBaseUrl allows the *NonceGeneration* concept to be more general and modular. When you connect different these two together such as in NonceGeneration.generate (context: shortUrlBase), having different names and stating it in context: shortUrlBase provides much more clarity at the expense of succintness.  

3. **Inclusion of request**. Why is the *request* action included in the first two syncs but not the third one?  

The first two syncs *generate* and *register* need information from the *request* action. This means that the then action can only occur once the *request* action has occurred which is enforced by including it in the when. *ExpiringResource.setExpiry* only requires shortUrl which is given to us by the *UrlShortening.register* so we only need that action to run. Thus, requiring the *request* action to run would be unnecessary.  

4. **Fixed domain**. Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?  

We can change *NonceGeneration.generate* and *UrlShortening.register* to the following lines below. That way no matter what the *request* action puts for shortUrlBase, we enforce that it is "bit.ly."  

NonceGeneration.generate (context: "bit.ly")  
UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit.ly", targetUrl)  

5. **Adding a sync**. These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the *ExpiringResource* and *URLShortening* concepts.  

- **sync** deleteExpired  
- **when** ExpiringResource.expireResource (): (resource: shortUrl)  
- **then** UrlShortening.delete (shortUrl)  

## Extending the Design  
1. Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state). It should not be necessary to make any changes to the existing concepts.  

- **concept** ShortUrlOwner [User]
- **purpose** to associate shortUrls with the user who made them
- **principle** 
    - the user who registers a shortUrl is saved as its owner;
    - given the shortUrl, its owner can be found;
    - the shortUrl and its owner can be deleted;
- **state**
    - a set of ShortUrls with
        - an owner User
- **actions**
    - linkUser (shortUrl: String, owner: User)
        - **requires** no owner currently exists for shortUrl
        - **effect** saves shortUrl with owner User
    - verifyOwner (shortUrl: String, owner: User): (verifiedShortUrl: String)
        - **requires** shortUrl to exist and have matching owner User
        - **effect** returns associated shortUrl as verifiedShortUrl

- **concept** Analytics [User]
- **purpose** to count the number of accesses to a link
- **principle**
    - a count is started for shortUrl set to 0;
    - the associated count can be incremented;
    - the count can be read;
- **state**
    - a set of ShortUrls with
        - a count Integer
- **actions**
    - init (shortUrl: String)
        - **requires** a count does not exist for shortUrl
        - **effect** starts a count set to 0
    - increment (shortUrl: String)
        - **requires** a count exists for shortUrl
        - **effect** increment the count by 1
    - read (shortUrl: String): (count: Integer)
        - **requires** a counter exists for shortUrl
        - **effect** returns count


2. Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are expanded; and one when a user examines analytics.  

- **sync** initAnalytics
- **when**
    - Request.shortenUrl (user)
    - UrlShortening.register (): (shortUrl)
- **then**
    - ShortUrlOwner.linkUser (shortUrl, owner: user)
    - Analytics.init (shortUrl)

- **sync** linkAccessed
- **when**
    - UrlShortening.lookup (shortUrl): ()
- **then**
    - Analytics.increment (shortUrl)

- **sync** viewAnalytics
- **when**
    - Request.viewAnalytics (user, shortUrl)
    - ShortUrlOwner.verifyOwner (shortUrl, owner: User): (verifiedShortUrl: String)
- **then**
    - Analytics.read (verifiedShortUrl): (count: Integer)

3. As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:  
    - Allowing users to choose their own short URLs;  
        - We can allow this by using the following sync.
            - **sync** register
            - **when**
                - Request.shortenUrl (targetUrl, shortUrlBase, shortUrlSuffix)
            - **then**
                - UrlShortening.register (shortUrlSuffix, shortUrlBase, targetUrl)
    - Using the “word as nonce” strategy to generate more memorable short URLs;  
        - We can allow this by using the following modifications to *NonceGeneration*.
            - **concept** NonceGeneration [Context, DictionaryWords]  
            - **state**  
                - a set of DictionaryWords with  
                    - a commonword String  
            - **actions**  
                - generate (context: Context, dictionary: DictionaryWords) : (nonce: String)  
                    - **effect** returns a nonce that is not already used by this context and is in the dictionary  
    - Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;  
        - We can allow this by swapping out shortUrl for targetUrl in the Analytics concept, ShortUrlOwner concept, and their associated syncs as seen in **Extending the Design** parts 1 and 2. An example is shown below.
            - **sync** initAnalytics
            - **when**
                - Request.shortenUrl (user, targetUrl)
                - UrlShortening.register (targetUrl): ()
            - **then**
                - ShortUrlOwner.linkUser (targetUrl, owner: user)
                - Analytics.init (targetUrl)

            - **sync** linkAccessed
            - **when**
                - UrlShortening.lookup (): (targetUrl)
            - **then**
                - Analytics.increment (targetUrl)
    - Generate short URLs that are not easily guessed;  
        - The purpose of the UrlShortening concept is to create shorter or more memorable links. This goes against that purpose as hard to guess short URLs have more complexity to them which means they are longer and harder to remember. The new link would then still be hard to share. It should not be used as a way for security as that should be its own concept with better solutions.
    - Supporting reporting of analytics to creators of short URLs who have not registered as user.  
        - This should not be done as it would have too much security risk. Methods such as using IP address are easily bypassed with spoofing such that malicious parties can access this information easily. Secret links to analytics can get forwarded or logged and now access would be hard to manage. Without accounts, you can’t reliably prove who the owner is and who should be able to see a link’s data.
