# Design and Implementation

## Frontend

In WanderWise, we used exclusively kotlin and AndroidX Compose libraries for the UI. 
The application layout and multiple layout components are built using Jetpack Compose, which offers numerous advantages. Jetpack Compose uses a declarative syntax, reducing complexity and making the codebase easier to maintain. It has built-in support for managing UI state, helping to create reactive UIs that update automatically in response to state changes.

Compose integrates seamlessly with existing Android views, allowing for gradual migration of legacy code. It is fully supported by Android Studio, offering features like live previews and hot reloading for instant feedback on changes. The toolkit allows for the creation of reusable and customizable composable functions, and includes simple APIs for rich animations and transitions.

Designed with performance in mind, Jetpack Compose minimizes unnecessary recompositions and uses efficient rendering techniques. It also makes building accessible apps easier with built-in support for accessibility services. Regular updates from Google ensure it stays current with the latest Android advancements, and a growing ecosystem of libraries and tools supports its use.

For styling, we used *Material Design*, Google’s open-source design system. Icons in our application exclusively use Material icons, offering an easily recognisable and visualy appealing interface.


## Backend (Application Server Design)

In our POC, the burden of managing business logic was placed largely on the app,
namely database accesses and API requests to different endpoints. We would like
to centralize all of this functionality in our application servers, allowing the
frontend application to communicate via CRUD requests to a single endpoint that
we host (for most requests) This will imply

- A more performant and reactive application frontend: the user's device won't 
have to handle and complicated business logic such as filtering a large number 
of public itineraries.
- Clear separation between the frontend and backend: the backend can be updated
independently of the frontend, provided that the general API format is 
well-defined and consistent

We envisage implementing core backend functionality in blazingly-fast and 
memory-safe Rust.

### Authentication

We would like to use an authentication library compliant with OAuth 2.0 
standards, offloading sign-in to third-party applications instead of building
our own authentication from the ground up. This also enables the user to use
their account from another service instead of going through the process of
creating a new username-password combination that they can forget.

We will opt for a session-token authentication approach as opposed to JWT.
Given the stateless nature of JWT, we wouldn't be able to manage the lifetime of
a user session (for example, in the case that they wish to sign out). This
adds a certain burden to our application servers, as authenticated actions will
require additional database interaction.

### Responding to a Request from the Frontend

