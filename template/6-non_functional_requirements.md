# Non-Functional Requirements

## Security, privacy, and data retention policies


Our app utilizes Google Authentication via Firebase, which handles the management of passwords, emails, and refresh tokens. This alleviates the need for us to store these credentials. However, since our app accesses the user's location and photo gallery, we must request explicit permission for both. Currently, location permissions are requested, but we must also implement a mechanism to request gallery access permission to comply with privacy regulations.

WanderWise aims to facilitate the creation and discovery of itineraries nearby. It is essential to have a well-defined policy regarding the data we retain. Specifically, we store users' liked itineraries to provide personalized recommendations. We must clearly communicate to users that this data is retained to enhance their experience and usability.

For optimal functionality, our app requires access to the user's location and photo gallery. Users should have the option to grant or revoke these permissions at any time through the app's settings. Additionally, users should be able to select and display their preferred usernames, with no other personal information being necessary. If a user opts not to grant these permissions, itinerary recommendations will be based on popularity metrics such as the number of likes.

## Adoptions, Scalability and Availability

To ensure scalability, the app should display itineraries based on user preferences and location. If the user shares their location, itineraries should be displayed accordingly. The app must also offer an intuitive interface for modifying permissions and enabling or disabling notifications related to location tracking.

As the app relies heavily on Firebase, it is crucial that Firebase servers and databases are consistently available to maintain app functionality. Google handles the authentication process, requiring stable and reliable servers for connecting and storing user-sensitive data. For itinerary creation, continuous availability of Google Maps and Firebase services is necessary to ensure a seamless user experience. In case of connectivity issues, the app should allow users to access previously liked itineraries offline. Additionally, we need to implement failover mechanisms to handle system errors and maintain operational continuity.

