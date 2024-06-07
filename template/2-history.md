# History

The development of our proof-of-concept, in the context of the SwEnt course,
provided valuable insights on getting an appliction (including relevant 
backend) up and running. Most of the core features that we wanted to implement
were implemented, namely those related to itineraries themselves (creation, 
navigation, discovery).

Our POC is lacking in several areas, however, which will need to be addressed 
as we move towards an MVP that is ready to be released to users and pitched to
potential investors.

1. Social elements: our POC, although containing a lot of core functionality,
is lacking in terms of social elements. To create a more dynamic 
user-experience and ultimately gain traction, this will need to be prioritized
as it will require significant reworks to many parts of our existing frontend
and infrastructure.

1. Scalability: Our application's backend is heavily reliant on third party 
APIs. Some of these can't be avoided without significant investment (e.g. 
Google Maps and Directions APIs which proved invaluable and are difficult to
replicate), others can be implemented in a way that is better tailored to our
needs, namely storage. As we gain user-adoption, it becomes unsustainable to
host everything through third party services. It is better that we take this on
sooner rather than later