1. Receive authentication code (unstored) and make API request to relevant
authentication API (e.g. Google Auth). Once validity is confirmed, generate a
session token for the frontend. Only actions that require specific access rights
(`DELETE` an itinerary, `GET` user's private itineraries) will require that
the token be provided in the request header.
    a. Token is queried in the `USER` table of our database
    b. If no row exists, then respond with a `401: Unauthorized`, else we can
    check if the token corresponds to a user that has correct access rights for
    the action they are performing (e.g. a user should only be able to `DELETE`
    their own itinerary)
2. The application server makes API requests to Google Directions API, parsing
and caching the resulting polyline (as we discussed in chapter 7). Future 
previews of the same itinerary won't require further API requests.
3. The application server handles access of an itinerary, querying first the 
cache and then the main database if needed.
4. The application server maintains lightweight metrics of frequently accessed
itinerary `uid`s for cache maintainance. We don't want to be polluting an 
in-memory cache with *"one-hit-wonder"* itineraries, for example.

## Data Model

### Data Collection and Management

As this is an app designed to create and share itineraries, we need to store user information and itinerary details. Additionally, we use the user's location and photo gallery for enhanced functionality.

- **Profiles:**
  - Each profile is identified by a `user uid`, which serves as the key in the "profiles" collection.
  - Profiles contain the following information:
    - `username`
    - `list of liked itineraries` (stored as itinerary uids, acting as foreign keys to the "itineraries" collection)

- **Itineraries:**
  - Each itinerary is identified by an `itinerary uid`, which serves as the key in the "itineraries" collection.
  - Itinerary details include:
    - `user uid` (of the creator)
    - `list of locations`
    - Additional information:
      1. Description
      2. Number of likes
      3. Price
      4. Tags (maximum of 3)
      5. Title
      6. Visibility (private or public)

- **Images:**
  - Related to `profile pictures` and `itinerary pictures`.

- **Firestore:**
  - `profiles` collection: Contains user information.
  - `itineraries` collection: Contains itinerary information.

- **Firebase Storage:**
  - `images/itineraryPictures`: Stores images related to itineraries.
  - `images/profilePicture`: Stores profile pictures.

- **Firebase Authentication:**
  - Manages logged-in users. Passwords are securely stored by Google, only user emails are stored in our collections.

- **Data Sharing:**
  - Firestore is used for data sharing. The `profiles` collection consists of documents identified by `user uid`. Each profile document includes a list of `itinerary uids` for the itineraries liked by the user (those `uids` can be used to identify the corresponding itineraries in the collection).
  - The `itineraries` collection also consists of documents identified by `itinerary uid`, with a `user uid` field to link back to the user who created the itinerary.

- **Image Management:**
  - Images are stored in Firebase Storage and linked to itineraries and profiles using a unique `uid` (`user uid` or `itinerary uid`).

- **Caching:**
  - Caching is implemented for liked itineraries. Users can click a button on the like screen to save itineraries for offline access. (A more optimized approach would involve automatically storing liked itineraries during app usage.)

This organization ensures efficient data management, sharing, and caching, leveraging Firebase's robust backend services.

## Security Considerations

### Authentification

The app utilizes Google as its primary authentication provider to streamline the login process and enhance security. This integration allows users to sign in using their existing Google accounts, which not only simplifies the authentication process but also leverages Google's robust security measures including OAuth 2.0 protocol, secure access tokens and token handling. This also means that, by delegating authentification to Google, we minimize the amount of personnal data handled directly by our app.

### Personnal Data

- **Location tracking**: The app accesses user locations data strictly based on explicit user consent. Users are prompted to grant permission for location tracking when the app starts. The user's location is not stored and all information related to the user's position is discarded when the app closes or when the session ends. User location data are strictly used for app functionality during active session and are not shared with third parties.

- **Itinerary data**: By defaults, created itineraries can be shared publicly, allowing other users to view these itineraries as part of the app's social features. However, users have the option to set their itineraries to "private." When this option is selected, the itinerary is not visible to other users and is kept confidential. Users can also delete their itineraries at any time. All itineraries are securely stored on Firebase, a cloud service provider with robust security measures. It ensures data integrity and availability.

## Infrastructure and Deployment

### Development
- **Frontend development**: The app's UI and interaction with the database were created using Android Studio.
- **Backend infrastructure**: The app utilizes a Firestore database to manage its data.
- **Version Control**: The development team uses Git for version control.
- **CI/CD Pipeline**: To ensure the app's reliability, a CI/CD (Continuous Integration/Continuous Deployment) pipeline was set up with GitHub Actions.

### Testing
To thoroughly test the app's functionality, three types of tests were conducted:
- **Unit testing**: Testing individual components of a feature.
- **Integration testing**: Testing the interaction between different components of the app.
- **End-to-end testing**: Testing a feature of the app through a user flow to check its usability and correctness.

### Deployment
- **Backend server**: The backend server will be deployed on Firebase Hosting.
- **User access**: The app will be deployed to Google Play services to provide easy access for users.
- **Maintainability**: Firebase Analytics, Google Analytics, and Crashlytics will be used to detect and resolve issues related to the app's performance and stability.

## Test Plan

### Plugin Tests

Given the dynamic nature of our itinerary planner app, which features a variety of user-driven customizations, it is critical to ensure that each component or plugin functions seamlessly across different scenarios. Testing will involve:

- **Frontend Rendering**: Ensuring consistent and accurate rendering of the user interface as users switch between different views and functionalities like map integration, itinerary creation, and social sharing.

- **Component Compatibility**: Verifying that all plugins work in harmony without conflicts, maintaining app stability and user experience.

### API Integration Tests

Our app relies on third-party services for functionalities like Google authentication and map services. Regular integration tests are essential to:

- **Service Connectivity**: Routinely check the connectivity and response from these APIs to ensure that authentication flows and map functionalities are always operational.

- **Change Monitoring**: Implement monitoring tools to detect changes or deprecations in third-party services, allowing us to be proactive in addressing potential disruptions.

### Performance Testing

Considering the app’s social sharing and real-time location features, performance under load is critical. Testing in this area will include:

- **Traffic Simulation**: Stress test the app by simulating peak usage scenarios, such as during holiday planning periods when user activity is at its highest.

- **Concurrent Usage**: Ensure the system can handle a high number of concurrent users creating, updating, and sharing itineraries without performance degradation.

- **Latency Measurements**: Focus on critical operations like loading itineraries and updating user locations to keep a smooth user experience. Unfortunately, the duration of operations dependent on location can vary significantly due to several factors, such as network connectivity and traffic conditions.

